---
status: active
routeEnabled: false
description: 模块级 AI 扫盘五件套。
triggerWhen:
  - 模块扫盘
  - scan module
---
# Skill：scan-module-by-ai

## 1. 目的

在**不修改业务源码**的前提下，针对 `{projectRoot}` 内某一**逻辑模块**（由 `moduleId` 标识），在只读源码与配置前提下，于 `.ai/docs/modules/{moduleId}/` 下一次性生成五类文档。**模块真实行为以 `module-map` 与 `scanScope` 所覆盖的仓库路径及关键文件为准**；下列 Markdown 均为辅助上下文。

## 2. 适用场景

- 已在 `.ai/config/module-map.md` 中声明跨目录、多关键文件的模块，需生成或刷新模块级五件套。
- 单目录集中的能力边界（`directory-module`），可用 `moduleId` + 可选 `moduleRootRelative` 快捷扫描。
- 尚无 `module-map` 行时，基于现有索引推断候选范围并生成草稿，供人类在 `module-map` 中确认。

## 3. 词汇表（Agent 必须理解）

| 字段 / 概念 | 说明 |
|-------------|------|
| `moduleId` | 逻辑模块稳定标识；仅用于文档目录与索引主键，须为安全 slug（建议小写字母、数字、连字符），**不得**含 `/`、`..`、盘符。输出目录为 `.ai/docs/modules/{moduleId}/`。 |
| `moduleDisplayName` | 人类可读模块名称（中文或项目惯例短名）。 |
| `moduleType` | `logical-module`：范围主要来自 `module-map` 多路径；`directory-module`：范围主要来自单一 `moduleRootRelative` 或等价单路径；`inferred-module`：无 `module-map` 行时由索引推断。 |
| `scanScope` | 一两句中文（可含路径列举）概括本轮 Agent 应覆盖的边界与证据来源。 |
| `primaryPaths` | 相对 `{projectRoot}` 的目录路径列表（POSIX）；业务主体所在树。 |
| `relatedPaths` | 关联目录（共享工具、类型、适配层等）。 |
| `entryFiles` | 入口源文件路径列表。 |
| `registryFiles` | 注册、路由、插件清单等路径列表。 |
| `configFiles` | 与模块强相关的配置文件路径列表。 |
| `excludePaths` | 可选；明确不纳入本轮归纳的路径前缀或文件。 |
| `readOnlyPaths` | 可选；只读参考、不当作实现主体的路径。 |
| `testPaths` | 可选；测试树根或关键 spec 路径。 |
| `sourceEvidence` | 写入 front matter 的证据列表；每项建议含 `path` 与简短 `note`（YAML 列表）。 |
| `reviewedByHuman` | 布尔；人类审阅后置 `true`。 |
| `status` | 文档草稿程度；从索引推断且无 `module-map` 时必须为 `inferred`。 |
| `needsHumanConfirmation` | 布尔；`inferred-module` 或缺少 `module-map` 支撑时为 `true`。 |

## 4. module-map 最小契约（人类可维护）

`{projectRoot}/.ai/config/module-map.md` 建议按模块分小节或使用表格，每个逻辑模块至少包含：

- **必填**：`moduleId`、`moduleDisplayName`。
- **扫描范围**：`primaryPaths`、`relatedPaths`、`entryFiles`、`registryFiles`、`configFiles`（可为空列表但小节须存在或表格列可空）。
- **可选**：`excludePaths`、`readOnlyPaths`、`testPaths`。
- **元信息**：`scanScope`（短中文说明）；可选 `sourceEvidence`（路径 + 说明）。

Agent 以**人类可读表格/小节**解析为准，不要求复杂机器格式。

## 5. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根。 |
| `{skillLibraryRoot}` | 公共库根。 |
| `{moduleId}` | **必填或可推导**：优先直接提供；若缺省则必须提供 `{moduleRootRelative}`，由 §8 步骤 1 推导（见该节）。 |
| `{moduleRootRelative}` | **可选**（与 `moduleId` 二选一即可，亦可同时提供以锚定单目录）。单一目录相对 `{projectRoot}` 的路径，正斜杠，无 `..`。若提供且 `module-map` 中无该 `moduleId` 对应行，则设 `moduleType: directory-module`，`primaryPaths: [{moduleRootRelative}]`。 |
| `{moduleDisplayName}` | **可选**；缺省时可用 `moduleId` 或 `module-map` 中的显示名。 |

## 6. 输出

以下路径均在 `{projectRoot}/.ai/docs/modules/{moduleId}/` 下（**不再**使用源码树镜像路径作为模块文档目录）：

| 文件 | 模板 |
|------|------|
| `overview.md` | [templates/docs/module-overview.template.md](../../templates/docs/module-overview.template.md) |
| `deep-dive.md` | [templates/docs/module-deep-dive.template.md](../../templates/docs/module-deep-dive.template.md) |
| `constraints.md` | [templates/docs/module-constraints.template.md](../../templates/docs/module-constraints.template.md) |
| `change-guide.md` | [templates/docs/module-change-guide.template.md](../../templates/docs/module-change-guide.template.md) |
| `risk-points.md` | [templates/docs/module-risk-points.template.md](../../templates/docs/module-risk-points.template.md) |

## 7. 前置条件

- `{projectRoot}/.ai/` 已存在。
- 可读五个模块文档模板与下列 Rule：`{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`、`{skillLibraryRoot}/rules/base/source-of-truth-rule.md`、`{skillLibraryRoot}/rules/documentation/structured-doc-writing-rule.md`、`{skillLibraryRoot}/rules/documentation/human-readable-doc-rule.md`、`{skillLibraryRoot}/rules/documentation/agent-context-budget-rule.md`、`{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、`{skillLibraryRoot}/rules/project/module-is-logical-boundary-rule.md`、`{skillLibraryRoot}/rules/project/scan-readonly-and-artifacts-rule.md`。
- `{projectRoot}/.ai/config/language-policy.md` 存在（与 `scan-project-by-ai` 惯例一致）。

## 8. 执行步骤

1. **规范化**：校验 `moduleId` 安全字符；若缺省 `moduleId` 仅提供 `moduleRootRelative`，则令 `moduleId` 为该路径末段经 slug 化后的值（冲突时须人类可辨后缀，并在正文待确认项说明）。拒绝路径中含 `..`。
2. **解析扫描范围**（优先级）：
   - **优先**：读取 `{projectRoot}/.ai/config/module-map.md`，定位 `moduleId` 对应块，提取 `primaryPaths`、`relatedPaths`、`entryFiles`、`registryFiles`、`configFiles` 及可选字段；合成 `scanScope` 与 `sourceEvidence`。
   - **若无 map 行**：读取 `.ai/indexes/module-index.md`、`.ai/indexes/file-index.md`、`.ai/indexes/directory-index.md`（存在则读），推断候选 `primaryPaths` / 关键文件；设 `moduleType: inferred-module`，`status: inferred`，`needsHumanConfirmation: true`。
   - **若仅** `moduleRootRelative` **且无 map**：设 `primaryPaths` 含该路径，`moduleType: directory-module`；若 `module-map` 亦无冲突声明，`needsHumanConfirmation` 可为 `false`，但须在正文说明范围仅限该目录。
3. **存在性**：对 `primaryPaths` 中每个路径验证 `{projectRoot}/<path>` 存在（文件或目录）；不存在的路径记入待确认项，不得虚构内容。
4. **创建目录**：`{projectRoot}/.ai/docs/modules/{moduleId}/`。
5. **占位符**：准备 `{{MODULE_ID}}`、`{{MODULE_DISPLAY_NAME}}`、`{{MODULE_TYPE}}`、`{{SCAN_SCOPE_TEXT}}`、`{{SCANNED_AT_ISO}}`、YAML 列表占位（路径类）、`sourceEvidence` 列表；`{{MODULE_ROOT_RELATIVE}}` 仅在 `directory-module` 时填主锚目录，否则可为 `""`。
6. **清单**（预算分摊）：对每个 `primaryPaths` 目录分别列举**下一层**条目，每目录最多 **40** 名称，总计列举条目展示在笔记中不超过 **120** 条（超出则截断并标注 `truncated`）。
7. **深读**（共用一批阅读结果，五份产出禁止重复通读全树）：
   - 优先：`entryFiles`、`registryFiles`、`configFiles` 中已存在且可读的项，最多 **8** 个文件。
   - 若仍不足 8，自各 `primaryPaths` 一层子项中启发式补足（对外接口、路由、服务命名等），总数仍 ≤ **8**。
   - 单文件 ≤ **260** 行或 **24KiB**（超限则头 **160** + 尾 **80** 行）。
8. **实例化五模板**：填充正文；`constraints` / `change-guide` / `risk-points` 中无依据的条目须标「待确认」或「推断」。
9. **写入五文件**：分别校验 YAML front matter（含 `moduleId`、`moduleDisplayName`、`moduleType`、`scanScope`、`scanSkill: scan-module-by-ai`、`scannedAt`）。若为 `inferred-module` 或 §8 判定为自索引推断且无 `module-map` 支撑：设 `status: inferred`、`needsHumanConfirmation: true`；否则通常 `status: draft`、`needsHumanConfirmation: false`（除非正文要求人工确认架构边界）。
10. **合并更新索引**（若对应文件存在；禁止无备份整体替换人类大段手工内容）：
    - `module-index.md`：按 `moduleId` 找行或追加，补全 `moduleDisplayName`、`moduleType` 及五篇文档列链接（`docs/modules/{moduleId}/overview.md` 至 `risk-points.md`）。
    - `directory-index.md`：对本轮涉及的目录路径行，合并或追加 `relatedModuleIds` 中的 `moduleId`。
    - `file-index.md`：对本轮深读的每个文件追加或合并一行；**同一 `filePathRelative` 可再增一行以绑定第二 `moduleId`**，或在模板约定列中用逗号分隔多 `moduleId`（与项目侧表头一致）。
    - 若某索引不存在：**不**由本 Skill 强制创建完整项目索引；在回复与最后一篇产出正文末提醒先执行 `scan-project-by-ai`。若合并失败须在回复中说明。

## 9. 读取范围

| 类型 | 范围 | 上限 |
|------|------|------|
| 配置 / 声明 | `module-map.md`、三索引、`language-policy.md` | 各 1 次深读，单行宽表格适度截断 |
| 目录清单 | 每个 `primaryPaths` 下一层 | 每目录 40 条，合计笔记 ≤120 条 |
| 源文件深读 | `entryFiles`/`registryFiles`/`configFiles` 与启发式补足 | **8** 文件；单文件行/字节见步骤 7 |

**不**默认递归读满所有 `relatedPaths` 子树；更深覆盖可另起本轮或 `scan-file-by-ai`。

## 10. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/docs/modules/{moduleId}/overview.md` |
| `{projectRoot}/.ai/docs/modules/{moduleId}/deep-dive.md` |
| `{projectRoot}/.ai/docs/modules/{moduleId}/constraints.md` |
| `{projectRoot}/.ai/docs/modules/{moduleId}/change-guide.md` |
| `{projectRoot}/.ai/docs/modules/{moduleId}/risk-points.md` |
| `{projectRoot}/.ai/indexes/module-index.md`（仅合并更新） |
| `{projectRoot}/.ai/indexes/directory-index.md`（仅合并更新） |
| `{projectRoot}/.ai/indexes/file-index.md`（仅合并更新） |

| 禁止 |
|------|
| 写入 `{skillLibraryRoot}/**` |
| 修改 `{projectRoot}` 下业务源码或非 `.ai/` 路径 |
| 在 `.ai/docs/modules/` 下写入与 `{moduleId}` 不一致的旁路目录名 |

## 11. 禁止事项

- 虚构对外 API（未出现于已读文件则标待确认）。
- 将本 Skill 产出写入公共库。
- 单次深读文件数超过 §9（违反 `agent-context-budget-rule`）；超出预算时拆分为多次执行并标注部分覆盖。

## 12. 完成标准

- [ ] 五个 Markdown 均已写入且 front matter 含 `moduleId`、`moduleDisplayName`、`moduleType`、`scanScope`、`scanSkill: scan-module-by-ai`、`scannedAt`、`reviewedByHuman`；路径类列表字段与 `sourceEvidence` 合法 YAML。
- [ ] `moduleType` 为 `inferred-module` 或缺少 `module-map` 支撑时：`status: inferred` 且 `needsHumanConfirmation: true`；否则 `status` 为 `draft`（或项目约定值）且 `needsHumanConfirmation` 与 §8 推断规则一致。
- [ ] 每份文档含模板所载「事实源声明」或等价表述（多路径 / `module-map` 优先）。
- [ ] `overview.md` 含「对外接口」节且列表带来源路径。
- [ ] `risk-points.md` 含风险表模板结构（允许填「无已知风险」但须说明依据不足）。
- [ ] 索引合并已尝试或已在回复说明原因。

## 13. 失败处理

| 情况 | 处理 |
|------|------|
| `moduleId` 非法或与文件系统保留名冲突 | 中止；提示人类重命名。 |
| `primaryPaths` 全部不存在 | 中止或仅写五文件正文说明缺口；`needsHumanConfirmation: true`。 |
| 选中 0 个可读源文件 | 仍生成五文件，正文说明数据不足；待确认项列出需人类在 `module-map` 中补充 `entryFiles`。 |
| 某一模板缺失 | 中止；已写入文件在回复中列出。 |
