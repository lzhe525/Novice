# doc-impact-after-code-change-rule

## 适用主体

执行 `skills/docs/check-doc-impact-after-change.md` 的 Agent，以及编排代码变更后文档门禁的人类。

## 规则陈述

1. **触发**：在业务源码或协作流程已发生变更、且项目已存在 `{projectRoot}/.ai/` 时，应优先通过 `check-doc-impact-after-change` 产出结构化影响结论，再决定是否调用 `update-docs-after-change`。与 `create-code-by-pattern` 编排时，建议在改码并落盘 `pattern-code-generation-result.md` 之后执行本检查 Skill。
2. **变更事实来源优先级**（须在 `doc-impact-report.md` 的「证据与读取范围」中写明实际采用了哪几层）：
   - **P1**：`.ai/reports/pattern-code-generation-result.md`、`.ai/reports/pattern-code-generation-plan.md`、`.ai/state/pattern-code-generation-status.md`（若存在且与本轮相关）。
   - **P2**：编排或用户显式给出的报告路径，须位于 `{projectRoot}` 下；只读；缺失则记为缺口，**不得虚构**变更列表。
   - **P3**：版本控制弱证据（例如 `git diff --name-only` / `git status --short`）：仅当 P1/P2 无法还原已修改文件路径时使用，且须在报告中将据此得出的结论标为 **`uncertain` 风险升高**，并列入「不确定项」。
3. **七类判断**（每项取值 `yes` / `no` / `uncertain`）：是否需更新项目文档（`.ai/docs/project/**`）、模块文档（`.ai/docs/modules/**`）、文件文档（`.ai/docs/files/**`）、索引（`.ai/indexes/**`）、Pattern Pack（`.ai/pattern-packs/**`）、项目本地 Skill（`.ai/skills/project-local/**`）。每项须附一句理由与至少一条证据路径（相对 `{projectRoot}`，保持原文）。
4. **`uncertain` 触发条件**（示例，非穷尽）：变更路径无法从 P1/P2 解析；`moduleId` / `codeTypeId` 与索引或 Pack 不一致；文档与磁盘事实明显冲突但未深读源码；触及 `danger-zones` 或未在 Pack 中声明的联动范围。
5. **只读边界**：本规则所指的检查阶段**不得**修改业务源码，**不得**直接改写 `.ai/docs/**`、`.ai/indexes/**`、`.ai/pattern-packs/**`、`.ai/skills/project-local/**`（检查 Skill 仅写入报告与状态两文件，见 `project-local-output-rule`）。

## 允许路径

- 检查 Skill 的产出：`.ai/reports/doc-impact-report.md`、`.ai/state/doc-impact-status.md`（须与对应 Skill 一致）。

## 禁止事项

- 无证据将七类判断全部标为 `yes` 或全部标为 `no`。
- 将推测写为确定事实（与 `human-readable-doc-rule`、`source-of-truth-rule` 冲突）。

## 与其它 Rule 的关系

- 落实 `project-local-output-rule` 的写入白名单。
- 与 `doc-update-policy-rule`、`doc-sync-scope-rule`、`doc-staleness-rule` 衔接：检查只产出判断与建议清单；定点更新由 `update-docs-after-change` 执行。
- 遵守 `agent-context-budget-rule`：路径与文件列表须有上限策略并在报告中说明截断。
