# agent-entry-thin-adapter-rule

## 适用主体

编写或更新 `AGENTS.md`（HACF 区块）、`CLAUDE.md`、`.cursor/rules/hacf.mdc`（或 `agent-entry-policy.md` 声明的同等极薄 Cursor Rule）的 Agent。

## 极薄定义

1. **指针优先**：正文核心信息仅限引导阅读 `.ai/entry/AI_ENTRY.md`；可含少量编号约束（如「产物仅写入 `.ai/`」「公共库只读」），**不得**展开 Skill 目录、Pattern 列表或模块文档。
2. **受控区块**：HACF 可自动合并的段落必须位于成对的 `<!-- HACF_AGENT_DEFAULT_ENTRY:BEGIN -->` 与 `<!-- HACF_AGENT_DEFAULT_ENTRY:END -->` 之间（`AGENTS.md` 仍沿用既有 `<!-- AI-SKILLKIT:BEGIN/END -->` 时，以 `create-or-update-agents-md` 为准，不强制改为本标记）。
3. **篇幅启发式**（与 `agent-context-budget-rule` 精神一致）：单文件总行数建议不超过约 **80 行**；明显超过且非受控区含大量二级标题 / 大段 fenced 代码疑似整篇 Skill 时，视为**非极薄**，须人工收敛。

## 禁止写入 Agent 专属入口的内容

- 完整 HACF 公共库 Rule、Skill 正文拷贝。
- 项目扫描结果、Pattern Pack、模块深读文档、索引表。
- 项目私有密钥、内网地址、未脱敏日志。

## 允许的唯一「间接」引用

- 允许一句内指向根目录 `AGENTS.md`，**仅当**该文件已存在且可被验证含有 `.ai/entry/AI_ENTRY.md` 或 `AI_ENTRY.md` 子串；不得用 `AGENTS.md` 代替阅读 `AI_ENTRY.md` 作为长期唯一入口而不维护 `AI_ENTRY` 链接语义（推荐仍直接列出 `AI_ENTRY.md`）。

## 失败与人工

- 若 `BEGIN`/`END` 标记不成对或仅存在一侧，**不得**自动修复整文件；须请人类整理后重跑 `ensure-agent-default-entry`。

## 与其它 Rule 的关系

- 与 `single-ai-context-directory-rule`：CLAUDE / Cursor Rule 与本规则一致的极薄例外。
- 与 `agent-default-entry-rule`：流程与边界；本规则约束**写什么**。
