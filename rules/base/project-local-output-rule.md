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

**扫盘（第二阶段）**在仅执行 `skills/scan/**` 所列 Skill 时，允许在下列路径创建或更新文档（须与各 Skill「写入位置」节一致）：

- `{projectRoot}/.ai/docs/**` 下 Markdown；
- `{projectRoot}/.ai/indexes/**` 下索引类 Markdown（例如 `scan-project-by-ai` 生成的 `module-index.md`、`directory-index.md`、`file-index.md`；`scan-module-by-ai` 可合并更新上述索引）；
- `{projectRoot}/.ai/state/scan-status.md`（项目级扫盘状态，由 `scan-project-by-ai` 维护）。

**Pattern Pack 校验（第三阶段）**在仅执行 `skills/pattern/validate-pattern-pack.md` 时，允许在下列路径创建或更新文档（须与该 Skill「写入位置」节一致）：

- `{projectRoot}/.ai/state/pattern-validation-status.md`
- `{projectRoot}/.ai/reports/pattern-validation-report.md`（可自动创建 `.ai/reports/` 目录）

**不得**在 `.ai/` 外散落上述扫盘产物。

不在上列且非用户显式要求的源码修改，**不得**新建。

## 禁止事项

- 以「方便 Agent」为由在 `{projectRoot}` 根或 `src/` 下新增与上表同类的协作文档。
- 在未引入对应 Skill 前创建 `.ai/reports/` 并写入报告；**例外**：已引入 `validate-pattern-pack` 时，仅允许写入该 Skill 明示的 `pattern-validation-report.md`。

## 与其它 Rule 的关系

- 依赖 `single-ai-context-directory-rule` 的单根目录原则。
- 与 `read-source-before-edit-rule` 区分：后者针对**源码**读取；本 Rule 针对**协作文档写入位置**。
