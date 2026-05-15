---
status: active
routeEnabled: false
description: 识别模块内候选代码类型并写入索引。
triggerWhen:
  - 代码类型识别
  - detect code types
---
# Skill：detect-code-types-by-ai

## 1. 目的

在**不修改业务源码**的前提下，针对 `{projectRoot}` 内某一已扫描模块（`moduleId`），基于模块文档、索引、`module-map` 与 **scanScope 内真实源码样例**，识别并记录**固定代码类型**的候选清单（名称、用途、证据、结构、可变部分、命名规律、依赖与注册/声明/导出点等），为后续 Pattern Pack 抽象做准备。

**本轮仅产出候选代码类型与证据说明**，不实现 Pattern 抽象、不生成业务代码、不执行 Placement 或 Develop。

## 2. 适用场景

- 已完成 `scan-module-by-ai`（或等价地已具备模块级 `overview` / `deep-dive` / `constraints` / `change-guide` 四篇文档），需在同一 `moduleId` 下归纳重复出现的「编码形态」。
- 需在写入 Pattern 相关产物之前，先固化「代码类型」候选与**证据强弱**，供人类或后续 Skill 消费。
- 逻辑模块跨多路径时，仍须以 `module-map` 与 `scanScope` 为准，在预算内抽样深读，避免通读全树。

## 3. 与其它 Skill 关系

- **须先于本 Skill**：`scan-project-by-ai`（索引与项目上下文）、`scan-module-by-ai`（至少产出下列四篇模块文档）。
- **必填读取**：`.ai/docs/modules/{moduleId}/overview.md`、`deep-dive.md`、`constraints.md`、`change-guide.md`。
- **可选读取**：同目录下 `risk-points.md`（若存在且有助于标注风险相关代码类型，可打开；**不**替代必填四篇）。
- **解析扫描范围**：与 `scan-module-by-ai` 一致，**优先** `.ai/config/module-map.md` 中该 `moduleId` 块，其次再参考索引（见 `module-is-logical-boundary-rule`）。

## 4. 词汇表（Agent 必须理解）

| 字段 / 概念 | 说明 |
|-------------|------|
| `codeTypeId` | 候选代码类型的稳定标识（建议小写字母、数字、连字符）；**不得**含 `/`、`..`、盘符。用于索引主键与文档锚点（如 `## codeType-foo-bar`）。 |
| `displayName` | 代码类型的中文显示名（与 `codeTypeId` 配套）。 |
| `evidenceStrength` | `strong` / `medium` / `weak`；判定口径见 `code-type-detection-rule.md`。 |
| `status`（条目级） | 有充分已读源码与互证支撑时可写 `draft`；**主要依赖推断或文档**时必须 `inferred`。 |
| `needsHumanConfirmation` | 布尔；弱证据、单文件结构、边界不清等须为 `true`（见 Rule）。 |
| `abstractability` | 是否**适合**抽象为 Pattern Pack 的评估结论（是/否/条件）；**非** Pack 已实现声明。 |
| `multiFileCoordination` | 该代码类型是否通常需要多文件联动（是/否及说明）。 |
| `scanScope` | 本轮应覆盖的边界：优先来自 `module-map` 与模块文档 front matter / 正文；与 `scan-module-by-ai` 中含义对齐。 |

## 5. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根。 |
| `{skillLibraryRoot}` | 公共 HACF 库根（仅用于读取 Rule / 模板路径，**不得**写入项目分析结果）。 |
| `{moduleId}` | **必填**；逻辑模块标识，须为安全 slug。 |

## 6. 输出

| 文件 | 模板 / 说明 |
|------|-------------|
| `code-types.md` | [templates/docs/code-types.template.md](../../templates/docs/code-types.template.md) |
| `code-type-index.md` | [templates/indexes/code-type-index.template.md](../../templates/indexes/code-type-index.template.md) |
| `code-type-detection-status.md` | 见 **§10 状态文件约定**（无单独模板文件）。 |

上述路径均相对于 `{projectRoot}/.ai/`：

- `docs/modules/{moduleId}/code-types.md`
- `indexes/code-type-index.md`
- `state/code-type-detection-status.md`

## 7. 前置条件

- `{projectRoot}/.ai/` 已存在。
- 下列模块文档**可读**（缺一则按 §13 失败处理）：  
  `.ai/docs/modules/{moduleId}/overview.md`、`deep-dive.md`、`constraints.md`、`change-guide.md`。
- 可读取本 Skill 使用的 Rule 与模板：  
  `{skillLibraryRoot}/rules/pattern/code-type-detection-rule.md`、  
  `{skillLibraryRoot}/templates/docs/code-types.template.md`、  
  `{skillLibraryRoot}/templates/indexes/code-type-index.template.md`，以及  
  `{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`、  
  `{skillLibraryRoot}/rules/base/source-of-truth-rule.md`、  
  `{skillLibraryRoot}/rules/documentation/structured-doc-writing-rule.md`、  
  `{skillLibraryRoot}/rules/documentation/human-readable-doc-rule.md`、  
  `{skillLibraryRoot}/rules/documentation/agent-context-budget-rule.md`、  
  `{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、  
  `{skillLibraryRoot}/rules/base/project-local-output-rule.md`、  
  `{skillLibraryRoot}/rules/project/scan-readonly-and-artifacts-rule.md`、  
  `{skillLibraryRoot}/rules/project/module-is-logical-boundary-rule.md`。  
- `{projectRoot}/.ai/config/language-policy.md` 存在（与 Scan 阶段惯例一致）。

## 8. 执行步骤

1. **规范化 `moduleId`**：校验安全字符；拒绝含 `..`、盘符、路径分隔符的输入。非法则中止（§13）。
2. **读取模块四文档**：依次打开 §7 所列四篇；提取 `scanScope`、路径列表、对外接口与约定等与「重复结构」相关的线索。
3. **解析扫描范围**（与 `scan-module-by-ai` 同序）：  
   - **优先**：读取 `{projectRoot}/.ai/config/module-map.md`，定位 `moduleId`，收集 `primaryPaths`、`relatedPaths`、`entryFiles`、`registryFiles`、`configFiles` 及可选排除项。  
   - **辅助**：读取 `.ai/indexes/module-index.md`、`.ai/indexes/file-index.md`、`.ai/indexes/directory-index.md`（若不存在，在 `code-types.md` 与状态中说明「索引缺失」，不得虚构索引内容）。  
4. **存在性**：对将用于深读的路径验证 `{projectRoot}/<path>` 存在；不存在则记入待确认项，不得虚构文件内容。
5. **选取源码样例（深读）**：在 **scanScope** 内按下列顺序填充深读列表，总数 **≤ 8** 个文件：  
   `entryFiles` → `registryFiles` → `configFiles` → 自各 `primaryPaths` 启发式补足（命名启发：`*Service*`、`*Controller*`、`*Route*`、`*Module*`、`*Plugin*` 等，以项目惯例为准）。  
   **鼓励**：对同一候选代码类型尽量覆盖 **≥ 2** 个样例文件以增强证据；若客观只能 1 个文件，该类型在条目中须说明原因，且通常 `evidenceStrength: weak`、`needsHumanConfirmation: true`（见 Rule）。  
6. **深读预算**（与 `scan-module-by-ai` §8 步骤 7 对齐）：单文件 ≤ **260** 行或 **24KiB**；超限则读头 **160** 行 + 尾 **80** 行，并在正文「备注」说明截断。  
7. **归纳候选代码类型**：对每个候选填写 §9 识别字段表所列维度；为每条设置 `codeTypeId`、`displayName`、`evidenceStrength`、`status`、`needsHumanConfirmation`。  
   - 凡主要依赖推断、或未满足 Rule 中 `strong` 证据条件且缺乏互证的，**必须** `status: inferred`。  
   - 单一样例且无注册点/导出点/第二文件互证的，**不得**使用「稳定模式」类表述；`evidenceStrength` 至少为 `weak`，且 **`needsHumanConfirmation: true`**。  
8. **实例化 `code-types.md`**：用模板填充；总览表与分节一一对应；锚点建议 `## codeType-{codeTypeId}`。整文 front matter 中 `scanSkill: detect-code-types-by-ai`、`moduleId`、`scannedAt` 必填；若模块内多数条目为 `inferred` 或存在大量待确认，可将文档级 `needsHumanConfirmation` 设为 `true` 并在正文说明。  
9. **合并 `code-type-index.md`**：若不存在，自模板新建；若已存在，按 `moduleId` + `codeTypeId` 更新或追加行，**禁止**静默删除人类行。`codeTypesDoc` 列指向 `docs/modules/{moduleId}/code-types.md`（可加锚点）。  
10. **写入 `code-type-detection-status.md`**：按 §10 更新或覆盖（仅该状态文件可整文件覆盖为最新一次运行元数据，但建议在回复中提醒人类若曾手工编辑状态文件则需 diff）。  
11. **校验**：YAML front matter 合法；路径与标识符未翻译；无 Pattern Pack 目录或生成物产生。

## 9. 读取范围

| 类型 | 范围 | 上限 / 约定 |
|------|------|-------------|
| 模块文档（必填） | `overview.md`、`deep-dive.md`、`constraints.md`、`change-guide.md` | 各 1 次完整阅读（过大则分段仍须覆盖标题与关键小节） |
| 模块文档（可选） | `risk-points.md` | 1 次，可跳过 |
| 配置 / 索引 | `module-map.md`、`module-index.md`、`file-index.md`、`directory-index.md` | 各 1 次；缺失则记录，不伪造 |
| 源码深读 | `entryFiles` / `registryFiles` / `configFiles` 与启发式补足 | **≤ 8** 文件；单文件行/字节见 §8 步骤 6 |

**不**默认递归读满所有 `relatedPaths`；更深覆盖应分次执行本 Skill 并在状态中标注「部分覆盖」。

### 9.1 每个候选代码类型须覆盖的识别字段

- 固定代码类型名称（`codeTypeId` + 中文 `displayName`）  
- 代码类型用途  
- 触发依据（已读样例 / 文档 / 注册点等）  
- 样例文件（路径保持原文）  
- 固定结构  
- 可变部分  
- 命名规律  
- 依赖关系  
- 注册点 / 声明点 / 导出点  
- 是否需要多文件联动（`multiFileCoordination`）  
- 是否适合抽象为 Pattern Pack（评估结论，非实现）  
- 是否需要人工确认（与 `needsHumanConfirmation` 一致）  
- **`status`**、**`evidenceStrength`**

## 10. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/docs/modules/{moduleId}/code-types.md` |
| `{projectRoot}/.ai/indexes/code-type-index.md` |
| `{projectRoot}/.ai/state/code-type-detection-status.md` |

| 禁止 |
|------|
| 写入 `{skillLibraryRoot}/**` |
| 修改 `{projectRoot}` 下 `.ai/` **以外** 的任意文件（含业务源码） |
| 在 `.ai/` 下创建 Pattern Pack 目录结构或生成「可执行抽象」代码 |

### 10.1 `code-type-detection-status.md` 约定

文件位于 `{projectRoot}/.ai/state/code-type-detection-status.md`。建议 YAML front matter 键：

- `lastCodeTypeDetectionAt`：ISO8601 字符串  
- `lastCodeTypeDetectionModuleId`：本轮 `moduleId`  
- `detectionSkill`：`detect-code-types-by-ai`  
- `codeTypesDocRelative`：例如 `docs/modules/{moduleId}/code-types.md`  
- `codeTypeIndexRelative`：`indexes/code-type-index.md`  

正文建议小节：

- **本轮摘要**：时间、`moduleId`、产出路径列表  
- **深读文件列表**（相对 `{projectRoot}` 的路径，保持原文）  
- **预算与截断**：是否达到 8 文件上限、哪些文件截断  
- **推荐下一步**：例如「人类确认 `code-types.md` 中带 `needsHumanConfirmation: true` 的条目」或「无」  
- **是否需要人工确认**：是/否及简述原因  

## 11. 禁止事项

- 实现 Pattern Pack、Placement、Develop，或生成替换业务源码的代码。  
- 将项目分析结果写入公共库。  
- 把**单个**样例文件直接当作全模块稳定模式且不标注证据强弱与待确认（违反 `code-type-detection-rule`）。  
- 虚构未读文件中的 API、注册点或依赖。  
- 单次深读文件数超过 §9（违反 `agent-context-budget-rule`）；超出须分次执行并标注部分覆盖。  
- 在仅推断的情况下省略 `status: inferred`。

## 12. 完成标准

- [ ] `code-types.md` 已写入，front matter 含 `docType: module-code-types`、`moduleId`、`scanSkill: detect-code-types-by-ai`、`scannedAt`、`reviewedByHuman`、`sourceOfTruth: repository`，YAML 合法。  
- [ ] 每个候选代码类型覆盖 §9.1 全部字段，且含 `evidenceStrength`、`status`、`needsHumanConfirmation`。  
- [ ] 凡推断归纳：`status: inferred`；凡须开发者确认：`needsHumanConfirmation: true`（含弱证据单文件情形，见 Rule）。  
- [ ] 正文中文描述；路径、类名、方法名、枚举名等保持原文。  
- [ ] `code-type-index.md` 已新建或合并更新，表内行与 `code-types.md` 一致，主键 `moduleId` + `codeTypeId` 无重复冲突（同模块同 id 应更新而非重复追加）。  
- [ ] `code-type-detection-status.md` 已更新，含本轮 `moduleId`、时间与深读列表。  
- [ ] 未创建 Pattern Pack；未修改业务源码。

## 13. 失败处理

| 情况 | 处理 |
|------|------|
| `moduleId` 非法 | 中止；提示人类更正。 |
| 四篇模块文档之一缺失 | 中止；提示先执行 `scan-module-by-ai` 或补全缺失文件。 |
| `module-map.md` 与三索引均不可用 | 可仅依赖四篇模块文档与有限路径深读继续，但候选条目须偏保守：`evidenceStrength` 不高于 `medium`，且多数应 `status: inferred`、`needsHumanConfirmation: true`；状态中说明依据不足。 |
| 选中 0 个可读源文件 | 仍生成 `code-types.md`（可为空表或单条「数据不足」说明）、更新索引与状态；**不得**虚构样例；待确认项列出需在 `module-map` 中补充的 `entryFiles` 等。 |
| 模板或 Rule 缺失 | 中止；在回复中列出缺失的 `{skillLibraryRoot}` 路径。 |
| 预算内无法覆盖全部意图类型 | 写入当前可支撑候选；状态与 `code-types.md` 备注中标明「部分覆盖」；建议人类分次再跑或缩小 `moduleId` 范围。 |

## 14. 权威 Rule

证据强弱、单文件限制、`status` / `needsHumanConfirmation` 的强制语义以 [{skillLibraryRoot}/rules/pattern/code-type-detection-rule.md](../../rules/pattern/code-type-detection-rule.md) 为准。
