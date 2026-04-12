# oh-my-opencode Setup Prompt

来源：`/home/wjiwji/projects/ai_prompts/oh-my-opencode.md`

## 用途

给 LLM Agent 一份安装和配置 `oh-my-opencode` 的执行提示，重点覆盖：

- 先问清订阅信息
- 非交互安装命令怎么拼
- provider 认证怎么做
- Gemini / Claude / OpenAI / Copilot / OpenCode Zen / Z.ai 组合配置

## 关键执行流

1. 先问用户有没有 Claude Pro/Max、OpenAI/ChatGPT Plus、Gemini、GitHub Copilot、OpenCode Zen、Z.ai Coding Plan
2. 检查 `opencode` 是否已安装
3. 运行 `bunx oh-my-opencode install --no-tui ...`
4. 验证 `opencode --version` 与 `~/.config/opencode/opencode.json`
5. 按 provider 逐个执行 `opencode auth login`

## 关键命令模板

```bash
bunx oh-my-opencode install --no-tui \
  --claude=<yes|no|max20> \
  --openai=<yes|no> \
  --gemini=<yes|no> \
  --copilot=<yes|no> \
  --opencode-zen=<yes|no> \
  --zai-coding-plan=<yes|no>
```

## Prompt 里的关键信息

- 原 prompt 里把 provider 优先级写得比较清楚：Native > GitHub Copilot > OpenCode Zen > Z.ai Coding Plan
- 原 prompt 里还补了 Gemini Antigravity OAuth、Claude OAuth、Copilot fallback、Z.ai 和 OpenCode Zen 的模型分配思路
- 原 prompt 最后还带了验收、使用提醒、广告和 star 引导这类收尾动作

## 原文入口

- 原始文件：`/home/wjiwji/projects/ai_prompts/oh-my-opencode.md`

## 备注

- 这页保留的是可执行摘要，不再把整份长 prompt 全量贴一遍
- 如果后面要公开长期维护，建议定期回看其中的模型名、版本号和 provider 说明
