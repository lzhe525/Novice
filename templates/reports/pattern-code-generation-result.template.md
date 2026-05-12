---
docType: pattern-code-generation-result
generatedBy: ai
reviewedByHuman: false
skill: create-code-by-pattern
codeTypeId: "{{CODE_TYPE_ID}}"
moduleId: "{{MODULE_ID}}"
mode: "{{MODE}}"
resultGeneratedAt: "{{RESULT_GENERATED_AT_ISO}}"
appliedSourceChanges: "{{APPLIED_SOURCE_CHANGES}}"
validatorOverall: "{{VALIDATOR_OVERALL}}"
sourceOfTruth: repository
---

# Pattern 驱动代码生成 — 结果报告

> **事实源声明**：本报告记录本轮 Skill 执行与校验过程；业务行为以仓库源码与测试为准。

## 总览

| 字段 | 值 |
|------|-----|
| `codeTypeId` | {{CODE_TYPE_ID}} |
| `moduleId` | {{MODULE_ID}} |
| `mode` | {{MODE}} |
| 报告生成时间（UTC） | {{RESULT_GENERATED_AT_ISO}} |
| 是否已应用源码变更 | **{{APPLIED_SOURCE_CHANGES}}**（`true` / `false`） |
| `validator.md` 执行总览 | **{{VALIDATOR_OVERALL}}**（`pass` / `failed` / `skipped` / `not_run`） |

## 执行摘要

<!-- 三至八句中文：计划是否落地、触及路径数、阻塞原因 -->

## 已触及文件

<!-- 列出本轮创建或修改的源码相对路径；若未改源码写「无」 -->

## `validator.md` 清单执行记录

<!-- 逐项：检查项简述 | pass / fail / skipped | 备注或命令摘要 -->

| 项 | 结果 | 备注 |
|----|------|------|
| | | |

## 文档更新提示

<!-- 是否建议更新 `.ai/docs/modules/...` 或 `indexes`；无则写「无」 -->

{{DOCS_UPDATE_HINT}}

## 证据与备注

<!-- 路径截断、pattern-index 缺失、人类未确认等 -->
