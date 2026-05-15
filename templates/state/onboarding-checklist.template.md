---
generatedAt: "{{GENERATED_AT_ISO}}"
lastUpdatedAt: "{{LAST_UPDATED_AT_ISO}}"
projectNameOrSlug: "{{PROJECT_NAME_OR_SLUG}}"
---

# onboarding-checklist

> **用途**：引导式接入的**可勾选**进度表；与 `guide-project-onboarding` 各阶段对应。勾选须以事实为准（文件存在、人类已确认等）。

## 使用说明

- Agent 仅在完成对应事实后勾选 `- [x]`；不得虚勾。
- 「负责」列：主责 Skill 或「人类」。

## 接入清单（1–14）

| 步骤 | 项 | 负责 | 状态 |
|------|----|------|------|
| 1 | 已确认 `{projectRoot}` 为业务仓库根 | `guide-project-onboarding` + 人类 | - [ ] |
| 2 | 已确认 `{skillLibraryRoot}` 相对路径（如 `../HACF`） | `guide-project-onboarding` + 人类 | - [ ] |
| 3 | 已执行或等价完成 `load-skill-library`（含子 Skill 效果，含 `ensure-agent-default-entry`） | `load-skill-library` | - [ ] |
| 4 | 根目录 `AGENTS.md` 已创建或更新且引用 `AI_ENTRY.md` | `create-or-update-agents-md` | - [ ] |
| 5 | 已创建 `.ai/` 目录树（Bootstrap MVP 范围） | `initialize-project-ai-context` | - [ ] |
| 6 | 已生成 `.ai/entry/`（`AI_ENTRY.md`、`SKILLKIT_LINK.md`） | `initialize-project-ai-context` | - [ ] |
| 7 | 已选择或接受默认目标 Agent，并已执行 `ensure-agent-default-entry`（见 `.ai/state/agent-entry-status.md`） | `ensure-agent-default-entry` | - [ ] |
| 8 | 已执行 `check-project-readiness` 并存在最新 `readiness.md` | `check-project-readiness` | - [ ] |
| 9 | 人类已确认项目基础信息（见 `onboarding-questions.md` 第一节） | 人类 | - [ ] |
| 10 | 人类已确认硬约束、高风险区、源码/忽略/只读目录与语言策略（见 `onboarding-questions.md` 第二节） | 人类 | - [ ] |
| 11 | 已更新 `.ai/state/onboarding-status.md` | `guide-project-onboarding` | - [ ] |
| 12 | 已更新 `.ai/state/onboarding-checklist.md`（与本表一致） | `guide-project-onboarding` | - [ ] |
| 13 | 已生成或更新 `.ai/state/onboarding-questions.md` | `guide-project-onboarding` | - [ ] |
| 14 | 已告知用户下一步应执行的 Skill（可引用 `recommend-next-onboarding-step`） | `guide-project-onboarding` | - [ ] |

## 备注

<!-- 部分步骤由 load 一次完成时，可在此说明「步骤 3–6 同批完成」等；若 load 已含 ensure，步骤 7 可与步骤 3 一并满足 -->
