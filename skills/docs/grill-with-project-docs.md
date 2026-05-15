# Skill：grill-with-project-docs

## 1. 目的

在**不修改业务源码、不修改公共 HACF Skill 库**的前提下，围绕用户提出的方案或设计意图，结合项目已写入 `{projectRoot}/.ai/` 的文档与索引，通过**单问单答**的追问与对照，澄清术语、边界与可核对事实；仅在用户明确要求固化结论时，将纪要写入白名单路径下的单一 Markdown 文件。

## 2. 适用场景

- 用户希望「拷问」「对齐」「压力测试」某一方案，使其与项目内已有 AI 协作文档一致。
- 扫盘或适配器文档已存在，需要在变更前统一词汇与模块边界表述。
- 用户希望用具体场景暴露接口或数据边界上的模糊地带（**不**进入完整架构改造或工单拆分流程）。
- 用户明确要求将本轮已达成共识的结论写入 `.ai/docs/project/design-grill-notes.md`（与「固化结论」同义）。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根目录。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；**仅用于读取** Rule 与 Skill，**不得**写入。 |
| `{userProposal}` | 用户待拷问的方案、决策列表或问题陈述（可由对话提供）。 |
| `{persistNotes}` | 可选。仅当与用户**明确要求固化结论**等价（例如明确说「写入 design-grill-notes」或「固化到项目文档」）时，允许执行 §6 中的写盘步骤；否则默认**不落盘**。 |

## 4. 输出

| 输出类型 | 说明 |
|----------|------|
| Agent 对话 | 默认主产出：一次一个关键问题、附推荐立场或选项，等待用户回答后再继续。 |
| `{projectRoot}/.ai/docs/project/design-grill-notes.md` | **仅当**与用户明确要求固化结论等价且已执行 §6 写盘步骤时：创建或按小节合并更新；不得整份覆盖扫盘主文档（如 `overview.md`、`deep-dive.md`）。 |

执行含写盘步骤时，须在回复中列出实际写入的相对 `{projectRoot}` 路径。

## 5. 前置条件

- `{projectRoot}/.ai/` 目录存在（Bootstrap 已完成或等价结构已建立）；若不存在，不得创建除本 Skill「写入位置」外的其它 `.ai/` 子树以外的业务文件。
- 建议只读：`{skillLibraryRoot}/rules/base/source-of-truth-rule.md`、`{skillLibraryRoot}/rules/base/project-local-output-rule.md`、`{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、`{skillLibraryRoot}/rules/documentation/agent-context-budget-rule.md`、`{skillLibraryRoot}/rules/base/read-source-before-edit-rule.md`。
- 若需核对实现细节：对业务源码**仅只读**打开，**禁止**编辑。

## 6. 执行步骤

1. **拉远视角（zoom-out）**：基于 `.ai/docs/project/overview.md`、`deep-dive.md`、`adapter.md`（若存在）与 `.ai/indexes/module-index.md` 等，用项目文档中的模块名与关系，给出与 `{userProposal}` 相关的一小段「地图」（调用方向或数据流方向即可），避免引入与项目文档无关的新造模块名。
2. **建立对照基线**：阅读 `.ai/entry/AI_ENTRY.md`、极薄 `{projectRoot}/AGENTS.md` 中指向入口的段落；抽样阅读与方案最相关的 `.ai/docs/modules/<moduleId>/**` 或 `.ai/docs/files/**`（有则读，无则记缺口但不虚构）。
3. **单问单答循环**：  
   - 每次只提**一个**关键问题；可给出简短「若选 A / 若选 B」式推荐及理由。  
   - 若用户用语与已有文档标题或术语冲突，直接指出并请求定夺。  
   - 若问题可通过只读源码核对，则打开相关文件验证；发现与文档矛盾时，引用路径说明矛盾，**不得**为弥合矛盾而改源码或改扫盘五件套全文。
4. **场景探针（轻量）**：在讨论关系或边界时，构造**一个**小而具体的场景（输入 / 状态 / 期望结果），用于暴露歧义；不展开完整诊断或测试矩阵。
5. **垂直切片思维（仅口述）**：如需拆分工作，仅作为对话建议，说明「每一竖切应能独立演示或验证」的原则；**不得**创建 issue、PRD 或调用外部工单系统。
6. **可选写盘**：当且仅当用户**明确要求固化结论**（与 `{persistNotes}` 输入等价）时：  
   - 若 `design-grill-notes.md` 不存在，可新建；若已存在，在文首或「变更记录」小节**追加**本轮日期与结论摘要，或按既有标题合并段落。  
   - 写入内容须可被 §2–§4 已读文档或只读源码证据支持；禁止编造与源码不一致的「事实」。  
   - **禁止**重写或清空 `.ai/docs/project/overview.md`、`deep-dive.md` 等扫盘主产物。
7. **收尾**：在对话中列出仍开放的问题（若有）及建议下一步阅读的 `.ai/` 相对路径（不粘贴长篇原文）。

## 7. 读取范围

| 路径（相对 `{projectRoot}/`，除非注明） | 说明 |
|------------------------------------------|------|
| `.ai/entry/AI_ENTRY.md`、`.ai/entry/SKILLKIT_LINK.md` | 入口与公共库链接 |
| `AGENTS.md` | 极薄入口，仅抽样 |
| `.ai/docs/project/**`、`.ai/docs/modules/**`、`.ai/docs/files/**` | 项目 AI 文档事实 |
| `.ai/indexes/*.md` | 索引 |
| 与讨论相关的业务源码文件 | **只读**、有针对性 |
| `{skillLibraryRoot}/rules/**` | 按需只读 |

## 8. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/docs/project/design-grill-notes.md`（**仅当**用户明确要求固化结论且该 Skill 已执行写盘步骤；**不得**据此 Skill 覆盖或整份重写 `overview.md`、`deep-dive.md`、`adapter.md`、`adapter-evidence.md` 等扫盘主文档） |

| 禁止 |
|------|
| 修改 `{projectRoot}` 下任意业务源码 |
| 覆盖或整份重写 `.ai/docs/project/overview.md`、`deep-dive.md`、`adapter.md`、`adapter-evidence.md` 等扫盘主文档 |
| 修改 `.ai/reports/**`、`.ai/state/**` 下任意文件（本 Skill 未授权其它路径） |
| 写入或修改 `{skillLibraryRoot}/**` |
| 在未满足「用户明确要求固化结论」且未执行写盘步骤时，创建或更新 `design-grill-notes.md` |

## 9. 禁止事项

1. **不得**在本 Skill 中实现完整缺陷诊断循环、完整 TDD 红绿重构、issue tracker 对接或 PRD 发布。  
2. **不得**要求用户执行 shell、CLI 或脚本以完成本 Skill。  
3. **不得**在业务项目根目录创建 `CONTEXT.md` 或独立 ADR 目录树（本框架以 `.ai/` 为协作根）。  
4. **不得**将项目私有扫描结果、密钥或内网细节复制到 `{skillLibraryRoot}`。

## 10. 完成标准

- [ ] 对话已体现「单问单答」节奏；若读过文档或源码，结论可追溯到具体路径。  
- [ ] 若未显式要求固化：未写入 `design-grill-notes.md`。  
- [ ] 若显式要求固化：`design-grill-notes.md` 仅为追加或合并更新，且未破坏扫盘主文档。  
- [ ] 未修改业务源码与公共库。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| `.ai/` 不存在 | 中止写盘；仅在对话中继续拷问，并提示先完成 Bootstrap。 |
| 关键文档均缺失 | 在对话中标注证据缺口；不写与源码无法核对的事实性断言；`{persistNotes}` 为真时仅记录「待扫盘补充」类元信息。 |
| 用户拒绝固化 | 不执行写盘步骤。 |

## 权威 Rule

协作产物位置以 [`{skillLibraryRoot}/rules/base/project-local-output-rule.md`](../../rules/base/project-local-output-rule.md) 为准；事实性以 [`{skillLibraryRoot}/rules/base/source-of-truth-rule.md`](../../rules/base/source-of-truth-rule.md) 为准。
