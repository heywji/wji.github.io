# 手册 02：Wayland 下的 ydotoold 配套、多后端选择与落地

## 目标

把前台文本发送工具从“只支持 `xdotool` 的单后端实现”，升级成“X11 / Wayland 可分流的多后端实现”。

核心结论：

- X11 / XWayland 下优先考虑 `xdotool`
- Wayland 下不能只写一句 “兼容”，必须把 `ydotoold` 和 `ydotool` 这条链路真正接起来

## 为什么旧实现会卡住

旧实现最容易出问题的地方不是 GUI，而是默认假设：

> 只要装了 `xdotool`，输入层就能工作。

这个假设在 Wayland 下不稳。真正需要补的是：

1. 会话检测
2. 后端抽象
3. `ydotoold` 配套服务
4. socket 探测
5. 明确错误提示

## 必须先检查的环境项

先让 AI 在本地检查这些项目，再开始编码：

```bash
echo "$XDG_SESSION_TYPE"
which xdotool
which ydotool
id -u
ls -l /run/user/$(id -u)/.ydotool_socket
```

这里至少要回答清楚：

- 当前到底是 `x11` 还是 `wayland`
- `xdotool` 是否存在
- `ydotool` 是否存在
- 当前用户 socket 是否存在

## 推荐的后端抽象

至少拆成这三个角色：

- `BaseInputBackend`
- `XdotoolBackend`
- `YdotoolBackend`

每个后端至少实现这些接口：

- `is_supported()`
- `is_ready()`
- `send_text()`
- `send_key()`
- `explain_failure()`

这样 GUI 和 CLI 都不需要知道底层细节。

## ydotoold 配套服务

Wayland 方案里，`ydotoold` 不是附属品，而是关键依赖。

一个可工作的 systemd 服务示意：

```ini
[Unit]
Description=ydotool daemon
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/ydotoold -p /run/user/1001/.ydotool_socket -P 0666
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

实现层真正关心的是：

- socket 路径可预测
- 客户端可访问
- 失败时知道应该去哪里查

## 客户端后端选择策略

推荐执行顺序：

1. 检测会话类型
2. 如果是 `X11` 或 `XWayland`，优先尝试 `xdotool`
3. 如果是 `Wayland`，优先尝试 `ydotool`
4. 调用 `ydotool` 之前先检查 socket 是否存在
5. 如果 socket 不在默认位置，允许使用 `YDOTOOL_SOCKET`
6. 如果主后端失败，输出明确诊断，不要静默吞掉

## 默认支持的能力

在多后端阶段，也建议只先覆盖这些能力：

- 普通文本
- `<ENTER>`
- `<TAB>`
- `<ESC>`
- `<CTRL+C>`
- `<CTRL+V>`

不要在第一轮 Wayland 适配里把所有组合键都扩出去。

## AI 落地时必须完成的实现项

1. 写一个会话检测函数
2. 写一个后端工厂
3. 写一个 `YdotoolBackend` 的 ready 检测
4. 支持默认 socket 路径探测
5. 支持通过环境变量覆盖 socket
6. 把错误信息从 “发送失败” 改成分层错误

## 错误信息最少要分到这个粒度

- `xdotool` 未安装
- `ydotool` 未安装
- `ydotoold` 未启动
- socket 不存在
- socket 权限不足
- 当前会话类型不匹配
- 所选后端存在但当前不可用

## 测试矩阵

至少跑这四类测试：

### 测试 1：X11 正常路径

- 检测到 `x11`
- 选择 `xdotool`
- 发送普通文本成功

### 测试 2：Wayland + ydotoold 正常

- 检测到 `wayland`
- 发现 `ydotool`
- 发现 socket
- 发送普通文本成功

### 测试 3：Wayland + ydotoold 未启动

- 检测到 `wayland`
- `ydotool` 存在
- socket 不存在
- 返回明确错误，而不是卡死

### 测试 4：缺失依赖

- 缺少 `xdotool` 或 `ydotool`
- GUI 和 CLI 都能给出可读错误

## 完成标志

这份手册完成后，工具应该已经从“能跑的 demo”进化到“知道自己为什么能跑，以及为什么跑不起来”的工程形态。

再往下一步，才值得把这些要求整理成一个可以直接交给 AI 的执行 prompt。
