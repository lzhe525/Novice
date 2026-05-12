# pattern-validation-rule

## 适用主体

执行 `validate-pattern-pack`、阅读 `{projectRoot}/.ai/pattern-packs/{codeTypeId}/` 下 Pattern Pack 草稿或校验报告的 Agent 与人类维护者。

## 规则陈述

### 写入与只读边界

1. 校验 Skill **仅**允许写入 `validate-pattern-pack` Skill「写入位置」表中列出的 `{projectRoot}/.ai/**` 路径；**不得**修改 Pack 内任一文件、**不得**修改业务源码、**不得**修改 `{skillLibraryRoot}/**`。
2. **不得**将 `reviewedByHuman` 改为 `true`，**不得**覆盖 Pack 正文中人类已填写的确认结论或签字类内容（校验产出仅限状态与报告两文件）。

### 与公共库及事实源

3. Pack 正文不得指示将项目专有 Pattern、扫描结果或业务路径写入 `{skillLibraryRoot}`（落实 `public-skill-library-purity-rule`）。
4. YAML front matter 与正文路径须遵守 `frontmatter-format-rule`；不得声称与仓库文件不一致的「已存在」路径（落实 `source-of-truth-rule`）。

### Pack 文件与 front matter（检查项 1–5）

5. **目录**：`{projectRoot}/.ai/pattern-packs/{codeTypeId}/` 必须存在；不存在则检查项 1 为 **failed**，后续依赖该目录的检查项可标 **skipped** 并说明原因。
6. **必需文件**（检查项 2）：下列四文件均须存在且为可读 Markdown：  
   `pattern.md`、`validator.md`、`examples.md`、`docs.md`。缺一为 **failed**。
7. **front matter**（检查项 3）：上述每个文件开头须存在合法 YAML front matter（第一行 `---`、闭合 `---`、闭合后空一行再接正文）；无法解析为合法 YAML 则为 **failed**。
8. **`status`**（检查项 4）：每个 Pack 文件 front matter 中的 `status` 须为 **`draft`**、**`active`**、**`deprecated`** 之一（仅此三值）；缺失或非法为 **failed**。
9. **`reviewedByHuman`**（检查项 5）：须显式为布尔 **`true`** 或 **`false`**；缺失、非布尔或字符串混用为 **failed**。

### `status` 与 `reviewedByHuman` 一致性（检查项 6–7）

10. **检查项 6**：任一 Pack 文件若为 `status: draft` 且 `reviewedByHuman: true`，**failed**。
11. **检查项 7**：任一 Pack 文件若为 `status: active` 且 `reviewedByHuman` 不为 `true`，**failed**。
12. **`deprecated`**：`reviewedByHuman` 可为 `true` 或 `false`，不设与 `draft`/`active` 相同的强制组合；若与其它文件同一 Pack 内 front matter 不一致，记 **warning** 并在报告中说明。

### `pattern.md` 最低内容（检查项 8）

13. 正文（不含 front matter）须**显式**覆盖下列五类信息，允许用二级标题或小节标题表述，也允许用加粗引导词 + 段落；Agent 校验时以「标题或首行含下列关键词之一」为匹配辅助：  
    - **固定结构**（关键词示例：`固定结构`、固定结构）  
    - **可变部分**（`可变部分`、可变、扩展点）  
    - **命名规则**（`命名规则`、命名约定、命名规范）  
    - **文件位置规则**（`文件位置`、放置路径、目录约定）  
    - **多文件联动**（`多文件联动`、多文件、联动、注册点、导出点）  
    缺任一类为 **failed**。

### `validator.md` 可执行清单（检查项 9）

14. 正文须包含**可执行**检查清单，至少满足下列之一：  
    - 不少于 **3** 条 Markdown 任务列表项 `- [ ]` 或 `- [x]`；或  
    - 不少于 **3** 条编号步骤（`1.` / `1)` 形式）且每步为可观察、可验证动作；或  
    - 带语言标记的**可复制**代码围栏（如 `bash`、`powershell`、`sh`）内给出可运行检查命令。  
    不满足为 **failed**。

### `examples.md` 样例与证据（检查项 10）

15. 须列举至少 **1** 个**仓库相对路径**（指向 `{projectRoot}` 下真实文件或目录的约定写法，如 `` `src/...` `` 或 Markdown 链接路径），并配有**证据说明**（为何该样例支撑本 Pattern，非空段落）。  
    无任何仓库相对路径则为 **failed**；有路径但证据极短（例如单句无指向具体结构）为 **warning**。

### `docs.md` 双受众（检查项 11）

16. 须同时体现**人类阅读**与 **Agent 执行** 两类受众，至少满足下列之一：  
    - 两个独立小节标题分别含「人类」与「Agent」（或「执行者」）；或  
    - 表格中含「人类 / Agent」或「读者 / 执行」列说明；或  
    - 首段明确写清「本文面向人类审阅者与自动化 Agent 的用途分工」。  
    不满足为 **warning**。

### 单样例过度归纳（检查项 12）

17. 若 `examples.md` 中可解析的**不同**样例文件路径少于 **2** 个，且 `pattern.md` 出现强烈绝对化措辞（如「所有」「必须一律」「唯一方式」等）且无「待确认」「建议」缓冲：检查项 12 为 **warning**；若同时缺少多文件联动说明且声称覆盖全仓库模式，可为 **failed**（由 Agent 在报告中引用具体句）。  
    若样例不少于 **2** 个且有互证描述，检查项 12 倾向 **pass**。

### 缺失人工确认项（检查项 13）

18. 在 `pattern.md`、`docs.md` 或 `examples.md` 任一 front matter 或正文中，须出现 **`needsHumanConfirmation`**（YAML 或正文键名）或明确小节/列表「待人工确认」「需人工确认」。  
    当 Pack 内**所有**文件 `status` 均为 `draft` 时，若三文件均不满足本条，为 **warning**；若 `status` 已为 `active` 仍完全缺失，为 **failed**。

### 引用路径存在性（检查项 14）

19. 仅从下列来源收集待校验路径字符串，**总数上限 40**（超出部分记 **warning**「未校验截断」）：  
    - 四份 Pack 文件正文中的 Markdown 链接目标、反引号包裹的类 Unix 风格相对路径；  
    - `` `...` `` 内形如 `src/`、`apps/`、`packages/` 等以项目根为根的相对路径。  
20. 排除：`http(s)://`、`mailto:`、`#` 锚点、`{{` 占位符、明显模板占位、`..` 段、`skillLibraryRoot` 字面路径。  
21. 对每个候选路径，若规范化后指向 `{projectRoot}/<path>` 的文件或目录**不存在**，检查项 14 记 **failed**（列出首个失败路径即可在同项汇总，其余列附录）；若为可选资源且文中已标「若存在」可降为 **warning**。

### 公共库污染风险（检查项 15）

22. 若在 Pack 正文中发现下列任一，检查项 15 为 **failed**：  
    - 明示将项目内容**提交、同步、复制**到 `{skillLibraryRoot}` 或「HACF 公共库」「公共 Skill 库」的步骤；  
    - 将具体业务项目绝对路径与 `skills/`、`rules/`、`templates/`（公共库布局）组合成写入目标；  
    - 要求把本项目 `.ai/pattern-packs/` 下文件**移动**到公共库目录树。  
23. 若仅泛述「上游通用 Pattern 在公共库」而无写入指令，为 **pass**。

### 汇总语义

24. **failed**：至少一项检查为 **failed**。  
25. **warning**：无 **failed**，但至少一项为 **warning**。  
26. **pass**：无 **failed** 且无 **warning**（或约定将信息类记为 pass 时须在报告备注）。  
27. **`canSubmitForHumanReview`**：当且仅当无 **failed**、且检查项 6–7 未触发、且检查项 15 未触发时为 `true`；否则为 `false`。

## 须同时遵守的其它 Rule

- `rules/base/frontmatter-format-rule.md`
- `rules/base/source-of-truth-rule.md`
- `rules/base/public-skill-library-purity-rule.md`
- `rules/base/project-local-output-rule.md`
- `rules/project/scan-readonly-and-artifacts-rule.md`
- `rules/documentation/agent-context-budget-rule.md`（路径枚举上限见上文）

## 禁止事项

- 通过校验操作反向修改 Pack 草稿或业务源码。
- 在校验报告中伪造未执行的存在性检查。

## 与其它 Rule 的关系

- 与 `code-type-detection-rule` 互补：后者定义代码类型候选；本 Rule 定义 Pack 草稿结构与证据的校验口径。
- 与 `public-skill-library-purity-rule` 互补：检查项 15 将纯洁性要求落到可判定启发式。
