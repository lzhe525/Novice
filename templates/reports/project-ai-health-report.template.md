---
docType: project-ai-health-report
generatedBy: ai
reviewedByHuman: false
healthCheckSkill: check-project-ai-health
overallHealth: "{{OVERALL_HEALTH}}"
checkedAt: "{{CHECKED_AT_ISO}}"
criticalCount: {{CRITICAL_COUNT}}
highCount: {{HIGH_COUNT}}
mediumCount: {{MEDIUM_COUNT}}
lowCount: {{LOW_COUNT}}
blockingIssuesCount: {{BLOCKING_ISSUES_COUNT}}
humanInterventionRequired: {{HUMAN_INTERVENTION_REQUIRED}}
sourceOfTruth: repository
---

# 项目 AI 协作框架健康检查报告

> **事实源声明**：本报告为 `check-project-ai-health` 的检查记录，非业务源码说明。路径存在性以运行检查时 `{projectRoot}` 下文件系统为准。

## 总体结论

<!-- 一至三句中文，含 overallHealth：blocked / degraded / healthy -->

当前总体健康度：**{{OVERALL_HEALTH}}**。

## 阻塞项

<!-- 仅列出 critical 与 Rule 定义的阻塞类 high；若无写「无」 -->

## 按严重等级分类的问题清单

| 严重等级 | 领域编号 | 说明 | 证据路径（相对项目根） |
|----------|----------|------|------------------------|
| <!-- critical / high / medium / low --> | <!-- H1–H11 --> | | |

## 可用能力判断

| 能力 | 判断 | 理由 |
|------|------|------|
| 扫盘类 Skill（`skills/scan/**`） | <!-- pass / conditional / blocked --> | |
| `validate-pattern-pack` | | |
| `ensure-agent-default-entry`（多 Agent 极薄入口） | | |
| `create-code-by-pattern` | | |
| 项目本地 Skill（`.ai/skills/project-local/**`） | | |

## 建议修复顺序

<!-- 有序列表：先解阻塞项，再处理 high、medium -->

1.

## 是否需要人工处理

**{{HUMAN_INTERVENTION_REQUIRED}}**（`true` / `false`）

<!-- 展开与 overallHealth、阻塞项一致的理由 -->

## 附录：路径存在性抽检（可选）

<!-- 列出已校验路径及结果；若无则写「无」 -->

## 附录：Pattern Pack / 项目本地 Skill 枚举摘要（可选）

<!-- Pack 目录列表、project-local Skill 列表 -->

## 证据与备注

<!-- 如 pattern-validation-status 交叉引用、路径校验截断、前置文件缺失等 -->
