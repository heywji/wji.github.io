# 项目：Gemini API 中转站

## 这是什么

这是一个给 Bot、浏览器前端和其他本地项目复用的 Gemini API 中转站项目，同时支持：

- Web 界面聊天
- RESTful API
- 多模态输入
- Gemini 图片生成模型

## 来源

当前项目主要来自这些本地实现：

- `wenkangnas_gemini_api/README.md`
- `wenkangnas_gemini_api/config.php`
- `wenkangnas_gemini_api/chat.php`
- `wenkangnas_gemini_api/api.php`
- `wenkangnas_gemini_api/ui.html`

## 项目目标

做一个能部署到 NAS 上的 Gemini 中转服务，满足这些需求：

- 浏览器可直接使用
- Agent 可通过 REST API 调用
- 支持会话历史
- 支持上传图片、视频、PDF、音频
- 支持包括 Nano Banana 系列在内的图片生成模型

## 架构要点

核心链路是：

```text
浏览器 / Agent -> Nginx + PHP -> Gemini API
                          |
                        MySQL
```

也就是说，这不是一个单纯前端页，而是一个：

- 后端可代理 Gemini
- 前端可展示文本和图片
- 数据库可保存会话

的完整小系统。

## 执行重点

### 1. 先把服务端跑起来

至少要先确认：

- PHP 环境可用
- `pdo_mysql`、`curl`、`fileinfo` 已启用
- 上传目录有写权限
- Nginx 入口不会被静态 `index.html` 抢走

### 2. 再配 Gemini 代理参数

这类项目最容易卡在：

- API Key
- 代理设置
- Token 认证
- NAS 到 Google API 的连通性

### 3. 再去补图片生成模型体验

图片生成这条线至少要覆盖：

- 模型选择
- 图片渲染
- 下载
- 全屏预览

## 适合拆成的手册主题

后面如果继续补，可以拆成：

1. NAS 部署手册
2. Gemini API 代理与认证手册
3. 图片生成模型接入手册
4. Web UI 与 REST API 使用手册

## 验收标准

至少应当能验证：

1. 文本对话正常
2. 会话历史能保存
3. 通过图片生成模型能拿到图片结果
4. 前端能渲染图片并允许下载
5. Agent 能通过 REST API 正常调用

## 备注

这一页只做项目入口，不把本地的 Token、数据库密码、代理地址、API Key 直接搬到博客里。
