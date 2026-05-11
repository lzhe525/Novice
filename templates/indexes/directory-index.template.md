---
docType: directory-index
status: draft
generatedBy: ai
reviewedByHuman: false
lastReviewedAt: ""
sourceOfTruth: repository
scanSkill: scan-project-by-ai
generatedAt: "{{SCANNED_AT_ISO}}"
---

# 目录扫描索引（Directory Index）

> **事实源声明**：本文件记录**项目级受控扫盘**过程中识别的重要目录与扫描元数据，**非**完整仓库目录树。目录可与多个逻辑模块（`moduleId`）关联；实现以仓库文件系统为准。

## 使用方式（Agent）

1. 优先结合 `.ai/state/scan-status.md` 判断本轮是否仍有 `truncated` 或 `partial` 的重要目录。
2. 对未扫完的目录，建议后续执行 **`scan-module-by-ai`**，并传入 **`moduleId`**（必填）；可附带 `directoryPathRelative` 供人类对照。
3. 结合 `.ai/config/module-map.md` 核对 `relatedModuleIds` 是否与声明一致。

## 重要目录（本轮）

Agent 在 `scan-project-by-ai` 执行后填写；无重要目录则保留表头并写一行「无」。

| directoryPathRelative（相对 `{projectRoot}`） | relatedModuleIds | 触发原因（标签或简述） | 扫描状态 | truncated（是/否） | 备注 |
|-----------------------------------------------|------------------|------------------------|----------|---------------------|------|
| <!-- 示例：src/core --> | <!-- 示例：billing, auth 或 — --> | <!-- 如：module-map、关键文件、用户任务提及 --> | <!-- completed / partial / skipped --> | <!-- 是/否 --> | <!-- 可选：局部预算耗尽、已达重置次数上限等 --> |

## 其他说明

<!-- 人类或 Agent 追加：全局重置次数已用尽、忽略规则冲突等 -->
