# 项目：Nanobanana AI 时尚工作室

## 这是什么

这是一个围绕服装图片生成展开的独立项目，目标是：

- 上传一张普通服装图
- 通过 Gemini 图片生成模型
- 一次产出多个视角的时尚效果图

它不是单纯的 API 调用示例，而是一个完整的小产品：

- 上传
- 参数设置
- 生成
- 画廊展示
- 历史作品集
- 模型切换

## 来源

当前项目主要来自这些本地实现：

- `wenkangnas_nanobanana/README.md`
- `wenkangnas_nanobanana/src/`
- `wenkangnas_nanobanana/api/`
- `wenkangnas_nanobanana/vite.config.js`

## 当前能力

从现有实现看，这个项目已经覆盖：

- 上传服装图片
- 选择模特性别与族裔
- 输入创意描述
- 选择风格滤镜
- 调用 Gemini 生成 4 种视角：
  - 正面全景
  - 背面展示
  - 面料细节
  - 完整穿搭
- 保存历史作品与元数据
- 在 `Nano Banana Pro` 与 `Nano Banana 2` 之间切换

## 技术栈

- 前端：React + Vite + React Router + Zustand
- 后端：PHP
- 数据库：MySQL / MariaDB
- AI：Gemini 图片生成模型
- 部署：Apache + `.htaccess`

## 为什么它适合放进 Vibe Coding

因为它不是一份零散 prompt，而是一整条可以复现的工作流：

- 前端页面结构明确
- 后端接口明确
- 模型入口明确
- 产物保存路径明确
- 部署方式明确

这正适合继续沉淀成“别人 fork 之后也能做出类似工具”的项目手册。

## 后续适合拆成的手册

1. 前端页面和状态管理手册
2. 图片生成 API 接入手册
3. Gemini 模型切换与 prompt 结构手册
4. Apache / PHP / 数据库存储部署手册

## 验收标准

至少应当能验证：

1. 上传一张图片后能正常进入生成流程
2. 能产出多个视角的结果图
3. 模型切换会影响生成路径
4. 历史记录可以被查看
5. 作品元数据会被保存

## 备注

这一页只做项目入口，不直接抄本地 `.env`、中转 token、数据库口令或其他敏感配置。
