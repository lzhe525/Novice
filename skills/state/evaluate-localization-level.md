# Skill：evaluate-localization-level

## 1. 目的

依据当前项目中 **active** 的 Pattern Pack、**active** 的项目本地 `create-*` Skill、以及 `skill-index.md` 的一致性，计算**本地化等级** `L0`～`L3`，写入 `localization-level.md`；并**合并更新** `skillkit-status.md` 的 YAML front matter（追加 `localizationLevel` 与 `localizationEvaluatedAt`），使 SkillKit 状态与本地化评估保持同步。

## 2. 适用场景

- 完成或变更了 Pattern 激活、项目本地 Skill 晋升/激活、`skill-index.md` 后，需刷新「项目与公共库解耦程度」的可视化状态。
- 人类希望在单文件中查看当前等级依据与缺口（不写业务码）。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；**仅读取** Rule，**不得**写入。 |

## 4. 输出

| 路径（相对 `{projectRoot}/.ai/`） | 说明 |
|----------------------------------|------|
| `state/localization-level.md` | 本地化等级结论、判定依据表、与上一轮的差异（若可检测）。 |
| `state/skillkit-status.md` | **合并**更新 YAML：保留 `libraryVersion`、`linkedAt`、`projectState`、`lastCheck` 等既有键；写入或更新 **`localizationLevel`**（`L0`/`L1`/`L2`/`L3`）、**`localizationEvaluatedAt`**（UTC ISO8601）；**不得**删除人类在正文中的既有段落（本 Skill **不**依赖正文追加等级说明）。 |

## 5. 前置条件

- `{projectRoot}/.ai/` 存在。
- 建议只读：`{skillLibraryRoot}/rules/base/project-local-output-rule.md`、`{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、`{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`。
- `skillkit-status.md` 应已存在（Bootstrap）；若缺失，可先按 `initialize-project-ai-context` 创建后再运行本 Skill，或由本 Skill 用同模板实例化后再合并（若选择实例化，须在 `localization-level.md` 注明来源）。

## 6. 执行步骤

1. **采集事实**（只读）：  
   - `.ai/state/skillkit-status.md`（解析 front matter）。  
   - `.ai/entry/SKILLKIT_LINK.md` 或 `AI_ENTRY.md`：确认 SkillKit 链接是否可解析（用于 `L0` 语义）。  
   - `.ai/state/scan-status.md`；`.ai/indexes/module-index.md` 或 `directory-index.md` 或 `file-index.md`：**任一生成扫盘索引存在且非空表/非占位**则满足 `L1` 扫盘维度。  
   - `.ai/indexes/code-type-index.md` 中 `## Pattern Pack 状态`：是否存在 **`patternPackStatus: active`** 的行。  
   - `.ai/skills/project-local/create-*.md`：枚举 glob；逐个解析 front matter。  
   - `.ai/indexes/skill-index.md` 中 `## 项目本地 Skill` 表（若存在）。
2. **计算 `localizationLevel`**（取满足条件的**最高**一级，且高等级须隐含低等级条件）：  
   - **`L0`**：`skillkit-status.md` 存在且 `linkedAt` 可解析，或 `SKILLKIT_LINK.md` 存在且含 `skillLibraryRootRelative`。  
   - **`L1`**：在 `L0` 基础上，扫盘索引（见步骤 1）表明至少一项扫盘产物已建立。  
   - **`L2`**：在 `L1` 基础上，`code-type-index.md` 的 `## Pattern Pack 状态` 中存在至少一行 `patternPackStatus` 为 **`active`**（或与该表设计等价的列值）。  
   - **`L3`**：在 `L2` 基础上，存在某 `codeTypeId` 使得：  
     - 文件 `skills/project-local/create-{codeTypeId}.md` 存在且 front matter 为 **`status: active`** 且 **`reviewedByHuman: true`**；且  
     - `skill-index.md` 中存在对应行，`status` 为 **`active`**，`skillRelativePath` 为 `skills/project-local/create-{codeTypeId}.md`。  
   若 `L3` 条件部分满足（文件 active 但索引缺行）：**降级为 `L2`**，并在 `localization-level.md` 的 `gaps` 中列出「索引不一致」。
3. **写入** `state/localization-level.md`：须含 YAML front matter（`localizationLevel`、`evaluatedAt`、`skillkitVersionHint` 可选）与 Markdown 正文「依据表」（每级一行：条件、是否满足、证据路径）。
4. **合并写入** `state/skillkit-status.md`：更新 YAML 中 `localizationLevel` 与 `localizationEvaluatedAt`；可选将 `lastCheck` 更新为当前 UTC ISO8601；**不得**将 `projectState` 从人类已设置的 `blocked` 擅自改为 `loaded`；**不得**删除 `libraryVersion` / `linkedAt`。
5. 回复中列出两文件相对 `.ai/` 的路径及本轮 `localizationLevel`。

## 7. 读取范围

| 路径 | 说明 |
|------|------|
| `.ai/state/skillkit-status.md` | 读并合并写回。 |
| `.ai/state/scan-status.md` | 判断 `L1`。 |
| `.ai/indexes/module-index.md`、`directory-index.md`、`file-index.md` | 任选可读；判断 `L1`。 |
| `.ai/indexes/code-type-index.md` | 判断 `L2`。 |
| `.ai/indexes/skill-index.md` | 判断 `L3` 与索引一致性。 |
| `.ai/skills/project-local/*.md` | 判断项目本地 Skill 状态。 |
| `.ai/entry/SKILLKIT_LINK.md`、`AI_ENTRY.md` | 判断 `L0`。 |
| `{skillLibraryRoot}/rules/**` | 按需只读。 |

## 8. 写入位置

| 允许 |
|------|
| `state/localization-level.md` |
| `state/skillkit-status.md` | **仅**合并 YAML front matter：追加或更新 `localizationLevel`、`localizationEvaluatedAt`；可选将 `lastCheck` 更新为当前 UTC ISO8601；**不得**在正文追加新小节（等级说明以 `localization-level.md` 为准）。 |

## 9. 禁止事项

1. **不得**修改业务源码或 `AGENTS.md`（除非另有独立 Skill）。  
2. **不得**修改 `pattern-packs/**` 内 Pack 正文或 front matter。  
3. **不得**将 `localizationLevel` 写为表外任意字符串（仅 `L0`～`L3`）。  
4. **不得**为抬升等级而伪造索引表行。  
5. **不得**向 `{skillLibraryRoot}` 写入任何项目内容。

## 10. 完成标准

- [ ] 已只读采集 §6 所列证据。  
- [ ] `localization-level.md` 已写入且含依据表。  
- [ ] `skillkit-status.md` YAML 已含 `localizationLevel` 与 `localizationEvaluatedAt`，且未破坏既有键语义。  
- [ ] 未修改业务源码。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| `.ai/` 不存在 | 中止；提示 Bootstrap。 |
| `skillkit-status.md` 缺失且无法安全创建 | 仅写 `localization-level.md` 并标 `skillkitStatus: missing`；回复说明。 |
| front matter 解析失败 | 尽力保留原文件正文；将新键写在 `localization-level.md` 备注中并标 `blocked`；回复警告。 |
| 索引文件均缺失 | 无扫盘证据时 `localizationLevel` 最高为 `L0`；若仅有链接无索引则在依据表中列缺口。 |

**字段说明**：`skillkit-status.md` 允许追加的 YAML 键以 [`{skillLibraryRoot}/templates/project-ai-context/skillkit-status.template.md`](../../templates/project-ai-context/skillkit-status.template.md) 中「扩展字段」为准。
