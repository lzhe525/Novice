---
docType: doc-impact-report
generatedBy: ai
reviewedByHuman: false
impactEvaluatedAt: "{{IMPACT_EVALUATED_AT_ISO}}"
impactCheckSkill: check-doc-impact-after-change
needsUpdateProjectDocs: "{{NEEDS_UPDATE_PROJECT_DOCS}}"
needsUpdateModuleDocs: "{{NEEDS_UPDATE_MODULE_DOCS}}"
needsUpdateFileDocs: "{{NEEDS_UPDATE_FILE_DOCS}}"
needsUpdateIndexes: "{{NEEDS_UPDATE_INDEXES}}"
needsUpdatePatternPack: "{{NEEDS_UPDATE_PATTERN_PACK}}"
needsUpdateProjectLocalSkill: "{{NEEDS_UPDATE_PROJECT_LOCAL_SKILL}}"
uncertaintyCount: {{UNCERTAINTY_COUNT}}
sourceOfTruth: repository
---

# 代码变更后文档影响报告

> **事实源声明**：本报告为 `check-doc-impact-after-change` 的影响判断记录，非业务源码说明。路径与存在性以评估时 `{projectRoot}` 下只读证据为准。

## 总体结论

<!-- 一至三句中文，概括变更来源、是否建议进入 update-docs-after-change、主要风险 -->

## 输入证据

| 优先级 | 来源 | 是否采用 | 说明 |
|--------|------|----------|------|
| P1 | `.ai/reports/pattern-code-generation-result.md` 等三件套 | <!-- 是/否/部分 --> | |
| P2 | 显式提供的其它报告路径 | | |
| P3 | git 等弱证据 | | |

## 已解析的变更路径（相对项目根）

<!-- 列表或表格；若截断须说明上限 -->

## 七类判断

| 类别 | 判断 | 理由（一句） | 证据路径（相对项目根） |
|------|------|--------------|------------------------|
| 项目文档 `.ai/docs/project/**` | <!-- yes / no / uncertain --> | | |
| 模块文档 `.ai/docs/modules/**` | | | |
| 文件文档 `.ai/docs/files/**` | | | |
| 索引 `.ai/indexes/**` | | | |
| Pattern Pack `.ai/pattern-packs/**` | | | |
| 项目本地 Skill `.ai/skills/project-local/**` | | | |

## 建议更新清单（供 update-docs-after-change 解析）

<!-- 表格：目标路径（相对项目根，须在 .ai/ 下） | 变更类型（merge_frontmatter / patch_section / append_changelog / mark_stale_notice 等） | 依据（报告章节） | 与七类判断对应 -->

| 目标路径 | 变更类型 | 依据 | 备注 |
|----------|----------|------|------|

## 不确定项

<!-- 若无写「无」；若有则每条含：描述、建议下一步（人工 / scan-* / 补充报告） -->

## 附录：读取范围摘要

<!-- 列出本轮只读打开的主要 `.ai/` 与源码路径（若有），遵守 agent-context-budget -->
