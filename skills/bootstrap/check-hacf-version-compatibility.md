---
status: active
routeEnabled: false
description: 比对公共 HACF 与项目框架版本兼容状态。
triggerWhen:
  - 版本兼容
  - hacf version
---
# Skill：check-hacf-version-compatibility

## 1. 目的

读取公共 HACF 的 `VERSION.md` 与 `capabilities/hacf-capabilities.yml`，结合项目 `.ai/entry/SKILLKIT_LINK.md` 与 `.ai/state/skillkit-status.md`，判定项目 **`localFrameworkVersion`** 与公共 **`currentVersion`** 是否一致、项目是否具备当前版本声明的 `requiredFiles`，并将结论**覆盖写入** `{projectRoot}/.ai/state/hacf-version-status.md`。

## 2. 适用场景

- `load-skill-library` 末尾编排执行，使新项目落地即有版本状态。
- 人类在拉取新版公共 HACF 后手动触发，确认 Agent 不会用新 Skill 操作旧结构。
- `guide-project-onboarding` 或 `check-project-ai-health` 前的快速门禁。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根目录绝对路径。 |
| `{skillLibraryRoot}` | 可选；若缺省则从 `{projectRoot}/.ai/entry/SKILLKIT_LINK.md` 解析（算法同 `check-project-readiness` §6 步骤 1：`rel` → `normalize({projectRoot}/rel)`）。 |

## 4. 输出

| 路径 | 说明 |
|------|------|
| `{projectRoot}/.ai/state/hacf-version-status.md` | 自模板实例化并**覆盖写入**。模板：[`templates/state/hacf-version-status.template.md`](../../templates/state/hacf-version-status.template.md)。 |

Agent 回复须含：本文件相对路径、解析得到的 `compatibilityStatus`、公共版本与本地版本字符串。

### 模板占位符

| 占位符 | 含义 |
|--------|------|
| `{{CHECKED_AT_ISO}}` | 本次检查 UTC ISO8601。 |
| `{{PUBLIC_FRAMEWORK_VERSION}}` | `{skillLibraryRoot}/VERSION.md` 首行非空文本；读取失败为 `unknown`。 |
| `{{LOCAL_FRAMEWORK_VERSION}}` | 优先 `{projectRoot}/.ai/state/skillkit-status.md` YAML 的 `localFrameworkVersion`；若缺失则使用同文件 `libraryVersion`；若文件不存在或均无则 `unknown`。 |
| `{{COMPATIBILITY_STATUS}}` | 见 §6.3。 |
| `{{PUBLIC_CAPABILITY_SCHEMA_VERSION}}` | 从 `hacf-capabilities.yml` 顶层读取 `schemaVersion`；若无该键则填 `n/a`。 |

## 5. 前置条件

- `{projectRoot}/.ai/entry/SKILLKIT_LINK.md` 存在（否则将 `compatibilityStatus` 置为 `incompatible` 并仍尝试写入状态文件，若目录可写）。
- Agent 对 `{projectRoot}/.ai/state/` 有写入权限（若不存在则创建 `.ai/state/`）。

## 6. 执行步骤

1. **解析 `{skillLibraryRoot}`**：同 `check-project-readiness` §6 步骤 1；失败则 `PUBLIC_FRAMEWORK_VERSION`=`unknown`，`COMPATIBILITY_STATUS`=`incompatible`，跳至步骤 5 仍写入状态文件（若可写）。
2. **读取公共版本**：读取 `{skillLibraryRoot}/VERSION.md` 首行非空文本为 `publicVer`。
3. **读取能力声明**：读取并解析 `{skillLibraryRoot}/capabilities/hacf-capabilities.yml`。若文件不存在或 YAML 无法解析：`COMPATIBILITY_STATUS`=`unknown`（若 `SKILLKIT_LINK` 亦无效可置 `incompatible`），`publicVer` 仍取自 `VERSION.md`；跳至步骤 5。
4. **读取本地版本**：读取 `{projectRoot}/.ai/state/skillkit-status.md` front matter：`localVer` = `localFrameworkVersion`，若缺失则 `libraryVersion`，再缺失则 `unknown`。
5. **一致性辅助检查**：若 `localVer` 与 `skillkit-status.libraryVersion` 同时存在且**不相等**，在 `hacf-version-status.md` 正文「详情」注明；若 `framework.currentVersion`（yml）与 `publicVer` 不一致，在正文注明（维护问题，状态倾向 `unknown`）。
6. **判定 `compatibilityStatus`（§6.3）**：
   - **`unknown`**：`publicVer` 为 `unknown`，或 yml 缺失/不可解析，或无法从磁盘读取 `skillkit-status` 且 `localVer` 为 `unknown`。
   - **`incompatible`**：`SKILLKIT_LINK` 无法解析出有效 `{skillLibraryRoot}`；或 `localVer` 非 `unknown` 且不在 yml 的 `versionOrder` 列表中；或 `publicVer` 非 `unknown` 且不在 `versionOrder` 中。
   - **`local_newer_than_public`**：`localVer` 与 `publicVer` 均在 `versionOrder` 中，且 `index(localVer) > index(publicVer)`（从旧到新列表）。
   - **`outdated`**：`index(localVer) < index(publicVer)`，或**同版本**但 `versions[publicVer].requiredFiles` 中任一路径在 `{projectRoot}` 下不存在（**排除**本 Skill 即将写入的 `.ai/state/hacf-version-status.md` 自身，以免首写前误报）。
   - **`up_to_date`**：`localVer == publicVer`（字符串相等）且均在 `versionOrder` 中，且当前版本所需 `requiredFiles` 全部存在；且 `localVer` 与 `libraryVersion` 无矛盾（若 `libraryVersion` 缺失则不因此降级）。
7. **写入**：按模板填充并**覆盖写入** `hacf-version-status.md`；正文「详情」「建议下一步」用简体中文简述事实与建议 Skill 路径（英文路径保持原文）。

### 6.3 状态枚举汇总

| 值 | 含义（简述） |
|----|----------------|
| `up_to_date` | 本地记录版本与公共一致且声明文件齐备。 |
| `outdated` | 本地落后于公共，或同版本但缺声明文件。 |
| `local_newer_than_public` | 本地声明版本在 `versionOrder` 中新于公共检出（少见）。 |
| `unknown` | 信息不足或能力文件不可用。 |
| `incompatible` | 链接无效、版本不在声明链，或矛盾不可安全继续自动化。 |

## 7. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/state/hacf-version-status.md` |
| 创建 `{projectRoot}/.ai/state/` 目录（若不存在） |

| 禁止 |
|------|
| 写入或修改 `{skillLibraryRoot}/**` |
| 修改 `skillkit-status.md`（由 `initialize-project-ai-context` / `apply-hacf-local-upgrade` 负责） |

## 8. 禁止事项

- **禁止**在未读取 `VERSION.md` 的情况下伪造 `PUBLIC_FRAMEWORK_VERSION`。
- **禁止**在本 Skill 中执行 `plan-hacf-local-upgrade` 或 `apply-hacf-local-upgrade`。

## 9. 完成标准

- [ ] `hacf-version-status.md` 已写入且 front matter 含 `checkedAt`、`publicFrameworkVersion`、`localFrameworkVersion`、`compatibilityStatus`。
- [ ] Agent 回复列出输出路径与状态枚举值。

## 10. 失败处理

| 情况 | 处理 |
|------|------|
| `.ai/state/` 不可写 | 中止；五段或普通回复说明权限；不写文件。 |
| `SKILLKIT_LINK` 缺失 | `incompatible`；若可写则仍写状态文件记录原因。 |

## 权威 Rule

以 [`{skillLibraryRoot}/rules/bootstrap/hacf-version-compatibility-rule.md`](../../rules/bootstrap/hacf-version-compatibility-rule.md) 为准。
