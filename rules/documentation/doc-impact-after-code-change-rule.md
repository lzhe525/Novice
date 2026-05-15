# doc-impact-after-code-change-rule

## 适用主体

执行代码修改方案的 Agent、执行 `skills/docs/check-doc-impact-after-change.md` 的 Agent，以及编排文档同步建议门禁的人类。

## 规则陈述

1. **建议门禁**：Agent 在确定代码修改方案、正式写入业务源码前，必须做一次轻量的「文档同步影响预判」。预判不要求写文件、不阻断改码，但须由 Agent 自行判断本轮方案是否预计影响项目文档、模块文档、文件文档、索引、Pattern Pack 或项目本地 Skill，并在修改方案、计划报告或最终回复中说明。
2. **正式检查 Skill 的触发**：`check-doc-impact-after-change` 是可选加强动作，不是每次改码后的强制步骤。当 Agent 预判影响明显、范围较大、文档疑似过期、实际改动超出原方案，或用户明确要求生成正式报告时，才应通过该 Skill 产出结构化影响结论，再决定是否调用 `update-docs-after-change`。与 `create-code-by-pattern` 编排时，优先在计划中记录预判；若实际应用源码后发现范围扩大，再建议或执行正式检查。
3. **预判类别**：轻量预判须覆盖六类对象：项目文档（`.ai/docs/project/**`）、模块文档（`.ai/docs/modules/**`）、文件文档（`.ai/docs/files/**`）、索引（`.ai/indexes/**`）、Pattern Pack（`.ai/pattern-packs/**`）、项目本地 Skill（`.ai/skills/project-local/**`）。每类可取「不影响 / 可能影响 / 需要同步 / 不确定」，并附一句理由。
4. **预判后的处置**：若全部为「不影响」，可继续改码且不额外更新 `.ai/` 文档；若存在「可能影响」「需要同步」或「不确定」，Agent 可选择轻量同步相关文档、标记文档可能过期，或建议运行 `check-doc-impact-after-change` 生成正式报告。同步写入仍须遵守 `doc-sync-scope-rule` 与 `doc-update-policy-rule`。
5. **变更事实来源优先级**（仅在执行 `check-doc-impact-after-change` 并写入 `doc-impact-report.md` 时适用；须在报告「证据与读取范围」中写明实际采用了哪几层）：
   - **P1**：`.ai/reports/pattern-code-generation-result.md`、`.ai/reports/pattern-code-generation-plan.md`、`.ai/state/pattern-code-generation-status.md`（若存在且与本轮相关）。
   - **P2**：编排或用户显式给出的报告路径，须位于 `{projectRoot}` 下；只读；缺失则记为缺口，**不得虚构**变更列表。
   - **P3**：版本控制弱证据（例如 `git diff --name-only` / `git status --short`）：仅当 P1/P2 无法还原已修改文件路径时使用，且须在报告中将据此得出的结论标为 **`uncertain` 风险升高**，并列入「不确定项」。
6. **七类正式判断**（仅在正式报告中使用；每项取值 `yes` / `no` / `uncertain`）：是否需更新项目文档（`.ai/docs/project/**`）、模块文档（`.ai/docs/modules/**`）、文件文档（`.ai/docs/files/**`）、索引（`.ai/indexes/**`）、Pattern Pack（`.ai/pattern-packs/**`）、项目本地 Skill（`.ai/skills/project-local/**`）。每项须附一句理由与至少一条证据路径（相对 `{projectRoot}`，保持原文）。
7. **`uncertain` 触发条件**（示例，非穷尽）：变更路径无法从 P1/P2 解析；`moduleId` / `codeTypeId` 与索引或 Pack 不一致；文档与磁盘事实明显冲突但未深读源码；触及 `danger-zones` 或未在 Pack 中声明的联动范围。
8. **只读边界**：正式检查阶段**不得**修改业务源码，**不得**直接改写 `.ai/docs/**`、`.ai/indexes/**`、`.ai/pattern-packs/**`、`.ai/skills/project-local/**`（检查 Skill 仅写入报告与状态两文件，见 `project-local-output-rule`）。

## 允许路径

- 正式检查 Skill 的产出：`.ai/reports/doc-impact-report.md`、`.ai/state/doc-impact-status.md`（须与对应 Skill 一致）。

## 禁止事项

- 无证据将七类判断全部标为 `yes` 或全部标为 `no`。
- 将推测写为确定事实（与 `human-readable-doc-rule`、`source-of-truth-rule` 冲突）。
- 将建议门禁解释为“每次改码后必须生成 `doc-impact-report.md`”。

## 与其它 Rule 的关系

- 落实 `project-local-output-rule` 的写入白名单。
- 与 `doc-update-policy-rule`、`doc-sync-scope-rule`、`doc-staleness-rule` 衔接：预判只形成 Agent 判断；正式检查只产出判断与建议清单；定点更新由 `update-docs-after-change` 执行。
- 遵守 `agent-context-budget-rule`：路径与文件列表须有上限策略并在报告中说明截断。
