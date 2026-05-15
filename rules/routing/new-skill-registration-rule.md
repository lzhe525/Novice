# new-skill-registration-rule

## 适用主体

在公共 HACF 库 `{skillLibraryRoot}` 或业务项目 `{projectRoot}/.ai/skills/project-local/` 中**新增**协作 Skill 的维护者、Agent 与编排流程。

## 规则陈述

新增公共 Skill 或激活项目本地 Skill 时，**必须**按下列顺序完成注册闭环；**不得**仅创建 Skill 文件即假定 `select-skill-for-task` 会自动命中。

### 注册步骤（公共库）

1. **创建 Skill 文件**：路径落在 `{skillLibraryRoot}/skills/**` 或 `{projectRoot}/.ai/skills/project-local/**`（项目本地）。
2. **添加合法 YAML front matter**（文件最顶部，置于 `# Skill：…` 标题之前），字段规范见下文「Front matter 规范」。
3. **标记 `status`**：合法值 `draft` | `planned` | `active` | `deprecated` | `disabled`。
4. **标记 `routeEnabled`**：布尔值；仅 `active` 且 `routeEnabled: true` 的 Skill 可参与自动路由。
5. **若 Skill 可直接被用户任务意图触发**：在 `{skillLibraryRoot}/routers/SKILL_ROUTER.md` 对应分组表中登记路径与说明。
6. **若 Skill 需要按任务信号自动触发**：在 `{skillLibraryRoot}/routers/TASK_TRIGGER_MATRIX.md` 登记；已实现且可强制触发的放入 **Available Skill Triggers**；尚未实现的**只能**放入 **Planned Skill Triggers**。
7. **若 Skill 尚未实现**：只能列入 Matrix 的 Planned 分区；**不得**标记为 `available` 或 `status: active` + `routeEnabled: true` 参与 `must_invoke`。
8. **执行路由校验**：完成登记后执行 `{skillLibraryRoot}/skills/router/validate-skill-routing.md`（或等价的结构化检查），scope 覆盖本次变更范围。
9. **修复所有 routing error**：校验报告为 `failed` 或存在 error 级项时，**不得**宣称该 Skill 已可自动路由；修复后重新校验直至通过或仅剩可接受的 warning。

### 项目本地 Skill 附加要求

- 人类确认激活后，**必须**在 `{projectRoot}/.ai/indexes/skill-index.md` 的 `## 项目本地 Skill` 表中 upsert 对应行（含 `skillRelativePath`、`status` 等）。
- 未在 `skill-index.md` 登记的 `.ai/skills/project-local/**/*.md` **不得**被 `select-skill-for-task` 自动命中。

### Front matter 规范

| 字段 | 必填 | 说明 |
|------|------|------|
| `status` | 是 | `draft` / `planned` / `active` / `deprecated` / `disabled` |
| `routeEnabled` | 是 | `true` 仅当允许自动路由；`draft`/`planned` 默认 `false` |
| `description` | `status: active` 时必填 | 一句话说明 Skill 目的 |
| `triggerWhen` | `status: active` 时必填 | YAML 字符串列表，归纳触发场景或任务信号 |
| `skillName` | 可选 | 展示名，缺省可用文件名 |

示例：

```yaml
---
status: active
routeEnabled: true
description: 在任务开始前依据 Router 与 Matrix 选择应执行的 Skill。
triggerWhen:
  - 任务路由预检
  - select skill
---
```

### 引导式注册

维护者可调用 `{skillLibraryRoot}/skills/router/register-new-skill.md` 生成 metadata 建议与 Router/Matrix 补丁草案；**不得**跳过人工确认即批量改写 Router。

## 禁止事项

- 仅创建 Skill 文件而不更新 Router / Matrix / skill-index（若适用）。
- 将未实现 Skill 列入 Matrix **Available** 或标记 `must_invoke`。
- 将 `planned` Skill 自动提升为 `available` 而不补齐文件与 metadata。
- 跳过 `validate-skill-routing` 即合并公共库路由变更。

## 与其它 Rule 的关系

- `skill-route-enabled-rule`：定义自动路由的充要条件。
- `skill-routing-validation-rule`：定义校验必须覆盖的检查项。
- `skill-routing-preflight-rule`：任务开始前仍须执行 `select-skill-for-task`；仅完成注册不替代预检。
- `mandatory-skill-trigger-rule`：Matrix `must_invoke` 须与 `skill-route-enabled-rule` 同时满足。
