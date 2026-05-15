# project-local-output-rule

## 适用主体

在 `{projectRoot}` 中生成或更新 AI 协作文档、状态、配置草稿的 Agent。

## 规则陈述

凡属于「AI 协作产物」（说明性文档、协作状态、为 Agent 准备的配置草稿、未来扫描生成的索引与报告等），**默认**只能写入：

`{projectRoot}/.ai/`

Bootstrap 阶段**已定义**、允许创建或更新的项目侧文件包括：

- `{projectRoot}/AGENTS.md`（极薄入口，见 `create-or-update-agents-md` Skill）
- `{projectRoot}/.ai/entry/AI_ENTRY.md`
- `{projectRoot}/.ai/entry/SKILLKIT_LINK.md`
- `{projectRoot}/.ai/state/skillkit-status.md`
- `{projectRoot}/.ai/state/readiness.md`
- `{projectRoot}/.ai/config/language-policy.md`

**Agent 默认极薄入口适配**在仅执行 `skills/bootstrap/ensure-agent-default-entry.md` 时，允许在下列路径创建或更新文档（须与该 Skill「写入位置」节一致；**仅**合并 `<!-- HACF_AGENT_DEFAULT_ENTRY:BEGIN -->` … `END` 受控区块或编排 `create-or-update-agents-md`，不得整文件覆盖用户自有内容）：

- `{projectRoot}/CLAUDE.md`（Claude Code 极薄入口）
- `{projectRoot}/.cursor/rules/hacf.mdc`（或该项目 `agent-entry-policy.md` 中 `cursorRulePath` 声明的**单文件** Cursor Project Rule，文件名默认 `hacf.mdc`；可自动创建 `.cursor/rules/` 目录）
- `{projectRoot}/.ai/state/agent-entry-status.md`
- `{projectRoot}/.ai/config/agent-entry-policy.md`

**引导式项目接入（Bootstrap）**在仅执行 `skills/bootstrap/guide-project-onboarding.md` 时，允许在下列路径创建或更新文档（须与该 Skill「写入位置」节一致）：

- `{projectRoot}/.ai/state/onboarding-status.md`
- `{projectRoot}/.ai/state/onboarding-checklist.md`
- `{projectRoot}/.ai/state/onboarding-questions.md`

**扫盘（第二阶段）**在仅执行 `skills/scan/**` 所列 Skill 时，允许在下列路径创建或更新文档（须与各 Skill「写入位置」节一致）：

- `{projectRoot}/.ai/docs/**` 下 Markdown；
- `{projectRoot}/.ai/indexes/**` 下索引类 Markdown（例如 `scan-project-by-ai` 生成的 `module-index.md`、`directory-index.md`、`file-index.md`；`scan-module-by-ai` 可合并更新上述索引）；
- `{projectRoot}/.ai/state/scan-status.md`（项目级扫盘状态，由 `scan-project-by-ai` 维护）。

**Pattern Pack 校验（第三阶段）**在仅执行 `skills/pattern/validate-pattern-pack.md` 时，允许在下列路径创建或更新文档（须与该 Skill「写入位置」节一致）：

- `{projectRoot}/.ai/state/pattern-validation-status.md`
- `{projectRoot}/.ai/reports/pattern-validation-report.md`（可自动创建 `.ai/reports/` 目录）

**Pattern 受控代码生成（第三阶段）**在仅执行 `skills/pattern/create-code-by-pattern.md` 时，允许在下列路径创建或更新文档（须与该 Skill「写入位置」节一致）：

- `{projectRoot}/.ai/reports/pattern-code-generation-plan.md`
- `{projectRoot}/.ai/reports/pattern-code-generation-result.md`
- `{projectRoot}/.ai/state/pattern-code-generation-status.md`

**Pattern 代码生成复盘**在仅执行 `skills/pattern/review-pattern-code-generation.md` 时，允许：

- `{projectRoot}/.ai/reports/pattern-code-generation-review.md`（可自动创建 `.ai/reports/`）

**项目本地 Skill 晋升**在仅执行 `skills/pattern/promote-project-local-skill.md` 时，允许：

- `{projectRoot}/.ai/skills/project-local/create-{codeTypeId}.md`（**仅**当该文件尚不存在时新建；可自动创建 `.ai/skills/project-local/`）
- `{projectRoot}/.ai/state/project-local-skill-promotion-status.md`

**项目本地 Skill 激活**在仅执行 `skills/pattern/activate-project-local-skill.md` 时，允许：

- `{projectRoot}/.ai/skills/project-local/create-{codeTypeId}.md`（**仅**合并 front matter）
- `{projectRoot}/.ai/indexes/skill-index.md`

**本地化等级评估**在仅执行 `skills/state/evaluate-localization-level.md` 时，允许：

- `{projectRoot}/.ai/state/localization-level.md`
- `{projectRoot}/.ai/state/skillkit-status.md`（**仅**合并 YAML，含 `localizationLevel`、`localizationEvaluatedAt` 等，须与该 Skill「写入位置」节一致）

**项目 AI 协作框架健康检查**在仅执行 `skills/health/check-project-ai-health.md` 时，允许在下列路径创建或更新文档（须与该 Skill「写入位置」节一致）：

- `{projectRoot}/.ai/reports/project-ai-health-report.md`（可自动创建 `.ai/reports/` 目录）
- `{projectRoot}/.ai/state/project-ai-health-status.md`

**代码变更后文档影响检查**在仅执行 `skills/docs/check-doc-impact-after-change.md` 时，允许在下列路径创建或更新文档（须与该 Skill「写入位置」节一致）：

- `{projectRoot}/.ai/reports/doc-impact-report.md`（可自动创建 `.ai/reports/` 目录）
- `{projectRoot}/.ai/state/doc-impact-status.md`

**代码变更后文档同步**在仅执行 `skills/docs/update-docs-after-change.md` 时，允许在下列路径创建或更新文档（须与该 Skill「写入位置」节一致）：

- `{projectRoot}/.ai/state/doc-impact-status.md`
- `{projectRoot}/.ai/docs/**`、`{projectRoot}/.ai/indexes/**`、`{projectRoot}/.ai/pattern-packs/**`、`{projectRoot}/.ai/skills/project-local/**` 下的 Markdown，**仅当**该相对路径已列入执行时所依据的 `{projectRoot}/.ai/reports/doc-impact-report.md` 正文「建议更新清单」表格（或该 Skill 认可的等价结构化列表）且变更类型符合 `doc-update-policy-rule`；**不得**据此 Skill 写入未列入清单的路径。

**方案与文档对齐拷问**在仅执行 `skills/docs/grill-with-project-docs.md` 时，允许在下列路径创建或更新文档（须与该 Skill「写入位置」节一致）：

- `{projectRoot}/.ai/docs/project/design-grill-notes.md`（**仅当**用户明确要求固化结论且该 Skill 已执行写盘步骤；**不得**据此 Skill 覆盖或整份重写 `overview.md`、`deep-dive.md`、`adapter.md`、`adapter-evidence.md` 等扫盘主文档）

**会话交接**在仅执行 `skills/productivity/create-agent-handoff.md` 时，允许在下列路径创建或更新文档（须与该 Skill「写入位置」节一致）：

- `{projectRoot}/.ai/state/agent-handoff.md`

**Skill 路由预检**在仅执行 `skills/router/select-skill-for-task.md` 时，允许在下列路径创建或更新文档（须与该 Skill「写入位置」节一致）：

- `{projectRoot}/.ai/state/last-skill-routing.md`（仅当 `{writeRoutingState}` 为 `true`）
- `{projectRoot}/.ai/state/hacf-version-status.md`（仅当版本兼容判定非 `up_to_date`；`up_to_date` 时不得因路由预检创建或覆盖）

**不得**在 `.ai/` 外散落上述扫盘产物。

不在上列且非用户显式要求的源码修改，**不得**新建。

## 禁止事项

- 以「方便 Agent」为由在 `{projectRoot}` 根或 `src/` 下新增与上表同类的协作文档。
- 在未引入对应 Skill 前创建 `.ai/reports/` 并写入报告；**例外**：已引入 `validate-pattern-pack` 时，仅允许写入该 Skill 明示的 `pattern-validation-report.md`；已引入 `create-code-by-pattern` 时，仅允许写入该 Skill 明示的 `pattern-code-generation-plan.md` 与 `pattern-code-generation-result.md`；已引入 `review-pattern-code-generation` 时，仅允许写入该 Skill 明示的 `pattern-code-generation-review.md`；已引入 `check-project-ai-health` 时，仅允许写入该 Skill 明示的 `project-ai-health-report.md`；已引入 `check-doc-impact-after-change` 时，仅允许写入该 Skill 明示的 `doc-impact-report.md`。

## 与其它 Rule 的关系

- 依赖 `single-ai-context-directory-rule` 的单根目录原则。
- 与 `read-source-before-edit-rule` 区分：后者针对**源码**读取；本 Rule 针对**协作文档写入位置**。
