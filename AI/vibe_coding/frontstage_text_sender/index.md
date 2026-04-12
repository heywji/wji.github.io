# 项目：前台文本发送助手

## 这是什么

这是一个本机前台辅助输入项目，用来解决：

- `remote-viewer`
- SPICE
- VNC
- 其他需要手动聚焦的桌面窗口

里复制粘贴不稳定的问题。

项目目标不是做后台工具，而是做一个：

- GUI + CLI 双模式
- 用户显式触发
- 用户自己切换焦点
- 文本和少量标签可发送

的本地前台文本发送助手。

## 来源

当前项目主要来自这些本地实现：

- `wji_clipboard/clipboard_injector.py`
- `wji_clipboard/README.md`
- `wenkangnas_xfyun/xfyun_gui.py`
- `wenkangnas_xfyun/ydotoold.service`

## 手册

1. [手册 01：前台文本发送助手的目标、边界与最小实现](../01_frontstage_text_sender_manual.md)
2. [手册 02：Wayland 下的 ydotoold 配套、多后端选择与落地](../02_wayland_backend_manual.md)
3. [手册 03：交给 Gemini / Cursor / Claude 的执行 Prompt 模板](../03_ai_execution_manual.md)

## 最终交付物

这一组手册最终希望 AI 做出：

- 一个 GUI + CLI 双模式工具
- 一个 `xdotool` / `ydotool` 可分流的输入后端层
- 一套 Wayland 下可诊断的 `ydotoold` 配套说明
- 一份本地可执行的测试清单

## 适合什么时候打开

当你要做这些事时，就打开这个项目：

- 想把本地文本可靠打进远端桌面窗口
- 想把 `xdotool` 单后端实现升级成 Wayland 可用版本
- 想写一个能直接交给 AI 落地的前台输入工具手册
