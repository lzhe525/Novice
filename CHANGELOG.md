# Changelog

## [0.2.5]

- 新增 **框架版本兼容与项目本地升级**：`capabilities/hacf-capabilities.yml`；Bootstrap `check-hacf-version-compatibility`、`plan-hacf-local-upgrade`、`apply-hacf-local-upgrade`；状态/报告模板与 `rules/bootstrap/hacf-*-rule.md` 三条。
- 项目侧 `skillkit-status.md` 增加 `localFrameworkVersion`；`load-skill-library` 编排版本检查；`guide-project-onboarding` 增加 `phase_check_hacf_version`；`check-project-ai-health` 增加 H4.1。
- 更新 `routers/SKILL_ROUTER.md`、`initialize-project-ai-context.md`、`check-project-readiness.md`、`project-ai-health-rule.md`。

## [0.2.4]

- 新增 **Docs**：`skills/docs/grill-with-project-docs.md`（对照 `.ai` 文档拷问、对齐方案；可选写入 `.ai/docs/project/design-grill-notes.md`）。
- 新增 **Productivity**：`skills/productivity/create-agent-handoff.md`（固定覆盖 `.ai/state/agent-handoff.md` 的会话交接）。
- 新增 **Rule**：`rules/skill/skill-description-trigger-rule.md`（公共 Skill 目的/触发描述与纯度约束）。
- 更新 `routers/SKILL_ROUTER.md`、`rules/base/project-local-output-rule.md`（登记 Router 与上述 `.ai/` 写入白名单）。

## [0.2.3]

- Pattern 第三阶段扩展：`skills/pattern/extract-code-pattern.md`（从样例抽象 Pack 四文件）；`create-code-by-pattern.md`（受控改源码）；`review-pattern-code-generation.md`；`promote-project-local-skill.md`、`activate-project-local-skill.md`（项目本地 Skill 草稿与激活）。
- 新增 **Health**：`skills/health/check-project-ai-health.md`；`rules/health/project-ai-health-rule.md`；模板 `templates/reports/project-ai-health-report.template.md`、`templates/state/project-ai-health-status.template.md`。
- 新增 **Bootstrap 引导式接入**：`skills/bootstrap/guide-project-onboarding.md`、`recommend-next-onboarding-step.md`；`rules/bootstrap/onboarding-stepwise-rule.md`、`onboarding-human-confirmation-rule.md`；模板 `templates/state/onboarding-status.template.md`、`onboarding-checklist.template.md`、`onboarding-questions.template.md`。
- 新增 **State**：`skills/state/evaluate-localization-level.md`；`templates/project-ai-context/skillkit-status.template.md` 增加 `localizationLevel` / `localizationEvaluatedAt` 字段说明。
- 更新 `routers/SKILL_ROUTER.md`、`rules/base/project-local-output-rule.md`（接入、健康检查、Pattern 生成/复盘、项目本地 Skill、本地化评估等写入路径）。

## [0.2.2]

- 新增第三阶段 Pattern：`skills/pattern/validate-pattern-pack.md`，校验 `.ai/pattern-packs/{codeTypeId}/` 四文件结构与合规性；产出 `.ai/state/pattern-validation-status.md`、`.ai/reports/pattern-validation-report.md`。
- 新增 `rules/pattern/pattern-validation-rule.md`；模板 `templates/state/pattern-validation-status.template.md`、`templates/reports/pattern-validation-report.template.md`。
- 更新 `routers/SKILL_ROUTER.md`、`rules/base/project-local-output-rule.md`（允许本 Skill 写入上述报告与状态路径）。

## [0.2.1]

- 自检补强第二阶段 Scan：`scan-project-by-ai` 现产出 `deep-dive.md`、`indexes/module-index.md`、`state/scan-status.md`；`scan-module-by-ai` 产出五件套文档；`scan-file-by-ai` 区分 `normal` / `critical`；`generate-project-adapter-by-ai` 增加 `adapter-evidence.md`。
- 新增模板：`templates/docs/project-deep-dive.template.md` 等模块子模板、`templates/indexes/module-index.template.md`、`templates/state/scan-status.template.md`、`templates/docs/project-adapter-evidence.template.md`。
- 更新 `agent-context-budget-rule`（索引按需加载）、`project-local-output-rule`、`SKILL_ROUTER.md`。

## [0.2.0]

- 新增第二阶段 **AI 扫盘与项目文档生成**：`skills/scan/scan-project-by-ai.md`、`scan-module-by-ai.md`、`scan-file-by-ai.md`、`generate-project-adapter-by-ai.md`。
- 新增文档模板 `templates/docs/`（`project-overview`、`module-overview`、`file-doc`、`project-adapter`）。
- 新增 Rule：`rules/base/frontmatter-format-rule.md`，`rules/documentation/` 下 `structured-doc-writing-rule`、`human-readable-doc-rule`、`agent-context-budget-rule`。
- 更新 `routers/SKILL_ROUTER.md` 与 `templates/project-ai-context/AI_ENTRY.template.md` 以引用扫盘能力与 front matter 规范（不改变 Bootstrap readiness 的 R1–R6 协议）。

## [0.1.0-mvp]

- 初始化公共 Skill 文档库（HACF）与 Bootstrap 最小闭环：`load-skill-library`、`create-or-update-agents-md`、`initialize-project-ai-context`、`check-project-readiness`。
- 新增基础 Rule（`rules/base/`）、Router（`routers/SKILL_ROUTER.md`）及 agents / project-ai-context / config 模板。
