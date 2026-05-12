# pattern-human-confirmation-rule

## 适用主体

执行 `extract-code-pattern`、审阅或维护 `{projectRoot}/.ai/pattern-packs/{codeTypeId}/**` 的 Agent 与人类维护者。

## 规则陈述

### Agent 不可自动完成的门控

1. **不得**将任一 Pattern Pack 文件（`pattern.md`、`validator.md`、`examples.md`、`docs.md`）的 front matter 中 `reviewedByHuman` 设为 `true`。仅人类在审阅仓库中实际内容后可手动改为 `true`。
2. **不得**将上述文件的 `status` 设为 `active` 作为 `extract-code-pattern` 的产出；本 Skill 初始值必须为 `status: draft`。
3. **不得**在正文或 front matter 中声称「已由人类审阅」或同等含义，除非人类已在外部流程中完成修改。

### 与 `needsHumanConfirmation` 的联动

4. 当 `needsHumanConfirmation: true` 时，Pack 与 `pattern-extraction-status.md` 须在「待人工确认项」中列出需人类决策的具体问题（空泛提示不足）。
5. Agent **不得**因 `needsHumanConfirmation: true` 而跳过写入草稿；应写入草稿并醒目标注待确认。

### 人类审阅前的使用边界

6. 在 `reviewedByHuman` 仍为 `false` 时，任何 Agent 应将 Pack 视为**草稿**：不得将其作为唯一依据对生产源码做大范围自动改写；若需改代码，仍须遵守 `read-source-before-edit-rule` 与项目流程。

## 须同时遵守的其它 Rule

- `rules/pattern/pattern-extraction-rule.md`
- `rules/pattern/pattern-pack-structure-rule.md`
- `rules/base/frontmatter-format-rule.md`
- `rules/base/source-of-truth-rule.md`

## 与其它 Rule 的关系

- 与 `code-type-detection-rule` 中「推断与人工确认标记」精神一致，但本规则专门约束 **Pattern Pack** 的人类审阅字段与 `status` 晋升。
