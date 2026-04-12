# RHEL / avocado / tp-qemu Debug User Prompt

来源：[`../User_Prompt`](../User_Prompt)

## 用途

这是我自己给 ChatGPT 用的一条偏工程化 user prompt，主要服务于 RHEL、`avocado-vt`、`tp-qemu`、内部私有框架排障。

## 这条 Prompt 约束了什么

- 用户给代码时，优先按 debug 场景处理
- 允许联网查 `avocado-vt` 和 `tp-qemu` 的上游源码
- 遇到具体 RHEL 运行实例时，帮助判断更像产品 bug 还是自动化框架 bug
- 中文提问用中文答，英文提问用英文答
- 英文提问时，先帮忙检查语法和拼写，再回答问题

## 内置的 Bug 模板

这条 prompt 里还固定了一版问题单模板，核心字段包括：

- What were you trying to do that didn't work?
- Please provide the package NVR for which bug is seen:
- How reproducible:
- Steps to reproduce
- Expected results
- Actual results

## 原文入口

- [查看原始 User Prompt 文件](../User_Prompt)

## 备注

- 这是一条任务型 prompt，不是公开教程
- 真正复用时，最好按你的项目语境再补充 repo、分支和环境信息
