---
libraryVersion: "{{SKILLKIT_VERSION}}"
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
| `linkedAt` | 最近一次成功完成 `load-skill-library` 或链接校验的 UTC ISO8601 时间。 |
| `projectState` | MVP 建议取值：`loaded`、`configuring`、`blocked`（语言策略未人工确认等）。 |
| `lastCheck` | 最近一次执行 `check-project-readiness` 的时间。 |

## 人工可读摘要

- 公共库版本：`{{SKILLKIT_VERSION}}`
- 上次链接校验：`{{GENERATED_AT_ISO}}`
