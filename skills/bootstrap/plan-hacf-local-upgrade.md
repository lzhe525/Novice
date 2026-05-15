---
status: active
routeEnabled: false
description: 生成项目本地框架升级计划。
triggerWhen:
  - 升级计划
  - plan hacf upgrade
---
# Skill：plan-hacf-local-upgrade

## 1. 目的

在 `hacf-version-status.md` 表明项目落后于公共能力矩阵（或人类明确要求）时，生成**只读计划**文件 `.ai/reports/hacf-version-upgrade-plan.md`，列出跨版本需补齐的能力与文件、自动与人工项及建议顺序；**不**修改其它 `.ai/` 文件。

## 2. 适用场景

- `compatibilityStatus` 为 `outdated`。
- 人类希望在执行 `apply-hacf-local-upgrade` 前审阅差额。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根。 |
| `{skillLibraryRoot}` | 可选；缺省从 `SKILLKIT_LINK.md` 解析。 |

## 4. 输出

| 路径 | 说明 |
|------|------|
| `{projectRoot}/.ai/reports/hacf-version-upgrade-plan.md` | **覆盖写入**。模板：[`templates/reports/hacf-version-upgrade-plan.template.md`](../../templates/reports/hacf-version-upgrade-plan.template.md)。 |

### 模板占位符

| 占位符 | 含义 |
|--------|------|
| `{{PLANNED_AT_ISO}}` | UTC ISO8601。 |
| `{{LOCAL_FRAMEWORK_VERSION}}` | 计划基准：来自 `skillkit-status.md` 的 `localFrameworkVersion` 或回退 `libraryVersion`。 |
| `{{PUBLIC_FRAMEWORK_VERSION}}` | `VERSION.md` 首行与 yml `framework.currentVersion` 交叉校验后取值。 |
| `{{SECTION_1_LOCAL_VERSION}}`～`{{SECTION_9_ORDER}}` | 简体中文正文块，对应计划九节。 |

## 5. 前置条件

- `{skillLibraryRoot}/capabilities/hacf-capabilities.yml` 可读；否则中止并提示先修复公共库。
- `{projectRoot}/.ai/state/hacf-version-status.md` 建议已存在（若不存在，须先运行 `check-hacf-version-compatibility`）；若缺失则本 Skill 可先提示运行检查，或基于 `skillkit-status` 与 yml 推导并在计划正文注明「未读取 hacf-version-status」。

## 6. 执行步骤

1. 解析 `{skillLibraryRoot}`；读取 `VERSION.md`、`hacf-capabilities.yml`、`hacf-version-status.md`（若存在）、`skillkit-status.md`。
2. 根据 yml `versionOrder` 计算从 `localVer` 到 `publicVer` 需跨越的版本序列（不含 `localVer` 若与下一版本连续则列出中间版本边界说明）。
3. 对每个跨越目标版本，汇总 `versions[v].requiredProjectCapabilities` 相对上一版本的新增能力 id，并从 `capabilityDefinitions` 附中文说明。
4. 扫描 `versions[publicVer].requiredFiles`（及跨越版本中新增的 requiredFiles 并集），标记项目缺失路径。
5. **第 6 节「可安全自动创建项」**：缺失且由 `autoRemediate: true` 且**不**落在 `hacf-upgrade-no-overwrite-rule` 禁止树内的路径/能力。
6. **第 7 节「必须人工确认项」**：`requiresHumanConfirmation: true` 的能力（例如 `language-policy` 内容确认），以及任何非自动补缺之配置审阅。
7. **第 8 节「禁止自动覆盖项」**：列出 Rule 固定禁止路径；说明已存在文件一律跳过。
8. **第 9 节「建议升级顺序」**：先运行 `check-hacf-version-compatibility` → 人类确认计划 → `apply-hacf-local-upgrade`；语言策略等须人类在 apply 之后仍按 `check-project-readiness` 流程确认。
9. 创建 `{projectRoot}/.ai/reports/`（若不存在），覆盖写入计划文件。

## 7. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/reports/hacf-version-upgrade-plan.md` |
| 创建 `{projectRoot}/.ai/reports/` |

| 禁止 |
|------|
| 修改 `hacf-version-status.md`、`skillkit-status.md`、`.ai/entry/**`、`.ai/config/**`（本 Skill 仅写计划） |
| 修改 `{skillLibraryRoot}/**` |

## 8. 禁止事项

- **禁止**在本 Skill 内执行文件补缺或调用 `apply-hacf-local-upgrade`。
- **禁止**覆盖项目私有文档或 Pattern Pack。

## 9. 完成标准

- [ ] 计划文件含用户要求的九节内容（对应模板占位 SECTION_*）。
- [ ] 路径与版本号保持原文。

## 10. 失败处理

| 情况 | 处理 |
|------|------|
| yml 不可读 | 中止；不写计划。 |
| 无法解析本地版本 | 在计划 §1 写 `unknown`，§3–§5 基于保守假设列出风险；仍写计划但正文强调须先修复 `skillkit-status`。 |

## 权威 Rule

[`rules/bootstrap/hacf-local-upgrade-rule.md`](../../rules/bootstrap/hacf-local-upgrade-rule.md)、[`rules/bootstrap/hacf-upgrade-no-overwrite-rule.md`](../../rules/bootstrap/hacf-upgrade-no-overwrite-rule.md)。
