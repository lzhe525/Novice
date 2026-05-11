---
docType: module-index
status: draft
generatedBy: ai
reviewedByHuman: false
lastReviewedAt: ""
sourceOfTruth: repository
scanSkill: scan-project-by-ai
generatedAt: "{{SCANNED_AT_ISO}}"
---

# 模块文档索引（Module Index）

> **事实源声明**：本索引仅列出 `.ai/docs/modules/` 下**已生成**的文档路径与仓库模块根路径的映射，**非**源码索引。实现以各模块源码目录为准。

## 使用方式（Agent）

1. 会话开场**不要**通读整个 `.ai/docs/`；先读本文件与 `.ai/docs/project/adapter.md`。
2. 按任务只打开索引表中与当前任务相关的 **1–3** 行所指向的 Markdown 路径。

## 模块条目

Agent 在完成 `scan-module-by-ai` 后，将下列表格补全为有效 Markdown 行；尚未扫盘的模块不要虚构链接。

| moduleRootRelative（仓库） | overview | deep-dive | constraints | change-guide | risk-points |
|----------------------------|----------|-----------|-------------|--------------|-------------|

## 项目级文档快捷链接

| 文档 | 相对 `{projectRoot}/.ai/` 路径 |
|------|--------------------------------|
| Project overview | `docs/project/overview.md` |
| Project deep-dive | `docs/project/deep-dive.md` |
| Project adapter | `docs/project/adapter.md` |
| Adapter evidence | `docs/project/adapter-evidence.md` |
