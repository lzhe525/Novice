---
status: active
routeEnabled: false
description: 检查 Bootstrap 就绪情况并引导人工确认语言策略。
triggerWhen:
  - 就绪检查
  - readiness
---

# Skill：check-project-readiness

## 1. 目的

校验 Bootstrap MVP 所需路径与引用是否就绪；**强制检查** `.ai/config/language-policy.md` 是否已由人类确认；将结果写入 `{projectRoot}/.ai/state/readiness.md`，并更新 `{projectRoot}/.ai/state/skillkit-status.md` 中的 `projectState` 与 `lastCheck`。

## 2. 适用场景

- `load-skill-library` 流程末尾自动执行。
- 人类在修改 `SKILLKIT_LINK.md` 或语言策略后手动触发复查。
- CI 或本地脚本由 Agent 代为执行（仍须人类确认语言策略，Agent 不得伪造）。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根。 |
| `{skillLibraryRoot}` | 可选；若缺省则从 `{projectRoot}/.ai/entry/SKILLKIT_LINK.md` 解析（见执行步骤）。 |

## 4. 输出

| 路径 | 说明 |
|------|------|
| `{projectRoot}/.ai/state/readiness.md` | 创建或整体替换为本次检查结果（Markdown + checklist）。 |
| `{projectRoot}/.ai/state/skillkit-status.md` | 更新 front matter：`projectState`、`lastCheck`；必要时更新 `libraryVersion`。 |

## 5. 前置条件

- `{projectRoot}/.ai/entry/SKILLKIT_LINK.md` 已存在（否则本检查直接失败）。
- Agent 具备对上述输出路径的写入权限。

## 6. 执行步骤

1. **解析 `{skillLibraryRoot}`**：
   - 读取 `{projectRoot}/.ai/entry/SKILLKIT_LINK.md`。
   - 从正文表格或字段 `skillLibraryRootRelative` 取得相对路径字符串 `rel`（例如 `../HACF`）。
   - 计算绝对路径：`resolved = normalize({projectRoot} + '/' + rel)`（实现时按 OS 规范化；文档中记为 `{skillLibraryRoot}`）。
   - 验证文件存在：`{skillLibraryRoot}/ENTRY.md`、`{skillLibraryRoot}/VERSION.md`。任一不存在则记录失败项 `R1`。
2. **检查入口文件**：
   - 验证 `{projectRoot}/.ai/entry/AI_ENTRY.md` 存在；否则 `R2`。
3. **检查根 `AGENTS.md`**：
   - 验证 `{projectRoot}/AGENTS.md` 存在；否则 `R3`。
   - 读取全文，必须包含子串 `.ai/entry/AI_ENTRY.md` 或 `AI_ENTRY.md`；否则 `R4`。
4. **检查语言策略（MVP 人工门控）**：
   - 读取 `{projectRoot}/.ai/config/language-policy.md`；不存在则 `R5`。
   - 解析 YAML front matter（`---` 包裹的第一段）：
     - 若 `reviewedByHuman` 不为布尔值 `true`（含缺失、false、字符串 `"false"`），标记 `R6`：要求人类将 metadata 更新为至少包含：`status: active`、`reviewedByHuman: true`、`confidence: confirmed`、`lastReviewedAt: <YYYY-MM-DD>`（参见该文件正文「Human confirmation」节）。
5. **汇总 `projectState`**（写入 `skillkit-status.md`）：
   - 若 `R1`–`R6` 任一为真：`projectState = blocked`。
   - 若全部通过且语言策略已确认：`projectState = constraints_ready`（MVP 下表示「Bootstrap 关键人工项已完成」；**非**设计文档全量 `ready`）。
   - 若仅结构通过但语言未确认：`projectState = configuring`。
6. **写入 `readiness.md`**（路径：`{projectRoot}/.ai/state/readiness.md`）：
   - 使用以下固定结构（将方括号内替换为实际布尔或说明）：

```markdown
---
generatedAt: <ISO8601 UTC>
projectState: <configuring|constraints_ready|blocked>
---

# Readiness（Bootstrap MVP）

## 检查结果摘要

| 编号 | 检查项 | 状态 |
|------|--------|------|
| R1 | `{skillLibraryRoot}/ENTRY.md` 与 `VERSION.md` 可解析且存在 | pass / fail |
| R2 | `.ai/entry/AI_ENTRY.md` 存在 | pass / fail |
| R3 | `AGENTS.md` 存在 | pass / fail |
| R4 | `AGENTS.md` 引用 `AI_ENTRY.md` | pass / fail |
| R5 | `.ai/config/language-policy.md` 存在 | pass / fail |
| R6 | `language-policy.md` 已人工确认（`reviewedByHuman: true`） | pass / fail |

## 人类待办（仅当存在 fail）

<!-- 列出具体动作，例如打开哪个文件、改哪些字段 -->

## 后续阶段配置（勿在 MVP 自动创建）

以下文件属于框架后续能力，**本 Skill 不得**因检查缺失而自动创建：

- `.ai/config/project-profile.md`
- `.ai/config/hard-constraints.md`
- `.ai/config/danger-zones.md`
- `.ai/config/architecture-rules.md`
- `.ai/config/coding-style.md`
- `.ai/config/domain-glossary.md`
- `.ai/config/module-map.md`
- `.ai/config/code-type-registry.md`
- `.ai/config/lifecycle-map.md`
- `.ai/config/code-placement.md`

当相关 Skill 在未来版本引入后，再在 `readiness.md` 中扩展对应检查行。
```

7. **更新 `skillkit-status.md`**：
   - 读取 `{projectRoot}/.ai/state/skillkit-status.md`；若不存在则从 `{skillLibraryRoot}/templates/project-ai-context/skillkit-status.template.md` 实例化（占位符同 `initialize-project-ai-context`）。
   - 写入或合并 YAML front matter：
     - `libraryVersion`：来自 `{skillLibraryRoot}/VERSION.md` 的首行纯文本版本号（与 `initialize-project-ai-context` 中 `{{SKILLKIT_VERSION}}` 规则一致）。
     - `localFrameworkVersion`：若 front matter 中该键缺失，则补为与 `libraryVersion` 相同；若已存在，则**不得降低**（与 `initialize-project-ai-context` §6 步骤 5 对 `localFrameworkVersion` 的规则一致）。
     - `lastCheck`：当前 UTC ISO8601。
     - `projectState`：步骤 5 的值。
   - 保留正文「人工可读摘要」小节并刷新其中版本与时间行。

## 7. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/state/readiness.md` |
| `{projectRoot}/.ai/state/skillkit-status.md` |

| 禁止 |
|------|
| 修改 `{skillLibraryRoot}/**` |
| 因检查失败而删除人类已有文档 |
| 自动将 `language-policy.md` 标为已确认 |

## 8. 禁止事项

- **禁止**在未由人类修改文件的前提下，将 `language-policy.md` 内 `reviewedByHuman` 写为 `true`。
- **禁止**为「凑齐全量设计」而在 `.ai/config/` 下批量创建空壳配置。

## 9. 完成标准

- [ ] `readiness.md` 存在且含 R1–R6 表格，每项为 `pass` 或 `fail`。
- [ ] `skillkit-status.md` 中 `lastCheck` 与 `projectState` 已更新。
- [ ] 若存在 `fail`，「人类待办」节中有**可执行**步骤（文件路径 + 字段名）。

## 10. 失败处理

| 情况 | 处理 |
|------|------|
| `SKILLKIT_LINK.md` 缺失 | `projectState=blocked`；`readiness.md` 中 R1–R6 均 fail 或注明不适用；提示先执行 `initialize-project-ai-context`。 |
| 无法解析 YAML | 对 `language-policy.md` 记 `R6=fail`；待办中写明「请使用有效 YAML front matter」，并指引人类或 Agent 阅读 `{skillLibraryRoot}/rules/base/frontmatter-format-rule.md` 修正格式。 |
| 无写权限 | 中止；输出路径；不声称检查已完成。 |
