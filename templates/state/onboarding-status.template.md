---
generatedAt: "{{GENERATED_AT_ISO}}"
lastUpdatedAt: "{{LAST_UPDATED_AT_ISO}}"
currentPhase: "phase_confirm_roots"
projectRootConfirmed: false
skillLibraryRelativePathConfirmed: false
loadSkillLibraryDone: false
agentsMdVerified: false
aiDirectoryVerified: false
entryDirectoryVerified: false
agentDefaultEntryDone: false
readinessChecked: false
hacfVersionChecked: false
hacfVersionBlocking: false
humanBasicsConfirmed: false
humanConstraintsBundleConfirmed: false
onboardingArtifactsSynced: false
handoffDone: false
projectNameOrSlug: "{{PROJECT_NAME_OR_SLUG}}"
---

# onboarding-status

> **事实源声明**：本文件记录 HACF **引导式项目接入**进度与阶段门控，不替代 `readiness.md` 或 `skillkit-status.md`；事实以仓库内对应文件为准。

## 当前阶段（`currentPhase`）

| 取值 | 含义 |
|------|------|
| `phase_confirm_roots` | 确认 `{projectRoot}` 与公共库相对路径 |
| `phase_run_load_skill_library` | 执行或引导执行 `load-skill-library` |
| `phase_check_hacf_version` | 执行或引导执行 `check-hacf-version-compatibility`，写入 `.ai/state/hacf-version-status.md` |
| `phase_verify_agents` | 确认根目录 `AGENTS.md` 已正确引用入口 |
| `phase_verify_ai_tree` | 确认 `.ai/` 目录存在 |
| `phase_verify_entry` | 确认 `.ai/entry/` 与链接文件 |
| `phase_ensure_agent_default_entry` | 选择目标 Agent 并执行 `ensure-agent-default-entry` |
| `phase_run_readiness` | 执行或引导执行 `check-project-readiness` |
| `phase_confirm_project_basics` | 人类确认项目基础信息 |
| `phase_confirm_constraints_bundle` | 人类确认硬约束、高风险区、目录策略等 |
| `phase_sync_onboarding_artifacts` | 同步更新 `onboarding-status` / `checklist` / `questions` |
| `phase_handoff_next` | 交接：告知下一步应执行的 Skill |

当前解析值：见上方 YAML 的 `currentPhase` 字段（实例化后由 Agent 与人类协作更新）。

## 摘要

- 项目简称：`{{PROJECT_NAME_OR_SLUG}}`
- 最近更新（UTC）：{{LAST_UPDATED_AT_ISO}}

## 关联状态文件

- 清单：`.ai/state/onboarding-checklist.md`
- 待答问题与答复摘要：`.ai/state/onboarding-questions.md`
- Bootstrap 就绪：`.ai/state/readiness.md`
- SkillKit 摘要：`.ai/state/skillkit-status.md`
- 框架版本比对：`.ai/state/hacf-version-status.md`
- Agent 默认入口：`.ai/state/agent-entry-status.md`、`.ai/config/agent-entry-policy.md`

## 最近人类动作（可选）

<!-- 人类或 Agent 简述上一轮确认内容，例如「已确认 projectRoot=D:\work\foo」 -->

## 备注

<!-- 阻塞、路径解析失败、待下一轮处理项 -->
