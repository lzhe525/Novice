---
version: 1
reviewedByHuman: false
# 执行 ensure-agent-default-entry 且 targetAgent 为 all 时，默认会尝试写入下列 Agent 对应入口（仍受「极薄」与受控区块约束）
enabledAgents:
  - codex
  - cursor
  - claude-code
  - generic
# 健康检查（check-project-ai-health H1）对下列 Agent 强制要求对应文件存在且含 HACF 受控区块与 AI_ENTRY 指针；留空则不强校验 CLAUDE / Cursor（仅沿用既有 AGENTS + .ai/entry 检查）
mandatoryAgents: []
# Cursor Project Rule 单文件相对路径（默认 hacf.mdc）
cursorRulePath: .cursor/rules/hacf.mdc
---

# agent-entry-policy

## 用途

声明本项目希望维护的 **Agent 默认极薄入口** 范围，以及健康检查是否强制要求某些入口文件。

## 各 Agent 与落盘路径

| Agent 标识 | 说明 | 默认落盘路径 |
|------------|------|----------------|
| `codex` / `generic` | 读取根目录 `AGENTS.md` | `AGENTS.md` |
| `claude-code` | Claude Code 项目说明 | `CLAUDE.md` |
| `cursor` | Cursor Project Rules | `cursorRulePath`（默认 `.cursor/rules/hacf.mdc`） |

## 人工确认

若调整 `mandatoryAgents` 或收窄 `enabledAgents`，建议由人类审阅后视需要置 `reviewedByHuman: true`。
