# 项目：Xfyun Wayland 实时语音输入

## 这是什么

这是一个基于讯飞实时语音转写接口的 Linux 桌面 GUI 工具，目标是：

- 一边录音
- 一边实时转写
- 再把文字直接贴到当前光标所在位置

它是一个典型的 Vibe Coding 项目，因为它把这些层叠在一起了：

- WebSocket 实时语音识别
- 音频采集
- GUI 交互
- Wayland 下的输入模拟
- 长时会话重连

## 来源

当前项目主要来自这些本地实现：

- `wenkangnas_xfyun/README.md`
- `wenkangnas_xfyun/xfyun_gui.py`
- `wenkangnas_xfyun/ydotoold.service`

## 目标能力

这个项目当前已经验证过的方向包括：

- 实时语音转写
- GUI 极简折叠模式
- Wayland 下用 `ydotool` 做输入 fallback
- 通过模拟粘贴把识别结果送到当前光标位置
- 处理“回车 / 换行 / Over”等语音指令
- 长时录音自动重连

## 执行重点

### 1. 先解决环境依赖

至少要先确认：

- `portaudio`
- `PyQt6`
- `pyaudio`
- `websocket-client`
- `pyperclip`
- `pyautogui`
- `ydotool`

都具备。

### 2. Wayland 下先把 ydotoold 跑起来

这条线的关键不是 GUI，而是输入后端能不能工作。

在 Wayland GNOME 下，核心前提是：

- `ydotoold` 已启动
- socket 可访问
- 客户端能探测到 `YDOTOOL_SOCKET`

### 3. 再去处理讯飞 IAT 的帧发送逻辑

如果鉴权和中间帧逻辑不对，即使麦克风正常也拿不到稳定结果。

### 4. 最后再调自动粘贴体验

验证顺序建议是：

1. 先确认识别结果能稳定返回
2. 再确认 GUI 状态切换正常
3. 再确认输入模拟能把结果贴到目标窗口

## 建议拆成的手册主题

如果后面要继续写细，可以拆成：

1. Wayland 依赖与 `ydotoold` 部署
2. 讯飞 WebSocket 鉴权与音频帧逻辑
3. GUI 极简模式与自动粘贴流程
4. 语音命令与长时重连

## 验收标准

至少应当能验证：

1. 语音说出后有实时文本结果
2. 焦点放在编辑器或浏览器时，文本能自动上屏
3. Wayland 下不用手工切到 X11 也能工作
4. 说“回车”或“换行”时能触发正确动作
5. 长时间使用时能自动重连

## 备注

这一页只做项目入口，不镜像本地 README 里的敏感配置。真正落地时请用你自己的 AppID、API Key、API Secret。
