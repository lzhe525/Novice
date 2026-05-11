---
projectSlug: {{PROJECT_NAME_OR_SLUG}}
generatedAt: {{GENERATED_AT_ISO}}
---

# AI_ENTRY

本文件是项目内 Agent 协作的**入口索引**，路径：`{projectRoot}/.ai/entry/AI_ENTRY.md`。

## 推荐阅读顺序

1. 本文件（当前页）
2. 同目录 `SKILLKIT_LINK.md`（解析 `{skillLibraryRoot}`）
3. `../config/language-policy.md`（语言策略；须人工确认后 `reviewedByHuman: true`）
4. `../state/readiness.md`（就绪检查结果）
5. `../state/skillkit-status.md`（库版本与状态摘要）

## 公共库内关键路径（相对 `{skillLibraryRoot}`）

以下路径需先根据 `SKILLKIT_LINK.md` 解析出 `{skillLibraryRoot}` 后再拼接读取：

| 用途 | 相对路径 |
|------|------------|
| Bootstrap 接入 | `skills/bootstrap/load-skill-library.md` |
| 更新根入口 | `skills/bootstrap/create-or-update-agents-md.md` |
| 初始化 `.ai` | `skills/bootstrap/initialize-project-ai-context.md` |
| 就绪检查 | `skills/bootstrap/check-project-readiness.md` |
| Skill 路由表 | `routers/SKILL_ROUTER.md` |
| 基础 Rule | `rules/base/public-skill-library-purity-rule.md` 等 |
| Markdown front matter 格式 | `rules/base/frontmatter-format-rule.md` |
| 项目级扫盘 | `skills/scan/scan-project-by-ai.md` |
| 项目 Adapter | `skills/scan/generate-project-adapter-by-ai.md` |
| 模块扫盘 | `skills/scan/scan-module-by-ai.md` |
| 单文件扫盘 | `skills/scan/scan-file-by-ai.md` |
| 文档结构化 | `rules/documentation/structured-doc-writing-rule.md` |

## 变量约定

- `{projectRoot}`：含本文件的项目根目录（`AI_ENTRY.md` 位于 `{projectRoot}/.ai/entry/`）。
- `{skillLibraryRoot}`：由 `SKILLKIT_LINK.md` 中 `skillLibraryRootRelative` 解析得到的公共库根目录。

## 能力范围提示

- **Bootstrap**：`.ai/entry`、`state`、`config` 与根目录 `AGENTS.md` 由 Bootstrap 系列 Skill 维护；就绪检查仍以 `check-project-readiness` 为准。
- **扫盘（第二阶段）**：公共库提供 `skills/scan/` 下 Skill；**所有**生成文档仅可写入 `{projectRoot}/.ai/docs/**`。**源码为事实源**，文档为辅助上下文。
- **未提供**：Pattern、Placement、`develop`、`review`、框架升级等 Skill；若任务需要，请使用项目本地 Skill 或等待后续公共库版本。
