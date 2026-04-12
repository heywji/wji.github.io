# Vibe Coding

这里不是单个项目的说明页，而是一个项目目录。

我会把自己已经跑通过、并且可以继续交给 AI 复现的项目，按“项目入口 + 可执行手册”的方式往这里放。这样后面无论是 Gemini、Cursor、Claude，还是我自己回来复盘，都能快速找到对应项目。

## 这个目录里放什么

这里放的是这类东西：

- 我已经本地做出来的 AI 项目
- 用 Gemini 做出来的项目
- 用 Cursor 做出来的项目
- 后续还会继续补充的 Vibe Coding 项目

它们不一定都属于同一技术栈，但都满足一个共同点：

> 不只是灵感或讨论，而是已经有本地实现、依赖、工作流、验证方法，可以继续沉淀成手册。

## 项目目录

### 1. [前台文本发送助手](./frontstage_text_sender/index.md)

关键词：

- `remote-viewer`
- `xdotool`
- `ydotoold`
- Wayland
- GUI + CLI

这是当前手册最完整的一条线，里面已经拆出了目标、后端设计和 AI 执行 prompt。

### 2. [Xfyun Wayland 实时语音输入](./xfyun_wayland_asr/index.md)

关键词：

- 讯飞 IAT
- WebSocket
- Wayland
- `ydotool`
- 实时转写

这条线的重点是：实时语音输入、Wayland 下自动上屏、以及 `ydotoold` 配套。

### 3. [Nanobanana AI 时尚工作室](./nanobanana_studio/index.md)

关键词：

- Nano Banana
- 图片生成
- 时尚广告图
- React
- PHP

这条线的重点是：上传服装图、生成多视角时尚图、保存历史作品与元数据。

### 4. [Cursor Agent Browser 登录方案](./cursor_agent_browser_login/index.md)

关键词：

- Cursor
- MCP Browser
- 验证码
- 登录流程
- Playwright

这条线的重点是：用 Cursor Agent Browser 完成登录工作流，并保留本地等价验证方案。

### 5. [Sojiang Survey Assistant](./sojiang_survey_assistant/index.md)

关键词：

- Survey
- Playwright
- Gemini / OpenAI / Claude
- 验证码
- 自动回填

这条线的重点是：登录、验证码、题目提取、LLM 候选答案、自动回填和测试产物。

### 6. [Gemini API 中转站](./nano_banana_gemini_proxy/index.md)

关键词：

- Gemini API
- NAS
- PHP
- 代理
- Web UI
- REST API

这条线的重点是：Gemini 中转、认证、会话历史和上层项目复用。

## 后续扩展方式

后面如果再加新的项目，继续按下面这个结构放：

1. 项目入口页
2. 项目来源文件
3. 目标和边界
4. 可执行步骤
5. 验收标准
6. 故障排查

这样这个目录才会越来越像“项目手册库”，而不是一次性记录。
