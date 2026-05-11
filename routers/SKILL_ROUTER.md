# SKILL_ROUTER（MVP）

本文件位于**公共 Skill 库** `{skillLibraryRoot}/routers/SKILL_ROUTER.md`，供 Agent 在已解析 `{skillLibraryRoot}` 后做任务到 Skill 的路径映射。本 Router **不得**复制到业务项目的 `.ai/` 中；项目内应通过 `.ai/entry/SKILLKIT_LINK.md` 回到公共库路径后读取。

表中「Skill 路径」均相对于 `{skillLibraryRoot}`，使用正斜杠。

| 任务意图（中文） | Skill 路径 | 说明 |
|------------------|------------|------|
| 将公共库接入当前项目、建立 `.ai/` 与极薄 `AGENTS.md` | `skills/bootstrap/load-skill-library.md` | 编排子 Skill，完成 Bootstrap 最小闭环 |
| 仅创建或更新项目根 `AGENTS.md`（极薄入口或 AI-SKILLKIT 区块） | `skills/bootstrap/create-or-update-agents-md.md` | 不触碰 `.ai/` 内其它文件 |
| 仅初始化项目 `.ai/entry/`、`.ai/state/`、`.ai/config/` 及模板实例化文件 | `skills/bootstrap/initialize-project-ai-context.md` | 不创建 `docs/`、`indexes/` 等非 MVP 目录 |
| 检查 Bootstrap 就绪情况并引导人工确认语言策略 | `skills/bootstrap/check-project-readiness.md` | 写入 `.ai/state/readiness.md`，更新状态 |

若任务超出上表（例如扫盘、Pattern、Placement），当前 MVP **未提供**对应 Skill；Agent 应停止并告知需后续版本能力。
