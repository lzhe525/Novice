---
docType: pattern-code-generation-plan
generatedBy: ai
reviewedByHuman: false
skill: create-code-by-pattern
codeTypeId: "{{CODE_TYPE_ID}}"
moduleId: "{{MODULE_ID}}"
mode: "{{MODE}}"
planGeneratedAt: "{{PLAN_GENERATED_AT_ISO}}"
sourceOfTruth: repository
---

# Pattern 驱动代码生成 — 变更计划

> **事实源声明**：本计划用于约束后续受控改码范围；实现细节以 `{projectRoot}` 下已读源码为准，不得仅凭 Pattern 正文替代读源。

## 总览

| 字段 | 值 |
|------|-----|
| `codeTypeId` | {{CODE_TYPE_ID}} |
| `moduleId` | {{MODULE_ID}} |
| `mode` | {{MODE}} |
| 计划生成时间（UTC） | {{PLAN_GENERATED_AT_ISO}} |
| 用户需求摘要 | {{USER_REQUIREMENT_SUMMARY}} |

## 前置与门禁摘要

<!-- blocked / ready 等一句话；例如 pattern-index 缺失、Pack 非 active、skillkit blocked -->

{{GATE_SUMMARY}}

## 拟变更文件清单

<!-- 每行：仓库相对路径 | 操作（新增/修改）| 与 Pattern 小节或联动说明的对应关系 -->

| 路径（相对项目根） | 操作 | 依据（Pattern / 样例 / 模块文档） |
|--------------------|------|-------------------------------------|
| <!-- 示例占位，执行时删除本行 --> | | |

## 分文件变更意图

<!-- 分小节或列表：每文件拟做什么，引用已读事实 -->

## 风险与待确认项

<!-- 高风险路径、证据不足、validator 可能无法执行的环境等 -->

## 文档同步提示

<!-- 若结构变化可能影响 `.ai/docs/`，列出建议人类跟进项；无则写「无」 -->
