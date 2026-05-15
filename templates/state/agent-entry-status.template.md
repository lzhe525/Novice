---
docType: agent-entry-status
generatedBy: ensure-agent-default-entry
lastRunAt: "{{LAST_RUN_AT_ISO}}"
targetAgentsResolved: "{{TARGET_AGENTS_RESOLVED}}"
---
# agent-entry-status

> **事实源声明**：本文件记录最近一次 **Agent 默认极薄入口** 适配执行情况，不替代 `AGENTS.md` / `CLAUDE.md` / Cursor Rule 正文；事实以项目根下对应文件为准。

## 本轮摘要

- 执行时间（UTC）：{{LAST_RUN_AT_ISO}}
- 解析后的 `targetAgent` 集合：`{{TARGET_AGENTS_RESOLVED}}`

## 各入口结果

| 目标 | 路径（相对项目根） | 结果 |
|------|-------------------|------|
| codex / generic | `AGENTS.md` | <!-- created / updated / skipped / blocked --> |
| claude-code | `CLAUDE.md` | |
| cursor | `.cursor/rules/hacf.mdc`（或 policy 中 `cursorRulePath`） | |

## 备注

<!-- 失败原因、需人工处理的 BEGIN/END 不成对等 -->
