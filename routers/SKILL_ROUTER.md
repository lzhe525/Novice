# SKILL_ROUTER

本文件位于**公共 Skill 库** `{skillLibraryRoot}/routers/SKILL_ROUTER.md`，供 Agent 在已解析 `{skillLibraryRoot}` 后做任务到 Skill 的路径映射。本 Router **不得**复制到业务项目的 `.ai/` 中；项目内应通过 `.ai/entry/SKILLKIT_LINK.md` 回到公共库路径后读取。

表中「Skill 路径」均相对于 `{skillLibraryRoot}`，使用正斜杠。

## Router / 预检

任务开始前（代码理解/修改、调试、规划、测试、文档、Pattern、项目本地 Skill 相关），Agent **须**先完成 Skill 路由预检。预检包含两部分：先按 `check-hacf-version-compatibility` 口径做版本兼容判定（`up_to_date` 仅提醒且不写 `hacf-version-status.md`，非一致状态才写入并提醒），再做 Skill 路由决策；版本状态不阻断路由且不自动升级。

| 用途 | 相对路径 |
|------|----------|
| 任务 Skill 路由预检（版本提醒 + 路由编排） | `skills/router/select-skill-for-task.md` |
| 任务信号 → Skill / fallback | `routers/TASK_TRIGGER_MATRIX.md` |
| 预检义务 Rule | `rules/routing/skill-routing-preflight-rule.md` |
| 强制触发级别 Rule | `rules/routing/mandatory-skill-trigger-rule.md` |
| 路由启用条件 Rule | `rules/routing/skill-route-enabled-rule.md` |
| 新 Skill 注册 Rule | `rules/routing/new-skill-registration-rule.md` |
| 路由校验 Rule | `rules/routing/skill-routing-validation-rule.md` |

**细查顺序**：`TASK_TRIGGER_MATRIX`（信号匹配）→ 本表（能力登记）→ 项目本地 `.ai/indexes/skill-index.md`（**仅**登记行，不扫描目录；见 `skill-route-enabled-rule`）。

仅 `TASK_TRIGGER_MATRIX` **Available**（`status: available`）且 Skill 文件存在、`status: active`、`routeEnabled: true` 时 **必须先读 Skill 再执行**；**Planned** 仅执行 fallback，**不得**强制读取不存在的 Skill。

可选预检状态：`{projectRoot}/.ai/state/last-skill-routing.md`。版本不一致或无法确认一致时，预检还会写入或覆盖 `{projectRoot}/.ai/state/hacf-version-status.md`；版本一致时不因预检刷新该文件。

## Router（路由维护）

| 任务意图（中文） | Skill 路径 | 说明 |
|---|---|---|
| 根据当前任务选择应该使用的项目本地或公共 Skill | `skills/router/select-skill-for-task.md` | 读取项目本地 skill-index、公共 Router / Trigger Matrix，执行任务路由预检 |
| 检查公共库和项目本地 Skill 路由配置是否一致 | `skills/router/validate-skill-routing.md` | 检查 Router、Trigger Matrix、Skill metadata、routeEnabled 和路径存在性 |
| 引导注册一个新增 Skill 到 HACF 路由体系 | `skills/router/register-new-skill.md` | 检查 Skill metadata，并引导更新 Router / Trigger Matrix |

## Bootstrap

| 任务意图（中文） | Skill 路径 | 说明 |
|------------------|------------|------|
| 将公共库接入当前项目、建立 `.ai/`、极薄 `AGENTS.md` / `CLAUDE.md` / Cursor Rule、就绪检查与**框架版本兼容状态** | `skills/bootstrap/load-skill-library.md` | 编排子 Skill（含 `ensure-agent-default-entry`、`check-hacf-version-compatibility`），完成 Bootstrap 最小闭环 |
| 仅创建或更新项目根 `AGENTS.md`（极薄入口或 AI-SKILLKIT 区块） | `skills/bootstrap/create-or-update-agents-md.md` | 不触碰 `.ai/` 内其它文件 |
| 仅初始化项目 `.ai/entry/`、`.ai/state/`、`.ai/config/` 及模板实例化文件 | `skills/bootstrap/initialize-project-ai-context.md` | 不创建 `docs/`、`indexes/` 等非 MVP 目录 |
| 检查 Bootstrap 就绪情况并引导人工确认语言策略 | `skills/bootstrap/check-project-readiness.md` | 写入 `.ai/state/readiness.md`，更新状态 |
| 引导式完成 HACF 项目接入落地流程（分阶段、人工门控、维护 `onboarding-*.md`） | `skills/bootstrap/guide-project-onboarding.md` | 编排 `load-skill-library` / 子 Skill；**不**进入 Scan / Pattern / Develop；与 `load-skill-library` 相比偏「逐步向导」 |
| 根据当前项目状态推荐下一步应执行的 Skill（只读） | `skills/bootstrap/recommend-next-onboarding-step.md` | 读取 `.ai/state/onboarding-status.md` 等；**不**写入文件；可在 `guide-project-onboarding` 完成后独立调用 |
| 检查并生成各 Agent 极薄默认入口（`AGENTS.md` / `CLAUDE.md` / Cursor Rule）、更新 `agent-entry-status.md` | `skills/bootstrap/ensure-agent-default-entry.md` | 编排 `create-or-update-agents-md`；受控区块 `HACF_AGENT_DEFAULT_ENTRY`；**不**改公共库与业务源码 |
| 比对公共 HACF 与项目 `localFrameworkVersion` / 声明文件，写入 `.ai/state/hacf-version-status.md` | `skills/bootstrap/check-hacf-version-compatibility.md` | 读取 `VERSION.md`、`capabilities/hacf-capabilities.yml`、`SKILLKIT_LINK.md`、`skillkit-status.md`；**不**改公共库 |
| 生成项目本地框架升级计划（只写 `.ai/reports/hacf-version-upgrade-plan.md`） | `skills/bootstrap/plan-hacf-local-upgrade.md` | 须已有兼容检查结论；**不**自动补缺、不执行 apply |
| 在人类确认计划后仅补齐缺失框架文件（冲突写 `hacf-version-upgrade-conflicts.md`） | `skills/bootstrap/apply-hacf-local-upgrade.md` | **须**用户明确同意；遵守 `hacf-upgrade-no-overwrite-rule`；成功后写 `hacf-version-upgrade-result.md` 并刷新版本状态 |

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
| 复盘一次 `create-code-by-pattern` 的计划/结果/状态与报告涉及源码（只读） | `skills/pattern/review-pattern-code-generation.md` | 写入 `.ai/reports/pattern-code-generation-review.md`；**不**属于 `skills/review/**` 大能力 |
| 将稳定可重复的 Pattern 生成流程沉淀为项目本地 Skill 草稿 | `skills/pattern/promote-project-local-skill.md` | 写入 `.ai/skills/project-local/create-{codeTypeId}.md`（**初始** `status: draft`）与 `.ai/state/project-local-skill-promotion-status.md` |
| 人类确认后激活项目本地 Skill 并维护索引 | `skills/pattern/activate-project-local-skill.md` | 合并 `create-{codeTypeId}.md` front matter 为 `active`；更新 `.ai/indexes/skill-index.md` |

## Docs（文档影响与同步）

| 任务意图（中文） | Skill 路径 | 说明 |
|------------------|------------|------|
| 正式判断代码变更对 `.ai/` 文档、索引、Pattern Pack、项目本地 Skill 的影响并产出报告与状态 | `skills/docs/check-doc-impact-after-change.md` | 用于显式要求正式报告、Agent 预判影响明显或不确定时；读取变更报告（优先 Pattern 三件套等）、只读变更文件与相关 `.ai/` 产物；**仅**写入 `.ai/reports/doc-impact-report.md`、`.ai/state/doc-impact-status.md`；**不**改源码、**不**直接改其它文档 |
| 依据 `doc-impact-report` 中的建议清单定点更新 `.ai/` 内文档（不含业务源码与公共库） | `skills/docs/update-docs-after-change.md` | **须**在 `check-doc-impact-after-change` 之后或已有有效报告时执行；**仅**更新清单列出的 `.ai/docs/**`、`.ai/indexes/**`、`.ai/pattern-packs/**`、`.ai/skills/project-local/**` 及 `doc-impact-status.md`；禁止全量重扫式重写；`uncertain` 项默认跳过 |
| 对照 `.ai` 文档与索引拷问、对齐方案；可选将纪要写入 `design-grill-notes.md` | `skills/docs/grill-with-project-docs.md` | 单问单答；默认对话产出；**仅**在用户明确要求固化时写入 `.ai/docs/project/design-grill-notes.md`；不改业务源码与扫盘主文档全文 |

普通改码流程采用建议门禁：Agent 在确定修改方案后、正式写入源码前先做文档同步影响预判；若预判或实际结果显示影响明显、范围较大或存在不确定项，再编排 `check-doc-impact-after-change`，之后按需 `update-docs-after-change`。

## Productivity

| 任务意图（中文） | Skill 路径 | 说明 |
|------------------|------------|------|
| 将会话上下文压缩为固定路径交接文档，供下一 Agent 续写 | `skills/productivity/create-agent-handoff.md` | 覆盖写入 `.ai/state/agent-handoff.md`；用路径引用代替重复长文；不依赖 shell 临时文件；不改业务源码与公共库 |

## Health

| 任务意图（中文） | Skill 路径 | 说明 |
|------------------|------------|------|
| 检查项目本地 `.ai/` 协作框架健康（入口、单目录约定、配置、状态、文档、索引、Pattern Pack、项目本地 Skill、风险、闭环） | `skills/health/check-project-ai-health.md` | 只读检查；写入 `.ai/reports/project-ai-health-report.md`、`.ai/state/project-ai-health-status.md`；不修复、不改源码、不改公共库 |

## State / 本地化

| 任务意图（中文） | Skill 路径 | 说明 |
|------------------|------------|------|
| 根据 active Pattern Pack、项目本地 Skill 与 `skill-index` 更新本地化等级与 SkillKit 摘要字段 | `skills/state/evaluate-localization-level.md` | 写入 `.ai/state/localization-level.md`；合并更新 `.ai/state/skillkit-status.md` 中 `localizationLevel` / `localizationEvaluatedAt` |

## 未提供的能力

Placement、`skills/develop/**`、`skills/review/**` 等**不在**本表范围内（本表已列出的 Pattern / 生成复盘 / 项目本地 Skill / **框架跨版本对齐**三条 Bootstrap Skill 除外）；`review-pattern-code-generation` 仅为 **Pattern 代码生成复盘**，**不是** `skills/review/**`；Agent 应停止或改用项目本地 `.ai/skills/project-local/`（若存在）。
