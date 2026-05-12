---
docType: pattern-validation-report
generatedBy: ai
reviewedByHuman: false
validationSkill: validate-pattern-pack
codeTypeId: "{{CODE_TYPE_ID}}"
overallResult: "{{OVERALL_RESULT}}"
canSubmitForHumanReview: "{{CAN_SUBMIT_FOR_HUMAN_REVIEW}}"
blockingFailureCount: {{BLOCKING_FAILURE_COUNT}}
validatedAt: "{{VALIDATED_AT_ISO}}"
sourceOfTruth: repository
---

# Pattern Pack 校验报告

> **事实源声明**：本报告为 `validate-pattern-pack` 的校验记录，非业务源码说明。路径存在性以运行校验时 `{projectRoot}` 下文件系统为准。

## 总览

| 字段 | 值 |
|------|-----|
| `codeTypeId` | {{CODE_TYPE_ID}} |
| Pack 目录（相对 `.ai/`） | `pattern-packs/{{CODE_TYPE_ID}}/` |
| 校验时间（UTC） | {{VALIDATED_AT_ISO}} |
| 总体结果 | **{{OVERALL_RESULT}}** |
| 是否可提交人工确认 | **{{CAN_SUBMIT_FOR_HUMAN_REVIEW}}** |
| `blockingFailureCount`（与 `activate-pattern-pack` 门禁一致） | **{{BLOCKING_FAILURE_COUNT}}** |

## 检查项结果（1–15）

| 编号 | 检查项 | 结果 |
|------|--------|------|
| 1 | Pattern Pack 目录是否存在 | <!-- pass / failed / warning / skipped --> |
| 2 | 必需文件是否存在 | |
| 3 | 各文件合法 YAML front matter | |
| 4 | `status` 取值合法 | |
| 5 | `reviewedByHuman` 为布尔 | |
| 6 | `draft` 不得 `reviewedByHuman: true` | |
| 7 | `active` 必须 `reviewedByHuman: true` | |
| 8 | `pattern.md` 含固定结构、可变部分、命名规则、文件位置规则、多文件联动 | |
| 9 | `validator.md` 含可执行检查清单 | |
| 10 | `examples.md` 含样例路径与证据说明 | |
| 11 | `docs.md` 兼顾人类阅读与 Agent 执行 | |
| 12 | 单样例过度归纳风险 | |
| 13 | 缺失人工确认项 | |
| 14 | 引用路径在项目中存在 | |
| 15 | 无引导将项目私有 Pattern 写入公共 HACF 库 | |

## 失败项清单

<!-- 每项：检查项编号、说明、建议修复文件（相对 `{projectRoot}/.ai/`） -->

- （若无则写「无」）

## 警告项清单

<!-- 每项：检查项编号、说明、建议修复文件 -->

- （若无则写「无」）

## 路径存在性检查附录

<!-- 列出已校验的候选路径及结果；若无则写「无」 -->

## 证据与备注

<!-- 引用 `code-type-index.md` 或前置文件缺失说明等 -->
