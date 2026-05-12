---

docType: code-type-index

status: draft

generatedBy: ai

reviewedByHuman: false

lastReviewedAt: ""

sourceOfTruth: repository

scanSkill: detect-code-types-by-ai

generatedAt: "{{SCANNED_AT_ISO}}"

---



# 代码类型索引（Code Type Index）



> **事实源声明**：本索引**不是**源码事实源；以各模块下 `.ai/docs/modules/{moduleId}/code-types.md` 为准。索引行为「跨模块检索表」，便于 Agent 按 `moduleId` / `codeTypeId` 定位候选类型与证据强弱。



## 使用方式（Agent）



1. 按需打开本表中的 **1–3** 行所指向的 `code-types.md`，不要默认通读所有模块文档。

2. 模块边界与扫描路径仍以 `.ai/config/module-map.md` 与 `scan-module-by-ai` 产出为准。



## 合并更新规则（强制）



- 主键为 **`moduleId`** + **`codeTypeId`**。执行 `detect-code-types-by-ai` 后，对本轮 `moduleId` 相关行进行**查找替换或追加**；**禁止**无备份整体覆盖人类已维护的大段内容。

- **不得**为尚未执行代码类型识别的模块虚构 `codeTypeId` 行。

- 同一 `codeTypeId` 在单模块内应唯一；跨模块允许同名 `codeTypeId`（以 `moduleId` 区分）。



## 代码类型条目



Agent 在完成 `detect-code-types-by-ai` 后，将下列表格补全为有效 Markdown 行；锚点列建议指向 `docs/modules/{moduleId}/code-types.md` 内小节 `## codeType-{codeTypeId}`（若该锚点不存在则链至文件即可）。



| moduleId | codeTypeId | displayName | evidenceStrength | status | needsHumanConfirmation | codeTypesDoc |

|----------|------------|-------------|-------------------|--------|------------------------|--------------|

| | | | | | | |



## 项目级文档快捷链接（可选）



| 文档 | 相对 `{projectRoot}/.ai/` 路径 |

|------|--------------------------------|

| Module index | `indexes/module-index.md` |

| File index | `indexes/file-index.md` |

| Directory index | `indexes/directory-index.md` |


