---
status: active
routeEnabled: false
description: 人工确认后将 Pattern Pack 从 draft 激活为 active。
triggerWhen:
  - 激活 Pattern Pack
  - activate pattern pack
---
# Skill：activate-pattern-pack

## 1. 目的

在开发者**已显式确认**、Pattern Pack **文件齐备**且 **`validate-pattern-pack` 无 blocking** 的前提下，将 `{projectRoot}/.ai/pattern-packs/{codeTypeId}/` 下四篇协作文档的 YAML front matter 从 `draft` 等预备态**激活为 `active`**，写入人类已审阅与信心级别，并同步 `.ai/indexes/code-type-index.md` 中的 **Pattern Pack 状态**小节与 `.ai/state/pattern-activation-status.md`。

**本轮不**实现 `create-code-by-pattern`、不生成或修改业务源码、不执行 Placement / Develop。

## 2. 适用场景

- 已完成 Pattern Pack 草稿（四件套）且已运行 `validate-pattern-pack`，报告 `blockingFailureCount` 为 `0`。
- 开发者已阅读 Pack 与校验报告，并愿意在对话或调用参数中提供**符合 Rule 的** `humanConfirmation` 原句。
- 需将某 `codeTypeId` 的 Pack 标记为可依赖的 `active` 协作态，供后续（未来）代码生成或审查类 Skill 消费。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；**仅用于读取** Rule 与模板路径，**不得**写入。 |
| `{codeTypeId}` | 目标代码类型标识，须为安全 slug：**不得**含 `/`、`..`、路径分隔符、盘符等。 |
| `{humanConfirmation}` | 开发者给出的**确认原文字符串**（非空）；判定口径见 `pattern-human-confirmation-rule.md`。 |

## 4. 输出

| 产物 | 说明 |
|------|------|
| 四篇 Pack 文档 front matter 更新 | `pattern.md`、`validator.md`、`examples.md`、`docs.md` 各合并写入 `status`、`reviewedByHuman`、`confidence`、`lastReviewedAt`（见 §6）。 |
| `indexes/code-type-index.md` | 在 `## Pattern Pack 状态` 表中对目标 `codeTypeId` **upsert** 一行（见 `pattern-pack-activation-rule.md`）。 |
| `state/pattern-activation-status.md` | 依据 [templates/state/pattern-activation-status.template.md](../../templates/state/pattern-activation-status.template.md) 实例化或覆盖为**最新一次**激活元数据。 |

上述路径均相对于 `{projectRoot}/.ai/`。

## 5. 前置条件

1. `{projectRoot}/.ai/` 存在。
2. 下列文件**存在且可读**：  
   `.ai/pattern-packs/{codeTypeId}/pattern.md`、  
   `.ai/pattern-packs/{codeTypeId}/validator.md`、  
   `.ai/pattern-packs/{codeTypeId}/examples.md`、  
   `.ai/pattern-packs/{codeTypeId}/docs.md`。
3. 已执行 **`validate-pattern-pack`**（由后续 HACF Skill 或项目流程提供），且 `.ai/reports/pattern-validation-report.md` 存在，且按 `pattern-pack-activation-rule.md` **无 blocking failure**（`blockingFailureCount: 0`）。
4. `{humanConfirmation}` 满足 `pattern-human-confirmation-rule.md`；不满足则**禁止**激活。
5. Agent 可读取：  
   `{skillLibraryRoot}/rules/pattern/pattern-pack-activation-rule.md`、  
   `{skillLibraryRoot}/rules/pattern/pattern-human-confirmation-rule.md`、  
   `{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`、  
   `{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、  
   `{skillLibraryRoot}/rules/base/project-local-output-rule.md`、  
   `{skillLibraryRoot}/templates/state/pattern-activation-status.template.md`。

## 6. 执行步骤

1. **校验 `codeTypeId`**：拒绝含 `..`、`/`、`\`、盘符（如 `C:`）及空字符串；非法则中止（§11），不写任何文件。
2. **校验 `humanConfirmation`**：按 `pattern-human-confirmation-rule` 执行 `trim`、长度、`codeTypeId` 字面量与句式 A/B；不通过则**立即拒绝**，不写任何文件。
3. **检查四件套路径**：确认 §5 中四个文件存在；缺一即中止（§11）。
4. **读取并解析校验报告**：打开 `.ai/reports/pattern-validation-report.md`，读取 YAML front matter 中的 `blockingFailureCount`。若不等于 `0` 或不符合 `pattern-pack-activation-rule`：中止（§11）。
5. **读取索引**：打开 `.ai/indexes/code-type-index.md`。若在主表「代码类型条目」中出现 **多于一行** 具有相同 `codeTypeId`（不同 `moduleId`）：中止（§11），提示消歧。
6. **合并更新四篇 Pack 的 front matter**（不改正文）：对每个文件仅更新或追加下列键：  
   - `status: active`  
   - `reviewedByHuman: true`  
   - `confidence: confirmed`  
   - `lastReviewedAt: <执行当日日期，YYYY-MM-DD>`  
   遵守 `frontmatter-format-rule` 与 `pattern-pack-activation-rule` 的合并策略。
7. **更新 `code-type-index.md`**：  
   - **不得**为激活而修改主表「代码类型条目」中既有 `status` / `needsHumanConfirmation` 等列的语义。  
   - 若不存在二级标题 `## Pattern Pack 状态`，在文档适当位置（建议紧接主表之后）创建该节与表头。  
   - 在 `## Pattern Pack 状态` 表中对 `{codeTypeId}` 行 upsert：`patternPackStatus` 为 `active`，`patternPackRelativePath` 为 `pattern-packs/{codeTypeId}/`，`lastActivatedAt` 为 ISO8601 UTC 字符串（与 `pattern-activation-status` front matter 一致），`activatedBySkill` 为 `activate-pattern-pack`。
8. **写入 `pattern-activation-status.md`**：用模板填充占位符 `{{ACTIVATED_AT_ISO}}`、`{{CODE_TYPE_ID}}` 等，可覆盖该状态文件为最新一次运行结果；正文摘要须反映报告 `blockingFailureCount` 与本轮写入路径。

## 7. 读取范围

| 文件 | 说明 |
|------|------|
| `.ai/pattern-packs/{codeTypeId}/pattern.md` | 读取以合并 front matter。 |
| `.ai/pattern-packs/{codeTypeId}/validator.md` | 同上。 |
| `.ai/pattern-packs/{codeTypeId}/examples.md` | 同上。 |
| `.ai/pattern-packs/{codeTypeId}/docs.md` | 同上。 |
| `.ai/reports/pattern-validation-report.md` | 读取 front matter 中 `blockingFailureCount` 等。 |
| `.ai/indexes/code-type-index.md` | 读取以检查歧义并更新 `## Pattern Pack 状态`。 |

## 8. 写入位置

| 允许写入（相对 `{projectRoot}/.ai/`） |
|----------------------------------------|
| `pattern-packs/{codeTypeId}/pattern.md`（仅 front matter 合并） |
| `pattern-packs/{codeTypeId}/validator.md`（仅 front matter 合并） |
| `pattern-packs/{codeTypeId}/examples.md`（仅 front matter 合并） |
| `pattern-packs/{codeTypeId}/docs.md`（仅 front matter 合并） |
| `indexes/code-type-index.md`（仅允许按 Rule 更新 `## Pattern Pack 状态` 及必要小节标题创建） |
| `state/pattern-activation-status.md` |

| 禁止写入 |
|----------|
| `{skillLibraryRoot}/**` |
| `{projectRoot}` 下 `.ai/` **以外**任意路径（含业务源码） |
| 其它 `codeTypeId` 的 `pattern-packs/**` |

## 9. 禁止事项

1. 在 `humanConfirmation` 未满足 `pattern-human-confirmation-rule` 时仍执行激活或写入 `confidence: confirmed` / `reviewedByHuman: true`。
2. 在 `blockingFailureCount ≠ 0` 或报告缺失有效 `blockingFailureCount` 时激活。
3. 修改业务源码或 `{skillLibraryRoot}` 内文件。
4. 执行 `create-code-by-pattern` 或任何形式的业务代码自动生成。
5. 修改与目标 `{codeTypeId}` **无关**的 Pattern Pack 文件或索引行。
6. 将用户未确认的推断规则或 Pack 标记为 `confirmed`。
7. 为激活而篡改 `code-type-index.md` 主表中代码类型识别态列的含义。

## 10. 完成标准

- [ ] `humanConfirmation` 已校验通过；未通过时无任何写入。
- [ ] 四篇 Pack 文件 front matter 已合并包含：`status: active`、`reviewedByHuman: true`、`confidence: confirmed`、`lastReviewedAt` 为当日 `YYYY-MM-DD`，YAML 合法。
- [ ] `pattern-validation-report.md` 已读且 `blockingFailureCount == 0`。
- [ ] `code-type-index.md` 中 `## Pattern Pack 状态` 表对目标 `codeTypeId` upsert 正确，且主表识别态未被违规修改。
- [ ] `pattern-activation-status.md` 已按模板写入并与本轮参数一致。
- [ ] 未修改 `.ai/` 外源码；未写入公共库。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| `codeTypeId` 非法 | 中止；提示合法 slug 规则。 |
| `humanConfirmation` 不明确或未包含 `codeTypeId` 字面量 | **拒绝激活**；不写任何文件。 |
| 四件套缺一 | 中止；列出缺失路径。 |
| `pattern-validation-report.md` 不存在 | 中止；提示先执行 `validate-pattern-pack`。 |
| `blockingFailureCount` 缺失、非整数或 `≠ 0` | **不得激活**；说明须先修复校验或报告。 |
| 主表中同一 `codeTypeId` 多行 | 中止；提示人类消歧。 |
| 模板或 Rule 缺失 | 中止；列出缺失的 `{skillLibraryRoot}` 路径。 |

**权威 Rule**：人工确认句式以 [{skillLibraryRoot}/rules/pattern/pattern-human-confirmation-rule.md](../../rules/pattern/pattern-human-confirmation-rule.md) 为准；激活条件、报告门禁、索引与 front matter 合并以 [{skillLibraryRoot}/rules/pattern/pattern-pack-activation-rule.md](../../rules/pattern/pattern-pack-activation-rule.md) 为准。
