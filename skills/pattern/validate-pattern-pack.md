# Skill：validate-pattern-pack

## 1. 目的

在**不修改业务源码、不修改 Pattern Pack 草稿、不激活 Pattern** 的前提下，校验 `{projectRoot}/.ai/pattern-packs/{codeTypeId}/` 下 Pattern Pack 是否**结构完整、字段合规、证据与引用可核查**，并写入**仅用于校验结论**的状态与报告，供判断是否可进入**人工确认**环节。

## 2. 适用场景

- 已在项目 `.ai/` 下起草某 `codeTypeId` 的 Pattern Pack（四文件：`pattern.md`、`validator.md`、`examples.md`、`docs.md`），需在合并或推广前做一次**静态校验**。
- 人类或 Agent 在「激活」或代码生成之前，需要明确的 **pass / failed / warning** 与**可提交人工确认**判定。
- 与 `detect-code-types-by-ai` 衔接：索引与模块侧 `code-types.md` 已存在时，可将本 Skill 作为 Pack 定稿前的质量闸。本 Skill **不**创建 Pack，**不**修改公共库。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根目录。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；用于读取 Rule 与模板，**不得**写入校验结果至此树。 |
| `{codeTypeId}` | Pattern Pack 目录名；须为安全标识（建议小写字母、数字、连字符）。**禁止**含 `/`、`\\`、`..`、盘符或空字符串。 |

## 4. 输出

| 路径（相对 `{projectRoot}/.ai/`） | 模板 |
|----------------------------------|------|
| `state/pattern-validation-status.md` | [`templates/state/pattern-validation-status.template.md`](../../templates/state/pattern-validation-status.template.md) |
| `reports/pattern-validation-report.md` | [`templates/reports/pattern-validation-report.template.md`](../../templates/reports/pattern-validation-report.template.md) |

执行完成后应在回复中列出上述**两个**输出路径（相对 `.ai/`）。

## 5. 前置条件

- 已解析 `{skillLibraryRoot}` 与 `{projectRoot}`。
- `{projectRoot}/.ai/` 已存在；若不存在则中止（见 §11）。
- 可读权威 Rule：`{skillLibraryRoot}/rules/pattern/pattern-validation-rule.md`。
- 建议只读加载：`{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`、`{skillLibraryRoot}/rules/base/source-of-truth-rule.md`、`{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、`{skillLibraryRoot}/rules/base/project-local-output-rule.md`、`{skillLibraryRoot}/rules/project/scan-readonly-and-artifacts-rule.md`、`{skillLibraryRoot}/rules/documentation/agent-context-budget-rule.md`。
- 若模板缺失则中止（见 §11）。

## 6. 执行步骤

1. **规范化 `{codeTypeId}`**：去除首尾空白；若含 `..`、`/`、`\\`、盘符模式（如 `^[A-Za-z]:`）或不符合项目惯例的非法字符，则**不**写入 Pack 内任何文件，仅写入失败状态与报告后中止。
2. **读取前置与上下文文件**（见 §7）：依次尝试读取；缺失则记入报告「证据与备注」，**不**虚构内容。`skillkit-status.md` 或 `AI_ENTRY.md` 缺失不阻塞校验 Pack 本身，但须在报告中标注。
3. **存在性**：判断 `{projectRoot}/.ai/pattern-packs/{codeTypeId}/` 是否存在；不存在则检查项 1 **failed**，2–15 中对四文件的依赖项标 **skipped** 并说明。
4. **四文件读取**：若目录存在，依次读取 `pattern.md`、`validator.md`、`examples.md`、`docs.md`。缺一记 **failed**。
5. **front matter 与枚举**：对每个已读 Pack 文件解析 YAML front matter；按 `pattern-validation-rule` 校验检查项 3–7（含 `draft`/`active` 与 `reviewedByHuman` 约束）。`deprecated` 的 `reviewedByHuman` 按 Rule 不强制互斥。
6. **语义结构**：按 `pattern-validation-rule` 对检查项 8–11 做标题/关键词/清单形态扫描；记录 **pass** / **failed** / **warning**。
7. **风险与确认项**：执行检查项 12–13（单样例过度归纳、人工确认声明）。
8. **路径存在性（有界）**：从四文件正文中按 Rule 收集候选相对路径，**最多 40** 条；对每个路径在 `{projectRoot}` 下验证文件或目录是否存在；记录检查项 14。超出部分未验须在报告备注「截断」。
9. **公共库污染**：按 Rule 检查项 15 扫描正文是否指示向 `{skillLibraryRoot}` 写入项目私有内容。
10. **合并索引线索（可选）**：读取 `indexes/code-type-index.md`，若存在与当前 `codeTypeId` 相关行，可在报告「证据与备注」中引用；**不得**因索引与 Pack 不一致而自动修改索引或 Pack。
11. **汇总**：若任一项为 **failed**，则 `overallResult: failed`；否则若有 **warning** 则 `overallResult: warning`；否则 `overallResult: pass`。按 Rule 计算 `canSubmitForHumanReview`（无 failed、且 6、7、15 未触发失败语义等）。
12. **写入**：确保 `{projectRoot}/.ai/state/`、`{projectRoot}/.ai/reports/` 存在；用两模板实例化并**覆盖写入** `pattern-validation-status.md` 与 `pattern-validation-report.md` 全文。占位符：`{{VALIDATED_AT_ISO}}`（UTC ISO8601）、`{{CODE_TYPE_ID}}`、`{{OVERALL_RESULT}}`、`{{CAN_SUBMIT_FOR_HUMAN_REVIEW}}`（小写 `true`/`false`）、`{{FAILED_COUNT}}`、`{{WARNING_COUNT}}`（非负整数字符串）、`{{BLOCKING_FAILURE_COUNT}}`（**与 `{{FAILED_COUNT}}` 相同**，为检查项 1–15 中结果为 **failed** 的条数，供 `activate-pattern-pack` 读取 `blockingFailureCount`）。

## 7. 读取范围

| 路径（相对 `{projectRoot}/`，除非注明） | 说明 |
|------------------------------------------|------|
| `.ai/entry/AI_ENTRY.md` | 入口与约定上下文 |
| `.ai/state/skillkit-status.md` | 库链接与项目状态摘要 |
| `.ai/config/language-policy.md` | 与文档语言策略对齐参考 |
| `.ai/indexes/code-type-index.md` | 可选交叉引用，缺失则注明 |
| `.ai/pattern-packs/{codeTypeId}/pattern.md` | Pack 核心 |
| `.ai/pattern-packs/{codeTypeId}/validator.md` | 检查清单 |
| `.ai/pattern-packs/{codeTypeId}/examples.md` | 样例与证据 |
| `.ai/pattern-packs/{codeTypeId}/docs.md` | 双受众说明 |
| `{skillLibraryRoot}/rules/pattern/pattern-validation-rule.md` | 判定口径 |
| `{skillLibraryRoot}/templates/state/pattern-validation-status.template.md` | 状态模板 |
| `{skillLibraryRoot}/templates/reports/pattern-validation-report.template.md` | 报告模板 |

**路径存在性校验**仅针对 Pack 四文件与（若引用）索引正文中出现的显式路径，**上限 40** 条待验路径，遵守 `agent-context-budget-rule`。

## 8. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/state/pattern-validation-status.md` |
| `{projectRoot}/.ai/reports/pattern-validation-report.md` |

| 禁止 |
|------|
| 修改 `{projectRoot}` 下 `.ai/pattern-packs/**` 内任意文件 |
| 修改 `{projectRoot}` 下 `.ai/**` 内除上表两文件外的其它文件（除非用户另有显式任务） |
| 写入或修改 `{skillLibraryRoot}/**` |
| 修改业务源码 |

## 9. 禁止事项

1. **不得**修改业务源码。  
2. **不得**自动激活 Pattern Pack 或生成替换业务代码。  
3. **不得**修改公共 HACF 库（`{skillLibraryRoot}`）任何文件。  
4. **不得**将 Pack 或任何文档中的 `reviewedByHuman` 改为 `true`。  
5. **不得**覆盖 Pack 正文中人类已有确认内容；本 Skill **仅**覆盖写入 §8 所列两个校验产出文件。  
6. **不得**将校验结果写入 `.ai/` 以外路径。

## 10. 完成标准

- [ ] `pattern-validation-status.md` 已写入，front matter 含 `lastPatternValidationAt`、`validationSkill: validate-pattern-pack`、`codeTypeId`、`patternPackRelativePath`、`overallResult`（`pass` / `failed` / `warning`）、`canSubmitForHumanReview`、`failedCount`、`warningCount`，YAML 合法。  
- [ ] `pattern-validation-report.md` 已写入，front matter 含 `blockingFailureCount`（与 `{{FAILED_COUNT}}` 一致、YAML 合法整数），含检查项 **1–15** 结果表、失败项清单、警告项清单，并给出**建议修复文件**（相对 `.ai/` 的 Pack 路径为主）。  
- [ ] 能明确判断 **pass / failed / warning** 及 **是否可提交人工确认**。  
- [ ] 未修改 Pack 草稿与源码；未激活 Pattern。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| `{codeTypeId}` 非法 | 写入 `overallResult: failed`、`canSubmitForHumanReview: false`；报告说明原因；不读取 Pack。 |
| `.ai/` 不存在 | 中止；回复提示先 Bootstrap；不写报告（若无法创建目录）。 |
| `pattern-validation-rule.md` 或任一模板缺失 | 中止；回复列出缺失的 `{skillLibraryRoot}` 路径。 |
| Pack 目录或必需文件缺失 | 对应检查项 **failed**；其余依赖项 **skipped**；仍写入两产出文件以便人类一次性看到缺口。 |
| 部分文件 front matter 非法 | 检查项 3 **failed**；在失败清单中列出文件相对路径。 |
| 路径校验达到 40 条上限仍有剩余 | 检查项 14 已验部分如实记录；备注「未验路径截断」**warning**。 |

## 权威 Rule

检查项 1–15 的 **failed / warning / pass / skipped** 语义与汇总规则以 [`{skillLibraryRoot}/rules/pattern/pattern-validation-rule.md`](../../rules/pattern/pattern-validation-rule.md) 为准。
