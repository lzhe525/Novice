---
appliedAt: "{{APPLIED_AT_ISO}}"
previousLocalFrameworkVersion: "{{PREVIOUS_LOCAL_FRAMEWORK_VERSION}}"
newLocalFrameworkVersion: "{{NEW_LOCAL_FRAMEWORK_VERSION}}"
publicFrameworkVersion: "{{PUBLIC_FRAMEWORK_VERSION}}"
outcome: "{{OUTCOME}}"
---

# hacf-version-upgrade-result

> 由 `apply-hacf-local-upgrade` 在人工确认且执行成功后写入。路径与版本号保持原文。

## 摘要

- 执行时间（UTC）：{{APPLIED_AT_ISO}}
- 升级前本地框架版本：{{PREVIOUS_LOCAL_FRAMEWORK_VERSION}}
- 升级后本地框架版本：{{NEW_LOCAL_FRAMEWORK_VERSION}}
- 当时公共版本：{{PUBLIC_FRAMEWORK_VERSION}}
- 结果：`{{OUTCOME}}`（`success` / `partial` / `aborted`）

## 已创建或补缺的框架路径

<!-- 列表：仅列自动创建且非冲突项 -->

## 未执行项与原因

<!-- 须人工确认而未执行、或跳过 -->

## 关联报告

- 若存在冲突记录：`.ai/reports/hacf-version-upgrade-conflicts.md`
