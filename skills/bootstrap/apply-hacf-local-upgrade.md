---
status: active
routeEnabled: false
description: 在人类确认后补齐缺失框架文件。
triggerWhen:
  - 应用升级
  - apply hacf upgrade
---
# Skill：apply-hacf-local-upgrade

## 1. 目的

在已存在 `.ai/reports/hacf-version-upgrade-plan.md` 且人类**明确同意**执行的前提下，**仅补齐**计划中列为可安全自动创建、且路径不在禁止覆盖列表内的**框架**缺失文件；不覆盖已存在文件；冲突写入 `.ai/reports/hacf-version-upgrade-conflicts.md`；成功后写入结果报告并刷新版本状态与 `skillkit-status.md` 中的 `localFrameworkVersion` / `libraryVersion`。

## 2. 适用场景

- 已审阅 `hacf-version-upgrade-plan.md`，人类在对话中明确授权执行（须可被审计地引用或复述）。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根。 |
| `{skillLibraryRoot}` | 可选；缺省从 `SKILLKIT_LINK.md` 解析。 |
| **人类确认** | **必填**：用户原文同意执行本 Skill 或对计划的无异议确认。 |

## 4. 输出

| 路径 | 说明 |
|------|------|
| `{projectRoot}/.ai/reports/hacf-version-upgrade-result.md` | 成功或 `partial` 时**覆盖写入**。模板：[`templates/reports/hacf-version-upgrade-result.template.md`](../../templates/reports/hacf-version-upgrade-result.template.md)。 |
| `{projectRoot}/.ai/reports/hacf-version-upgrade-conflicts.md` | 存在跳过/禁止/已存在冲突时**覆盖或追加写入**（本 Skill 约定为**覆盖写入**整文件以最新一次执行为准，正文须含历史摘要可选）。模板：[`templates/reports/hacf-version-upgrade-conflicts.template.md`](../../templates/reports/hacf-version-upgrade-conflicts.template.md)。 |
| `{projectRoot}/.ai/state/hacf-version-status.md` | 成功后由 Agent 调用 `check-hacf-version-compatibility` 的逻辑**等效刷新**：可直接再次按 `check-hacf-version-compatibility` §6 执行一遍以覆盖写入；或合并更新 front matter 使状态趋向 `up_to_date`。 |
| `{projectRoot}/.ai/state/skillkit-status.md` | 合并更新 front matter：`localFrameworkVersion`、`libraryVersion` 均设为 `{skillLibraryRoot}/VERSION.md` 首行；刷新 `linkedAt` 为当前 UTC ISO8601（与 `initialize-project-ai-context` 语义一致）。 |

### 结果模板占位符

| 占位符 | 含义 |
|--------|------|
| `{{APPLIED_AT_ISO}}` | UTC ISO8601。 |
| `{{PREVIOUS_LOCAL_FRAMEWORK_VERSION}}` | 执行前 `localFrameworkVersion` 或回退值。 |
| `{{NEW_LOCAL_FRAMEWORK_VERSION}}` | 执行后与公共对齐的版本。 |
| `{{PUBLIC_FRAMEWORK_VERSION}}` | `VERSION.md` 首行。 |
| `{{OUTCOME}}` | `success` / `partial` / `aborted`。 |

## 5. 前置条件

- `{projectRoot}/.ai/reports/hacf-version-upgrade-plan.md` 存在且可读。
- 用户已在**本轮或可被引用的上文**中明确同意执行；否则**立即中止**。
- 已只读阅读 [`rules/bootstrap/hacf-upgrade-no-overwrite-rule.md`](../../rules/bootstrap/hacf-upgrade-no-overwrite-rule.md)。

## 6. 执行步骤

1. **校验同意**：无明确人类同意则中止，`OUTCOME`=`aborted`，不写 result（或写简短 aborted 说明至 result，由 Agent 选择其一并在完成标准中自洽；**推荐**不写 result 仅回复）。
2. **解析计划**：提取须创建的路径列表（仅限计划 §6 与 yml `autoRemediate: true` 且文件系统缺失者）。
3. **逐项处理**：
   - 若目标路径匹配禁止覆盖前缀（`hacf-upgrade-no-overwrite-rule`）：**跳过**，记入 conflicts 表。
   - 若文件已存在：不得覆盖；记入 conflicts 表（原因：`existing_file_skip`）。
   - 若缺失且为模板类（如 `hacf-version-status.md`）：自 `{skillLibraryRoot}/templates/state/hacf-version-status.template.md` 实例化，占位符规则与 `initialize-project-ai-context` 一致：`{{CHECKED_AT_ISO}}` 等按 `check-hacf-version-compatibility` 表；可先写占位再由步骤 7 覆盖。
   - 若缺失且为 Bootstrap 已知模板（`AI_ENTRY`、`SKILLKIT_LINK`、`skillkit-status`、`language-policy`）：使用 `{skillLibraryRoot}/templates/project-ai-context/*.template.md` 与 `templates/config/language-policy.template.md`，占位符计算同 `initialize-project-ai-context.md` §6 步骤 1（`{{SKILLKIT_VERSION}}` 取自 `VERSION.md` 首行；`{{SKILLKIT_RELATIVE_PATH}}` 从现有 `SKILLKIT_LINK.md` 表格读取；`{{PROJECT_NAME_OR_SLUG}}` 从目录名或 `skillkit-status` 推断并经保守默认）。
4. **不得**创建 `.ai/docs/**`、`.ai/pattern-packs/**`、`.ai/skills/project-local/**` 内容除非计划明确且路径不在禁止树（默认不创建）。
5. **写入 conflicts**（若存在任一冲突/跳过）：用模板填充 `hacf-version-upgrade-conflicts.md`。
6. **写入 result**：汇总创建列表、`OUTCOME`（全部成功 `success`；部分跳过 `partial`）。
7. **更新 `skillkit-status.md`**：合并 front matter `localFrameworkVersion` 与 `libraryVersion` 为公共 `VERSION.md` 首行；**不得**将 `language-policy.md` 的 `reviewedByHuman` 改为 `true`。
8. **重新运行兼容检查**：打开并严格执行 `check-hacf-version-compatibility.md` §6，覆盖写入 `hacf-version-status.md`。

## 7. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/reports/hacf-version-upgrade-result.md` |
| `{projectRoot}/.ai/reports/hacf-version-upgrade-conflicts.md` |
| `{projectRoot}/.ai/state/hacf-version-status.md` |
| `{projectRoot}/.ai/state/skillkit-status.md` |
| 计划中允许的、且非禁止树的**新建**框架文件 |
| 创建缺失父目录 |

| 禁止 |
|------|
| 覆盖「禁止覆盖」Rule 所列路径已有文件 |
| 修改 `{skillLibraryRoot}/**` |
| 修改 `.ai/docs/**`、`.ai/pattern-packs/**`、`.ai/skills/project-local/**` |

## 8. 禁止事项

- **禁止**在无计划或人类未同意时写盘。
- **禁止**覆盖 `hard-constraints.md`、`danger-zones.md`。

## 9. 完成标准

- [ ] 若执行写盘：`hacf-version-upgrade-result.md` 已写入且含摘要（须引用升级计划路径与 `OUTCOME`）。
- [ ] `skillkit-status.md` 的 `localFrameworkVersion` 与公共 `VERSION.md` 首行一致（在成功路径下）。
- [ ] 已再次执行 `check-hacf-version-compatibility` 并更新 `hacf-version-status.md`。

## 10. 失败处理

| 情况 | 处理 |
|------|------|
| 计划缺失 | 中止；提示先 `plan-hacf-local-upgrade`。 |
| 全部项因冲突跳过 | `OUTCOME`=`partial` 或 `aborted`；conflicts 写原因。 |

## 权威 Rule

[`rules/bootstrap/hacf-local-upgrade-rule.md`](../../rules/bootstrap/hacf-local-upgrade-rule.md)、[`rules/bootstrap/hacf-upgrade-no-overwrite-rule.md`](../../rules/bootstrap/hacf-upgrade-no-overwrite-rule.md)。
