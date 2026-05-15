# mandatory-skill-trigger-rule

## 适用主体

执行 `skills/router/select-skill-for-task.md` 或在任务开始前做 Skill 路由决策的 Agent。

## 规则陈述

本规则定义**任务信号**与 **Skill 路径**之间的强制级别。完整信号关键词与 fallback 文案以 `{skillLibraryRoot}/routers/TASK_TRIGGER_MATRIX.md` 为准；存在性以磁盘探测为准。

### 强制级别

| 级别 | 含义 |
|------|------|
| `must_invoke` | Matrix §1（`status: available`）且 Skill 文件存在，且目标 Skill 满足 `skill-route-enabled-rule`（`status: active`、`routeEnabled: true`、已登记）；预检后 **必须先读该 Skill** 再执行主体任务。 |
| `suggest_fallback` | Matrix §2（`status: planned`）或 Skill 文件不存在；**不得** `must_invoke`；执行 Matrix 中 fallback 说明。 |
| `n/a` | 无匹配信号；预检结论为 `none`，普通处理。 |

### 信号 → Skill → 强制级别

| 任务信号（示例） | Skill 路径（相对 `{skillLibraryRoot}`） | Matrix 分区 | 强制级别（文件存在时） |
|------------------|----------------------------------------|-------------|------------------------|
| 需求不清、术语模糊、业务规则不完整、拷问、对齐方案 | `skills/docs/grill-with-project-docs.md` | §1 Available | `must_invoke` |
| bug、异常、报错、不符合预期、复现失败 | `skills/develop/diagnose-bug-with-context.md` | §2 Planned | `suggest_fallback` |
| 大范围功能、跨模块变更、竖切、拆分交付 | `skills/planning/split-change-into-vertical-slices.md` | §2 Planned | `suggest_fallback` |
| 轻量 PRD、变更说明、需求大纲 | `skills/planning/create-change-prd.md` | §2 Planned | `suggest_fallback` |
| 不理解某文件/类/函数在系统中的位置、从单文件拉远视角 | `skills/scan/zoom-out-from-file.md` | §2 Planned | `suggest_fallback` |
| 测试方案、补测试、行为测试计划 | `skills/testing/plan-behavior-tests.md` | §2 Planned | `suggest_fallback` |
| 会话交接、切换 Agent、总结上下文、handoff | `skills/productivity/create-agent-handoff.md` | §1 Available | `must_invoke` |
| 基于 Pattern 生成代码、按 Pattern 改码、`codeTypeId` 同类实现 | `skills/pattern/create-code-by-pattern.md` | §1 Available | `must_invoke` |
| 正式文档影响报告、运行 doc impact、生成 `doc-impact-report`、执行 `check-doc-impact-after-change` | `skills/docs/check-doc-impact-after-change.md` | §1 Available | `must_invoke` |

### 项目本地覆盖

当 `{projectRoot}/.ai/indexes/skill-index.md` 中 `## 项目本地 Skill` 表存在 `status: active` 行，且其 `skillRelativePath` 或 `codeTypeId` 与当前任务信号一致时：

- **优先** `must_invoke` 项目本地路径（相对 `{projectRoot}/.ai/`），高于上表公共 Skill。
- 项目本地 Skill 文件须通过存在性探测；不存在则回退公共同意图行，若公共为 `planned` 则 `suggest_fallback`。

### 多信号冲突

1. 项目本地 Skill > 公共 Skill。
2. 更具体意图 > 更泛化意图（例：`生成 doc-impact-report` > `文档拷问`）。
3. 若仍冲突，在预检输出 `evidence` 中列出全部命中信号，并选择 **对用户目标最直接** 的一项；必要时向用户确认一项。

### 配置错误

若 Matrix §1 标记为 `available`，但 `{skillLibraryRoot}/<Skill路径>` 与项目本地覆盖路径均不存在：

- 预检结论 **必须** 含 `decision: config_error`。
- Agent **不得** 自行编造该 Skill 的执行步骤。

## 禁止事项

- 对 §2 Planned 行使用 `must_invoke`。
- 在文件不存在时假装「已读 Skill」。
- 将普通代码修改前的文档同步预判升级为强制调用 `check-doc-impact-after-change`；预判义务由 `doc-impact-after-code-change-rule` 约束，正式报告仅在显式意图或 Agent 判断需要加强时调用。
- 将本表复制进业务项目 `.ai/` 作为 override（应始终回读公共库 Matrix）。

## 与其它 Rule 的关系

- 由 `skill-routing-preflight-rule` 要求在任务开始前应用本规则。
- 与 `project-local-priority-rule` 一致：项目本地层优先于本表公共路径。
- 与 `skill-route-enabled-rule` 一致：`must_invoke` 还须满足该 Rule 中的全部启用条件。
- 与 `skill-routing-validation-rule` 一致：维护者应定期校验本表与磁盘及 metadata 一致。
