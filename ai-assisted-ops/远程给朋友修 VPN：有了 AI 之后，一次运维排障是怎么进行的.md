# 远程给朋友修 VPN：有了 AI 之后，一次运维排障是怎么进行的

[TOC]

> 朋友坐在我家客厅，MacBook 上的 OpenVPN Connect 死活连不上我 NAS 的 VPN。我没碰她的电脑，只把一条 `ssh` 命令丢给了 AI agent（Claude Code）。
>
> 一个小时后：VPN 通了；她的设备在 NAS 上有了专属的出口策略；桌面上多了三个带图标的小程序，双击就能用。
>
> 这篇文章记两件事：**故障本身是怎么一层层剥开的**，以及**有了 AI 之后，这类工作和以前到底哪里不一样了**。
>
> 文中所有公网 IP、密码、商业服务商品牌、个人域名与真实用户名均已抹去或改写，只保留可复用的结构。网段沿用[前两篇](../home-network-topology/家庭网络分层拓扑：从远端笔记本到双出口的完整链路.html)的写法：内网 `192.168.0.0/24`，VPN `172.x.y.0/24`。

---

## 一、TL;DR

一次"连不上"背后叠了**四层**互不相关的问题，每剥掉一层症状就变一次：

| 层 | 症状 | 根因 |
|---|---|---|
| 1 | `Server poll timeout`（TCP） | 客户端 `.ovpn` 写的是 `proto tcp-client`，服务端只开 **UDP** |
| 2 | 改成 UDP 仍超时 | 人在家里局域网，拨的却是公网域名 —— 路由器 **hairpin NAT 不通** |
| 3 | 加了内网 `remote` 仍只打公网 IP | OpenVPN Connect 把服务器地址**缓存在自己的数据库**里，改磁盘上的 `.ovpn` 不生效 |
| 4 | `UDP send exception: send: Broken pipe` | 同机的另一个商业 VPN（WireGuard）kill-switch 在拦包；断开后残留状态仍拦，**换了一条网络链路才消失** |

连通之后又追加了两件事：

- **仅这一台设备**的流量不走家里的旁路网关 OpenWrt（其他 VPN 用户不变）：`client-config-dir` 固定 IP + 一条 `pref 0` 的 `ip rule` + 写进 DSM 定时任务持久化。
- 给一个不懂命令行的用户交付**桌面工具**：一键代理开关（带测速、退出恢复）、一个只走代理的 Gemini 独立浏览器、一个远程桌面入口。方案迭代了三版。

全程 **20:59 → 22:05**，一个会话，同时操作三台机器（我的 Mac、她的 Mac、NAS）。

---

## 二、背景

- **服务端**：群晖 NAS 上的 OpenVPN Server，`proto udp`，端口 `3389`，地址池 `172.x.y.0/24`，`topology subnet`，账号密码认证（`username-as-common-name` + `duplicate-cn`，所有人共用一个账号）。
- **拓扑**：NAS 上有一张策略路由表 —— `from 172.x.y.0/24 lookup 100`，table 100 的默认路由指向旁路网关 OpenWrt（`192.168.0.254`），VPN 用户的流量由它分流出境；NAS 自身走主路由 `192.168.0.1`。这套东西的来龙去脉见[《家庭网络分层拓扑》](../home-network-topology/家庭网络分层拓扑：从远端笔记本到双出口的完整链路.html)。
- **客户端**：朋友的 MacBook（macOS 15），OpenVPN Connect 3.8，`.ovpn` 是从 DSM 导出后**别人手改过**的。机器上还装着另一个商业 VPN、一个代理工具、一个企业 VDI 客户端和远程控制软件。
- **我的角色**：坐在旁边，不碰她的电脑。所有操作通过 `sshpass ... ssh` 由 AI 执行；NAS 也是 ssh + sudo。

---

## 三、排障时间线

按真实顺序写，每一步都是"看到什么 → 怀疑什么 → 用什么命令证实/证伪"。

### 3.1 第一层：协议不对

先看客户端日志（`~/Library/Application Support/OpenVPN Connect/log/ovpn.log`）：

```
Contacting <公网IP>:3389 via TCP
EVENT: WAIT
Server poll timeout, trying next remote entry...
```

再看 `.ovpn`：

```
remote <ddns域名> 3389
proto tcp-client
```

而服务端在 NAS 上：

```sh
$ netstat -lnu | grep 3389
udp6  0  0 :::3389  :::*
$ nc -vz 192.168.0.4 3389        # TCP
Connection refused
```

**TCP 永远握不上。** 这一步 AI 是先查了我的记忆库（后面第六章会讲），已经知道"服务端是 UDP 3389"，所以第一眼就盯住了 `proto` 行。

### 3.2 第二层：在家里打公网 IP

改成 UDP 后还是超时。这时注意到一个细节：

```sh
$ curl ifconfig.me       # 在她的 Mac 上
<和我家一样的公网IP>
```

她连的是我家 Wi-Fi，拨的却是 DDNS 域名 —— 从内网打自己路由器的公网 IP，靠的是 hairpin NAT，而我家路由器不支持。这个坑在我记忆库里也有一条："公网 DDNS 在家 hairpin 不通会静默卡死"。

修法直觉上很简单：在 `.ovpn` 里加一行内网地址，让它优先：

```
remote 192.168.0.4 3389 udp
remote <ddns域名> 3389 udp
```

### 3.3 第三层：客户端不读你改的文件

改完再连，日志里依然**只有公网 IP**，`192.168.0.4` 一次都没出现过。

OpenVPN Connect 3 导入 profile 时，会把解析出来的 `remoteHost` / `serverList` 存进自己的 `config.json`（一个 redux persist 状态）。之后连接用的是这份缓存，磁盘上的 `.ovpn` 只是个备份。所以：

> **改 OpenVPN Connect 的配置，必须在 app 里删掉 profile 重新导入**，而不是改文件。

重新导入后，日志终于出现 `Contacting 192.168.0.4:3389 via UDP`。然后症状变了。

### 3.4 第四层：Broken pipe

```
Connecting to [192.168.0.4]:3389 via UDP
UDP send exception: send: Broken pipe
UDP send exception: send: Broken pipe
...
```

**包在本机就没发出去。** 查她 Mac 上的网络状态：

```sh
$ scutil --nc list
* (Connected)     ... "<商业VPN> (Wireguard)"
$ netstat -rn | grep default
default   link#19   UCSg   utun4
```

另一个商业 VPN 正处于连接状态，`utun4` 抢了默认路由，它的 kill-switch 会丢掉一切不走隧道的流量。断开它 —— **依然 Broken pipe**。

接下来是一段真正的"排障"：

- `pfctl -s info` → pf 是 Disabled
- `sysctl net.cfil` → 内容过滤器 `active_count: 0`
- `systemextensionsctl list` → 那个商业 VPN 的系统扩展处于 `activated waiting for user`（半激活）
- 系统日志里发现 **OpenVPN Connect 进程自己发的 TCP SYN 也没有回应**（`SYN in/out: 0/11`），但同一台机器上 `curl`、Python 的 UDP socket 都正常 —— 这是**针对单个进程**的现象
- 机器上还跑着企业 VDI 客户端（root 权限的网络服务）、远程控制软件、四个残留的 `utun` 接口

到这里我叫停了 AI。它给出的候选根因都有道理，但每验证一个要五分钟，而我们的目标是"让 VPN 通"，不是"给这台机器写病历"。

**人做的决策**：让她把网络切到手机热点（同时 Wi-Fi 还连着我家、但那个 Wi-Fi 没网关），用 DDNS 域名走公网进来 —— 这才是她日常真正的使用场景。切完之后：

```
EVENT: CONNECTED vpn@<ddns域名>:3389 via /UDP on utun4/172.x.y.2/
```

**通了。** EPIPE 的最终根因没有钉死（候选：半激活的系统扩展残留、VDI 客户端的网络钩子、`en0` 无网关时 ovpnagent 计算 bypass route 出错）。这里诚实地记一笔：不是每个故障都值得追到底，**换链路后消失**是一个足够的工程结论。

### 3.5 教训小结

1. 日志里的"超时"和"Broken pipe"是**两种完全不同的失败**：前者包出去了没回应，后者包根本没出去。看清这一点能省一半时间。
2. 客户端 GUI 有自己的状态库时，**改文件不等于改配置**。
3. 一台机器上同时装着 3 个会接管网络栈的软件（商业 VPN、代理、VDI），任何一个的残留都可能拦别人的包。排障前先 `scutil --nc list` + `ifconfig | grep utun`。
4. **在服务端也要看**：NAS 上 `netstat -lnu`、`openvpn.conf`、连接日志（`synovpnlog.db`）三样合起来才能说"服务端没问题"。

---

## 四、第二阶段：仅这一台设备不走 OpenWrt

VPN 通了之后，新需求：这位朋友的流量**不要**经过旁路网关 OpenWrt，直接从主路由出去；其他 VPN 用户保持不变。

### 4.1 为什么不能直接做

策略路由按**源地址**匹配。现在所有人共用一个账号（`duplicate-cn`），拿到的 IP 是动态的 `.2/.3/.4`，从 NAS 看根本分不清谁是谁。所以第一步是**让这台设备有一个稳定的源地址**。

### 4.2 三个动作

**① 专属账号 + `client-config-dir` 固定 IP**

DSM 里给她建一个独立用户（授予 VPN Server 权限），然后在 OpenVPN 配置里启用 per-client 配置目录：

```sh
# /usr/syno/etc/packages/VPNCenter/openvpn/openvpn.conf 追加
client-config-dir /usr/syno/etc/packages/VPNCenter/openvpn/ccd
```

```sh
# ccd/<用户名>
ifconfig-push 172.x.y.200 255.255.255.0
push-remove dhcp-option                    # 不给她推旁路网关的 DNS
push "dhcp-option DNS 192.168.0.1"         # 改推主路由
```

两个细节：

- 用 `push-remove` 而不是 `push-reset`。`push-reset` 会把服务端自动生成的 `topology subnet`、`route-gateway`、`ping`/`ping-restart` 一起清掉，客户端会拿到一个残缺的配置。`push-remove` 只精确摘掉一项（OpenVPN ≥ 2.4）。
- `ccd` 文件名 = 客户端 CN。因为服务端开了 `username-as-common-name`，所以文件名就是**用户名**。

**② 一条排在前面的 `ip rule`**

```sh
ip rule add from 172.x.y.200/32 lookup main pref 0
```

```
0:  from all lookup local
0:  from 172.x.y.200 lookup main        ← 新加，查 main 表 → default via 192.168.0.1
1:  from 172.x.y.0/24 lookup 100        ← 原有，其他 VPN 用户 → OpenWrt
```

为什么是 `pref 0`：Linux 内核插入同优先级规则时，**排在已有同优先级规则之后**（`fib_nl_newrule` 找的是第一条 `pref > new` 的位置）。写 `pref 1` 会落在那条 `/24` 后面，永远匹配不到；写 `pref 0` 则正好落在 `local` 之后、`/24` 之前。

回程不用担心：NAS 回给 `.200` 的包源地址是 `172.x.y.1`，仍命中 `/24` 规则查 table 100，而 table 100 里有 `172.x.y.0/24 dev tun0`（[上一篇](../home-network-topology/家庭网络分层拓扑（二）：一条策略路由如何让 VPN 网关人间蒸发.html)就是修这条的）。

**③ 持久化**

`ip rule` 重启 NAS 就没了。我家 NAS 上已经有一个每 5 分钟跑一次、维护 table 100 的 DSM 定时任务（上一篇的产物），最合理的做法是把新规则加进同一个脚本，而不是再开一个。

DSM 的定时任务不在 crontab 里，是存在它自己的库里的。用 `synowebapi` 读出来、改、写回去：

```sh
synowebapi --exec api=SYNO.Core.TaskScheduler method=get version=4 id=7
# ...在 script 字段里插入下面这段，再 method=set 写回
```

```sh
DIRECT_HOSTS="172.x.y.200"
for h in $DIRECT_HOSTS; do
    ip rule list | grep -q "from ${h} lookup main" ||
        ip rule add from "${h}/32" lookup main pref 0
done
```

写回后立刻 `get` 一次核对字段确实在 —— 这类"写了但没生效"的坑上一篇踩过。

### 4.3 一个顺手的技巧：不登录也能验证密码

朋友不确定新账号的密码。DSM 的登录 API 对"密码错"和"密码对但没有登录权限"返回**不同的错误码**：

```sh
curl -G http://127.0.0.1:5000/webapi/entry.cgi \
  --data-urlencode api=SYNO.API.Auth --data-urlencode version=6 \
  --data-urlencode method=login --data-urlencode account=<用户> \
  --data-urlencode passwd=<待验证密码>
# {"error":{"code":402}}   ← 密码对，只是这个账号没有 DSM 桌面权限
# {"error":{"code":400}}   ← 账号或密码错
```

一次请求就知道答案，不用去改密码。

### 4.4 完成后的拓扑

```
朋友的 MacBook
  │ 底层：手机热点（或任意外网）
  │ OpenVPN / UDP 3389 → <ddns域名>，专属账号
  ▼
utun  172.x.y.200/24 ──────────────► NAS tun0 172.x.y.1
                                        │
                     ip rule ───────────┤
        pref 0  from .200 → main ──────►│ default via 192.168.0.1 ──► 主路由直出   ← 她
        pref 1  from /24  → table 100 ─►│ default via 192.168.0.254 ► OpenWrt 分流  ← 其他人
```

---

## 五、第三阶段：给不懂命令行的人交付工具

她的真实需求其实只有一句话："我要能用 Gemini。" 出境流量在我家是 OpenWrt 上的一个 SOCKS 端口（`192.168.0.254:1080`）负责的，所以要做的是：**让她的浏览器在需要时走这个 SOCKS**。

方案迭代了三版，每一版的否决理由都值得记。

### 5.1 第一版：系统代理开关（被否）

一个 AppleScript 小程序，双击后用 `networksetup -setsocksfirewallproxy` 把系统 SOCKS 代理指过去，弹一个窗；点"关闭并退出"或 ⌘Q 时恢复。

做对的部分：

- 开启前把每个网络服务的原始值（启用状态/服务器/端口）记到 `~/Library/Application Support/<工具>/state.psv`，退出时逐项还原，并在她机器上实测 on→off 一轮，四个服务全部精确回到 `Enabled: No / Server 空 / Port 0`。
- 上次被强杀没恢复的话，下次启动**不覆盖**已有记录，退出仍回到最初状态。
- 启动先 `nc -z` 探测代理可达，不通就提示"先连 Wi-Fi 或 VPN"，不改任何东西。

踩的坑：

- `networksetup -setsocksfirewallproxy <svc> "" 0` 会把服务器设成字面量 `0`；要清空得写 `"" ""`。
- 状态文件用 Tab 分隔，`IFS=$'\t' read` 会把**连续的 Tab 合并**（Tab 是空白类 IFS），空字段导致列错位。换成 `|` 分隔。
- AppleScript 里 `POSIX path of (path to me) & "…"` 偶发 `-1700`（can't make … into type），拆成两句赋值就稳了。
- `osacompile` 生成的小程序里带了一个 `Assets.car`，macOS 15 优先从它读图标，换 `applet.icns` 不生效；删掉 `Assets.car`、去掉 `Info.plist` 的 `CFBundleIconName`、重新签名才行。

被否的理由（用户原话）："**太危险了**。" 系统级代理影响所有应用，忘了关、或者小程序崩了，整台机器的流量就都在走那个口。对一个不看菜单栏的用户，这个风险不对称。

### 5.2 第二版：拆成"代理开关"和"Gemini"

代理开关保留，做成 VPN 工具那种小白模式：开启 → 自动跑一轮测试 → 弹窗显示

```
✅ Google 可达，延迟 580 ms
⬇️ 下载速度约 18.9 Mbps
🌍 出口：US Los Angeles
[重新测试]  [关闭代理并退出]
```

Gemini 单独一个图标，双击时检查系统代理没开就先拉起开关、等它开好，再开浏览器。**但这没有解决"系统代理"本身的风险**，只是把它藏得更深了。

### 5.3 第三版：Gemini 专用的隔离 Chrome（最终）

正确的模型是：**代理只作用于一个进程，且这个进程的生命周期和窗口一致。** Chrome 恰好都能做到：

```sh
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --user-data-dir="$HOME/Library/Application Support/GeminiChrome" \   # 独立 profile，和她日常的 Chrome 互不干扰
  --proxy-server="socks5://192.168.0.254:1080" \                       # 只有这个实例走代理
  --remote-debugging-port=9333 \                                        # 给看门狗用
  --no-first-run --no-default-browser-check --disable-sync \
  --app=https://gemini.google.com/app                                   # app 模式，无地址栏
```

三个要点：

1. **系统代理一个字节都不碰。** `scutil --proxy` 全程 `SOCKSEnable: 0`。
2. **DNS 不泄漏。** Chrome 对 `socks5://` 代理是把域名交给代理端解析的。一开始还加了 `--host-resolver-rules="MAP * ~NOTFOUND, EXCLUDE 192.168.0.254"` 做双保险，但 Chrome 会为它弹"不受支持的命令行标记"横幅，去掉了。
3. **关窗即退出。** macOS 上 Chrome 关掉最后一个窗口不会退出进程。解决办法是一个看门狗：每 2 秒查 `http://127.0.0.1:9333/json/list`，`"type": "page"` 数量为 0 就 `kill` 这个实例。不需要辅助功能权限，不需要 UI 脚本。

端到端验证的方式也值得记：不是"看起来关了"，而是通过 DevTools 协议模拟关窗（`/json/close/<id>`），然后确认进程 4 秒内消失、`launcher.pid` 被清理。

### 5.4 最终桌面

| 图标 | 做什么 |
|---|---|
| Gemini（官方图标） | 隔离的代理 Chrome；关窗 = 一切消失 |
| 代理开关（盾牌+锁） | 可选：其他应用也要走代理时用；带测速；退出恢复 |
| 远程桌面 VDI（客户端原版图标） | Safari 打开企业 VDI 登录页，Touch ID 自动填密码 |
| OpenVPN Connect / 商业 VPN | 替身 |

图标来源：Gemini 用 App Store 的 1024px 原图按 macOS 圆角规范裁；VDI 直接拿客户端 bundle 里的 `.icns`；代理开关用 PIL 画了一个盾牌 + 锁。

---

## 六、有了 AI 以后，变化在哪

这一节是写这篇的真正原因。上面的技术内容放在三年前我也能做出来，**但我不会去做** —— 尤其是第四、五章。变化不在"能不能"，在**边际成本**。

### 6.1 排障半径：从"我的机器"到"我能 ssh 到的一切"

整个会话里 AI 同时操作三台机器：我的 Mac（跑 agent）、她的 Mac（`sshpass ssh`）、NAS（ssh + sudo）。每一次"看客户端日志 → 看服务端监听 → 看 NAS 路由表 → 回到客户端"的跨机器往返，以前是我在三个终端窗口之间切换、复制粘贴、在脑子里拼图；现在是一次工具调用并行发出，结果并排回来。

第三章里客户端说 Broken pipe、同时服务端确认 UDP 3389 在听、同时我自己的机器 `nc` 测端口 —— 三个视角一分钟内对齐。**并行**是最直接的加速。

### 6.2 外置记忆：AI 比我更记得我家的网络

会话一开始，AI 先去查了我的记忆库（一个跨客户端共享的 MCP memory），搜到了四条相关记录：

- 服务端是 UDP 3389，不是 TCP
- table 100 的设计与"回程路由"的坑
- DDNS 在家 hairpin 不通
- DSM 定时任务存在两个地方，只查一个会误判

这四条**每一条都在这次派上了用场**。第一层故障几乎是秒判；第四章改定时任务时直接走了正确的存储位置。这些东西我自己写过、也在博客里发过，但如果是我亲手排查，大概率要重新想一遍"服务端到底是 UDP 还是 TCP 来着"。

博客顶上那句 *The brain has cache expiration too* 是写给未来的我的；现在多了一个读者，而且它真的读。

### 6.3 假设-证据循环更快，错得也更快 —— 人负责叫停

3.4 节是这次最值得反思的段落。AI 判断"另一个 VPN 的 kill-switch 是根因"—— 证据充分（默认路由被抢、utun4 在线），结论合理，**但断开后故障依旧**。它接着给出下一个候选、再下一个：pf、内容过滤器、系统扩展、NECP、flow divert、VDI 客户端……每一个都有对应的验证命令，每一个都能再花五分钟。

它不会自己停。这不是缺陷，是分工：**agent 的默认模式是穷尽假设空间，而"这个问题值不值得继续追"是目标层面的判断**，属于人。我叫停之后换了链路，问题消失，我们接受了"根因未钉死"这个结论继续往前走。

反过来说，如果我一个人排查，我可能在第二个候选就放弃了 —— 也可能在第五个候选上耗掉一晚上。AI 把"继续追"的成本压得很低，这让"停不停"这个决策变得更重要、也更清晰。

### 6.4 需求变更的成本趋近于一句话

回看这次的需求流：

1. "帮我看下为什么连不上"
2. "改成公网 IP"
3. "现在用的是手机热点，用 DDNS 域名解决"
4. "有办法不要流量往 OpenWrt 发吗？仅这个设备"
5. "用已有的 ho 用户"
6. "写一个工具，点击设置代理，关闭恢复如初"
7. "把 OpenVPN 快捷方式放桌面上"
8. "拆成代理开关和 Gemini 两个"
9. "图标换成真的"
10. "太危险了，做成 Chrome app，关闭就清掉"
11. "去掉那个横幅"

十一次变更，每次一句话，每次都在几分钟内落地并验证。以前这种对话发生在我和自己之间，落地的成本是"再花一个晚上"，于是大部分变更根本不会发生 —— 我会在第 1 步修好 VPN 之后说"行了，能用了"。

**AI 没有改变我想要什么，改变的是"想要"和"得到"之间的距离。** 第 4、6、9、10 步是以前绝对不会为朋友做的事。

### 6.5 从"修好"到"交付"

以前帮朋友修完东西，交付物是一段口头说明或一份 README。这次交付物是三个带图标的 `.app`，而且每一个都在她的机器上做了真实的端到端验证：

- 代理开关：on → 查 `scutil --proxy` → off → 逐项比对四个服务恢复原值
- Gemini：启动 → 查 DevTools 目标列表里有 `gemini.google.com/app` → 模拟关窗 → 确认进程退出、系统代理从未变过
- VDI：启动 → 读 Safari 前台标签 URL

**验证不是额外步骤，是每一步的一部分。** 这是 agent 工作方式里我最想保留到自己手工操作中的习惯。

### 6.6 代价：权限、边界与备份

诚实地列一下这次 AI 拿到了什么：她 Mac 的登录密码、NAS 的 sudo 密码、她新账号的密码、三台机器的 root 级操作。

对应的约束也要写清楚，这些是**人来定的**：

- 改 NAS 配置前先 `cp openvpn.conf openvpn.conf.bak-<时间戳>`；改她的 `.ovpn` 前先备份到桌面。
- 所有写入 NAS 的脚本必须**幂等**（`grep -q || ip rule add`），可以反复跑。
- 有不可逆或影响他人的操作时（建账号、重启 VPN Server 会踢掉现有用户），先问再做。这次有两处它停下来问了，都是对的。
- 会话结束后，密码文件删掉。

AI 让"做"变得便宜，也让"做错"变得便宜。备份和幂等不是形式，是这套工作方式的安全带。

### 6.7 时间线

| 时刻 | 事件 |
|---|---|
| 20:59 | 她第一次点 Connect，TCP 超时 |
| 21:04 | 定位 `proto tcp-client`，改 UDP + 内网 remote |
| 21:12 | 发现客户端缓存，重新导入 |
| 21:15 | Broken pipe；发现另一个 VPN 在线 |
| 21:20 | 换手机热点 |
| 21:22 | **CONNECTED** |
| 21:28 | ccd + ip rule + 定时任务写回，VPN Server 重启 |
| 21:31 | 以专属账号连接，拿到 `.200` |
| 21:36 | 第一版代理开关上线 |
| 21:44 | 桌面替身 |
| 21:54 | 三个工具 + 图标 |
| 22:00 | Chrome 隔离实例版 Gemini |
| 22:05 | 去掉横幅，收工 |

66 分钟。其中真正卡住的只有 3.4 节那十分钟。

---

## 七、可复用的排查清单

**OpenVPN 客户端连不上，按这个顺序：**

1. 客户端日志：是 `poll timeout`（包出去没回应）还是 `send exception`（包没出去）？
2. 服务端：`netstat -lnu` / `-lnt` 确认协议和端口；配置文件里的 `proto` / `port`。
3. 客户端 `.ovpn` 的 `proto` 是否与服务端一致。
4. 客户端和服务端是否在同一个 LAN（`curl ifconfig.me` 对比公网 IP）→ hairpin。
5. 如果改了 `.ovpn` 没效果：GUI 客户端是否有自己的状态缓存，需要删除重导入。
6. `scutil --nc list`、`ifconfig | grep utun`、`netstat -rn | grep default`：有没有别的 VPN/代理在线或残留。
7. 换一条网络链路（热点）再试，区分"这台机器的问题"和"这条链路的问题"。

**NAS 侧单设备例外，按这个顺序：**

1. 独立账号 + `client-config-dir` + `ifconfig-push` 固定 IP。
2. `push-remove` 精确摘掉不想推的选项，不要 `push-reset`。
3. `ip rule add from <ip>/32 lookup main pref 0`（注意同 pref 排序规则）。
4. 写进已有的自愈定时任务；写回后 `get` 一次核对。
5. 提醒未来的自己：DSM 界面一旦保存 VPN 设置，`client-config-dir` 会被抹掉。

**给非技术用户交付 macOS 小工具：**

1. 能不碰系统级设置就不碰；优先给单个进程传参数。
2. 生命周期要和用户看得见的东西（窗口）绑定。
3. 每一个"恢复原状"都要在目标机器上实测一轮 on→off 并比对。
4. `osacompile` 的图标：删 `Assets.car`、去 `CFBundleIconName`、`codesign --force --deep -s -`。

---

## 八、结语

这次最大的收获不是修好了一个 VPN。是清楚地看到了一种新的工作形态：**人提供意图、约束和叫停，agent 提供并行执行、外置记忆和逐步验证。** 十一句话，三台机器，一个小时，交付物是朋友能双击的东西。

三年前同样的下午，结局大概率是："我把 proto 改成 udp 了，你回家再试试。"
