---
libraryVersion: "{{SKILLKIT_VERSION}}"
localFrameworkVersion: "{{SKILLKIT_VERSION}}"
linkedAt: "{{GENERATED_AT_ISO}}"
projectState: "loaded"
lastCheck: "{{GENERATED_AT_ISO}}"
---

# skillkit-status

本文件记录当前项目与公共 Skill 库的链接状态摘要；详细就绪项见同目录下 `readiness.md`（由 `check-project-readiness` 维护）。

## 字段说明

| 字段 | 含义 |
|------|------|
| `libraryVersion` | 最近一次成功读取的 `{skillLibraryRoot}/VERSION.md` 首行版本文本。 |
| `localFrameworkVersion` | 项目侧记录的 HACF **框架同步版本**（与公共库对齐的目标版本；初始化时与 `libraryVersion` 相同；升级成功后由 `apply-hacf-local-upgrade` 与检查 Skill 刷新）。**不得**在无依据时降低该字段。 |
| `linkedAt` | 最近一次成功完成 `load-skill-library` 或链接校验的 UTC ISO8601 时间。 |
| `projectState` | MVP 建议取值：`loaded`、`configuring`、`blocked`（语言策略未人工确认等）。 |
| `lastCheck` | 最近一次执行 `check-project-readiness` 的时间。 |
| `localizationLevel` | **可选**（由 `skills/state/evaluate-localization-level.md` 写入）：`L0`～`L3`，表示项目 AI 协作本地化程度。 |
| `localizationEvaluatedAt` | **可选**：最近一次执行 `evaluate-localization-level` 的 UTC ISO8601 时间。 |

## 人工可读摘要

- 公共库版本：`{{SKILLKIT_VERSION}}`
- 上次链接校验：`{{GENERATED_AT_ISO}}`
