# 手册 03：交给 Gemini / Cursor / Claude 的执行 Prompt 模板

## 目标

这份手册不是讲原理，而是给一个可以直接复制给 AI 的执行模板。

要求是：

- 让 AI 读取本地已有实现
- 按手册 01 和手册 02 的边界重构
- 最终做出和我当前工作流一致的本地前台文本发送工具

## 使用前提

在把 prompt 丢给 AI 之前，你自己先确认三件事：

1. 目标是本机前台辅助输入，不是后台工具
2. 需要支持 X11 / Wayland 双路径
3. 本地已有参考实现和 `ydotoold` 配套资料可读

## 可直接使用的 Prompt

```text
你是一名资深 Python 桌面应用工程师。请基于我本地已有代码，重构一个“前台文本发送助手”，用于解决 remote-viewer / SPICE / VNC 会话里复制粘贴失效的问题。目标是本机前台、显式确认的辅助输入工具。

请先阅读这些本地参考实现：
1. /home/projects/wji_clipboard/clipboard_injector.py
2. /home/projects/wji_clipboard/README.md
3. /home/projects/wenkangnas_xfyun/README.md
4. /home/projects/wenkangnas_xfyun/xfyun_gui.py
5. /home/projects/wenkangnas_xfyun/ydotoold.service

请先输出你从这些文件里提炼出的约束，再开始设计和编码。

目标功能：
1. 保留 GUI + CLI 双模式
2. 支持文本框输入、stdin 输入、剪贴板加载
3. 保留发送前延迟和字符间隔
4. 只做前台发送，由用户自己切换目标窗口焦点
5. 支持多输入后端：
   - X11 / XWayland 优先使用 xdotool
   - Wayland 优先使用 ydotool
   - ydotool 路径下要检查 ydotoold 和 socket
   - 支持 /run/user/$UID/.ydotool_socket
   - 必要时支持 YDOTOOL_SOCKET

功能边界：
1. 不要实现后台驻留
2. 不要实现静默自动发送
3. 不要实现剪贴板监听或篡改
4. 不要实现全局键盘钩子
5. 不要实现远程控制

第一版标签白名单：
- <ENTER>
- <TAB>
- <ESC>
- <CTRL+C>
- <CTRL+V>

实现要求：
1. GUI 和 CLI 共享一套核心发送逻辑
2. 输入执行层必须抽象成后端接口
3. 必须实现会话检测
4. 必须实现后端选择策略
5. 必须给出明确错误信息，而不是笼统的 send failed

请按这个顺序输出：
1. 本地代码分析结论
2. 重构设计
3. 完整代码
4. 依赖安装说明
5. ydotoold.service 部署说明
6. 测试步骤
7. 与旧版实现的差异说明
```

## 我希望 AI 回答里必须出现的内容

如果 AI 没有回答这些点，就说明 prompt 还没压实：

1. 如何检测当前会话是 `x11` 还是 `wayland`
2. `xdotool` 与 `ydotool` 的选择规则
3. `ydotoold` 未启动时如何报错
4. socket 如何探测
5. GUI 和 CLI 是否共用同一套核心逻辑
6. 第一版只支持哪些标签

## 最后验收

可以用下面这套问题做最终验收：

1. AI 是否先读了本地实现，而不是凭空重写
2. AI 是否把输入后端单独抽象出来
3. AI 是否把 Wayland 配套服务写清楚
4. AI 是否给出了真实可运行的测试步骤
5. AI 是否保持了前台、显式确认的边界

如果这五个问题有任何一个答不清楚，就不要急着接受第一版结果。
