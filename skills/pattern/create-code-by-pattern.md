---
status: active
routeEnabled: true
description: 基于已激活 Pattern Pack 生成或修改同类代码。
triggerWhen:
  - Pattern 生成
  - 按 Pattern 改码
  - codeTypeId 实现
---
# Skill：create-code-by-pattern

## 1. 目的

在**项目本地已激活（`active`）**的 Pattern Pack 前提下，基于 `{userRequirement}` 对**同类代码**做受控的**计划先行**生成或修改：先落盘变更计划，在 `apply-after-confirmation` 下经人类确认后再改源码；改后按 Pack 内 `validator.md` 执行检查清单，并写入结果报告与状态文件。

## 2. 适用场景

- 某 `codeTypeId` 已完成 `validate-pattern-pack` 与 `activate-pattern-pack`，需在**声明过的联动文件**范围内新增或调整实现。
- 人类希望先看**变更计划**再决定是否落码（`plan-only`），或已审阅计划并授权执行（`apply-after-confirmation`）。
- 与 Placement、Develop、Review、框架升级类流程**无关**的本 Skill 单独编排场景。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根目录。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；用于读取 Rule 与模板，**不得**写入生成产物至此树。 |
| `{codeTypeId}` | Pattern Pack 目录名；须为安全标识（建议小写字母、数字、连字符）。**禁止**含 `/`、`\\`、`..`、盘符或空字符串。 |
| `{moduleId}` | 逻辑模块标识；安全 slug，**禁止**含 `/`、`..`、盘符等。 |
| `{userRequirement}` | 自然语言需求描述（中文或项目约定语言）。 |
| `{mode}` | **`plan-only`**：只产出计划与报告，**不**改业务源码。 **`apply-after-confirmation`**：产出计划后，仅在人类确认通过且门禁满足时修改源码并跑 validator。 |
| `{planApplyConfirmation}` | 当 `{mode}` 为 `apply-after-confirmation` 且本轮**将要**写入业务源码时**必填**：开发者给出的确认**原文字符串**；判定口径见 `pattern-apply-confirmation-rule.md`。`plan-only` 或未进入改码步骤时省略。 |

## 4. 输出

| 路径（相对 `{projectRoot}/.ai/`） | 模板 |
|----------------------------------|------|
| `reports/pattern-code-generation-plan.md` | [`templates/reports/pattern-code-generation-plan.template.md`](../../templates/reports/pattern-code-generation-plan.template.md) |
| `reports/pattern-code-generation-result.md` | [`templates/reports/pattern-code-generation-result.template.md`](../../templates/reports/pattern-code-generation-result.template.md) |
| `state/pattern-code-generation-status.md` | [`templates/state/pattern-code-generation-status.template.md`](../../templates/state/pattern-code-generation-status.template.md) |

在 `apply-after-confirmation` 且确认与门禁均通过时，另包括 **Pattern 声明范围内**的业务源码文件的新增或修改（路径因项目而异，须写入计划与结果报告清单）。

执行完成后应在回复中列出上述**三个** `.ai/` 产出路径（相对 `.ai/`）及（若有）已修改源码的相对 `{projectRoot}` 路径列表。

## 5. 前置条件

- 已解析 `{skillLibraryRoot}` 与 `{projectRoot}`。
- `{projectRoot}/.ai/` 存在；若不存在则中止（见 §13）。
- 可读权威 Rule：  
  `{skillLibraryRoot}/rules/pattern/pattern-application-rule.md`、  
  `{skillLibraryRoot}/rules/pattern/pattern-generated-code-validation-rule.md`、  
  `{skillLibraryRoot}/rules/pattern/pattern-apply-confirmation-rule.md`。
- 建议只读加载：  
  `{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`、  
  `{skillLibraryRoot}/rules/base/source-of-truth-rule.md`、  
  `{skillLibraryRoot}/rules/base/read-source-before-edit-rule.md`、  
  `{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、  
  `{skillLibraryRoot}/rules/base/project-local-output-rule.md`、  
  `{skillLibraryRoot}/rules/documentation/agent-context-budget-rule.md`、  
  `{skillLibraryRoot}/rules/pattern/pattern-validation-rule.md`（用于 `active` / `reviewedByHuman` 语义对齐）。
- §4 所列三个模板文件须存在；缺失则中止（见 §13）。

## 6. 执行模式

| `mode` | 行为摘要 |
|--------|----------|
| `plan-only` | 完成读源、门禁检查、生成并写入 **计划** 与 **结果**、更新 **状态**；**不**修改业务源码。 |
| `apply-after-confirmation` | 同上先生成计划；若门禁与人类确认均满足，则**仅**修改计划所列且 Pattern 已声明范围内的源码；随后执行 `validator.md` 清单并写入结果与状态。任一门禁失败或确认不通过：**不**改源码，结果中标 `blocked` / `planned` 等。 |

## 7. 执行步骤

1. **规范化 `{codeTypeId}` / `{moduleId}`**：`trim`；若含 `..`、`/`、`\\`、盘符模式或空字符串，则**不**改源码，仅写入失败/阻塞态产出后中止（见 §13）。  
2. **首读** `{projectRoot}/AGENTS.md`（极薄入口与项目约定）。  
3. **必须**读取 `{projectRoot}/.ai/entry/AI_ENTRY.md`。  
4. **必须**读取 `{projectRoot}/.ai/state/skillkit-status.md`；按 `pattern-application-rule` 判断 `projectState: blocked` 时对「应用源码」的限制。  
5. **确认项目状态允许使用 Pattern 生成能力**：若 Bootstrap 明显未就绪（如 `.ai/entry` 缺失），中止；`blocked` 时允许继续 **仅** `.ai/` 内计划/报告/状态写入，但 **apply** 不得改源码。  
6. **必须**读取 `{projectRoot}/.ai/indexes/pattern-index.md` 与 `{projectRoot}/.ai/indexes/code-type-index.md`；若 `pattern-index.md` 不存在：`apply-after-confirmation` **禁止**改源码；`plan-only` 仍写入计划但 `overallStatus` 为 `blocked` 并在报告注明前置缺口。  
7. **必须**确认 `{projectRoot}/.ai/pattern-packs/{codeTypeId}/` 存在；否则写入失败态计划/结果后中止。  
8. **必须**确认 Pattern Pack **已 active**：读取四文件 front matter，四者 `status: active` 且 `reviewedByHuman: true`（与 `pattern-validation-rule` 检查项 7 一致）；否则不应用源码，记 **blocked**。  
9. **必须**读取：  
   `.ai/pattern-packs/{codeTypeId}/pattern.md`、  
   `.ai/pattern-packs/{codeTypeId}/validator.md`、  
   `.ai/pattern-packs/{codeTypeId}/examples.md`、  
   `.ai/pattern-packs/{codeTypeId}/docs.md`。  
10. **必须**读取模块文档：  
   `.ai/docs/modules/{moduleId}/overview.md`、  
   `.ai/docs/modules/{moduleId}/constraints.md`、  
   `.ai/docs/modules/{moduleId}/change-guide.md`、  
   `.ai/docs/modules/{moduleId}/risk-points.md`；缺失则记入计划「缺口」，不虚构。  
11. **收集联动源码路径**（来自 Pack 四文件正文，上限 **40** 条候选），并对**计划将修改或新建**的每个路径在 `{projectRoot}` 下实读或确认不存在；**禁止只根据 Pattern 文档修改代码**，源码为事实源。  
12. **生成变更计划**：用计划模板实例化并**覆盖写入** `reports/pattern-code-generation-plan.md`（占位符含 `{{CODE_TYPE_ID}}`、`{{MODULE_ID}}`、`{{MODE}}`、`{{PLAN_GENERATED_AT_ISO}}`、`{{USER_REQUIREMENT_SUMMARY}}`、`{{GATE_SUMMARY}}` 等）。  
13. 若 `{mode}` 为 **`plan-only`**：写入 `reports/pattern-code-generation-result.md`（标明未应用源码）、`state/pattern-code-generation-status.md`（如 `overallStatus: planned`）；**结束**，不改源码。  
14. 若 `{mode}` 为 **`apply-after-confirmation`**：  
    - 校验 `{planApplyConfirmation}` 符合 `pattern-apply-confirmation-rule.md`；  
    - 再确认前述允许改源码的门禁均已满足（含 `pattern-index` 存在、Pack 四文件 **active**、`projectState` 未按 Rule 禁止 apply 等）；  
    - 全部通过则**仅**按计划清单修改 Pattern 已声明范围内的源码；  
    - **禁止**修改 Pattern 未声明的无关文件、禁止写入 `{skillLibraryRoot}`、禁止未经确认的高风险文件（见 `pattern-application-rule`）。  
15. 若已修改源码：**必须**按 `validator.md` 正文清单逐项执行，并按 `pattern-generated-code-validation-rule.md` 记入结果报告。  
16. **必须**写入 `reports/pattern-code-generation-result.md`。  
17. **必须**更新 `state/pattern-code-generation-status.md`。  
18. 若代码结构变化可能影响 `.ai/docs/**`：**仅在**对话与结果报告中提示是否需要更新文档；**不要**自动大范围更新 `.ai/docs/`。

## 8. 读取范围

| 路径（相对 `{projectRoot}/`，除非注明） | 说明 |
|------------------------------------------|------|
| `AGENTS.md` | 项目入口与人类约定；**第一步读取**。 |
| `.ai/entry/AI_ENTRY.md` | 入口与约定上下文。 |
| `.ai/state/skillkit-status.md` | 库链接与 `projectState`。 |
| `.ai/indexes/pattern-index.md` | Pattern 相关索引；缺失按 §7 / §13 处理。 |
| `.ai/indexes/code-type-index.md` | 代码类型与可选 `## Pattern Pack 状态`。 |
| `.ai/pattern-packs/{codeTypeId}/pattern.md` | Pack 核心。 |
| `.ai/pattern-packs/{codeTypeId}/validator.md` | 改后检查清单。 |
| `.ai/pattern-packs/{codeTypeId}/examples.md` | 样例与路径证据。 |
| `.ai/pattern-packs/{codeTypeId}/docs.md` | 双受众说明。 |
| `.ai/docs/modules/{moduleId}/overview.md` | 模块概览。 |
| `.ai/docs/modules/{moduleId}/constraints.md` | 约束。 |
| `.ai/docs/modules/{moduleId}/change-guide.md` | 变更指引。 |
| `.ai/docs/modules/{moduleId}/risk-points.md` | 风险点。 |
| 计划将触及的**业务源码**路径 | 须在编辑前逐路径实读。 |
| `{skillLibraryRoot}/rules/pattern/pattern-application-rule.md` 等 | 见 §5。 |
| `{skillLibraryRoot}/templates/reports/pattern-code-generation-plan.template.md` 等 | 见 §4。 |

## 9. 写入位置

| 允许 |
|------|
| Pattern 指定的新文件或联动文件（**仅限** `pattern-application-rule` 允许的源码集合内）。 |
| `{projectRoot}/.ai/reports/pattern-code-generation-plan.md` |
| `{projectRoot}/.ai/reports/pattern-code-generation-result.md` |
| `{projectRoot}/.ai/state/pattern-code-generation-status.md` |

| 禁止 |
|------|
| 写入或修改 `{skillLibraryRoot}/**` |
| 修改 Pattern **未声明**的无关业务文件 |
| 在 `.ai/` **外**创建 AI 协作文档（`AGENTS.md` 除外的人类显式任务） |
| **未经确认**的高风险文件（见 `pattern-application-rule`） |
| 自动大范围更新 `.ai/docs/**`（仅允许提示） |

## 10. 禁止事项

1. **不得**在 Pack 为 `draft` 或未四文件一致 `active` 时以本 Skill 为据批量改生产源码。  
2. **不得**跳过「先写计划」步骤直接改码。  
3. **不得**在 `apply-after-confirmation` 下缺少合法 `{planApplyConfirmation}` 时改源码。  
4. **不得**仅凭 Pattern 正文推断源码内容而不实读文件。  
5. **不得**实现或调用 Placement、Develop、Review、框架升级类 Skill 职责。  
6. **不得**将具体项目私有业务代码**写死**在本 Skill 正文中作为模板。  
7. **不得**将 `reviewedByHuman` 于 Pack 内改为 `false` 或降级 `status`（本 Skill **不**修改 Pack 四文件 front matter，除非项目另有显式任务）。

## 11. 校验流程

1. 改码完成后（或未改码则跳过执行仅记 `not_run`），打开 `.ai/pattern-packs/{codeTypeId}/validator.md`。  
2. 对其正文中每一条可解析检查项：执行或核对，在 `pattern-code-generation-result.md` 的清单表中记录 **pass** / **fail** / **skipped** 与备注。  
3. 汇总 `validatorOverall`：`pass`（无 fail）、`failed`（存在 fail）、`not_run`（未改码且未跑）、`skipped`（环境原因整体未跑，须在备注说明）。  
4. 细则与记录义务见 [`{skillLibraryRoot}/rules/pattern/pattern-generated-code-validation-rule.md`](../../rules/pattern/pattern-generated-code-validation-rule.md)。

## 12. 完成标准

- [ ] 已读取 `AGENTS.md`、`.ai/entry/AI_ENTRY.md`、`.ai/state/skillkit-status.md`。  
- [ ] 已读取 `pattern-index.md` 与 `code-type-index.md`，缺失按规则落盘 **blocked** 语义。  
- [ ] 已确认 `pattern-packs/{codeTypeId}/` 存在且四文件为 **active** 协作态。  
- [ ] 已读取 Pack 四文件与模块四文档；已实读将修改的源码路径。  
- [ ] `pattern-code-generation-plan.md` 已写入。  
- [ ] `plan-only` 下未改业务源码；`apply-after-confirmation` 下仅在确认与门禁通过后改声明范围内源码。  
- [ ] 若已改码：已执行 `validator.md` 清单并写入结果表。  
- [ ] `pattern-code-generation-result.md` 与 `pattern-code-generation-status.md` 已更新。  
- [ ] 未写入公共库；未自动大范围更新 `.ai/docs/`。

## 13. 失败处理

| 情况 | 处理 |
|------|------|
| `{codeTypeId}` 或 `{moduleId}` 非法 | 不写源码；尽力写入三产出文件标明 `blocked` 与原因；否则仅在回复说明。 |
| `{projectRoot}/.ai/` 不存在 | 中止；提示先 Bootstrap；不写产出（若目录无法创建）。 |
| 权威 Rule 或任一模板缺失 | 中止；回复列出缺失的 `{skillLibraryRoot}` 路径。 |
| Pack 目录缺失或四文件缺一 / `status` 非全 `active` | **不改源码**；计划/结果/状态标 **blocked** 与缺口。 |
| `pattern-index.md` 缺失且 `mode` 为 `apply-after-confirmation` | **不改源码**；可写计划；结果说明须先补齐索引或流程。 |
| `projectState: blocked` 且 `mode` 为 `apply-after-confirmation` | **不改源码**；允许写 `.ai/` 计划与报告。 |
| `{planApplyConfirmation}` 不合法 | **不改源码**；结果中标明确认未通过。 |
| `validator.md` 存在 **fail** 项 | 结果中 `validatorOverall: failed`；**不**自动回滚；列出待修复项。 |

## 权威 Rule

受控应用范围、路径上限、blocked 语义与人类确认格式以 [`{skillLibraryRoot}/rules/pattern/pattern-application-rule.md`](../../rules/pattern/pattern-application-rule.md) 与 [`{skillLibraryRoot}/rules/pattern/pattern-apply-confirmation-rule.md`](../../rules/pattern/pattern-apply-confirmation-rule.md) 为准；改后清单记录以 [`{skillLibraryRoot}/rules/pattern/pattern-generated-code-validation-rule.md`](../../rules/pattern/pattern-generated-code-validation-rule.md) 为准。
