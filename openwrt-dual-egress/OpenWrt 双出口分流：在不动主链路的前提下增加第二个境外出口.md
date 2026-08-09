# OpenWrt 双出口分流：在不动主链路的前提下增加第二个境外出口

[TOC]

> 本文记录一次在**已承担生产出口**的 OpenWrt 路由器上，安全地接入第二个境外 WireGuard 出口，并按域名分流的完整思路与方法。
>
> 核心诉求：主出口是全家的生命线，**绝对不能被第二出口覆盖或拖垮**；第二出口只用来把 YouTube、Spotify 这类大流量按域名单独引走。
>
> 文中所有服务器 IP、商业 VPN 服务商品牌、个人域名均已抹去，只保留可复用的方法。

---

## 全流程 Step-by-Step 总览

赶时间的话，照这个顺序走就行；每一步的详细说明见对应章节。**顺序本身就是安全保证**——先只读、再旁路、再最小上线、最后扩展。

- **Step 0｜看清现状**（第一、二章）：判断主出口是"路由"还是"透明代理"，只读命令调查，认出防泄漏规则。
- **Step 1｜定架构**（第三章）：确认第二出口放在代理平面，立下"绝不覆盖主出口"的铁律。
- **Step 2｜备份 + 回滚脚本**（第九章 §1-2）：动手前先能退回去。
- **Step 3｜取 WireGuard 配置**（第四章）：用服务商会员后台的配置生成器，**不要**跑官方一键脚本。
- **Step 4｜旁路验证**（第五章）：独立 sing-box 实例 + 额外 SOCKS 端口，测直连/链式两条底层路径，不碰生产。
- **Step 5｜最小规则上线**（第六章前半）：建节点 + 分流节点，先只放一个"金丝雀"域名，重启代理验证。
- **Step 6｜扩展分流域名**（第六章后半）：确认无误后把 YouTube/Spotify 等批量加进**同一条**规则。
- **Step 7｜多节点冗余 + 自动切换**（第七章）：下载节点池 + cron 故障切换脚本。
- **Step 8｜定 Fail-safe 策略**（第八章）：第二出口挂＝DROP；主出口挂＝绝不自动换国家。
- **Step 9｜收尾验证**（第九章）：核对路由表/规则未变、防泄漏仍在、两个出口 IP 正确。

---

## 一、为什么"加一条路由"这个直觉是错的

大多数人想到"给路由器加第二个出口"，脑子里的图是这样的：

```
                    +--> 默认路由 --> 主出口
终端 --> OpenWrt ---|
                    +--> 第二条路由 --> 新出口
```

于是去研究 `ip route` / `ip rule` / PBR（策略路由）/ mwan3。**但动手前必须先搞清楚：你现有的"主出口"到底是不是路由层实现的。**

我的这台机器调查下来发现：所谓"主出口"根本不是路由层的 default route，而是一套**全局透明代理（TPROXY）**。它的真实数据流是：

```
终端流量 / 路由器本机流量
      │  （被防火墙 mangle 表打上 mark）
      ▼
  ip rule: fwmark 0x1  →  独立路由表
      ▼
  TPROXY 重定向到本机代理内核（xray）监听端口
      ▼
  代理协议封装  →  境外服务器  →  Internet
```

也就是说：**所有流量在进入 main 路由表决策之前，就已经被防火墙 mark + TPROXY 劫走了。** main 表里那条 `default via 网关` 只是给代理内核自己连境外服务器用的底层链路，终端流量根本不经过它做决策。

这个发现直接推翻了"加第二条路由"的方案。因为：

1. 你在 main 表 / PBR 里怎么折腾第二条路由，普通流量都感知不到——它们早被 TPROXY 截走了。
2. 想按**域名**分流更是无从谈起：路由层只认 IP，不认域名。

> **结论：如果你的主出口是透明代理实现的，那么"第二出口"正确的位置不在路由层，而在代理平面里——把它做成代理内核里的第二个 outbound 节点，用代理软件自带的分流（shunt）按域名区分。**

这一步"先搞清楚现状再动手"是整件事里最重要的判断。不要照搬教程直接装 PBR 包。

---

## 二、调查阶段：只读，不改

在动任何配置之前，先把现状摸清楚。全部是只读命令，不会影响正在跑的生产流量：

```bash
ubus call system board         # 系统/内核/架构
ip addr ; ip route show        # 接口与 main 表
ip route show table all        # 所有路由表（关键！能看出有没有独立表）
ip rule show ; ip -6 rule show # 策略路由规则（关键！fwmark 规则暴露 TPROXY）
nft list ruleset               # 防火墙全量规则（mangle/TPROXY/mark 都在这）
wg show                        # 有没有现成 WireGuard
uci show network ; uci show firewall
opkg list-installed | grep -iE 'wireguard|pbr|mwan|xray|sing|passwall'
```

重点看三样东西，它们能直接告诉你主出口是不是透明代理：

- **`ip rule` 里有没有 `fwmark ... lookup <表号>`**：有，基本就是 TPROXY 分流。
- **`nft list ruleset` 里有没有 `mangle` 链 + `tproxy` / `meta mark set` 动作**：这是透明代理的指纹。
- **`ip route show table all` 里除了 main 有没有独立小表**（如 `local default dev lo table 100`）：TPROXY 的配套。

我的机器上还发现一条**自制"防泄漏"规则**（在 forward 链上 DROP 掉内网直连外网的流量），意思是"代理挂了宁可断网也不许裸奔"。**这种规则要认出来并保留**，它是你现有安全模型的一部分，别手滑删了。

---

## 三、目标架构

```
                              ┌─ 默认（未命中规则）──► 主出口（原有，保持不动）
终端/本机 ─ TPROXY ─► 代理内核分流 ─┤
                              └─ 命中域名规则 ───────► 第二出口（WireGuard 境外节点）
```

几条铁律：

- main 路由表、IPv4 `ip rule`、防火墙 zone、防泄漏规则——**一个字节都不改**。第二出口永远不可能变成系统默认出口。
- 第二出口挂掉 → 只有命中分流规则的域名受影响（表现为连接失败），主出口完全无关。
- 主出口挂掉 → 金融、默认流量**不会**自动切到第二出口（宁可暂时不通，也不要突然换国家 IP）。
- 不装 PBR / mwan3，不重启整个 network（避免 SSH 断线），唯一的服务中断点是重启代理服务（秒级）。

---

## 四、取得 WireGuard 配置的正确姿势

很多商业 VPN 服务商提供官方的 OpenWrt 客户端脚本，但**大多数这类脚本会无条件接管默认路由**——典型做法是往 main 表注入两条 `/1` 路由：

```
0.0.0.0/1   dev <wg>
128.0.0.0/1 dev <wg>
```

两条 `/1` 合起来覆盖整个 `0.0.0.0/0`，而且比默认路由更"具体"（前缀更长），会**压过**你原有的默认路由。很多客户端还会同时改 DNS、装 kill switch，且**没有开关可以关掉这些行为**。

> **所以：不要跑服务商的官方一键脚本。** 只需要它的"静态 WireGuard 配置"——现在正规服务商的会员后台基本都有一个 **WireGuard 配置生成器**：浏览器本地生成密钥对，私钥不上传（只把公钥注册到账号），当场生成标准的 `[Interface] / [Peer]` 配置文件。

生成的配置长这样（敏感信息已抹去）：

```ini
[Interface]
Address = 10.x.x.x/32, fd00:.../128
DNS = 10.x.x.1, fd00:...::1
PrivateKey = <私钥，只存本地>

[Peer]
PublicKey = <服务器公钥>
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = <服务器地址>:<端口>
PersistentKeepalive = 20
```

我们只取里面的 `PrivateKey / PublicKey / Endpoint / Address / DNS`，**AllowedIPs 里的 `0.0.0.0/0` 只是 WireGuard 协议字段，不会真的注入系统路由表**——因为我们不把它交给内核 WireGuard 接口，而是喂给代理内核当一个 outbound。

把配置传到路由器专门的目录并收紧权限，**私钥全程不要打印到终端或日志**：

```bash
ssh root@路由器 'mkdir -p /etc/hideout/pool && chmod 700 /etc/hideout/pool'
scp 配置.conf root@路由器:/etc/hideout/pool/nodeA.conf
ssh root@路由器 'chmod 600 /etc/hideout/pool/*.conf'
# 传完立刻删掉本地下载的副本
```

---

## 五、旁路验证：先别碰生产

在把节点接进生产代理之前，先起一个**独立的 sing-box 实例**，用一个额外的 SOCKS 端口单独测这条隧道。这样验证过程完全不影响正在跑的主出口。

关键点：WireGuard 走 UDP，从境内直连境外 UDP 端口有时会被干扰。所以要测两条底层路径：

1. **直连**：WireGuard 的 `Endpoint` 直接从本地网络出去。
2. **链式（前置代理）**：让 WireGuard 的握手和数据先走你现有的主出口代理，再到境外节点（sing-box 的 `detour` 字段）。

sing-box 测试配置骨架（用户态 WireGuard endpoint + 两个 SOCKS 入口分别对应两条路径）：

```jsonc
{
  "endpoints": [
    { "type": "wireguard", "tag": "wg-direct",
      "address": ["10.x.x.x/32", "fd00:.../128"],
      "private_key": "...", "mtu": 1380,
      "peers": [{ "address": "<endpoint>", "port": <端口>,
                  "public_key": "...", "allowed_ips": ["0.0.0.0/0","::/0"],
                  "persistent_keepalive_interval": 20 }] },
    { "type": "wireguard", "tag": "wg-chained", "detour": "socks-main",
      /* 其余同上，握手经主出口 */ }
  ],
  "inbounds": [
    { "type": "socks", "tag": "in-direct",  "listen": "127.0.0.1", "listen_port": 10808 },
    { "type": "socks", "tag": "in-chained", "listen": "127.0.0.1", "listen_port": 10809 }
  ],
  "outbounds": [
    { "type": "socks", "tag": "socks-main", "server": "127.0.0.1", "server_port": <主出口本地SOCKS端口> },
    { "type": "direct", "tag": "direct" }
  ],
  "route": { "rules": [
    { "inbound": ["in-direct"],  "outbound": "wg-direct" },
    { "inbound": ["in-chained"], "outbound": "wg-chained" }
  ], "final": "direct" }
}
```

验证命令（看到的 IP 应该是境外节点，而不是主出口）：

```bash
sing-box check -c test.json && sing-box run -c test.json &
curl --socks5-hostname 127.0.0.1:10808 https://ifconfig.co/ip   # 直连路径
curl --socks5-hostname 127.0.0.1:10809 https://ifconfig.co/ip   # 链式路径
curl https://ifconfig.co/ip                                     # 现网：应仍是主出口
```

我实测下来**直连 UDP 通且速度不错**（几十 Mbps 级别），就用直连；如果你那边直连不通，就用链式。测完把测试实例杀掉、临时文件删掉，生产还是干干净净的。

> 顺带记一个坑：主出口代理的本地 SOCKS 如果 **UDP relay 不通**，链式路径会失败——因为 WireGuard 是 UDP。这种情况要么开启 SOCKS 的 UDP 支持，要么老实用直连。

---

## 六、接入生产代理 + 域名分流

验证通过后，把这个 WireGuard 节点正式加进生产代理（这里以 PassWall2 为例，它的 xray 后端原生支持 WireGuard 协议节点）。用 `uci` 脚本从 `.conf` 里提取字段、建节点：

```bash
CONF=/etc/hideout/pool/nodeA.conf
PK=$(sed -n 's/^PrivateKey *= *//p' $CONF)
PUB=$(sed -n 's/^PublicKey *= *//p' $CONF)
ADDR=$(sed -n 's/^Address *= *//p' $CONF); A4=${ADDR%%,*}; A6=${ADDR##*,}
EP=$(sed -n 's/^Endpoint *= *//p' $CONF)

uci set 代理.nodeA=nodes
uci set 代理.nodeA.type='Xray'
uci set 代理.nodeA.protocol='wireguard'
uci set 代理.nodeA.address="${EP%:*}"
uci set 代理.nodeA.port="${EP##*:}"
uci set 代理.nodeA.wireguard_public_key="$PUB"
uci set 代理.nodeA.wireguard_secret_key="$PK"
uci add_list 代理.nodeA.wireguard_local_address="$A4"
uci add_list 代理.nodeA.wireguard_local_address="$A6"
uci set 代理.nodeA.wireguard_mtu='1380'
uci set 代理.nodeA.wireguard_keepAlive='20'
```

再建一个**分流（shunt）节点**：默认走主出口，命中规则的域名走第二出口。**先只加一条最小可验证的规则**（比如一个 IP 查询站点当"金丝雀"），确认整条链路通了再扩：

```bash
# 分流规则：初期只放一个金丝雀域名
uci set 代理.MediaRule=shunt_rules
uci set 代理.MediaRule.remarks='MediaRule'
uci set 代理.MediaRule.network='tcp,udp'
uci set 代理.MediaRule.domain_list='ifconfig.co'

# 分流节点：默认=主出口，MediaRule=第二出口
uci set 代理.split=nodes
uci set 代理.split.protocol='_shunt'
uci set 代理.split.default_node='<主出口节点ID>'
uci set 代理.split.MediaRule='nodeA'

uci set 代理.@global[0].node='split'   # 全局切到分流节点
uci commit 代理
/etc/init.d/代理 restart               # 唯一的秒级中断点
```

验证（默认仍是主出口，金丝雀域名走第二出口）：

```bash
curl https://ifconfig.me       # 应为主出口 IP
curl https://ifconfig.co/ip    # 应为第二出口（境外节点）IP
```

确认无误后，把真正要分流的域名批量加进**同一条规则**（用 geosite 分类最省事）：

```bash
uci set 代理.MediaRule.domain_list='geosite:youtube
geosite:spotify
domain:googlevideo.com
domain:ytimg.com
domain:youtubei.googleapis.com
domain:scdn.co
domain:spotifycdn.com'
uci commit 代理 && /etc/init.d/代理 restart
```

**金融、代码托管、AI 服务、工作相关域名——不写任何规则**，它们自然落到默认（主出口），符合"这些服务绝不自动换国家 IP"的要求。

> ⚠️ **一个重要的坑：不要让多条分流规则指向同一个 WireGuard 节点。** 用户态 WireGuard outbound 每条规则会建一个独立会话，同一把密钥的多个并发会话在服务端会互相打架（endpoint 漂移、丢包）。**所有域名都塞进同一条规则**即可。

---

## 七、多节点冗余 + 自动故障切换

单节点有个明显风险：那台境外服务器一挂，分流的域名就全崩。解决办法是**下载同一服务商的一批节点组成"节点池"**。

关键认知：同一个 WireGuard 密钥对，在服务商后台可以对**每个位置**分别生成配置——它们**共享同一把私钥和同一个隧道内网地址，只是 `Endpoint` 和服务器公钥不同**。所以下载一批（我下了十来个欧洲位置），逐个建成代理节点即可。

然后写一个**故障切换脚本**，用 cron 定时探测，节点挂了就沿"节点环"轮转到下一个。设计要点：

- **双金丝雀探测**：用两个独立的 IP 查询站点（都走第二出口规则），避免单站点抽风误判。
- **连续失败才切**：连续 2 次探测失败（约 6 分钟确认窗口）才动，避免瞬时抖动导致频繁重启代理。
- **整环走完仍失败就停手**：说明不是单节点问题（可能是金丝雀站点挂了、或上游/审查层面的问题），继续轮转只会无意义地反复重启，不如保持现状等恢复。
- **永不回落主出口**：切换只在节点池内部进行，跟"宁可断也不换国家"的原则一致。

脚本骨架：

```sh
#!/bin/sh
RING="nodeA nodeB nodeC ... nodeJ"     # 节点环
STATE=/tmp/failover.state
probe() {   # 双金丝雀，任一成功即算通
  curl -4 -s --max-time 10 https://金丝雀1/ip && return 0
  curl -4 -s --max-time 10 https://金丝雀2   && return 0
  return 1
}
FAILS=0; ROT=0; [ -f "$STATE" ] && . "$STATE"
if probe; then echo "FAILS=0"$'\n'"ROT=0" > "$STATE"; exit 0; fi   # 恢复即归零
FAILS=$((FAILS+1))
[ "$FAILS" -lt 2 ] && { echo "FAILS=$FAILS"$'\n'"ROT=$ROT" > "$STATE"; exit 0; }  # 未连续失败2次，先忍
NODECOUNT=$(echo $RING | wc -w)
[ "$ROT" -ge "$NODECOUNT" ] && exit 0    # 整环走完仍失败，停手保持现状
# 取环中下一个节点，改分流映射并重启代理
CUR=$(uci -q get 代理.split.MediaRule)
# ...在 RING 里找到 CUR 的下一个 NEXT...
uci set 代理.split.MediaRule="$NEXT"; uci commit 代理; /etc/init.d/代理 restart
echo "FAILS=0"$'\n'"ROT=$((ROT+1))" > "$STATE"
logger -t failover "switched: $CUR -> $NEXT"
```

挂到 cron：

```bash
echo '*/3 * * * * /etc/hideout/failover.sh' >> /etc/crontabs/root
/etc/init.d/cron restart
```

日志用 `logread | grep failover` 看切换动态。

---

## 八、Fail-safe：两种取舍

**第二出口挂掉时，命中规则的流量怎么办？** 两个方案：

- **A. DROP（推荐，零配置）**：分流规则固定指向第二出口节点，节点死则那些域名连接失败。零风险、绝不泄漏到别的出口。配合上面的自动切换，通常还没等你发现就已经切到别的节点了。
- **B. 回落主出口**：让规则节点自动切回主出口。**但这意味着 YouTube/Spotify 会在你不知情时突然变回主出口的国家 IP**，未必是你想要的。

我选 A。

**主出口挂掉时**：默认流量（金融、工作等）**不会**自动切到第二出口——因为它们压根没有分流规则，默认节点死了就是不通。这正是我们想要的："宁可这些服务暂时不可访问，也不要突然换一个国家的 IP 去登录银行。"

---

## 九、收尾验证清单

```bash
ip rule show                              # 应与备份一致：只有 local / fwmark→表 / main / default
ip route show                             # main 表默认路由未变
nft list chain inet fw4 <防泄漏链>         # 自制防泄漏规则仍在
curl https://ifconfig.me                  # 主出口 IP（未变）
curl https://ifconfig.co/ip               # 第二出口 IP（境外节点）
logread | grep failover | tail            # 故障切换正常
```

再补几个安全习惯：

1. **动手前先全量备份**：`/etc/config/{network,firewall,dhcp,代理}`、`/etc/nftables.d/`，以及 `ip route show table all` / `ip rule` / `nft list ruleset` 的快照，全部塞进 `/root/网络备份-时间戳/`。
2. **写一个一键回滚脚本**：恢复上述配置 + 重启代理服务（**不要** `network restart`，会断 SSH），确保配置出错能立刻退回原状。
3. **优先 `ifup/ifdown <接口>` 而不是 `/etc/init.d/network restart`**，避免 SSH 断线。
4. **私钥永不落终端/日志**，配置文件 `chmod 600`。

---

## 十、小结

这次的核心不是"怎么配 WireGuard"，而是三个判断：

1. **先摸清主出口的真实实现**（路由 vs 透明代理），它决定了第二出口该放在哪一层——照搬教程装 PBR 是南辕北辙。
2. **第二出口做成代理平面的一个 outbound 节点**，用软件自带的域名分流，天生不碰系统路由表，也天然用无污染的远程 DNS。
3. **主出口是 PRIMARY，第二出口是 SECONDARY**：后者的存在、故障、切换，都不允许以任何方式影响前者。

沿着"只读调查 → 旁路验证 → 最小规则上线 → 逐步扩展 → 加冗余与 fail-safe"这个顺序走，全程对生产链路零破坏，SSH 不断线，随时可一键回滚。
