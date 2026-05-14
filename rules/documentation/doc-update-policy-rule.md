# doc-update-policy-rule

## 适用主体

执行 `skills/docs/update-docs-after-change.md` 的 Agent。

## 规则陈述

1. **唯一依据**：对 `.ai/` 协作文档的写入须严格以最新 `.ai/reports/doc-impact-report.md` 中「建议更新清单」为准；**不得**超出该清单所列目标路径与操作类型自行扩大范围。
2. **定点更新**：优先采用合并 front matter、增补段落、在约定小节追加「变更同步记录」、按 `doc-staleness-rule` 添加过时提示等**轻量**操作；**禁止**以「同步」为由对 `.ai/docs/**` 或 `.ai/indexes/**` 做全目录批量重写或等同于重新执行 `scan-project-by-ai` / `scan-module-by-ai` 的全量重生成。
3. **`uncertain` 默认不执行**：报告中标记为 `uncertain` 或目标路径不明确的项，**默认跳过**；须在 `doc-impact-status.md`（或报告附录）记录跳过原因。
4. **可追溯**：每一处实际修改须在状态或报告附录中写明对应清单条目（例如表格行号或章节标题），便于人类审阅。
5. **Pattern Pack**：仅允许更新 `{projectRoot}/.ai/pattern-packs/{codeTypeId}/` 下说明类正文或 front matter 中与本次变更直接相关的片段；**不得**将业务项目内容写回 `{skillLibraryRoot}`（公共 HACF Skill 库）。

## 允许路径

- 与 `update-docs-after-change` Skill「写入位置」节一致：清单中显式列出的 `{projectRoot}/.ai/docs/**`、`.ai/indexes/**`、`.ai/pattern-packs/**`、`.ai/skills/project-local/**` 及 `.ai/state/doc-impact-status.md`。

## 禁止事项

- 修改 `{projectRoot}` 下任意业务源码。
- 修改或写入 `{skillLibraryRoot}/**`。
- 在无有效 `doc-impact-report.md` 或无法解析结构化清单时强行批量更新文档。

## 与其它 Rule 的关系

- 依赖 `doc-impact-after-code-change-rule` 的输出契约。
- 受 `doc-sync-scope-rule` 约束可写子树。
- 与 `read-source-before-edit-rule` 区分：本 Skill **不**以改写源码为手段；若需核对事实，仅只读打开源码或已有 `.ai/docs`。
