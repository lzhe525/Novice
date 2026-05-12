# Skill：extract-code-pattern

## 1. 目的

在**不修改业务源码**的前提下，针对 `{projectRoot}` 内某一已识别 `codeTypeId`（归属 `moduleId`），基于 `code-types.md`、模块文档、索引与**真实源码样例**，抽象代码生成模式，并在 `.ai/pattern-packs/{codeTypeId}/` 下生成 **Pattern Pack 草稿**（`pattern.md`、`validator.md`、`examples.md`、`docs.md`），同时更新 `.ai/state/pattern-extraction-status.md`。

**本轮仅完成 Pattern 抽象与项目本地草稿落盘**，不生成业务代码，不实现 `create-code-by-pattern`，不实现正式开发类 Skill，不创建 `status: active` 的 Pattern。

## 2. 适用场景

- 已执行 `detect-code-types-by-ai`（或等价地已具备 `.ai/docs/modules/{moduleId}/code-types.md` 且含目标 `codeTypeId`），需将单一代码类型沉淀为可审查的 Pattern Pack 草稿。
- 人类或 Agent 需在写生成类 Skill 之前，先固化「固定结构 / 可变部分 / 联动与校验清单」。
- 逻辑模块跨多路径时，仍须以 `module-map` 与 `scanScope` 为准，在预算内深读样例，避免通读全树。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；仅用于读取 Rule 与模板，**不得**写入项目分析结果或 Pattern 内容。 |
| `{moduleId}` | **必填**；逻辑模块标识，须为安全 slug（小写字母、数字、连字符），**不得**含 `/`、`..`、盘符。 |
| `{codeTypeId}` | **必填**；与 `code-types.md` 中条目一致的安全 slug，**不得**含 `/`、`..`、盘符。 |

## 4. 输出

| 文件 | 模板 |
|------|------|
| `pattern.md` | `{skillLibraryRoot}/templates/pattern-packs/pattern.template.md` |
| `validator.md` | `{skillLibraryRoot}/templates/pattern-packs/validator.template.md` |
| `examples.md` | `{skillLibraryRoot}/templates/pattern-packs/examples.template.md` |
| `docs.md` | `{skillLibraryRoot}/templates/pattern-packs/docs.template.md` |
| `pattern-extraction-status.md` | `{skillLibraryRoot}/templates/state/pattern-extraction-status.template.md` |

上述路径相对于 `{projectRoot}/.ai/`：

- `pattern-packs/{codeTypeId}/pattern.md`
- `pattern-packs/{codeTypeId}/validator.md`
- `pattern-packs/{codeTypeId}/examples.md`
- `pattern-packs/{codeTypeId}/docs.md`
- `state/pattern-extraction-status.md`

## 5. 前置条件

1. 当前项目已通过 HACF `load-skill-library` 接入（`.ai/entry/SKILLKIT_LINK.md`、`.ai/state/skillkit-status.md` 等可读）。
2. **阶段语义（可验证，不等同于 `skillkit-status.md` 的 `projectState` 枚举）**  
   - **`module_scanned`（等价条件）**：`.ai/docs/modules/{moduleId}/` 下至少存在并可读：`overview.md`、`deep-dive.md`、`constraints.md`、`change-guide.md`（与 `scan-module-by-ai` 产出对齐）。  
   - **`code_types_detected`（等价条件）**：存在 `.ai/docs/modules/{moduleId}/code-types.md`，且其中存在目标 `{codeTypeId}` 条目。  
   实际硬门禁以本条下列 3–4 为准。
3. 已存在 `.ai/docs/modules/{moduleId}/code-types.md`。
4. `code-types.md` 中存在目标 `{codeTypeId}` 对应节或表行。
5. **样例数量与证据**：目标类型在 `code-types.md` 中列出的可读样例路径若 **少于 2 个**，仍须生成全套草稿，但须标记 **`evidenceStrength: low`**（见 `pattern-extraction-rule.md`），且 Pattern Pack front matter 须为 `confidence: low` 与 `needsHumanConfirmation: true`。
6. 所有生成内容**只能**写入 `{projectRoot}/.ai/` 下本 Skill「写入位置」所列路径。

**须可读的 Rule / 模板（前置校验）**：

- `{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`
- `{skillLibraryRoot}/rules/base/source-of-truth-rule.md`
- `{skillLibraryRoot}/rules/base/language-policy-rule.md`
- `{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`
- `{skillLibraryRoot}/rules/base/project-local-output-rule.md`
- `{skillLibraryRoot}/rules/base/read-source-before-edit-rule.md`
- `{skillLibraryRoot}/rules/project/module-is-logical-boundary-rule.md`
- `{skillLibraryRoot}/rules/pattern/code-type-detection-rule.md`
- `{skillLibraryRoot}/rules/pattern/pattern-extraction-rule.md`
- `{skillLibraryRoot}/rules/pattern/pattern-pack-structure-rule.md`
- `{skillLibraryRoot}/rules/pattern/pattern-human-confirmation-rule.md`
- `{skillLibraryRoot}/rules/documentation/agent-context-budget-rule.md`
- `{skillLibraryRoot}/rules/documentation/structured-doc-writing-rule.md`
- `{skillLibraryRoot}/rules/documentation/human-readable-doc-rule.md`
- 本节 §4 所列五个模板文件。

## 6. 执行步骤

1. **解析与校验**：从本 Skill 路径反推 `{skillLibraryRoot}`；校验 `{projectRoot}/.ai/` 存在；校验 `moduleId`、`codeTypeId` 安全字符，拒绝含 `..`、路径分隔符、盘符的输入。
2. **读取入口与状态**：打开 `{projectRoot}/.ai/entry/AI_ENTRY.md`、`{projectRoot}/.ai/state/skillkit-status.md`、`{projectRoot}/.ai/state/readiness.md`。若 `skillkit-status` 显示 `projectState: blocked`，在回复中警告 Bootstrap 未就绪，但若人类显式要求仍可在 `.ai/` 内写草稿，并在 `pattern-extraction-status.md` 备注。
3. **读取配置与模块上下文**：`{projectRoot}/.ai/config/language-policy.md`、`{projectRoot}/.ai/config/module-map.md`；`.ai/docs/modules/{moduleId}/overview.md`、`deep-dive.md`、`constraints.md`、`change-guide.md`；`.ai/docs/modules/{moduleId}/code-types.md`；`.ai/indexes/code-type-index.md`、`.ai/indexes/file-index.md`（若不存在则记录于状态与 Pack「待人工确认项」，不得虚构索引内容）。
4. **解析 `codeTypeId` 条目**：从 `code-types.md` 提取该类型的显示名、用途、样例路径列表、`evidenceStrength`、`needsHumanConfirmation`、`status`、注册/声明/导出点、多文件联动等已有字段，作为抽象种子。
5. **解析 scanScope**（与 `detect-code-types-by-ai` / `scan-module-by-ai` 同序）：**优先** `module-map.md` 中该 `moduleId` 块的 `primaryPaths`、`relatedPaths`、`entryFiles`、`registryFiles`、`configFiles` 及可选排除项；**辅助**索引。遵守 `module-is-logical-boundary-rule.md`。
6. **构建深读列表**：以 `code-types.md` 中该类型「样例文件」为主；若不足，在 scanScope 内从 `entryFiles` → `registryFiles` → `configFiles` 启发式补足。深读文件总数 **≤ 8**；单文件 ≤ **260** 行或 **24 KiB**，超限则读头 **160** 行 + 尾 **80** 行，并在 `pattern-extraction-status.md` 与 Pack「备注」说明截断。
7. **深读样例**：对列表中每个路径验证 `{projectRoot}/<path>` 存在且可读；不存在则记入待确认项，不得虚构源码。
8. **证据与 front matter 决策**（与 `pattern-extraction-rule.md` 一致）：  
   - 若本轮实际纳入抽象且可引用的**不同样例文件数 < 2**：`evidenceStrength: low`；四个 Pack 文件 front matter：`status: draft`、`reviewedByHuman: false`、`confidence: low`、`needsHumanConfirmation: true`。  
   - 否则：`evidenceStrength` 按规则取 `medium` 或 `high`（见 `pattern-extraction-rule.md`）；Pack front matter：`status: draft`、`reviewedByHuman: false`、`confidence: medium`；`needsHumanConfirmation` 一般为 `false`，但若 `code-types` 条目为 `needsHumanConfirmation: true`、或存在合并冲突、或 `status: inferred`，则须 `needsHumanConfirmation: true`。  
   - **禁止**将 `reviewedByHuman` 设为 `true`，禁止将 Pack `status` 设为 `active`（见 `pattern-human-confirmation-rule.md`）。
9. **Pattern 抽象（须在正文中覆盖的维度）**：结合已读样例与模块文档，分析并写入 Pack（避免无依据断言）：  
   代码类型用途；固定结构；可变部分；命名规则；文件位置规则；依赖关系；import / using / 引用规律；继承、接口、注解、特性、装饰器等结构特征；注册点；声明点；导出点；配置联动点；测试联动点；新增同类代码时必须修改的文件；新增同类代码时禁止修改的文件；常见错误；需要人工确认的规则；后续是否适合沉淀为项目本地 Skill（评估性文字，非实现承诺）。
10. **已存在 Pack 处理**：若 `.ai/pattern-packs/{codeTypeId}/` 下任一目标文件已存在，**禁止**整文件静默覆盖。须先读现有内容，按 `pattern-extraction-rule.md` 合并（追加变更记录、冲突小节）或在 `pattern-extraction-status.md` 中写明「冲突与合并说明」。
11. **实例化模板**：填充 `pattern.md`、`validator.md`、`examples.md`、`docs.md` 及 `pattern-extraction-status.md`；`patternName` 建议与 `code-types.md` 中显示名一致或为其 slug 化可读名。
12. **写入**：创建目录 `{projectRoot}/.ai/pattern-packs/{codeTypeId}/`（若不存在）；写入或合并五个文件；校验 YAML front matter 合法且符合 `pattern-pack-structure-rule.md`。

### 6.1 抽象维度与产出映射（摘要）

| 抽象维度 | 主要落点 |
|----------|----------|
| 用途、固定/可变、命名、路径、依赖、import 规律、结构特征、注册/声明/导出、配置/测试联动、必改/禁改文件、常见错误、人工确认、本地 Skill 评估 | `pattern.md`、`docs.md` |
| 结构/命名/联动/注册导出/配置/测试/风险/完成标准 | `validator.md` |
| 样例列表、角色、固定/可变对照、证据强弱、不宜作模板 | `examples.md` |
| 本轮深读列表、证据、截断、合并冲突、推荐下一步 | `pattern-extraction-status.md` |

## 7. 读取范围

| 类型 | 路径（相对 `{projectRoot}`，正斜杠） | 上限 / 约定 |
|------|--------------------------------------|-------------|
| 入口与状态 | `.ai/entry/AI_ENTRY.md`、`.ai/state/skillkit-status.md`、`.ai/state/readiness.md` | 各 1 次 |
| 配置 | `.ai/config/language-policy.md`、`.ai/config/module-map.md` | 各 1 次 |
| 模块文档 | `.ai/docs/modules/{moduleId}/overview.md`、`deep-dive.md`、`constraints.md`、`change-guide.md` | 各 1 次 |
| 代码类型 | `.ai/docs/modules/{moduleId}/code-types.md` | 1 次 |
| 索引 | `.ai/indexes/code-type-index.md`、`.ai/indexes/file-index.md` | 存在则各 1 次 |
| 源码样例 | `code-types` 所列及启发式补足 | **≤ 8** 文件；单文件行/字节见 §6 步骤 6 |

**不**默认递归读满所有 `relatedPaths`；更深覆盖应分次执行并标注「部分覆盖」。

## 8. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/pattern-packs/{codeTypeId}/pattern.md` |
| `{projectRoot}/.ai/pattern-packs/{codeTypeId}/validator.md` |
| `{projectRoot}/.ai/pattern-packs/{codeTypeId}/examples.md` |
| `{projectRoot}/.ai/pattern-packs/{codeTypeId}/docs.md` |
| `{projectRoot}/.ai/state/pattern-extraction-status.md` |

| 禁止 |
|------|
| 写入 `{skillLibraryRoot}/**` |
| 修改 `{projectRoot}` 下 `.ai/` **以外** 的任意文件（含业务源码与构建产物） |
| 在 `.ai/` 下写入本表未列出的路径作为本 Skill 的「正式产出」（备注类合并仅允许落在上述五个路径内） |

## 9. 禁止事项

- 修改项目源码或非本 Skill 允许的 `.ai/` 外文件。
- 创建 `status: active` 的 Pattern Pack，或将 Pack 标记为 `reviewedByHuman: true`。
- 将项目 Pattern Pack 或摘要写入公共 HACF 库。
- 把**单个**样例文件直接当作全模块稳定规则且不标注证据强弱与待确认。
- 生成 `create-code-by-pattern`、正式开发 Skill、Placement 步骤或可执行「自动生成业务代码」指令。
- **静默覆盖**已有 Pattern Pack 整文件；已存在时必须合并或生成冲突说明（见 `pattern-extraction-rule.md`）。
- 虚构未读文件中的符号、注册点或依赖。
- 单次深读文件数超过 §7（须分次执行并标注部分覆盖）。

## 10. 完成标准

- [ ] `pattern-packs/{codeTypeId}/` 下四个 Markdown 均已写入（新建或按规则合并），且各文件含**合法 YAML front matter**，并满足 `pattern-pack-structure-rule.md` 对 `pattern.md` / `validator.md` / `examples.md` / `docs.md` 的必备节与键。
- [ ] 所有 Pack 文件 front matter 含：`status: draft`、`reviewedByHuman: false`、`confidence` 为 `medium` 或 `low`（与证据一致）；**不得**出现 `reviewedByHuman: true` 或 Pack 级 `status: active`。
- [ ] 样例 **< 2** 时：`evidenceStrength: low`（见状态与/或 Rule 约定字段）、`confidence: low`、`needsHumanConfirmation: true`。
- [ ] `pattern.md` 含：`codeTypeId`、`moduleId`、`patternName`、适用场景、不适用场景、固定结构、可变参数、命名规则、文件位置规则、多文件联动规则、生成步骤草案、禁止事项、待人工确认项。
- [ ] `validator.md` 含：结构检查清单、命名检查清单、文件联动检查清单、注册/声明/导出检查清单、配置检查清单、测试建议检查清单、风险检查清单、完成标准。
- [ ] `examples.md` 含：样例文件列表、每个样例的角色、固定结构部分、可变实现部分、证据较强样例、不应作为模板的样例。
- [ ] `docs.md` 含：给人类阅读的代码类型说明、给 Agent 执行的简明规则、新增此类代码的开发说明、常见坑点、待人工确认项。
- [ ] `pattern-extraction-status.md` 已更新，含本轮 `moduleId`、`codeTypeId`、时间、深读列表、证据/截断/合并冲突摘要。
- [ ] 正文中文说明；路径、类名、方法名等标识符保持原文。
- [ ] 未修改业务源码；未向公共库写入项目内容。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| `moduleId` 或 `codeTypeId` 非法 | 中止；提示人类更正。 |
| `code-types.md` 缺失或其中无 `{codeTypeId}` | 中止；提示先执行 `detect-code-types-by-ai` 或补全条目。 |
| 模块四文档之一缺失 | 中止；提示先执行 `scan-module-by-ai` 或补全。 |
| `module-map` 与索引均缺失或不可用 | 可依赖模块文档与 `code-types` 继续，但 Pack 须保守：`confidence: low` 或 `needsHumanConfirmation: true`，并说明依据不足。 |
| 选中 0 个可读样例文件 | 仍生成 Pack 骨架与状态，正文说明数据不足；待确认项列出需在 `module-map` / `code-types` 中补充的路径；`evidenceStrength: low`、`confidence: low`、`needsHumanConfirmation: true`。 |
| 模板或 Rule 缺失 | 中止；在回复中列出缺失的 `{skillLibraryRoot}` 路径。 |
| 已有 Pack 且合并无法自动完成 | 不覆盖；在 `pattern-extraction-status.md` 写清冲突与建议人类动作。 |

**权威 Rule**：证据口径、合并策略、Pack 结构与人工确认边界以 `{skillLibraryRoot}/rules/pattern/pattern-extraction-rule.md`、`pattern-pack-structure-rule.md`、`pattern-human-confirmation-rule.md` 为准；代码类型前置语义与检测阶段边界以 `code-type-detection-rule.md` 为准。
