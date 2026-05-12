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
| 项目级 AI 扫盘（overview + deep-dive + 索引 + 状态） | `skills/scan/scan-project-by-ai.md` | 写入 `.ai/docs/project/overview.md`、`.ai/docs/project/deep-dive.md`、`.ai/indexes/module-index.md`、`.ai/indexes/directory-index.md`、`.ai/indexes/file-index.md`、`.ai/state/scan-status.md` |
| 紧凑项目画像 + 证据表 | `skills/scan/generate-project-adapter-by-ai.md` | 写入 `.ai/docs/project/adapter.md`、`.ai/docs/project/adapter-evidence.md` |
| 模块级 AI 扫盘（五件套） | `skills/scan/scan-module-by-ai.md` | 写入 `.ai/docs/modules/<moduleId>/{overview,deep-dive,constraints,change-guide,risk-points}.md`；可选合并更新 `.ai/indexes/module-index.md`、`.ai/indexes/directory-index.md`、`.ai/indexes/file-index.md` |
| 单文件 AI 扫盘（普通 / 重点） | `skills/scan/scan-file-by-ai.md` | 写入 `.ai/docs/files/...` 下对应 `.md`；`fileImportance` 为 `normal` 或 `critical` |

## Pattern（第三阶段）

| 任务意图（中文） | Skill 路径 | 说明 |
|------------------|------------|------|
| 模块内识别候选代码类型并写入 `code-types.md` 与索引 | `skills/pattern/detect-code-types-by-ai.md` | 写入 `.ai/docs/modules/<moduleId>/code-types.md`、`.ai/indexes/code-type-index.md`、`.ai/state/code-type-detection-status.md` |
| 根据代码类型识别结果，在 `.ai/pattern-packs/{codeTypeId}/` 创建 Pattern Pack **草稿**（四文件，模板骨架） | `skills/pattern/create-pattern-pack-draft.md` | 读取 `docs/modules/<moduleId>/code-types.md` 与 `indexes/code-type-index.md`；不深读源码；冲突时不覆盖已有主文件，写入 `state/pattern-pack-draft-conflict-{codeTypeId}.md` |
| 基于 `code-types.md` 与样例源码抽象 Pattern Pack **草稿**（不写业务源码、不执行代码生成） | `skills/pattern/extract-code-pattern.md` | 深读样例与模块文档；写入 `.ai/pattern-packs/{codeTypeId}/pattern.md`、`validator.md`、`examples.md`、`docs.md` 及 `.ai/state/pattern-extraction-status.md`；已存在 Pack 时须合并或写冲突说明，禁止整文件静默覆盖；与 `create-pattern-pack-draft` 同路径时需约定执行顺序 |
| 校验 `.ai/pattern-packs/{codeTypeId}/` 下 Pattern Pack 草稿是否结构完整、可进入人工确认 | `skills/pattern/validate-pattern-pack.md` | 只读 Pack 四文件与前置上下文；写入 `.ai/state/pattern-validation-status.md`、`.ai/reports/pattern-validation-report.md`；不激活 Pattern、不改源码 |
| 在人工确认后将某 `codeTypeId` 的 Pattern Pack 从 draft 激活为 active | `skills/pattern/activate-pattern-pack.md` | 合并更新 `.ai/pattern-packs/{codeTypeId}/` 下四篇 `*.md` 的 front matter；更新 `.ai/indexes/code-type-index.md` 中 `## Pattern Pack 状态`；写入 `.ai/state/pattern-activation-status.md`；不生成业务代码 |
| 基于已激活 Pattern Pack 生成或修改同类代码 | `skills/pattern/create-code-by-pattern.md` | 读取 active Pattern Pack、源码样例和 validator；先生成计划，确认后受控修改源码 |

## 未提供的能力

Placement、`skills/develop/**`、`skills/review/**`、框架升级类 Skill 等**不在**本表范围内（本表已列出的 Pattern 能力除外）；Agent 应停止或改用项目本地 `.ai/skills/project-local/`（若存在）。
