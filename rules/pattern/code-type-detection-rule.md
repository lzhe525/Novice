# code-type-detection-rule

## 适用主体

执行 `detect-code-types-by-ai`、维护 `{projectRoot}/.ai/docs/modules/{moduleId}/code-types.md` 或 `{projectRoot}/.ai/indexes/code-type-index.md` 的 Agent 与人类维护者。

## 规则陈述

### 写入与只读边界

1. **项目侧产出**只能写入 `detect-code-types-by-ai` Skill「写入位置」表中列出的 `{projectRoot}/.ai/**` 路径。
2. 对 `{projectRoot}` 下业务源码与配置为 **只读**；**不得**为生成代码类型文档而修改 `.ai/` 以外的文件。
3. **不得**将具体项目的代码类型识别结果、索引或状态写入 `{skillLibraryRoot}`（与 `public-skill-library-purity-rule` 一致）。

### 文档语言与标识符

4. 正文以 **简体中文** 为主；**文件路径、仓库相对路径、类名、方法名、枚举名、API 字段名**等与源码或仓库一致，**不翻译**（与 `language-policy-rule` 及项目侧 `language-policy.md` 一致）。

### 推断与人工确认标记

5. **`status: inferred`**：候选代码类型的归纳**并非**由多重直接证据（见下条「证据强弱」）充分支持时，该条目在 `code-types.md` 与 `code-type-index.md` 中必须标为 `inferred`（或与整表约定一致的 YAML/表列）。
6. **`needsHumanConfirmation: true`**：凡存在架构边界不清、命名惯例不确定、证据偏弱、或单一样例即支撑结构判断等情况，必须将对应条目标为 `needsHumanConfirmation: true`。
7. 禁止在 front matter 或正文中声称与已读源码不一致的「确定事实」（落实 `source-of-truth-rule`）。

### 证据强弱（`evidenceStrength`）

8. 每个候选代码类型必须标注 `evidenceStrength`，取值建议为 `strong` | `medium` | `weak`，口径如下：
   - **`strong`**：至少 **2** 个样例文件呈现一致或可对齐的固定结构，且能与 `registryFiles` / 导出点 / 注册点之一形成互证（路径与标识符保持原文列举）。
   - **`medium`**：单文件内结构清晰，且与模块文档（如 `overview.md` / `deep-dive.md`）中的描述可互证，但跨文件重复证据不足。
   - **`weak`**：仅单文件片段、或主要依赖文档推断、或缺少注册点/导出点互证；**不得**将此类结果表述为「稳定模式」「已验证模式」或同等含义措辞。
9. **禁止**在「仅单文件样例、且无注册点/导出点/第二文件互证」时，将候选类型描述为稳定、生产级或已充分抽象就绪的 Pattern；此类情况至少为 `weak`，且通常须 `needsHumanConfirmation: true`。

### 与 Pattern Pack 的边界

10. 本阶段产出仅为 **候选代码类型** 与证据说明；**不得**创建 Pattern Pack、不得生成可执行抽象代码、不得执行 Placement 或 Develop 类步骤。
11. 「是否适合抽象为 Pattern Pack」仅为 **评估性文字**（是/否/条件化），不得写成已实现 Pack 或具体生成指令。
12. **`extract-code-pattern`** 可在具备本阶段产出（含 `code-types.md` 中目标 `codeTypeId`）后，于 `{projectRoot}/.ai/pattern-packs/{codeTypeId}/` 创建 Pattern Pack 草稿；须遵守 `pattern-extraction-rule.md`、`pattern-pack-structure-rule.md` 与 `pattern-human-confirmation-rule.md`。

## 须同时遵守的其它 Rule

- `rules/base/frontmatter-format-rule.md`
- `rules/base/source-of-truth-rule.md`
- `rules/documentation/structured-doc-writing-rule.md`
- `rules/documentation/human-readable-doc-rule.md`
- `rules/documentation/agent-context-budget-rule.md`
- `rules/base/public-skill-library-purity-rule.md`
- `rules/base/project-local-output-rule.md`（写入仅限 `.ai/` 下本 Skill 允许路径）
- `rules/project/scan-readonly-and-artifacts-rule.md`
- `rules/project/module-is-logical-boundary-rule.md`

## 禁止事项

- 将单个样例文件直接等同于全模块的稳定编码模式且不标注弱证据与待确认。
- 虚构未读文件中出现的注册点、导出点或依赖关系。

## 与其它 Rule 的关系

- 与 `module-is-logical-boundary-rule` 互补：后者定义模块边界如何确定；本 Rule 定义在已界定模块内如何负责任地标注代码类型候选与证据。
- 与 `scan-readonly-and-artifacts-rule` 互补：后者强调扫盘只读与 `.ai/` 产物位置；本 Rule 将同等约束延伸到 Pattern 阶段的代码类型识别。
