# Skill：create-pattern-pack-draft

## 1. 目的

在**不修改业务源码**、**不修改公共 HACF 库**的前提下，依据已完成或可用的 **代码类型识别** 产物（`code-types.md`、跨模块索引 `code-type-index.md`），在业务项目 `{projectRoot}/.ai/pattern-packs/{codeTypeId}/` 下创建 **Pattern Pack 草稿**四件套（`pattern.md`、`validator.md`、`examples.md`、`docs.md`），统一为 `draft`、`reviewedByHuman: false`，并将证据强弱映射为 `confidence`（仅 `low` / `medium` / `high`）。

**本轮仅创建草稿目录与 Markdown 骨架**，不实现 Pattern 激活、不生成或修改业务代码、不执行 Placement / Develop。

## 2. 适用场景

- 已执行 `detect-code-types-by-ai`（或等价地已具备 `.ai/docs/modules/{moduleId}/code-types.md` 与 `.ai/indexes/code-type-index.md`），需为某一 `codeTypeId` 落盘标准 Pattern Pack **草稿**结构，供人类后续补全或与其它流程衔接。
- 若项目后续引入 `extract-code-pattern` 等分析 Skill 且产出路径已约定，可在同模块下将分析结论**摘要**进本 Skill 生成的 `docs.md` / `pattern.md` 正文（可选读取，见 §7）；**不得**因该产物缺失而阻塞本 Skill 的主路径（仍以 `code-types.md` 为主）。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；**仅只读** Rule 与模板，**不得**写入。 |
| `{moduleId}` | 逻辑模块标识；须为安全 slug（禁止 `..`、路径分隔符、盘符等）。 |
| `{codeTypeId}` | 候选代码类型标识；同 `detect-code-types-by-ai` 约定（建议小写字母、数字、连字符）；禁止含 `/`、`..`、盘符。 |

## 4. 输出

| 情况 | 产出 |
|------|------|
| **成功**（目标 Pack 主文件均不存在，见 §8） | `.ai/pattern-packs/{codeTypeId}/pattern.md`、`validator.md`、`examples.md`、`docs.md`（由公共模板实例化）。 |
| **冲突**（目标目录已存在且其中任一主文件已存在） | **不**写入或覆盖四个主文件；写入 `.ai/state/pattern-pack-draft-conflict-{codeTypeId}.md`（冲突说明与合并建议，见 §6、§8）。 |

## 5. 前置条件

- `{projectRoot}/.ai/` 已存在。
- 下列文件**可读**：  
  - `{projectRoot}/.ai/docs/modules/{moduleId}/code-types.md`  
  - `{projectRoot}/.ai/indexes/code-type-index.md`
- 下列公共库文件**可读**（缺一按 §11 处理）：  
  - `{skillLibraryRoot}/rules/pattern/pattern-pack-structure-rule.md`  
  - `{skillLibraryRoot}/templates/pattern-packs/pattern.template.md`  
  - `{skillLibraryRoot}/templates/pattern-packs/validator.template.md`  
  - `{skillLibraryRoot}/templates/pattern-packs/examples.template.md`  
  - `{skillLibraryRoot}/templates/pattern-packs/docs.template.md`
- Agent 须同时遵守：`rules/base/frontmatter-format-rule.md`、`rules/base/source-of-truth-rule.md`、`rules/base/public-skill-library-purity-rule.md`、`rules/base/project-local-output-rule.md`、`rules/pattern/code-type-detection-rule.md`（与代码类型语义对齐）、`rules/project/scan-readonly-and-artifacts-rule.md`、`rules/project/module-is-logical-boundary-rule.md`，以及 `rules/documentation/` 下 `structured-doc-writing-rule.md`、`human-readable-doc-rule.md`、`agent-context-budget-rule.md`（与既有 Scan / Pattern 文档 Skill 一致）。

## 6. 执行步骤

1. **规范化 `moduleId` 与 `codeTypeId`**：校验安全字符；拒绝含 `..`、盘符、路径分隔符的输入。非法则中止（§11）。
2. **读取** `{projectRoot}/.ai/docs/modules/{moduleId}/code-types.md`：定位 `codeTypeId` 对应条目（总览表或明细节，锚点建议 `## codeType-{codeTypeId}`）。若不存在该条目，中止（§11）。
3. **读取** `{projectRoot}/.ai/indexes/code-type-index.md`：查找 `moduleId` + `codeTypeId` 行。若索引缺失该组合但 `code-types.md` 中确有该类型，允许继续，须在将写入的 `docs.md` 正文「备注」中说明「索引未同步」；若索引存在但与 `code-types.md` 中显示名或证据字段明显矛盾，**保守**处理：取 `code-types.md` 为准写草稿，并在 `docs.md` 备注中列出矛盾点，**不得**虚构索引中才有的字段。
4. **确定 `confidence`**（**禁止**使用 `confirmed`）：  
   - 条目 `evidenceStrength` 为 `strong` → `confidence: high`  
   - `medium` → `confidence: medium`  
   - `weak`、缺失、或条目 `status` 为 `inferred` 且缺乏更强互证 → `confidence: low`  
   若正文有额外强证据（多文件互证描述）而 front matter 未更新，可在不夸大前提下上调一档，但**仍不得** `confirmed`。
5. **冲突检测**（与 `pattern-pack-structure-rule.md` 一致）：  
   - 令 `packDir = {projectRoot}/.ai/pattern-packs/{codeTypeId}/`。  
   - 若 `packDir` **不存在**：创建该目录（仅 `.ai/` 下）。  
   - 若 `packDir` **已存在**，且其中**任一** `pattern.md`、`validator.md`、`examples.md`、`docs.md` **已存在**（视为已有 Pack 草稿或定稿片段）：**禁止**创建或覆盖这四个文件；**跳过步骤 6**，执行步骤 7。  
   - 若 `packDir` **已存在**但上述四文件**均不存在**（例如空目录）：视为安全，执行步骤 6。
6. **实例化四文件**（仅非冲突路径）：自 `{skillLibraryRoot}/templates/pattern-packs/*.template.md` 复制结构，替换占位符（如 `{{MODULE_ID}}`、`{{CODE_TYPE_ID}}`、`{{DISPLAY_NAME}}`、`{{CONFIDENCE}}`、`{{GENERATED_AT_ISO}}` 等，以模板内注释为准）；每文件 YAML front matter 须包含：`status: draft`、`reviewedByHuman: false`、`confidence`（步骤 4 结果）、`moduleId`、`codeTypeId`、`patternPackSkill: create-pattern-pack-draft`、`sourceOfTruth: repository`、`generatedAt`（或等价 ISO 时间键，与模板一致）。正文以**中文**为主，路径与代码标识符保持原文；从 `code-types.md` 摘要用途、样例路径、可变部分等到各文件对应小节，**不得**声称已读未读源码。
7. **冲突说明**（当步骤 5 判定为冲突）：写入或更新 `{projectRoot}/.ai/state/pattern-pack-draft-conflict-{codeTypeId}.md`：列出 `packDir` 下**现有文件**清单、说明为何不覆盖、给出**合并建议**（例如人工合并 front matter、保留人类编辑块、或改用新的 `codeTypeId` 子目录名后重跑）。若该冲突文件已存在，**允许覆盖整个冲突说明文件**（其非 Pack 本体），或在文件内**追加**带时间戳的小节二选一，须在执行时选一种并在该文件首段说明本次策略。
8. **校验**：四成功产出文件的 front matter 合法；`confidence` ∈ {`low`,`medium`,`high`}；未修改 `.ai/` 外文件与 `{skillLibraryRoot}`。

## 7. 读取范围

| 类型 | 路径（相对 `{projectRoot}` 或约定根） | 说明 |
|------|----------------------------------------|------|
| 必填 | `.ai/docs/modules/{moduleId}/code-types.md` | 主事实源，提取该 `codeTypeId` 条目与 `evidenceStrength` / `status` 等。 |
| 必填 | `.ai/indexes/code-type-index.md` | 交叉校验 `moduleId` + `codeTypeId`；缺失或不同步按 §6 处理。 |
| 必填 | `{skillLibraryRoot}/templates/pattern-packs/pattern.template.md` 等四个模板 | 实例化草稿。 |
| 必填 | `{skillLibraryRoot}/rules/pattern/pattern-pack-structure-rule.md` | 目录与字段约束。 |
| 可选 | `.ai/config/module-map.md` | **只读**；仅用于在 `docs.md` 中补充 scanScope / 路径摘要时引用；不得写入公共库。 |
| 可选 | `.ai/state/pattern-extraction-status.md` | 若已执行 `extract-code-pattern` 则可能存在；只读以了解同路径 Pack 是否已由该 Skill 写入，**不得**虚构。 |
| 可选 | 未来其它分析产物（项目内已约定路径的 Markdown） | 若存在且路径已在项目文档中约定，可读以加强 `docs.md` / `pattern.md` 摘要；**不得**虚构路径。 |

**不得**为填充草稿而深读或修改业务源码。

## 8. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/pattern-packs/{codeTypeId}/pattern.md`（无冲突时新建） |
| `{projectRoot}/.ai/pattern-packs/{codeTypeId}/validator.md`（同上） |
| `{projectRoot}/.ai/pattern-packs/{codeTypeId}/examples.md`（同上） |
| `{projectRoot}/.ai/pattern-packs/{codeTypeId}/docs.md`（同上） |
| `{projectRoot}/.ai/state/pattern-pack-draft-conflict-{codeTypeId}.md`（冲突时；可整文件覆盖或按 §6 追加小节） |

| 禁止 |
|------|
| 写入 `{skillLibraryRoot}/**` |
| 修改 `{projectRoot}` 下 `.ai/**` 以外任意文件（含业务源码） |
| 在已存在任一主文件时**覆盖** `pattern.md` / `validator.md` / `examples.md` / `docs.md` |

## 9. 禁止事项

- 修改业务源码或项目内 `.ai/` 以外的配置。  
- 修改或写入公共 HACF 库（`{skillLibraryRoot}`）。  
- 实现 Pattern 激活、Placement、Develop，或生成可执行替换源码的代码。  
- 将 Pack 或条目标为 `confirmed`，或将 `confidence` 设为 `confirmed`。  
- 将 `reviewedByHuman` 设为 `true`（本 Skill 产出必须为 `false`）。  
- 将初始 Pack 状态设为 `draft` 以外用于表示「已发布」的语义（本阶段统一 `draft`）。  
- 在冲突场景下强行覆盖已存在的四个主文件之一。

## 10. 完成标准

- [ ] 无冲突时：`pattern.md`、`validator.md`、`examples.md`、`docs.md` 四文件均已创建，且各文件 front matter YAML 合法。  
- [ ] 四文件（成功路径）均含：`status: draft`、`reviewedByHuman: false`、`confidence` 为 `low` / `medium` / `high` 之一且**非** `confirmed`、`moduleId`、`codeTypeId`、`patternPackSkill: create-pattern-pack-draft`、`sourceOfTruth: repository`。  
- [ ] 正文中文描述；路径、类名、方法名等与仓库一致**不翻译**。  
- [ ] 冲突时：四个主文件未被覆盖，且已写入 `pattern-pack-draft-conflict-{codeTypeId}.md` 含清单与合并建议。  
- [ ] 未修改源码；未修改公共库。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| `moduleId` 或 `codeTypeId` 非法 | 中止；提示更正输入。 |
| `code-types.md` 不存在或不可读 | 中止；提示先执行 `detect-code-types-by-ai` 或补全文件。 |
| `code-types.md` 中无该 `codeTypeId` | 中止；提示核对 `moduleId` / `codeTypeId` 或先更新代码类型识别。 |
| `code-type-index.md` 缺失 | 可继续；`docs.md` 备注说明索引缺失。 |
| 公共模板或 `pattern-pack-structure-rule.md` 缺失 | 中止；在回复中列出缺失的 `{skillLibraryRoot}` 路径。 |
| Pack 目录已存在且四主文件之一已存在 | 不覆盖；仅写冲突说明文件（§8）。 |
| `.ai/` 不存在 | 中止；提示先 Bootstrap。 |

## 权威 Rule

Pack 目录结构、字段语义与冲突策略以 [{skillLibraryRoot}/rules/pattern/pattern-pack-structure-rule.md](../../rules/pattern/pattern-pack-structure-rule.md) 为准。
