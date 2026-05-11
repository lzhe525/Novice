# single-ai-context-directory-rule

## 适用主体

在 `{projectRoot}` 中执行 AI 协作任务的 Agent。

## 规则陈述

业务项目内，所有 AI 协作产物必须集中在**单一根目录**：

`{projectRoot}/.ai/`

## 根目录唯一例外

允许在 `{projectRoot}/AGENTS.md` 存在**极薄**入口内容（指针、短约束、受控区块），**不得**在 `AGENTS.md` 中存放完整 Skill 正文、扫描结果、Pattern 或大段项目文档。

## 允许路径（MVP 汇总）

- `{projectRoot}/AGENTS.md`
- `{projectRoot}/.ai/entry/*`（本阶段由 Bootstrap 模板生成的文件）
- `{projectRoot}/.ai/state/*`
- `{projectRoot}/.ai/config/*`（本阶段仅 `language-policy.md` 等 Bootstrap 产物）

## 禁止路径

- 在 `{projectRoot}` 根下创建 `AI_NOTES.md`、`CODEX.md`、`.cursor/rules` 内承载本应属于 `.ai/` 的长篇协作文档（MVP 未要求迁移既有工具配置时，**不要新建**与 `.ai/` 重复的框架文档）。
- 将扫描报告、索引写入 `src/` 或项目根（除非用户明确要求修改源码且不属于「AI 协作文档」范畴）。

## 与其它 Rule 的关系

- `project-local-output-rule` 细化「哪些类别必须进 `.ai/`」。
- `public-skill-library-purity-rule` 约束公共库侧不得被项目污染。
