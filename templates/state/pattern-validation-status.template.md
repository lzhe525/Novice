---
lastPatternValidationAt: "{{VALIDATED_AT_ISO}}"
validationSkill: validate-pattern-pack
codeTypeId: "{{CODE_TYPE_ID}}"
patternPackRelativePath: "pattern-packs/{{CODE_TYPE_ID}}/"
overallResult: "{{OVERALL_RESULT}}"
canSubmitForHumanReview: "{{CAN_SUBMIT_FOR_HUMAN_REVIEW}}"
failedCount: "{{FAILED_COUNT}}"
warningCount: "{{WARNING_COUNT}}"
---

# pattern-validation-status

> **事实源声明**：本文件记录最近一次 **Pattern Pack 校验** 的元数据与结论摘要，不替代 Pack 正文；事实以 `{projectRoot}` 仓库内文件为准。

## 本轮摘要

- 校验时间（UTC）：{{VALIDATED_AT_ISO}}
- 执行 Skill：`validate-pattern-pack`
- `codeTypeId`：{{CODE_TYPE_ID}}
- Pack 目录（相对 `.ai/`）：`pattern-packs/{{CODE_TYPE_ID}}/`
- 总体结果：`{{OVERALL_RESULT}}`（`pass` / `failed` / `warning`）
- 是否可提交人工确认：`{{CAN_SUBMIT_FOR_HUMAN_REVIEW}}`（`true` / `false`）

## 计数

- 失败项数：{{FAILED_COUNT}}
- 警告项数：{{WARNING_COUNT}}

## 主要结论

<!-- 一至三句中文：例如「四文件齐全但 pattern.md 缺少多文件联动小节」 -->

## 推荐下一步

<!-- 例如：修正 `pattern-packs/{{CODE_TYPE_ID}}/pattern.md` 后重跑本 Skill；或「无」 -->

## 备注

<!-- 人类或 Agent：前置文件缺失、路径校验截断等 -->
