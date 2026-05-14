# doc-staleness-rule

## 适用主体

编写或更新 `.ai/docs/**`、`.ai/indexes/**`、`.ai/pattern-packs/**`、`.ai/skills/project-local/**` 的 Agent（含 `update-docs-after-change`）。

## 规则陈述

1. **过时标注**：当无法确认某段描述仍与源码一致、或仅基于弱证据（如仅 git 路径无内容核对）时，须在文档中使用明确中文措辞标注，例如：「**可能过时**（待确认）」「基于 `{日期}` 前事实；代码已变更，待扫盘或人工核对」。不得将未核实内容写为确定事实。
2. **与 `uncertain` 一致**：`doc-impact-report` 中某类判断为 `uncertain` 时，若在 `update-docs-after-change` 中仍写入相关小节，该小节应带过时或待确认提示，并指向报告中的不确定项编号或章节。
3. **优先重扫而非硬编**：当变更影响面大、模块边界不清、或索引与磁盘严重不一致时，报告与状态应**建议**调用 `scan-module-by-ai` / `scan-file-by-ai` / `scan-project-by-ai` 等扫盘 Skill，而不是由 `update-docs-after-change` 杜撰架构细节。
4. **轻量补丁优先**：能用一个短段落或「变更同步记录」说清时，不整篇替换扫描产物；整篇替换须有报告清单中的明确依据（例如「该文件由误生成需回滚」类人类指令，且仍建议人工在 Git 中审阅）。

## 允许路径

- `{projectRoot}/.ai/docs/**`、`.ai/indexes/**`、`.ai/pattern-packs/**`、`.ai/skills/project-local/**` 中经 `doc-impact-report` 授权更新的文件。

## 禁止事项

- 删除原文中人类已确认的结论而无报告依据。
- 用「已同步」等措辞掩盖仍不确定的内容。

## 与其它 Rule 的关系

- 与 `human-readable-doc-rule` 的不确定性措辞一致。
- 与 `doc-update-policy-rule` 的定点更新、禁止全量重生成一致。
- 与 `source-of-truth-rule` 一致：说明性文档非源码事实源。
