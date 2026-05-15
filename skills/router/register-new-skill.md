---
status: active
routeEnabled: true
description: 引导开发者将新 Skill 正确注册到 HACF 路由体系（metadata、Router、Matrix）。
triggerWhen:
  - 注册新 Skill
  - register skill
  - 新增 Skill 路由
---

# Skill：register-new-skill

## 1. 目的

引导维护者将**新增**或**待激活**的 Skill 按 HACF 标准注册进路由体系：校验文件与 front matter、按 `status` 决定 Router/Matrix 分区、生成 metadata 补丁与 Router/Matrix **拟插入片段**（不静默大块改写），并在注册流程末尾要求执行 `validate-skill-routing`。

## 2. 适用场景

- 在公共库 `{skillLibraryRoot}/skills/**` 新建 Skill 文件后。
- 在项目 `{projectRoot}/.ai/skills/project-local/**` 新建或晋升 Skill 后。
- 将 planned 能力实现为真实文件并拟升入 Matrix Available 前（须人工确认）。
- 用户显式要求「注册 Skill」「register new skill」。

**不适用**：

- 仅运行任务路由预检（应使用 `select-skill-for-task`）。
- 维护者要求全自动批量改写整个 Router 而无确认。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{skillPath}` | Skill 相对路径：公共库为相对 `{skillLibraryRoot}`（如 `skills/docs/foo.md`）；项目本地为相对 `{projectRoot}/.ai/`（如 `skills/project-local/create-x.md`）。 |
| `{skillName}` | 展示名。 |
| `{stage}` | 可选。`bootstrap` \| `scan` \| `pattern` \| `docs` \| `router` \| `project-local` 等，用于建议 Router 分组。 |
| `{status}` | `draft` \| `planned` \| `active` \| `deprecated` \| `disabled`。 |
| `{routeEnabled}` | `true` \| `false`；`draft`/`planned` 默认建议 `false`。 |
| `{description}` | 一句话目的；`active` 时必填。 |
| `{triggerWhen}` | 触发场景列表（YAML 列表或对话中的多条中文意图）。 |
| `{registerToRouter}` | `true` 时输出 `SKILL_ROUTER.md` 拟插入表行。 |
| `{registerToTriggerMatrix}` | `true` 时输出 `TASK_TRIGGER_MATRIX.md` 拟插入表行（分区由 `status` 决定）。 |
| `{projectRoot}` | 业务项目根；项目本地 Skill 时必填。 |
| `{skillLibraryRoot}` | 公共库根。 |

## 4. 输出

| Skill 位置 | 报告路径 |
|------------|----------|
| 公共库（`skillPath` 落在 `{skillLibraryRoot}/skills/**`） | `{skillLibraryRoot}/reports/new-skill-registration-report.md` |
| 项目本地 | `{projectRoot}/.ai/reports/new-skill-registration-report.md` |

报告须含：输入摘要、文件存在性、front matter 现状、建议补丁（YAML 块）、Router/Matrix 拟插入行、后续须执行的 `validate-skill-routing` 说明、注册结论（`ready` \| `blocked`）。

## 5. 前置条件

- `{skillPath}` 目标文件已创建或维护者确认即将创建（若不存在则步骤 1 记 blocked）。
- 只读：`rules/routing/new-skill-registration-rule.md`、`rules/routing/skill-route-enabled-rule.md`。
- 公共 Skill 须能解析 `{skillLibraryRoot}`；项目本地须能解析 `{projectRoot}`。

## 6. 执行步骤

1. **检查 `skillPath` 是否存在**：
   - 公共：`{skillLibraryRoot}/{skillPath}`
   - 项目本地：`{projectRoot}/.ai/{skillPath}`
   - 不存在 ⇒ 报告记 `blocked`，输出「须先创建文件」；不继续 Router 建议。
2. **检查 front matter**：读取文件首块 YAML；验证 `status`、`routeEnabled` 字段合法；若缺失或非法，生成**建议补丁**（完整 YAML 块，置于 `# Skill：` 之前）。
3. **必要 metadata**：若 `{status}` 为 `active` 且缺 `description` 或 `triggerWhen`，用输入参数生成建议值写入报告补丁区。
4. **判断 `status` 与 Matrix 分区**：
   - `active` + 文件存在 + `{routeEnabled: true}` ⇒ 可建议 **Available Skill Triggers**（仅当 `{registerToTriggerMatrix}` 为 true）。
   - `planned` ⇒ **只能**建议 **Planned Skill Triggers**；`routeEnabled` 建议 `false`；**不得**建议 `must_invoke`。
   - `draft` ⇒ 默认不得自动路由；`routeEnabled` 建议 `false`；可不登记 Matrix。
5. **`registerToRouter = true`**：根据 `{stage}` 建议 `SKILL_ROUTER.md` 分组（如 Bootstrap、Scan、Router（路由维护）等），输出**拟插入的一行 Markdown 表行**（任务意图、Skill 路径、说明），**不**直接改写文件。
6. **`registerToTriggerMatrix = true`**：输出拟插入表行，列含：触发场景、`skillPath`、`status`（available/planned）、强制级别（available 可为 `must_invoke` 或 `suggest`；planned **不得** must_invoke）、fallback（planned 必填；available 建议写文件缺失时的 config_error 说明）。
7. **项目本地**：若路径在 `.ai/skills/project-local/`，额外输出 `skill-index.md` 拟 upsert 行（`skillRelativePath`、`status`、`codeTypeId` 等占位由输入补全）。
8. **要求后续校验**：报告末尾固定写明：维护者应用补丁并更新 Router/Matrix 后，**必须**执行 `skills/router/validate-skill-routing.md`，`scope` 为 `public` / `project-local` / `all`（与本次 Skill 位置一致）。
9. **写入注册报告**并对话摘要：结论 `ready`（仅待人工应用补丁）或 `blocked`（缺文件或矛盾配置）。

## 7. 注册策略

| status | routeEnabled 建议 | Router | Matrix 分区 | 自动路由 |
|--------|-------------------|--------|-------------|----------|
| `draft` | `false` | 可选登记 | 不登记 Available | 否 |
| `planned` | `false` | 可选 | **仅** Planned + fallback | 否（仅 fallback） |
| `active` | `true`（若需自动命中） | 建议登记 | Available（若需信号触发） | 是（须校验通过） |
| `deprecated` / `disabled` | `false` | 可保留说明行 | 应从 Available 移除 | 否 |

未实现 Skill **不得**进入 Available；与 `new-skill-registration-rule` 一致。

## 8. 写入位置

| 允许 |
|------|
| `{skillLibraryRoot}/reports/new-skill-registration-report.md` |
| `{projectRoot}/.ai/reports/new-skill-registration-report.md` |

| 禁止 |
|------|
| 未经明确列出拟修改条目而**静默**改写整个 `SKILL_ROUTER.md` 或 `TASK_TRIGGER_MATRIX.md` |
| 修改业务源码 |
| 自动将 planned 提升为 available |

## 9. 禁止事项

1. 静默修改大量 Router / Matrix 内容；若修改，**必须**在报告中逐条列出将变更的条目。
2. 将未实现 Skill 登记为 Matrix Available。
3. 跳过 `validate-skill-routing` 建议。
4. 自动写入 Skill 正文业务逻辑或改码。

## 10. 完成标准

- [ ] 已检查 `skillPath` 存在性与 front matter。
- [ ] 已输出 metadata 建议补丁（若需要）。
- [ ] 若 `registerToRouter` / `registerToTriggerMatrix` 为 true，已给出拟插入表行而非整文件覆盖。
- [ ] 已写入注册报告并注明须执行 `validate-skill-routing`。
- [ ] 对话摘要含注册结论 `ready` 或 `blocked`。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| 文件不存在 | `blocked`；仅输出创建与 metadata 模板建议。 |
| `active` + `registerToTriggerMatrix` + 文件不存在 | `blocked`；建议先实现文件或改为 `planned`。 |
| 报告目录不可写 | 对话输出完整拟补丁内容；注明写盘失败。 |
| `status` 与 `routeEnabled` 矛盾（如 planned + routeEnabled true） | `blocked`；报告中说明须修正。 |

## 12. 与其它 Skill 的关系

- 注册闭环后续：**必须**调用 `validate-skill-routing`。
- 任务运行时路由由 `select-skill-for-task` 执行，本 Skill 不替代预检。
- 项目本地激活索引维护可参考 `skills/pattern/activate-project-local-skill.md`。
