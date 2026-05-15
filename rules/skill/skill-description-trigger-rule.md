# skill-description-trigger-rule

## 适用主体

在 `{skillLibraryRoot}` 内**新建或修订**公共 Skill 文件（`skills/**/*.md`）的维护者或 Agent。

## 规则陈述

HACF 公共库 Skill 以单文件 Markdown 为主，不强制使用 YAML `name` / `description` 头；**可发现性**依赖人类与 Agent 在正文中快速判断「做什么、何时用」。本规则约束**中文叙述**与**英文路径/标识符**的分工，使 Skill 易于检索且不与公共库纯度要求冲突。

## 格式与内容要求

1. **标题行**：文件顶部一级标题必须为 `# Skill：<skill-file-basename>`，其中 `<skill-file-basename>` 与文件名（不含 `.md`）一致，使用 `kebab-case` 英文。  
2. **目的（第 1 节）**：首段用一至两句中文说明**能力**与**典型触发**（例如依赖的前置产物、协作阶段、用户常用说法）。  
3. **适用场景（第 2 节）**：使用无序列表列出若干条**可检索的触发条件**（等价于「Use when…」信息，但用中文短句表达）；每条应可独立判断「是否该打开本 Skill」。  
4. **语言分工**：自然语言说明默认**中文**；文件路径、Skill 文件名、`{projectRoot}`、`{skillLibraryRoot}`、变量名、代码标识符、仓库内相对路径**英文**（或与现有 Skill 一致的约定英文）。  
5. **禁止过度承诺**：正文不得声称本 Skill 提供 issue tracker、git hooks、完整 TDD、完整系统诊断、`to-prd` / `to-issues` 式发布流程等，除非仓库内**另有**独立 Skill 且本 Skill 明确编排指向该 Skill。  
6. **可选 YAML front matter**：若项目侧或未来模板为 Skill 增加 front matter，须遵守 [`../base/frontmatter-format-rule.md`](../base/frontmatter-format-rule.md)。

## 与其它 Rule 的关系

- 与 [`../base/public-skill-library-purity-rule.md`](../base/public-skill-library-purity-rule.md) 一致：描述与示例**不得**诱导将业务项目私有扫描结果、密钥、内网地址或未脱敏日志写回 `{skillLibraryRoot}`。  
- 与 [`../base/project-local-output-rule.md`](../base/project-local-output-rule.md) 一致：若 Skill 会写入 `{projectRoot}/.ai/**`，须在各自 Skill 的「写入位置」节列出路径，且该路径须已被 `project-local-output-rule` 白名单收录。  
- 与 [`../documentation/human-readable-doc-rule.md`](../documentation/human-readable-doc-rule.md) 互补：本规则侧重触发与发现性，不改变文档可读性的一般要求。

## 允许路径

- `{skillLibraryRoot}/skills/**/*.md` 中由人类或受控 Agent 维护的 Skill 正文。

## 禁止路径

- 在 `{skillLibraryRoot}` 内保存任何**具体业务项目**的扫描全文、报告正文、密钥或仅适用于单一仓库的长篇私有说明（应放在 `{projectRoot}/.ai/` 下由对应 Skill 定义）。

## 自检清单（非强制执行字段）

维护者或 Agent 在合并前可核对：

- [ ] 「目的」首段是否同时交代**能力**与**典型触发**？  
- [ ] 「适用场景」是否含 2 条以上可独立判断的触发句？  
- [ ] 路径与文件名是否均为约定英文，且与 `SKILL_ROUTER`（若已登记）中的「Skill 路径」一致？  
- [ ] 是否未在正文暗示将项目私有内容写入公共库？
