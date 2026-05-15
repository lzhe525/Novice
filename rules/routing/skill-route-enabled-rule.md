# skill-route-enabled-rule

## 适用主体

执行 `{skillLibraryRoot}/skills/router/select-skill-for-task.md` 或在任务开始前做 Skill 路由决策的 Agent。

## 规则陈述

`select-skill-for-task` **只能**从下列白名单来源解析候选路径，且**只能**对同时满足全部启用条件的 Skill 给出 `decision: invoke` 或适用 `must_invoke`：

### 白名单来源（仅此四类）

1. `{projectRoot}/.ai/indexes/skill-index.md`（`## 项目本地 Skill` 表中已登记行）
2. `{skillLibraryRoot}/routers/SKILL_ROUTER.md`（表格中的 `` `skills/...` `` 路径）
3. `{skillLibraryRoot}/routers/TASK_TRIGGER_MATRIX.md`（**Available** 与 **Planned** 分区表格中的路径）
4. 项目本地路径**仅**能通过第 1 项 `skill-index` 进入候选集；**不得**通过列举 `.ai/skills/project-local/` 目录发现未登记 Skill

**禁止**全量扫描 `{skillLibraryRoot}/skills/**` 或任意目录后「自由选择」Skill；**禁止**将 `pattern-index.md`、`code-type-index.md` 作为 Skill 发现源（Pattern 上下文由命中后的 Skill 自行读取）。

### 启用条件（须全部满足方可自动路由）

| # | 条件 |
|---|------|
| 1 | Skill 文件在磁盘上**真实存在** |
| 2 | 文件首块 YAML front matter 中 `status: active` |
| 3 | `routeEnabled: true` |
| 4 | 路径已登记于 SKILL_ROUTER、TASK_TRIGGER_MATRIX（对应分区）或 skill-index（项目本地）之一 |
| 5 | 项目本地 Skill：`skill-index.md` 中对应行 `status` 为 `active`，且与文件 front matter 一致 |

### 不得强制路由的状态

下列 `status` 的 Skill **不得**被 `must_invoke` 或 `decision: invoke` 强制路由：

- `draft`
- `planned`
- `deprecated`
- `disabled`

Matrix **Planned** 分区中的路径即使文件不存在，也**仅**执行 fallback，**不得**强制执行。

### `routeEnabled: false` 的 Skill

- 可保留在 SKILL_ROUTER 中供人工查阅与跳转。
- `select-skill-for-task` **不得**对其输出 `invoke`。
- 若 Matrix **Available** 中登记了 `must_invoke` 但目标 Skill `routeEnabled: false` 或 `status` 非 `active`，预检应记 `config_error` 或排除并说明 metadata 不满足。

### Matrix 特殊情形

| 情形 | 行为 |
|------|------|
| Available + 文件不存在 | `decision: config_error`（routing error） |
| Available + metadata 不满足启用条件 | 排除或 `config_error`；说明原因 |
| Planned | 仅 fallback；向用户说明「该能力尚未实现」 |
| 无命中 | `decision: none`；允许普通分析，**须**说明未命中原因 |

## 禁止事项

- 引用未出现在白名单解析结果中的 Skill 路径。
- 对 `planned` / `draft` / 未登记项目本地 Skill 假装已读取或已执行。
- 在 `available` 文件缺失时自行编造 Skill 第 6 节执行步骤。

## 与其它 Rule 的关系

- `new-skill-registration-rule`：新增 Skill 时如何满足本规则。
- `mandatory-skill-trigger-rule`：`must_invoke` 以 Matrix 为准，但目标 Skill **还须**满足本规则全部启用条件。
- `skill-routing-validation-rule`：维护者定期校验本规则是否被配置违反。
