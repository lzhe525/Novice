# Skill：ensure-agent-default-entry

## 1. 目的

在 `{projectRoot}` 内**检查并生成或更新**各目标 Agent 的**极薄**默认入口文件，使 Codex / generic、Claude Code、Cursor 等在未显式读取 `.ai/entry/AI_ENTRY.md` 时仍能经各自工具链默认发现 HACF；同步写入 `.ai/state/agent-entry-status.md`；可选创建 `.ai/config/agent-entry-policy.md`。**不得**覆盖用户非受控内容，**不得**修改公共 HACF 库与业务源码。

## 2. 适用场景

- 已执行或即将执行任意 `{skillLibraryRoot}/skills/**` 下 HACF Skill，希望避免新对话绕过 `.ai/entry/AI_ENTRY.md`。
- `load-skill-library` 或 `guide-project-onboarding` 编排调用。
- 人类显式要求「补齐 Cursor / Claude / AGENTS 极薄入口」。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{skillLibraryRoot}` | 公共库根；本 Skill 路径为 `{skillLibraryRoot}/skills/bootstrap/ensure-agent-default-entry.md`，连续三次父目录解析得到 `{skillLibraryRoot}`。 |
| `{projectRoot}` | 业务项目根绝对路径；须已确认。 |
| `{targetAgent}` | 可选。取值：`codex`、`cursor`、`claude-code`、`generic`、`all`。**未提供**时：优先根据当前对话环境推断（例如在 Cursor 中优先包含 `cursor`）；无法可靠推断时，**须先询问用户**是否使用 `all`，得到确认后再写入。若用户确认使用 `all`，则按本节「`all` 语义」执行。 |
| 可选：用户禁止写入某路径 | 若用户禁止修改已有 `CLAUDE.md` 或 Cursor Rule，则跳过该项并在 `agent-entry-status.md` 记 `blocked`（用户拒绝）。 |

### `all` 语义

在已解析 `enabledAgents`（见执行步骤）前提下，`all` 表示对该集合内**每一种有独立落盘路径**的 Agent 执行一次适配；`codex` 与 `generic` **共享** `AGENTS.md`，只触发**一次** `create-or-update-agents-md` 编排。

## 4. 输出

| 路径 | 说明 |
|------|------|
| `{projectRoot}/AGENTS.md` | 经由编排 `create-or-update-agents-md` 创建或合并 `<!-- AI-SKILLKIT:BEGIN/END -->` 区块。 |
| `{projectRoot}/CLAUDE.md` | 新建全文或仅合并 `<!-- HACF_AGENT_DEFAULT_ENTRY:BEGIN/END -->` 区块。 |
| `{projectRoot}/.cursor/rules/hacf.mdc`（或 policy 中 `cursorRulePath`） | 新建或仅合并 HACF 受控区块；可创建 `.cursor/rules/` 目录。 |
| `{projectRoot}/.ai/state/agent-entry-status.md` | 每次执行后覆盖写入本轮摘要与各路径结果。 |
| `{projectRoot}/.ai/config/agent-entry-policy.md` | 若尚不存在且本轮需要解析 `enabledAgents` / `cursorRulePath`，则从模板实例化；已存在则**不**整文件覆盖，仅在本 Skill 需读取字段时只读解析。 |
| Agent 回复 | 中文摘要：解析后的 `targetAgent` 集合、各文件 `created`/`updated`/`skipped`/`blocked` 与待人工项。 |

## 5. 前置条件

- `{projectRoot}/.ai/entry/AI_ENTRY.md` 与 `{projectRoot}/.ai/entry/SKILLKIT_LINK.md` 已存在且可读。
- 下列公共库模板存在：
  - `{skillLibraryRoot}/templates/agent-entry/CLAUDE.hacf.template.md`
  - `{skillLibraryRoot}/templates/agent-entry/cursor-hacf-rule.template.mdc`
  - `{skillLibraryRoot}/templates/state/agent-entry-status.template.md`
  - `{skillLibraryRoot}/templates/config/agent-entry-policy.template.md`
- `{skillLibraryRoot}/skills/bootstrap/create-or-update-agents-md.md` 存在（用于编排 `AGENTS.md`）。

## 6. 执行步骤

1. **解析 `{skillLibraryRoot}`**（同其它 Bootstrap Skill）；校验 `ENTRY.md`、`VERSION.md` 可读。
2. **解析 `{targetAgent}`**：
   - 若调用方已显式传入合法枚举或 `all`（含由 `load-skill-library` / `guide-project-onboarding` 固定传入的 `all`），**直接采用**，不再要求环境推断或二次询问。
   - 若未传入：根据环境推断（示例：在 Cursor 对话中可默认包含 `cursor`；用户消息明确提到 Codex 可包含 `codex`）；若仍无法得到至少一个目标，**停止写入**，向用户列出选项并请其回复 `all` 或逗号分隔列表；**本轮结束**。
3. **只读**打开（不写入）：
   - `{projectRoot}/.ai/entry/AI_ENTRY.md`
   - `{projectRoot}/.ai/entry/SKILLKIT_LINK.md`
   - 若存在：`{projectRoot}/.ai/config/agent-entry-policy.md`
4. **读取 policy 与 `enabledAgents`**：
   - 若 `agent-entry-policy.md` 不存在：将 `{skillLibraryRoot}/templates/config/agent-entry-policy.template.md` **原样**写入 `{projectRoot}/.ai/config/agent-entry-policy.md`（占位符若无则不改写），然后重新只读解析；若创建失败则在回复中说明并继续（此时 `enabledAgents` 视为模板默认值）。
   - 若 YAML 中 `enabledAgents` 缺失或为空：视为 `codex`、`cursor`、`claude-code`、`generic` 全开。
5. **将 `{targetAgent}` 解析为集合 `S`**：
   - 若为 `all`：`S` = `enabledAgents` 集合（去重）。
   - 若为单一枚举：`S` = `{该枚举}`。
   - 将 `S` 与 `enabledAgents` **求交**（用户显式要求某 Agent 但若 policy 未启用该 Agent，则跳过该项并在状态文件记 `skipped`（policy 未启用））。
6. **`AGENTS.md`（`codex` 或 `generic` 在 `S` 中）**：
   - 打开 `{skillLibraryRoot}/skills/bootstrap/create-or-update-agents-md.md`，严格按其**第 6 节「执行步骤」**对 `{projectRoot}/AGENTS.md` 操作；**仅一步编排**，不重写其十节正文到本 Skill。
   - 若 `codex` 与 `generic` 同时在 `S` 中，仍只执行上述编排一次。
7. **`CLAUDE.md`（`claude-code` ∈ `S`）**：
   - 受控标记：`<!-- HACF_AGENT_DEFAULT_ENTRY:BEGIN -->` 与 `<!-- HACF_AGENT_DEFAULT_ENTRY:END -->`。
   - 文件不存在：读取 `{skillLibraryRoot}/templates/agent-entry/CLAUDE.hacf.template.md` 全文原样写入 `{projectRoot}/CLAUDE.md`。
   - 文件已存在：读全文；若 BEGIN/END **均不存在**：在文件末尾换行后追加与模板中同结构的受控区块（含 BEGIN/END 及其中文正文，须含 `.ai/entry/AI_ENTRY.md`）；若**仅存在一侧**标记：**不**写入该文件，记 `blocked`（标记不成对）；若两侧均存在：检查其间是否含 `AI_ENTRY.md` 或 `.ai/entry/AI_ENTRY.md`，不含则仅替换 BEGIN–END **之间**正文为模板中段落，保留两行边界标记。
8. **Cursor Rule（`cursor` ∈ `S`）**：
   - 解析 `cursorRulePath`：优先自 `agent-entry-policy.md` front matter 的 `cursorRulePath`，缺省为 `.cursor/rules/hacf.mdc`（相对 `{projectRoot}`，正斜杠）。
   - 确保父目录存在（如 `.cursor/rules/`）。
   - 目标文件逻辑同步骤 7：不存在则自 `{skillLibraryRoot}/templates/agent-entry/cursor-hacf-rule.template.mdc` 原样写入；已存在则按 **同一对** `HACF_AGENT_DEFAULT_ENTRY` 标记做追加或替换其中正文；标记不成对则 `blocked`。
9. **写入 `agent-entry-status.md`**：
   - 使用 `{skillLibraryRoot}/templates/state/agent-entry-status.template.md` 为骨架；将 `{{LAST_RUN_AT_ISO}}` 换为当前 UTC ISO8601；`{{TARGET_AGENTS_RESOLVED}}` 换为步骤 5 实际执行的 Agent 标识列表（逗号分隔）；填充「各入口结果」表单元格为 `created` / `updated` / `skipped` / `blocked` 之一及必要简注。
   - **覆盖写入** `{projectRoot}/.ai/state/agent-entry-status.md`。
10. **人类可读回复**：列出本轮 `S`、各路径结果与任何 `blocked` 原因。

### 受控区块内推荐正文（CLAUDE / Cursor）

须含项目相对路径 `.ai/entry/AI_ENTRY.md`；可含一两句「产物仅写入 `.ai/`」「公共库只读」；**不得**超出 `agent-entry-thin-adapter-rule` 允许的篇幅与内容。

## 7. 写入位置

| 允许 |
|------|
| `{projectRoot}/AGENTS.md`（仅经由编排 `create-or-update-agents-md`） |
| `{projectRoot}/CLAUDE.md` |
| `{projectRoot}/.cursor/rules/**` 下由 policy 指定**单文件**（默认 `hacf.mdc`） |
| `{projectRoot}/.ai/state/agent-entry-status.md` |
| `{projectRoot}/.ai/config/agent-entry-policy.md`（**仅当**文件尚不存在时自模板新建） |

| 禁止 |
|------|
| `{skillLibraryRoot}/**` 任意写入 |
| 业务源码路径（除上述允许项外） |
| 整文件覆盖已有 `agent-entry-policy.md` |
| 在 Agent 专属入口中写入扫描结果、Pattern、模块文档或完整 HACF Skill/Rule 正文 |

## 8. 禁止事项

- 覆盖 `AGENTS.md` / `CLAUDE.md` / Cursor Rule 中 BEGIN/END **之外**的用户正文。
- 修改 `<!-- AI-SKILLKIT:*-->` 区块的合并算法（应完全委托 `create-or-update-agents-md`）。
- 将 `generic` 与 `codex` 各执行一次 `create-or-update-agents-md` 导致重复劳动（合并为单次）。

## 9. 完成标准

- [ ] `agent-entry-status.md` 已写入且含本轮时间与结果表。
- [ ] 对本轮 `S` 中每一启用项，结果已记录为 `created`/`updated`/`skipped`/`blocked` 之一。
- [ ] 所有成功落盘的极薄入口均可搜索到 `.ai/entry/AI_ENTRY.md` 或 `AI_ENTRY.md` 子串（`AGENTS.md` 仍遵循 `create-or-update-agents-md` 完成标准）。

## 10. 失败处理

| 情况 | 处理 |
|------|------|
| `.ai/entry` 缺失 | 中止；提示先执行 `initialize-project-ai-context` 或 `load-skill-library`。 |
| 模板缺失 | 中止；列出缺失路径。 |
| `HACF_AGENT_DEFAULT_ENTRY` 标记不成对 | 不修改该文件；`blocked` 并提示人类整理。 |
| 无写权限 | 中止；标注路径。 |
| 用户拒绝修改某文件 | 该项 `blocked`；其余项可继续（若适用）。 |
