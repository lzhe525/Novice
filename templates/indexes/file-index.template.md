---
docType: file-index
status: draft
generatedBy: ai
reviewedByHuman: false
lastReviewedAt: ""
sourceOfTruth: repository
scanSkill: scan-project-by-ai
generatedAt: "{{SCANNED_AT_ISO}}"
---

# 文件与模块归属索引（File Index）

> **事实源声明**：本文件记录**源文件相对路径**与 **`moduleId`** 的归属关系，可与 `scan-file-by-ai` 在 `.ai/docs/files/` 下生成的单文件说明并存。同一文件可属于多个逻辑模块：允许**重复 `filePathRelative` 多行**（每行一个 `moduleId`），或在 `moduleIds` 列内使用**英文逗号**分隔多个 `moduleId`（项目任选其一约定，本表两种均合法）。

## 使用方式（Agent）

1. 判断某文件属于哪些能力边界时，先检索本表再读 `module-map.md`。
2. 合并更新时**不要**删除未理解的人类手工行；新增行追加在表末或按 `filePathRelative` 字母序插入。

## 文件条目

| filePathRelative（相对 `{projectRoot}`） | moduleIds | 备注 |
|------------------------------------------|-----------|------|
| <!-- 示例：src/app/main.ts --> | <!-- 示例：core 或 billing,api --> | <!-- 可选：推断 / 来自 module-map --> |

## 与 scan-module-by-ai 的衔接

`scan-module-by-ai` 在完成深读后，应为本轮已读且能确定归属的文件追加或合并 `moduleIds` 中的对应 `moduleId`；若无法确定归属则不要虚构 `moduleId`。
