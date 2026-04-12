# 项目：Sojiang Survey Assistant

## 这是什么

这是一个围绕问卷站点自动化整理出来的完整项目，当前能力包括：

- 登录站点
- 处理验证码
- 拉取问卷列表
- 提取题目
- 调用 LLM 生成候选答案
- 自动回填
- 控制是否最终提交

它不是一个单独脚本，而是一套多模块工作流。

## 来源

当前项目主要来自这些本地实现：

- `wenkangnas_sojiang/README.md`
- `wenkangnas_sojiang/main.py`
- `wenkangnas_sojiang/docs/MODULES.md`
- `wenkangnas_sojiang/docs/TESTING.md`
- `wenkangnas_sojiang/test_login_browser.py`

## 当前工作流

核心链路大致是：

1. 打开登录页
2. 输入账号密码
3. 处理验证码
4. 进入调查列表
5. 打开单份问卷
6. 从 DOM 提取题目和选项
7. 调用 Gemini / OpenAI / Claude 生成候选答案
8. 根据结构化建议自动回填
9. 根据配置决定手动或自动提交
10. 保存截图、JSON、调试产物

## 关键特点

这个项目已经具备比较完整的工程分层：

- 登录模块
- 验证码模块
- 问卷提取模块
- LLM 提供方模块
- 页面操作模块
- 产物与测试模块

从“Vibe Coding 项目”的角度看，它已经不只是一个 prompt 或一段浏览器自动化，而是一整套可继续维护的系统。

## 为什么适合收进这里

因为它满足几个关键条件：

- 有清晰的入口和模块结构
- 有配套 README
- 有测试文档
- 有多 provider 支持
- 有 artifacts 和调试产物

这些都很适合继续沉淀成项目手册。

## 后续适合拆成的手册

1. 登录与验证码手册
2. 问卷提取与页面回填手册
3. 多 LLM provider 接入手册
4. 缺陷测试与 artifacts 复盘手册

## 验收标准

至少应当能验证：

1. 能稳定登录
2. 验证码流程可控
3. 题目提取结果结构正确
4. LLM 建议可转成页面操作
5. 手动 / 自动提交策略生效
6. artifacts 能帮助复盘问题

## 备注

这一页只做项目入口，不写站点账号、密码、API key 或 `.env` 中的敏感配置。
