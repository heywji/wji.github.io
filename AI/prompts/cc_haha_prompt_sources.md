# CC-Haha Prompt Source Index

来源：`/home/wjiwji/projects/cc-haha`

## 用途

这不是单条 prompt，而是一整个 agent 项目的 prompt 源码集合。适合用来观察：

- tool prompt 是怎么拆文件的
- planner / memory / prompt suggestion 是怎么组织的
- 一个 CLI agent 项目如何把 prompt 写成可维护的代码资产

## 主要目录

- `src/tools/*/prompt.ts`
- `src/services/*/prompt.ts`
- `src/services/*/prompts.ts`
- `src/buddy/prompt.ts`
- `src/utils/ultraplan/prompt.txt`
- `src/constants/prompts.ts`

## 代表性文件

- `src/tools/BashTool/prompt.ts`
- `src/tools/WebSearchTool/prompt.ts`
- `src/tools/AgentTool/prompt.ts`
- `src/services/compact/prompt.ts`
- `src/services/SessionMemory/prompts.ts`
- `src/services/MagicDocs/prompts.ts`
- `src/buddy/prompt.ts`
- `src/utils/ultraplan/prompt.txt`

## 备注

- 这页只做索引，不把源码里的 prompt 全量搬到博客
- 这类 prompt 更适合按代码阅读，而不是按单页文档阅读
