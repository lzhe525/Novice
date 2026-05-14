# doc-sync-scope-rule

## 适用主体

执行 `skills/docs/check-doc-impact-after-change.md` 与 `skills/docs/update-docs-after-change.md` 的 Agent。

## 规则陈述

1. **项目内可评估 / 可同步的 `.ai/` 范围**（须在 `doc-impact-report` 与实际操作中保持一致）：
   - `.ai/docs/project/**`（项目级文档）
   - `.ai/docs/modules/**`（模块级文档）
   - `.ai/docs/files/**`（单文件级文档）
   - `.ai/indexes/**`（模块、目录、文件、代码类型、Pattern、Skill 等索引 Markdown）
   - `.ai/pattern-packs/{codeTypeId}/**`（项目侧 Pattern Pack 四文件，**非**公共库内模板）
   - `.ai/skills/project-local/**`（项目本地 Skill）
   - `.ai/state/doc-impact-status.md`（由本组 Skill 维护的执行与阻塞摘要）
2. **显式排除**（本组 Skill **不得**作为更新目标）：
   - `{skillLibraryRoot}/**`（公共 HACF Skill 库）
   - `{projectRoot}/AGENTS.md`（由 `create-or-update-agents-md` 等 Bootstrap Skill 专责）
   - `{projectRoot}/.ai/entry/**`、`.ai/config/**` 中未在 `doc-impact-report` 建议清单列出的文件（默认不碰）
   - 业务源码树（例如 `src/**`、应用项目约定的源码根）**不得**被 `update-docs-after-change` 修改
3. **检查 Skill 的写入上限**：除上述协作文档子树外，`check-doc-impact-after-change` **仅**允许新增或覆盖 `.ai/reports/doc-impact-report.md` 与 `.ai/state/doc-impact-status.md`（与 `project-local-output-rule` 一致）。

## 允许路径

- 与 `project-local-output-rule` 中为上述两 Skill 单列的白名单一致。

## 禁止事项

- 以文档同步为名修改公共库或业务源码。
- 将 `.ai/reports/` 下其它报告（如 `pattern-code-generation-plan.md`）作为 `update-docs-after-change` 的改写对象（除非未来单独 Skill 定义；当前**禁止**）。

## 与其它 Rule 的关系

- 与 `single-ai-context-directory-rule`、`project-local-output-rule` 一致：协作产物集中在 `.ai/`。
- 与 `public-skill-library-purity-rule` 一致：项目事实不进公共库。
