---
status: active
routeEnabled: true
description: 在任务开始前依据白名单 Router、Matrix 与 skill-index 选择应执行的项目本地或公共 Skill。
triggerWhen:
  - 任务路由预检
  - select skill
  - 路由 Skill
---

# Skill：select-skill-for-task

## 1. 目的

在业务任务主体开始前，先按 `check-hacf-version-compatibility` 的口径完成**版本兼容判定**，再**仅从白名单登记来源**解析候选 Skill，结合 front matter（`status: active`、`routeEnabled: true`）与磁盘存在性，判定是否应调用某一 **项目本地 Skill** 或 **公共 Skill**；对满足条件的 available Skill 给出 **必须先读再执行** 的决策；对 `planned` Skill 仅执行 Matrix 中的 fallback；对配置错误输出 `config_error`；可选将决策写入 `.ai/state/last-skill-routing.md`。

版本兼容判定仅用于提醒：`up_to_date` 时**不**写入 `{projectRoot}/.ai/state/hacf-version-status.md`；非 `up_to_date`（`outdated` / `unknown` / `incompatible` / `local_newer_than_public`）时，按 `check-hacf-version-compatibility` 模板写入或覆盖该状态文件并提醒用户。版本状态**不得**改变本 Skill 的路由决策结果，且**不得**自动生成升级计划或执行升级。

**不得**全量扫描 `{skillLibraryRoot}/skills/**` 或任意目录后自由选择 Skill；**不得**脑补未登记的 Skill 路径。

## 2. 适用场景

- 用户或编排即将进行：代码理解、代码修改、调试、规划、测试、文档、Pattern 相关、或项目本地 Skill 相关任务。
- Agent 已接入 HACF（或至少可解析 `{skillLibraryRoot}`），希望在改码 / 写盘 / 长篇规划前选定协作 Skill。
- 用户显式要求「路由 Skill」「预检 Skill」「select skill」。

**不适用**（可跳过本 Skill 或仅做极简公共层检索）：

- 正在执行本 Skill `select-skill-for-task` 自身（避免递归预检）。
- 纯 Bootstrap 接入且 `{projectRoot}/.ai/` 尚未建立，且任务仅为 `load-skill-library` 或其子 Skill。
- 用户明确禁止 Skill 路由，且任务与 HACF 协作无关的泛闲聊。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根目录。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；可由 `SKILLKIT_LINK.md` 解析，或从本 Skill 磁盘路径连续三次父目录解析（去掉 `select-skill-for-task.md`、`router`、`skills`）。 |
| `{taskSummary}` | 一两句中文，概括本轮任务目标。 |
| `{taskSignals}` | 可选。任务信号列表（中文关键词或意图标签），例如 `bug`、`改码后文档`、`Pattern生成`；缺省则由 Agent 从 `{taskSummary}` 与最近用户消息归纳。 |
| `{writeRoutingState}` | 可选。`true` 时覆盖写入 `.ai/state/last-skill-routing.md`；缺省为 `false`（仅对话输出）。 |

## 4. 输出

| 输出类型 | 说明 |
|----------|------|
| Agent 对话 | **必须**包含结构化决策摘要（见 §11 完成标准）。 |
| `{projectRoot}/.ai/state/last-skill-routing.md` | **仅当** `{writeRoutingState}` 为 `true` 且 `.ai/state/` 可写时覆盖写入。 |
| `{projectRoot}/.ai/state/hacf-version-status.md` | **仅当** 本轮版本兼容判定结果非 `up_to_date` 时，按 `check-hacf-version-compatibility` 模板写入或覆盖；`up_to_date` 时不得因本 Skill 创建或覆盖该文件。 |

### `last-skill-routing.md` 结构

```markdown
---
routedAt: <UTC ISO8601>
taskSummary: ...
decision: invoke|fallback|none|config_error
selectedSkillPath: <相对 skillLibraryRoot 或 .ai/ 的路径，无则 n/a>
skillStatus: available|planned|n/a
fallbackApplied: <中文简述或 n/a>
noMatchReason: <decision 为 none 或 fallback 时的原因摘要>
versionCompatibilityStatus: up_to_date|outdated|local_newer_than_public|unknown|incompatible|check_failed
publicFrameworkVersion: <VERSION.md 首行或 unknown>
localFrameworkVersion: <skillkit-status localFrameworkVersion/libraryVersion 回退值或 unknown>
versionReminder: <版本一致/建议 plan-hacf-local-upgrade/检查失败等中文摘要>
evidence:
  - <检索层与路径>
---
```

正文可选：一至两句中文说明下一步（例如「须先读 skills/docs/grill-with-project-docs.md」）。

## 5. 前置条件

- `{skillLibraryRoot}/ENTRY.md` 可读。
- 下列公共文件存在且可读：
  - `{skillLibraryRoot}/routers/TASK_TRIGGER_MATRIX.md`
  - `{skillLibraryRoot}/routers/SKILL_ROUTER.md`
  - `{skillLibraryRoot}/rules/routing/mandatory-skill-trigger-rule.md`
  - `{skillLibraryRoot}/rules/routing/skill-routing-preflight-rule.md`
  - `{skillLibraryRoot}/rules/routing/skill-route-enabled-rule.md`
  - `{skillLibraryRoot}/skills/bootstrap/check-hacf-version-compatibility.md`
  - `{skillLibraryRoot}/templates/state/hacf-version-status.template.md`
- 建议只读：`{skillLibraryRoot}/rules/routing/new-skill-registration-rule.md`、`{skillLibraryRoot}/rules/base/project-local-priority-rule.md`、`{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、`{skillLibraryRoot}/rules/base/project-local-output-rule.md`、`{skillLibraryRoot}/rules/documentation/agent-context-budget-rule.md`。
- 维护者校验路由配置时可读：`{skillLibraryRoot}/skills/router/validate-skill-routing.md`（本 Skill 运行时**不强制**执行校验，但配置错误时应提示维护者运行该校验 Skill）。
- 若 `{projectRoot}/.ai/entry/SKILLKIT_LINK.md` 存在，**须**以其解析 `{skillLibraryRoot}` 并与反推路径交叉校验；不一致时在输出中标注 `uncertain` 并优先采用 `SKILLKIT_LINK` 解析结果。

## 6. 执行步骤

1. **版本兼容判定（只读优先，按需写入）**：
   - 只读打开 `{skillLibraryRoot}/skills/bootstrap/check-hacf-version-compatibility.md`，按其 §6（含 §6.3 状态枚举）的同等口径读取 `{skillLibraryRoot}/VERSION.md`、`{skillLibraryRoot}/capabilities/hacf-capabilities.yml`、`{projectRoot}/.ai/entry/SKILLKIT_LINK.md`、`{projectRoot}/.ai/state/skillkit-status.md`，计算 `publicVer`、`localVer` 与 `compatibilityStatus`。
   - 若 `compatibilityStatus == up_to_date`：在本 Skill 输出中记录 `versionCompatibilityStatus: up_to_date` 与版本一致提醒；**不得**因本 Skill 创建、覆盖或刷新 `{projectRoot}/.ai/state/hacf-version-status.md`。
   - 若 `compatibilityStatus` 为 `outdated` / `unknown` / `incompatible` / `local_newer_than_public`：按 `check-hacf-version-compatibility` 的模板与占位符规则写入或覆盖 `{projectRoot}/.ai/state/hacf-version-status.md`；输出提醒须说明该状态只建议用户按需执行 `skills/bootstrap/plan-hacf-local-upgrade.md`，**不得**自动执行 `apply-hacf-local-upgrade.md`。
   - 若版本判定所需文件不可读或状态文件写入失败：输出 `versionCompatibilityStatus: check_failed`，在 `versionReminder` 说明原因；除 `SKILLKIT_LINK.md` 缺失导致无法安全解析项目接入状态外，不得因此改变后续路由决策。
2. **归一化任务信号**：合并 `{taskSignals}` 与用户最近消息，去重得到信号集合 `S`；将 `{taskSummary}` 记入输出上下文。
3. **白名单解析候选路径**（**仅此四类来源**，不得全库扫描）：
   - **L1** `{projectRoot}/.ai/indexes/skill-index.md`：解析 `## 项目本地 Skill` 表中各行 `skillRelativePath`（及 `status`）；仅 `status: active` 行进入候选 `C`，标注来源 `skill-index`。
   - **L2** `{skillLibraryRoot}/routers/SKILL_ROUTER.md`：提取表中所有 `` `skills/...` `` 路径，标注来源 `SKILL_ROUTER`。
   - **L3** `{skillLibraryRoot}/routers/TASK_TRIGGER_MATRIX.md`：分别解析 **Available Skill Triggers** 与 **Planned Skill Triggers** 分区表格路径，标注 `matrix-available` 或 `matrix-planned`。
   - **禁止**：列举 `.ai/skills/project-local/` 目录发现未登记 Skill；**禁止**将 `pattern-index.md`、`code-type-index.md` 作为 Skill 发现源。
4. **Matrix 信号匹配**：将 `S` 与 Matrix Available / Planned 各行「触发场景」列匹配；命中项并入候选集合 `M`（保留分区标签）。
5. **存在性探测**（对 `M` 与经信号过滤后的 `C` 中每条路径 `p`）：
   - 若来自 skill-index 或 `p` 为相对 `.ai/` 的项目本地路径：探测 `{projectRoot}/.ai/{相对路径}`。
   - 否则：探测 `{skillLibraryRoot}/{p}`。
   - 记录 `exists: true|false`。
6. **读取 front matter 门槛**（对 `exists: true` 的候选）：解析文件首块 YAML；记录 `status`、`routeEnabled`、`description`、`triggerWhen`。
   - 仅当 `status: active` **且** `routeEnabled: true` **且** 路径已出现在 L1/L2/L3 白名单解析结果中，标记为 `routeEligible: true`。
   - 否则标记为 `routeEligible: false` 并在内部记录排除原因（供 `noMatchReason` 使用）。
7. **应用强制触发规则**：只读 `mandatory-skill-trigger-rule.md` 与 `skill-route-enabled-rule.md`，结合 Matrix 分区、`exists`、`routeEligible` 判定：
   - Matrix **Available** + `exists: true` + `routeEligible: true` → `decision: invoke`（项目本地 skill-index 同名意图优先于公共路径）。
   - Matrix **Available** + `exists: false` → `decision: config_error`（**routing error**）；向用户说明 Matrix 与磁盘不一致；**禁止**脑补 Skill 执行步骤。
   - Matrix **Available** + `exists: true` + `routeEligible: false` → `decision: config_error` 或排除后 `none`；说明 metadata 未满足（如 `routeEnabled: false`）。
   - Matrix **Planned** → `decision: fallback`；向用户说明「该能力在公共库尚未以 Skill 落地（planned）」；执行该行 fallback；**不得**读取 planned Skill 文件或声称已执行。
   - SKILL_ROUTER 命中且 `routeEligible: true`、无 Matrix 冲突 → 可 `invoke`（非 Matrix must_invoke 时为建议级，须在 evidence 注明）。
   - 无命中且无可路由候选 → `decision: none`；**须**在 `noMatchReason` 列出原因（无信号匹配 / 未登记 / metadata 未满足 / 文件缺失等）。
8. **输出决策摘要**（对话必出）：`decision`、`selectedSkillPath`、`skillStatus`、`fallbackApplied`、`noMatchReason`（若适用）、`versionCompatibilityStatus`、`publicFrameworkVersion`、`localFrameworkVersion`、`versionReminder`、`evidence` 列表。
9. **可选写盘**：若 `{writeRoutingState}` 为 `true`，按 §4 结构覆盖写入 `last-skill-routing.md`。
10. **后续动作**：
   - `invoke`：Agent **必须**在继续任务主体前全文读取 `selectedSkillPath` 对应文件，并严格按其 **第 6 节「执行步骤」**（或等价编排节）执行。
   - `fallback`：按 Matrix Planned 说明在对话中完成；**不得**声称已执行 planned Skill。
   - `none`：按普通方式处理任务；**须**说明未命中原因；不声称经过某 Skill。
   - `config_error`：报告 routing error；建议维护者执行 `validate-skill-routing`；可经用户确认后降级为 `none` 但须保留 `config_error` 记录。

## 7. 路由优先级

在白名单来源内，任务信号匹配与候选裁决顺序（高 → 低）：

1. `{projectRoot}/.ai/indexes/skill-index.md`（`active` 且 `routeEligible`）
2. `{skillLibraryRoot}/routers/TASK_TRIGGER_MATRIX.md` — **Available** 分区（`must_invoke` 且 `routeEligible`）
3. `{skillLibraryRoot}/routers/TASK_TRIGGER_MATRIX.md` — **Planned** 分区（仅 fallback，不 invoke）
4. `{skillLibraryRoot}/routers/SKILL_ROUTER.md`（`routeEligible` 的登记路径，作能力补充映射）

**细查说明**：意图信号**优先**对照 Matrix Available / Planned；Router 补全未在 Matrix 逐条列举的 Bootstrap / Scan 等能力；项目本地 skill-index **始终**覆盖公共同意图 Skill。

**明确排除的来源**：`pattern-index.md`、`code-type-index.md`、未在 skill-index 登记的 `project-local/` 文件、对 `skills/**` 的全量目录扫描。

## 8. 强制触发规则

以 `{skillLibraryRoot}/rules/routing/mandatory-skill-trigger-rule.md`、`skill-route-enabled-rule.md` 与 `TASK_TRIGGER_MATRIX.md` 为准。

摘要：

- **仅** Matrix Available + 文件存在 + `status: active` + `routeEnabled: true` + 已登记时，适用 `must_invoke`。
- Matrix Planned **永不** `must_invoke`；仅 `suggest_fallback`。
- 多信号冲突：项目本地 Skill > 公共 available；同层取更具体意图。

## 9. fallback 策略

| 条件 | 行为 |
|------|------|
| `decision: none` | 无可用 Skill；普通处理；`noMatchReason` 必填；不声称经过某 Skill。 |
| `decision: fallback` | 执行 Matrix Planned 对应 fallback；说明 planned 未实现；**不**打开 planned 路径。 |
| `decision: config_error` | 报告 routing error；**不**假装读过 Skill；建议 `validate-skill-routing`；可经用户确认降级为 `none`。 |
| `versionCompatibilityStatus` 非 `up_to_date` | 写入或覆盖 `hacf-version-status.md` 并提醒用户；不改变 `decision`；不自动 `plan` / `apply`。 |
| `.ai/indexes/skill-index.md` 缺失 | 跳过 L1；仅用 L2–L3；在 `evidence` 注明缺口。 |
| `SKILLKIT_LINK.md` 缺失 | **中止**本 Skill，提示先执行 `load-skill-library`；不写 `last-skill-routing.md`。 |
| 发现 project-local 文件未在 skill-index 登记 | 在 `evidence` 记 warning「未登记，不参与自动路由」；**不得** invoke。 |

## 10. 禁止事项

1. **不得**全量扫描 `skills/**` 后自由选择 Skill。
2. **不得**虚构「项目本地已存在」或未在 skill-index 登记的 Skill。
3. **不得**对 `planned` Skill 假装已读取或已按其第 6 节执行。
4. **不得**在 `available` 文件缺失或 `routeEligible: false` 时自行编造 Skill 执行步骤。
5. **不得**将项目私有扫描结果、密钥或未脱敏内容写入 `{skillLibraryRoot}`。
6. **不得**在本 Skill 内修改业务源码。
7. **不得**在无用户授权时因预检而批量重写 `.ai/docs/**`。
8. **不得**引用未出现在白名单解析结果中的 Skill 路径。
9. **不得**在 `versionCompatibilityStatus: up_to_date` 时因本 Skill 创建或覆盖 `hacf-version-status.md`。
10. **不得**因版本不一致自动执行 `plan-hacf-local-upgrade` 或 `apply-hacf-local-upgrade`。

## 11. 完成标准

- [ ] 已仅从 §6 白名单四类来源解析候选，并在 `evidence` 中列出实际读取或探测的路径。
- [ ] 已完成版本兼容判定；`up_to_date` 时未写入 `hacf-version-status.md`，非 `up_to_date` 时已写入或说明写入失败。
- [ ] 已对候选做存在性探测与 front matter `routeEligible` 判定。
- [ ] 对话输出含：`decision`、`selectedSkillPath`（或 `n/a`）、`skillStatus`、`fallbackApplied`（或 `n/a`）、`noMatchReason`（`none`/`fallback` 时）、`versionCompatibilityStatus` 与 `versionReminder`。
- [ ] 若 `decision: invoke`，已明确下一步须先读的 Skill 路径。
- [ ] 若 `{writeRoutingState}` 为 `true`，已写入 `last-skill-routing.md` 或说明写盘失败原因。

## 12. 失败处理

| 情况 | 处理 |
|------|------|
| `{skillLibraryRoot}` 不可解析 | 停止；提示 Bootstrap；不产出 invoke。 |
| Matrix / 强制触发 Rule 文件缺失 | 停止；记 `config_error`；建议修复公共库。 |
| `.ai/` 不可读但公共库可读 | 降级：仅 L2–L3 + Matrix；跳过 skill-index。 |
| 版本兼容判定失败 | 输出 `versionCompatibilityStatus: check_failed` 与原因；若不影响公共 Router / Matrix 读取，继续给出路由决策。 |
| 非 `up_to_date` 但 `hacf-version-status.md` 写盘失败 | 对话输出仍须提醒版本状态与写盘失败；不阻塞后续 `invoke`。 |
| `last-skill-routing.md` 写盘失败 | 对话输出仍须完整；注明写盘失败，不阻塞后续 `invoke`。 |
| 用户任务属于纯 Bootstrap | 允许 `decision: none` 并注明「Bootstrap 豁免预检」。 |

## 13. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/state/last-skill-routing.md`（**仅当** `{writeRoutingState}` 为 `true`） |
| `{projectRoot}/.ai/state/hacf-version-status.md`（**仅当** 版本兼容判定非 `up_to_date`） |

| 禁止 |
|------|
| `{skillLibraryRoot}/**` 任意写入 |
| `{projectRoot}` 下业务源码 |
| `.ai/` 下除上表外的任意文件 |

## 14. 与其它 Skill 的关系

- 由 `skill-routing-preflight-rule` 要求在任务开始前调用本 Skill。
- 命中后可能编排 **routeEligible** 的 available Skill，或项目本地 skill-index 中 `active` 路径。
- 路由配置维护：`register-new-skill` → `validate-skill-routing`。
- 不替代 `recommend-next-onboarding-step`（后者只读、偏接入向导）。
