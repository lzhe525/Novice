---
checkedAt: "{{CHECKED_AT_ISO}}"
publicFrameworkVersion: "{{PUBLIC_FRAMEWORK_VERSION}}"
localFrameworkVersion: "{{LOCAL_FRAMEWORK_VERSION}}"
compatibilityStatus: "{{COMPATIBILITY_STATUS}}"
publicCapabilitySchemaVersion: "{{PUBLIC_CAPABILITY_SCHEMA_VERSION}}"
notes: ""
---

# hacf-version-status

> **事实源声明**：本文件记录 HACF **公共库版本**与项目侧 **`localFrameworkVersion`** 的比对结论；不替代 `skillkit-status.md` 全文。路径与版本号保持原文。

## 本轮摘要

- 检查时间（UTC）：{{CHECKED_AT_ISO}}
- 公共框架版本（`VERSION.md` 首行）：{{PUBLIC_FRAMEWORK_VERSION}}
- 本地记录版本（`skillkit-status.md` 的 `localFrameworkVersion`，缺失时回退逻辑见 `check-hacf-version-compatibility`）：{{LOCAL_FRAMEWORK_VERSION}}
- 兼容状态：`{{COMPATIBILITY_STATUS}}`（`up_to_date` / `outdated` / `local_newer_than_public` / `unknown` / `incompatible`）
- 能力声明 schema 版本（可选）：{{PUBLIC_CAPABILITY_SCHEMA_VERSION}}

## 详情

<!-- Agent 填写：缺失 `requiredFiles`、版本在 `hacf-capabilities.yml` 的 `versionOrder` 中位置、与 `libraryVersion` 是否冲突等 -->

## 建议下一步

<!-- 例如：状态为 outdated 时执行 `plan-hacf-local-upgrade.md`；为 incompatible 时先人工修复 SKILLKIT_LINK 或 skillkit-status -->
