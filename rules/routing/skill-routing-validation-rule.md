# skill-routing-validation-rule

## 适用主体

维护公共 HACF 库或项目 `.ai/` 协作框架的维护者；执行 `{skillLibraryRoot}/skills/router/validate-skill-routing.md` 的 Agent。

## 规则陈述

路由校验用于发现「Router / Matrix 登记」与「磁盘 Skill 文件及 metadata」之间的不一致。完整执行步骤以 `validate-skill-routing` Skill 为准；本 Rule 定义**必须检查**的项目。

### 必须检查的项目

1. **SKILL_ROUTER 路径存在性**：`{skillLibraryRoot}/routers/SKILL_ROUTER.md` 中声明的每个 `` `skills/...` `` 路径，在 `{skillLibraryRoot}/` 下须对应真实文件（或在校验报告中记为 error）。
2. **Trigger Matrix Available 路径存在性**：`TASK_TRIGGER_MATRIX.md` **Available Skill Triggers** 分区中每个 Skill 路径须存在；不存在记 **error**。
3. **Planned 不得强制**：Matrix **Planned Skill Triggers** 分区中条目**不得**标记 `must_invoke` 或含「必须使用」类强制措辞；违反记 **error**。
4. **routeEnabled 与登记一致性**：凡 Skill 文件 front matter 中 `routeEnabled: true`，须已登记于 SKILL_ROUTER **或** TASK_TRIGGER_MATRIX 之一；仅登记在 Router 且 `routeEnabled: false` 记 **info/warning**，不记 error。
5. **active Skill metadata 完整性**：`status: active` 的公共 Skill 须具备非空 `description` 与非空 `triggerWhen`（YAML 列表）；缺一项记 **error**。
6. **select-skill-for-task 语义一致性**：配置须保证未激活、未登记、`routeEnabled: false` 的 Skill 不会被 Matrix **Available** + `must_invoke` 组合误推为强制路由；矛盾记 **error**。
7. **幽灵引用**：Router 或 Matrix 指向不存在文件的公共 Skill 路径记 **error**。
8. **planned 被当作 available**：Planned 分区路径若同时出现在 Available 分区，或 Planned 路径文件存在却被标为 `planned` 且 Matrix 行含 `must_invoke`，记 **error**；Planned 路径文件已存在且拟升级为 Available 时，应走 `register-new-skill` + 人工确认，**不得**由校验 Skill 自动提升。

### 项目本地检查（scope 含 project-local 或 all 时）

- `skill-index.md` 中每行 `skillRelativePath` 对应 `{projectRoot}/.ai/` 下文件须存在。
- 表中 `status: active` 行须与文件 front matter 一致，且满足 `routeEnabled: true`（若参与自动路由）。
- `.ai/skills/project-local/**/*.md` 中**未**出现在 skill-index 的文件记 **warning**（未登记，不得自动路由）。
- 项目私有路径、密钥、未脱敏内容**不得**写入 `{skillLibraryRoot}/reports/` 下公共报告。

### 报告总体状态

| 状态 | 含义 |
|------|------|
| `passed` | 无 error；warning 可为零或已文档化可接受 |
| `warning` | 无 error；存在须维护者知晓的 warning |
| `failed` | 存在至少一条 error |

### 校验后义务

- 存在 error 时，相关 Skill **不得**参与自动路由，直至修复并重新校验。
- 校验 Skill **不得**自动修改 Router、Matrix 或 Skill metadata（仅输出报告与修复建议）。

## 禁止事项

- 校验通过时仍保留 Available 指向缺失文件的条目而不修复。
- 将校验报告中的项目私有信息写入公共库报告。
- 在校验流程中自动将 planned 改为 available。

## 与其它 Rule 的关系

- `new-skill-registration-rule`：第 8–9 步要求执行本 Rule 所约束的校验。
- `skill-route-enabled-rule`：本 Rule 检查的配置须落实该 Rule 的启用条件。
- `validate-skill-routing` Skill：本 Rule 的可执行实现。
