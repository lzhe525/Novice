# pattern-extraction-rule

## 适用主体

执行 `extract-code-pattern`、维护 `{projectRoot}/.ai/pattern-packs/{codeTypeId}/**` 或 `{projectRoot}/.ai/state/pattern-extraction-status.md` 的 Agent 与人类维护者。

## 规则陈述

### 写入与只读边界

1. **项目侧产出**只能写入 `extract-code-pattern` Skill「写入位置」表中列出的 `{projectRoot}/.ai/**` 路径。
2. 对 `{projectRoot}` 下业务源码与 `.ai/` 外配置为 **只读**；**不得**为生成 Pattern Pack 而修改 `.ai/` 以外的文件。
3. **不得**将具体项目的 Pattern Pack、状态全文或项目私有摘要写入 `{skillLibraryRoot}`（与 `public-skill-library-purity-rule` 一致）。

### 与代码类型检测的衔接

4. **须**以 `.ai/docs/modules/{moduleId}/code-types.md` 中对应 `{codeTypeId}` 条目为种子；不得跳过 `code-types.md` 凭空定义类型。
5. `detect-code-types-by-ai` **不得**创建 Pattern Pack；本规则仅约束 **提取** 阶段。两阶段产物边界与 `code-type-detection-rule.md` 第 10–11 条一致。

### 文档语言与标识符

6. 正文以 **简体中文** 为主；**文件路径、类名、方法名、枚举名、API 字段名**等与源码或仓库一致，**不翻译**（与 `language-policy-rule` 及项目侧 `language-policy.md` 一致）。

### 事实源与推断

7. Pattern 描述**不得**声称未读文件中存在的符号或注册关系（落实 `source-of-truth-rule`）。
8. 抽象须基于已读样例与已读 `.ai` 文档；无依据处须标「待确认」或「推断」。

### 抽象证据强度（`evidenceStrength`，本轮提取语义）

下列口径用于 `pattern-extraction-status.md` 及 Pack 正文对「本轮抽象所依据的样例文件数量与互证」的标注，**不替代** `code-types.md` 条目内的 `evidenceStrength`（strong / medium / weak）；若两者并存，须在 `docs.md` 或状态中说明二者关系。

9. **`evidenceStrength: low`**：实际纳入抽象且可引用的**不同样例源文件**少于 **2** 个，或主要依赖单文件/文档推断。
10. **`evidenceStrength: medium`**：至少 **2** 个不同样例源文件，结构可对齐，但注册/导出/配置互证不完整。
11. **`evidenceStrength: high`**：至少 **2** 个样例且与注册点、导出点或配置之一形成清晰互证（路径与标识符保持原文列举）。

### Pattern Pack front matter 与 `confidence`

12. 所有 Pack 文件（`pattern.md`、`validator.md`、`examples.md`、`docs.md`）须含合法 YAML front matter，且至少包含：  
    `status: draft`、`reviewedByHuman: false`、`confidence: medium` **或** `confidence: low`（与 `pattern-pack-structure-rule.md` 一致）。
13. 当本条第 9 款成立（样例 **< 2**）时：须 `confidence: low`，且须 `needsHumanConfirmation: true`。
14. 当第 9 款不成立且合并无强制人工项时：通常 `confidence: medium`；若 `code-types` 条目 `needsHumanConfirmation: true` 或条目 `status: inferred`，则 Pack front matter 仍须 `needsHumanConfirmation: true`。

### 已存在 Pattern Pack：合并与冲突

15. **禁止**在不读取旧文件的情况下，用新运行结果**整文件覆盖** `pattern.md`、`validator.md`、`examples.md`、`docs.md` 中任一的既有正文。
16. 若目录已存在：执行 Agent **须**先读取现有四个文件；新内容通过下列方式之一落地：  
    - **合并**：在文件末尾追加「变更记录」或「再次提取（ISO 时间）」小节，汇总新增/修正点；  
    - **冲突说明**：无法调和的断言不得强行覆盖；须在 `pattern-extraction-status.md` 的 **「冲突与合并说明」** 中逐条列出，并在各 Pack 的「待人工确认项」中重复或引用。
17. `pattern-extraction-status.md` 可整体覆盖为**最近一次**运行的元数据摘要，但须在正文保留或追加合并/冲突历史（至少保留上一轮摘要或指向人类 Git 历史由人类处理）。

### 禁止事项（提取阶段）

18. **不得**将单个样例文件直接等同于全仓库稳定规则且不标注证据与待确认。
19. **不得**在本阶段生成 `create-code-by-pattern`、正式开发 Skill 或可执行业务代码生成指令。
20. **不得**创建 `status: active` 的 Pattern，或将 Pack 的 `reviewedByHuman` 设为 `true`（见 `pattern-human-confirmation-rule.md`）。

## 须同时遵守的其它 Rule

- `rules/base/frontmatter-format-rule.md`
- `rules/base/source-of-truth-rule.md`
- `rules/base/language-policy-rule.md`
- `rules/base/public-skill-library-purity-rule.md`
- `rules/base/project-local-output-rule.md`
- `rules/base/read-source-before-edit-rule.md`
- `rules/project/module-is-logical-boundary-rule.md`
- `rules/pattern/code-type-detection-rule.md`
- `rules/pattern/pattern-pack-structure-rule.md`
- `rules/pattern/pattern-human-confirmation-rule.md`
- `rules/documentation/structured-doc-writing-rule.md`
- `rules/documentation/human-readable-doc-rule.md`
- `rules/documentation/agent-context-budget-rule.md`
- `rules/project/scan-readonly-and-artifacts-rule.md`

## 与其它 Rule 的关系

- 与 `code-type-detection-rule` 互补：检测阶段只产候选类型；提取阶段才允许创建 `.ai/pattern-packs/`。
- 与 `module-is-logical-boundary-rule` 互补：提取时的阅读与样例选取仍须尊重模块边界与 `module-map` 优先顺序。
