# onboarding-human-confirmation-rule

## 适用主体

执行 `guide-project-onboarding`、`recommend-next-onboarding-step` 的 Agent；填写或审阅 `{projectRoot}/.ai/state/onboarding-questions.md` 的人类。

## 规则陈述

### 缺信息与猜测

1. 当**未**从用户处获得明确答复时，Agent **必须停止**推进依赖该信息的阶段；不得猜测 `{projectRoot}`、`{skillLibraryRelativePath}`、硬约束、高风险区域、目录策略等关键事实。
2. 对「是否已执行某子 Skill」类事实，须以仓库内文件存在性与内容为准；不得仅凭对话声称已完成。

### 语言策略与人工字段

3. **禁止**在未由人类修改文件的前提下，将 `{projectRoot}/.ai/config/language-policy.md` 中 `reviewedByHuman` 写为 `true`，或伪造 `lastReviewedAt`、`confidence` 等审阅字段。
4. `readiness.md` 中 R6 及 `skillkit-status.md` 中 `projectState` 以 `check-project-readiness` 与真实文件为准；Agent 不得在摘要中声称「已就绪」若 R1–R6 仍存在 `fail`。

### 清单与问题文件

5. `onboarding-checklist.md` 中的勾选必须与事实一致；不得为人类未确认的步骤打勾。
6. `onboarding-questions.md` 中「人类答复摘要」须在人类提供答复（或明确授权摘录）后写入；不得编造答复。

### 与其它 Rule 的关系

- `rules/bootstrap/onboarding-stepwise-rule.md`（分阶段与五段输出）。
- `rules/pattern/pattern-human-confirmation-rule.md`（精神一致：门控与禁止代审；本规则专用于 **Bootstrap 接入向导** 与 onboarding 状态文件）。
- `rules/base/frontmatter-format-rule.md`。
