---
lastProjectAiHealthCheckAt: "{{CHECKED_AT_ISO}}"
healthCheckSkill: check-project-ai-health
overallHealth: "{{OVERALL_HEALTH}}"
criticalCount: "{{CRITICAL_COUNT}}"
highCount: "{{HIGH_COUNT}}"
mediumCount: "{{MEDIUM_COUNT}}"
lowCount: "{{LOW_COUNT}}"
humanInterventionRequired: "{{HUMAN_INTERVENTION_REQUIRED}}"
blockingIssuesCount: "{{BLOCKING_ISSUES_COUNT}}"
---

# project-ai-health-status

> **事实源声明**：本文件记录最近一次 **项目 AI 协作框架健康检查** 的元数据与结论摘要，不替代各状态文件正文；事实以 `{projectRoot}` 仓库内文件为准。

## 本轮摘要

- 检查时间（UTC）：{{CHECKED_AT_ISO}}
- 执行 Skill：`check-project-ai-health`
- 总体健康度：`{{OVERALL_HEALTH}}`（`blocked` / `degraded` / `healthy`）
- 是否需要人工处理：`{{HUMAN_INTERVENTION_REQUIRED}}`（`true` / `false`）

## 计数

- `critical`：{{CRITICAL_COUNT}}
- `high`：{{HIGH_COUNT}}
- `medium`：{{MEDIUM_COUNT}}
- `low`：{{LOW_COUNT}}
- 阻塞项数：{{BLOCKING_ISSUES_COUNT}}

## 主要结论

<!-- 一至三句中文 -->

## 推荐下一步

<!-- 例如：修复某路径后重跑 `check-project-ai-health`；或「无」 -->

## 备注

<!-- 路径截断、仅部分 Pack 扫描等 -->
