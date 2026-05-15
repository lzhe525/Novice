# TASK_TRIGGER_MATRIX

本文件位于**公共 Skill 库** `{skillLibraryRoot}/routers/TASK_TRIGGER_MATRIX.md`，供 `skills/router/select-skill-for-task.md` 在任务开始前做**信号 → Skill** 匹配。表中 Skill 路径均相对于 `{skillLibraryRoot}`，使用正斜杠。

**维护约定**（见 `rules/routing/new-skill-registration-rule.md`）：

- 仅当 Skill 文件在公共库磁盘中**真实存在**、front matter 为 `status: active` 且 `routeEnabled: true` 时，方可列入 **Available Skill Triggers** 并标记 `must_invoke`。
- 尚未实现的 Skill **只能**列入 **Planned Skill Triggers**；**不得** `must_invoke`、**不得**写「必须使用」。
- 登记变更后须执行 `skills/router/validate-skill-routing.md`。

---

## Available Skill Triggers（§1 可用触发）

以下条目经存在性校验且满足 `skill-route-enabled-rule` 后，适用 `must_invoke`（项目本地 skill-index 中同意图 `active` 行优先，见 `mandatory-skill-trigger-rule`）。

| 触发场景（中文关键词示例） | Skill 路径 | status | 是否强制 | 失败 fallback |
|---------------------------|------------|--------|----------|---------------|
| 拷问、对齐、压力测试方案、术语模糊、需求不清、业务规则不完整、设计拷问 | `skills/docs/grill-with-project-docs.md` | available | must_invoke | 若文件缺失或 metadata 不满足：预检 `config_error`，建议执行 `validate-skill-routing`；**不**编造 Skill 步骤 |
| 交接、handoff、切换 Agent、会话总结、下一 Agent 续写、固化上下文 | `skills/productivity/create-agent-handoff.md` | available | must_invoke | 同上 |
| Pattern 生成、按 Pattern 改码、同类代码生成、`codeTypeId` 实现 | `skills/pattern/create-code-by-pattern.md` | available | must_invoke | 同上；项目本地 `create-{codeTypeId}.md` 且 skill-index `active` 时优先本地路径 |
| 改码后文档影响、文档是否需同步、doc impact、变更后文档判断 | `skills/docs/check-doc-impact-after-change.md` | available | must_invoke | 同上 |

---

## Planned Skill Triggers（§2 计划中触发）

以下 Skill **尚未**在公共库实现（文件不存在或仍为 `planned`）；**禁止** `must_invoke`；**禁止**写「必须使用」；**禁止** Agent 假装已读取这些路径。命中信号时**仅**执行本表 fallback 列。

| 触发场景（中文关键词示例） | Skill 路径（planned） | status | Fallback 行为 |
|---------------------------|----------------------|--------|---------------|
| bug、异常、报错、堆栈、不符合预期、复现、诊断 | `skills/develop/diagnose-bug-with-context.md` | planned | 只读相关源码、日志与用户描述；列出假设与验证步骤；请用户补充复现路径；**不**声称已执行诊断 Skill |
| 大范围功能、跨模块、竖切、拆分交付、分期上线 | `skills/planning/split-change-into-vertical-slices.md` | planned | 在对话中说明竖切原则（每片可独立演示/验证）；给出 2–3 条建议切片；**不**创建 issue、工单或 PRD 文件 |
| 轻量 PRD、变更说明、需求大纲、范围界定 | `skills/planning/create-change-prd.md` | planned | 在对话中用简短 Markdown 大纲列出目标、范围、非目标、验收标准；**默认不落盘**；仅当用户明确要求写入项目文档时才写入 `.ai/` 且须另遵 `project-local-output-rule` |
| 不理解文件/类/函数位置、从单文件看系统、zoom-out、这文件干什么的 | `skills/scan/zoom-out-from-file.md` | planned | 只读 `.ai/docs`、`.ai/indexes` 与目标文件相邻源码；口述模块关系与调用方向；若索引缺失，建议执行 `skills/scan/scan-file-by-ai.md`（若公共库存在且 routeEligible） |
| 测试方案、补测试、行为测试、测试计划、怎么测 | `skills/testing/plan-behavior-tests.md` | planned | 列出关键行为与建议测试类型（单测/集成/手工）；标注风险点；**不**声称已执行完整 TDD 或测试 Skill |

---

## §3 存在性与 metadata 校验约定

1. `select-skill-for-task` 在推荐或 `must_invoke` 之前，**必须**探测目标文件并读取 front matter：
   - 公共 Skill：`{skillLibraryRoot}/<Skill路径>`
   - 项目本地 Skill：`{projectRoot}/.ai/<skillRelativePath>`（**仅**来自 `skill-index.md` 登记行）
2. **Available** 中 `status: available` 但文件不存在 → 预检结论为 `decision: config_error`（routing error）。
3. **Available** 中文件存在但 `status` 非 `active` 或 `routeEnabled: false` → `config_error` 或排除并说明 metadata 原因。
4. **Planned** → **永不**强制读取 Skill 文件；仅执行 §2「Fallback 行为」列。
5. Agent **不得**自行脑补 planned 或不符合 `skill-route-enabled-rule` 的 Skill 执行步骤。
6. 维护者定期或变更后执行 `skills/router/validate-skill-routing.md`。

---

## §4 与其它 Router / Rule 的关系

- **信号匹配**：以本 Matrix 为任务意图第一优先（Available / Planned 分区）。
- **能力登记**：`routers/SKILL_ROUTER.md` 补充 Bootstrap、Scan、Pattern 全流程等未在本 Matrix 逐条列举的能力。
- **预检 Skill**：`skills/router/select-skill-for-task.md`
- **注册与校验**：`skills/router/register-new-skill.md`、`skills/router/validate-skill-routing.md`
- **规则**：`rules/routing/skill-routing-preflight-rule.md`、`rules/routing/mandatory-skill-trigger-rule.md`、`rules/routing/skill-route-enabled-rule.md`、`rules/routing/new-skill-registration-rule.md`、`rules/routing/skill-routing-validation-rule.md`
