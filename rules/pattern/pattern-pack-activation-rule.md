# pattern-pack-activation-rule

## 适用主体

执行 `activate-pattern-pack` 的 Agent；审阅 `.ai/pattern-packs/{codeTypeId}/` 与 `.ai/reports/pattern-validation-report.md` 的人类维护者。

## 规则陈述

仅在**人工确认**、**文件齐备**、**校验报告无 blocking** 且 **`codeTypeId` 安全**时，允许将指定 Pattern Pack 的协作文档 front matter 更新为 `active` / `reviewedByHuman: true` / `confidence: confirmed`，并同步索引与激活状态文件。其它 Pattern Pack、业务源码与公共库须保持不触碰。

## Pattern Pack 必备文件

路径均相对 `{projectRoot}/.ai/`。对给定 `codeTypeId`，下列四个文件**必须存在且可读**：

- `pattern-packs/{codeTypeId}/pattern.md`
- `pattern-packs/{codeTypeId}/validator.md`
- `pattern-packs/{codeTypeId}/examples.md`
- `pattern-packs/{codeTypeId}/docs.md`

缺失任一文件则**不得激活**。

## 校验报告与 blocking failure

激活前**必须**已执行 `validate-pattern-pack`（由 HACF 后续 Skill 或项目约定提供），并存在报告文件：

`{projectRoot}/.ai/reports/pattern-validation-report.md`

### 判定「无 blocking failure」（须全部满足）

1. 报告文件存在且可解析为 Markdown（含 YAML front matter）。
2. Front matter 中须存在键 **`blockingFailureCount`**，且值为**非负整数**。
3. **`blockingFailureCount` 必须等于 `0`**。

若 `blockingFailureCount` 缺失、非整数或为负数：视为**报告不满足激活前置**，Agent **不得激活**（须提示先修复报告或重新执行 `validate-pattern-pack`）。

若 `blockingFailureCount` **大于 `0`**：存在 **blocking failure**，**不得激活**。

### 说明

本 Rule **不**规定 `validate-pattern-pack` 如何产生报告；仅规定激活门禁读取 `blockingFailureCount` 的口径，避免主观解读正文。

## Front matter 合并策略（四个 Pack 文件）

对 `pattern.md`、`validator.md`、`examples.md`、`docs.md`：

- **仅允许**合并更新下列键（值由 `activate-pattern-pack` 定义）：`status`、`reviewedByHuman`、`confidence`、`lastReviewedAt`。
- **不得**删除或改写其它已有 front matter 键，除非其与上述键同名（此时以上述激活值为准）。
- **不得**修改 YAML 闭合符 `---` 之外的**正文**（标题与段落保持原样）。
- 合并后的 front matter 须遵守 `{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`。

激活后上述四键的语义目标为：

- `status: active`
- `reviewedByHuman: true`
- `confidence: confirmed`
- `lastReviewedAt: <YYYY-MM-DD，与执行当日一致，取自 Agent 运行环境的「今日」日期>`

## 代码类型索引（`code-type-index.md`）

主表「代码类型条目」中列 `status` / `needsHumanConfirmation` 等表示 **detect-code-types-by-ai** 的识别态；**不得**为「激活 Pattern Pack」而篡改其语义。

须在 `indexes/code-type-index.md` 中维护独立小节（若不存在则创建）：

- 二级标题固定为：`## Pattern Pack 状态`
- 其下为窄表，列：`codeTypeId` | `patternPackStatus` | `patternPackRelativePath` | `lastActivatedAt` | `activatedBySkill`

规则：

- 主键为 **`codeTypeId`**（与本 Skill 目录名一致）。
- 激活时对该表做 **upsert**：仅更新或追加**目标** `codeTypeId` 行；`patternPackStatus` 写 `active`；`patternPackRelativePath` 写 `pattern-packs/{codeTypeId}/`；`lastActivatedAt` 写 ISO8601 或 `YYYY-MM-DD`（与 Skill 一致即可，须在 Skill 中写死一种）；`activatedBySkill` 写 `activate-pattern-pack`。
- **不得**修改非目标 `codeTypeId` 行；**不得**删除人类手工注释（除非与表行冲突且 Skill 明文要求替换——本 Skill 不要求删除注释）。

### 歧义

若在「代码类型条目」主表中出现 **多行相同 `codeTypeId`**（不同 `moduleId`）：仅依据 `codeTypeId` 无法唯一对应业务含义时，Agent **不得激活**，须在回复中要求人类合并索引或消歧后再执行。

## 写入边界

- 允许写入的路径须属于 `{projectRoot}/.ai/**`，且仅限 `activate-pattern-pack` Skill 所列文件。
- **禁止**写入 `{skillLibraryRoot}/**`。
- **禁止**修改 `{projectRoot}` 下 `.ai/` 以外的文件（含业务源码）。

## 与其它 Rule 的关系

- 人工确认口径以 `pattern-human-confirmation-rule.md` 为准。
- `public-skill-library-purity-rule`、`project-local-output-rule`：项目派生物不进公共库；本阶段写入仍须在 `.ai/` 内。
