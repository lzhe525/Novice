# hacf-local-upgrade-rule

## 适用主体

执行 `skills/bootstrap/plan-hacf-local-upgrade.md` 或 `skills/bootstrap/apply-hacf-local-upgrade.md` 的 Agent；阅读 `.ai/reports/hacf-version-upgrade-plan.md` 的人类维护者。

## 规则陈述

1. **顺序**：须先存在有效比对结论（`hacf-version-status.md`）；**计划**（`plan-hacf-local-upgrade`）**先于** **执行**（`apply-hacf-local-upgrade`）；禁止跳过计划直接批量写 `.ai/`。
2. **人工门控**：`apply-hacf-local-upgrade` **必须**取得用户明确同意（原文引用或清晰复述用户指令）后方可写盘；不得在对话中默认「用户已同意」。
3. **范围**：执行阶段**仅**补齐计划中列为可自动补缺的**框架**文件与目录；**不得**覆盖 `hacf-upgrade-no-overwrite-rule` 所列禁止路径；**不得**修改业务源码。
4. **与 onboarding 的关系**：`guide-project-onboarding` 在版本为 `outdated` / `incompatible` / `unknown` 时，须在五段输出中提示下一步 Skill，**不得**在同一轮内自动执行 `apply-hacf-local-upgrade`。
5. **产物**：冲突须写入 `.ai/reports/hacf-version-upgrade-conflicts.md`；成功须写入 `.ai/reports/hacf-version-upgrade-result.md` 并刷新 `hacf-version-status.md` 与 `skillkit-status.md` 中 `localFrameworkVersion`（及与公共对齐的 `libraryVersion`，由 `apply` Skill 定义）。

## 须同时遵守的其它 Rule

- `rules/bootstrap/hacf-upgrade-no-overwrite-rule.md`
- `rules/bootstrap/hacf-version-compatibility-rule.md`
- `rules/bootstrap/onboarding-human-confirmation-rule.md`（若与 onboarding 并发）
- `rules/base/project-local-output-rule.md`

## 禁止事项

- 将 `language-policy.md` 的 `reviewedByHuman` 代为改为 `true`。
- 覆盖已存在的框架文件内容以「对齐模板」（须记冲突并跳过，见 `hacf-upgrade-no-overwrite-rule.md`）。
