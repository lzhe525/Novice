# SKILL_ROUTER

本文件位于**公共 Skill 库** `{skillLibraryRoot}/routers/SKILL_ROUTER.md`，供 Agent 在已解析 `{skillLibraryRoot}` 后做任务到 Skill 的路径映射。本 Router **不得**复制到业务项目的 `.ai/` 中；项目内应通过 `.ai/entry/SKILLKIT_LINK.md` 回到公共库路径后读取。

表中「Skill 路径」均相对于 `{skillLibraryRoot}`，使用正斜杠。

## Bootstrap

| 任务意图（中文） | Skill 路径 | 说明 |
|------------------|------------|------|
| 将公共库接入当前项目、建立 `.ai/` 与极薄 `AGENTS.md` | `skills/bootstrap/load-skill-library.md` | 编排子 Skill，完成 Bootstrap 最小闭环 |
| 仅创建或更新项目根 `AGENTS.md`（极薄入口或 AI-SKILLKIT 区块） | `skills/bootstrap/create-or-update-agents-md.md` | 不触碰 `.ai/` 内其它文件 |
| 仅初始化项目 `.ai/entry/`、`.ai/state/`、`.ai/config/` 及模板实例化文件 | `skills/bootstrap/initialize-project-ai-context.md` | 不创建 `docs/`、`indexes/` 等非 MVP 目录 |
| 检查 Bootstrap 就绪情况并引导人工确认语言策略 | `skills/bootstrap/check-project-readiness.md` | 写入 `.ai/state/readiness.md`，更新状态 |

## Scan（第二阶段）

| 任务意图（中文） | Skill 路径 | 说明 |
|------------------|------------|------|
| 项目级 AI 扫盘（overview + deep-dive + 模块索引 + 状态） | `skills/scan/scan-project-by-ai.md` | 写入 `.ai/docs/project/overview.md`、`.ai/docs/project/deep-dive.md`、`.ai/indexes/module-index.md`、`.ai/state/scan-status.md` |
| 紧凑项目画像 + 证据表 | `skills/scan/generate-project-adapter-by-ai.md` | 写入 `.ai/docs/project/adapter.md`、`.ai/docs/project/adapter-evidence.md` |
| 模块级 AI 扫盘（五件套） | `skills/scan/scan-module-by-ai.md` | 写入 `.ai/docs/modules/<moduleRootRelative>/{overview,deep-dive,constraints,change-guide,risk-points}.md`；可选合并更新 `.ai/indexes/module-index.md` |
| 单文件 AI 扫盘（普通 / 重点） | `skills/scan/scan-file-by-ai.md` | 写入 `.ai/docs/files/...` 下对应 `.md`；`fileImportance` 为 `normal` 或 `critical` |

## 未提供的能力

Pattern、Placement、`skills/develop/**`、`skills/review/**`、框架升级类 Skill 等**不在**本表范围内；Agent 应停止或改用项目本地 `.ai/skills/project-local/`（若存在）。
