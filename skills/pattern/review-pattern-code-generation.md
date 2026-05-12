# Skill：review-pattern-code-generation

## 1. 目的

在**不修改** `create-code-by-pattern` 已落盘产物与业务源码的前提下，对一次受控代码生成进行**只读复盘**：对照计划、结果与状态文件，并实读报告中涉及的源码路径，输出独立复盘报告 `pattern-code-generation-review.md`，供后续沉淀项目本地 Skill 或改进 Pattern 流程使用。

## 2. 适用场景

- 已完成一轮 `create-code-by-pattern`（`plan-only` 或 `apply-after-confirmation`），已存在 `pattern-code-generation-plan.md`、`pattern-code-generation-result.md` 与 `pattern-code-generation-status.md`。
- 人类或 Agent 需结构化记录：计划与结果是否一致、validator 执行情况、门禁与确认是否合规、残留风险与建议（**不**在本 Skill 中改码或改已有三件套报告正文）。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根目录。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；**仅用于读取** Rule，**不得**写入。 |
| `{codeTypeId}` | 与本轮生成一致的代码类型标识；须为安全 slug：**不得**含 `/`、`\\`、`..`、盘符或空字符串。 |
| `{moduleId}` | 逻辑模块标识；安全 slug，**不得**含 `/`、`..`、盘符等。 |

## 4. 输出

| 路径（相对 `{projectRoot}/.ai/`） | 说明 |
|----------------------------------|------|
| `reports/pattern-code-generation-review.md` | 复盘报告；可**创建或覆盖**为本 Skill 的最新一次复盘实例。 |

报告建议结构（无独立模板时由本 Skill 约束）：

- YAML front matter：`reviewedSkill`（固定为 `create-code-by-pattern`）、`codeTypeId`、`moduleId`、`reviewGeneratedAt`（UTC ISO8601）、`overallReview`（如 `passed` / `issues` / `blocked`）。
- 正文：计划摘要、结果摘要、二者差异；`validatorOverall` 与检查项摘要；若涉及 `apply-after-confirmation`，记录确认与门禁是否在计划/结果中有据可查；从计划或结果中提取的**源码路径清单**及只读核对要点；残留风险与建议（**不**执行 Placement / Develop / `skills/review/**` / 框架升级）。

## 5. 前置条件

- 已解析 `{skillLibraryRoot}` 与 `{projectRoot}`。
- `{projectRoot}/.ai/` 存在。
- 建议只读加载：`{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、`{skillLibraryRoot}/rules/base/project-local-output-rule.md`、`{skillLibraryRoot}/rules/base/read-source-before-edit-rule.md`、`{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`。
- 若三件套中任一缺失，仍可按 §10 写入 `issues`/`blocked` 态复盘，但须在报告注明缺口。

## 6. 执行步骤

1. **规范化 `{codeTypeId}` / `{moduleId}`**：`trim`；非法则仅写入最小复盘（或仅在回复说明）并标 `blocked`（见 §10）。
2. **读取** `.ai/reports/pattern-code-generation-plan.md`、`.ai/reports/pattern-code-generation-result.md`、`.ai/state/pattern-code-generation-status.md`；**不得**改写上述文件。
3. 从计划与结果正文中解析「将修改 / 已修改 / 联动」的**业务源码路径**列表（去重，上限 **40** 条）；无法解析时于复盘中写明「路径清单不可从报告还原」。
4. 对列表中每个路径：在 `{projectRoot}` 下**只读**打开文件或确认不存在；**不得**编辑。将关键观察记入复盘（例如与计划操作类型是否一致、明显遗漏等），**禁止**大段抄写源码进复盘（仅允许短标识性摘录，遵守 `agent-context-budget-rule` 精神）。
5. 对照计划中的 `{mode}`、`{userRequirement}` 摘要（若有）与结果中的 `validatorOverall`、状态文件中的 `overallStatus` 等，记录一致性结论。
6. **写入** `.ai/reports/pattern-code-generation-review.md`（覆盖写入为最新一次本 Skill 产出）。
7. 在回复中列出该产出相对 `.ai/` 的路径。

## 7. 读取范围

| 路径（相对 `{projectRoot}/`，除非注明） | 说明 |
|------------------------------------------|------|
| `.ai/reports/pattern-code-generation-plan.md` | 只读。 |
| `.ai/reports/pattern-code-generation-result.md` | 只读。 |
| `.ai/state/pattern-code-generation-status.md` | 只读。 |
| 报告解析出的业务源码路径 | 只读实读。 |
| `.ai/pattern-packs/{codeTypeId}/validator.md` | 可选只读，用于核对结果中的检查项是否覆盖 validator 意图。 |
| `{skillLibraryRoot}/rules/**` | 仅按需只读，见 §5。 |

## 8. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/reports/pattern-code-generation-review.md`（可自动创建 `.ai/reports/`） |

| 禁止 |
|------|
| 修改 `{projectRoot}` 下**任意**业务源码 |
| 修改 `.ai/reports/pattern-code-generation-plan.md`、`.ai/reports/pattern-code-generation-result.md`、`.ai/state/pattern-code-generation-status.md` |
| 写入或修改 `{skillLibraryRoot}/**` |
| 在 `.ai/` 外创建协作文档（`AGENTS.md` 不在本 Skill 范围内） |

## 9. 禁止事项

1. **不得**以复盘为由回滚或自动修改已生成源码。  
2. **不得**将项目私有源码大段复制进公共 HACF 库；本 Skill 正文位于公共库，执行时产出**仅**在 `.ai/reports/`。  
3. **不得**调用或实现 Placement、Develop、`skills/review/**`、框架升级类流程。  
4. **不得**虚构报告中未出现的变更路径或 validator 结果。  
5. **不得**将 `pattern-human-confirmation-rule` 所指的 Pattern Pack 四文件作为本 Skill 的修改对象。

## 10. 完成标准

- [ ] 已只读读取计划、结果、状态三文件（或已在复盘中记录缺失）。  
- [ ] 已从报告抽取源码路径并尽量完成只读实读（或已说明无法抽取）。  
- [ ] `pattern-code-generation-review.md` 已写入且含 front matter 与结构化正文。  
- [ ] 未修改业务源码与三件套既有报告。  
- [ ] 未写入公共库。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| `{codeTypeId}` / `{moduleId}` 非法 | 不写或仅写极简 `blocked` 复盘；回复说明原因。 |
| `.ai/` 不存在 | 中止；提示先 Bootstrap。 |
| 三件套全部缺失 | 复盘 `overallReview: blocked`；正文列缺失路径；仍尽量创建 `pattern-code-generation-review.md`（若目录可写）。 |
| 部分报告缺失 | `overallReview: issues` 或 `blocked`；列出缺口。 |
| 源码路径超出上限 | 仅处理前 40 条并在复盘注明截断。 |
| Rule 与写入边界争议 | 以 `project-local-output-rule.md` 为写入边界；以 `read-source-before-edit-rule.md` 为只读源码义务。 |
