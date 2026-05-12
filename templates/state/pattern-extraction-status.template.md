---

lastPatternExtractionAt: "{{EXTRACTED_AT_ISO}}"

lastPatternExtractionModuleId: "{{MODULE_ID}}"

lastPatternExtractionCodeTypeId: "{{CODE_TYPE_ID}}"

extractionSkill: extract-code-pattern

patternPackDirRelative: ".ai/pattern-packs/{{CODE_TYPE_ID}}"

evidenceStrength: "{{EVIDENCE_STRENGTH}}"

confidence: medium

needsHumanConfirmation: false

---



# pattern-extraction-status



> **事实源声明**：本文件记录最近一次 **Pattern 提取** 元数据，不记录完整源码。事实以仓库文件为准。



## 最近一次提取



- 时间（UTC）：{{EXTRACTED_AT_ISO}}

- 执行 Skill：`extract-code-pattern`

- moduleId：`{{MODULE_ID}}`

- codeTypeId：`{{CODE_TYPE_ID}}`

- Pattern Pack 目录（相对 `.ai/`）：`pattern-packs/{{CODE_TYPE_ID}}/`

- evidenceStrength：{{EVIDENCE_STRENGTH}}

- confidence：medium（若样例不足 2 个，Agent 须改为 low 并设 needsHumanConfirmation: true）



## 深读文件列表



<!-- 相对 {projectRoot} 的路径，保持原文；每行一个 -->


- 



## 预算与截断



<!-- 是否达到 8 文件上限、哪些文件截断头尾；若无则写「无」 -->


- 无



## 冲突与合并说明



<!-- 若本次为合并运行：列出与旧 Pack 的冲突或未决项；若无则写「无」 -->


- 无



## 推荐下一步



<!-- 例如：人类审阅 pattern.md「待人工确认项」；或先补充 module-map 样例后再跑 -->


- 



## 是否需要人工确认



<!-- 是/否及简述原因 -->


- 否



## 备注



<!-- Agent 或人类追加 -->


