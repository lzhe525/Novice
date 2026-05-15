# Skill：recommend-next-onboarding-step

## 1. 目的

在**不写入项目文件**（本 Skill 不产生 `.ai/` 新内容）的前提下，读取项目本地状态 Markdown，判断当前最应执行的下一步 Skill 或人类动作，并给出可读理由与建议回复格式。

## 2. 适用场景

- `guide-project-onboarding` 的 `phase_handoff_next` 之后，人类希望快速获知「下一步点哪个 Skill」。
- 接入中途恢复工作，需根据现有文件推断阻塞点。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根绝对路径；须已确认或可解析。 |
| `{skillLibraryRoot}` | 公共库根；可由本 Skill 磁盘路径反推（同 `guide-project-onboarding`）。 |

## 4. 输出

Agent 面向用户的结构化回复（中文），**必须**包含以下小节（顺序固定）：

1. **当前阶段**（优先引用 `onboarding-status.md` 的 `currentPhase` 与中文解释；若文件缺失则说明「未开始引导」）。
2. **阻塞项**（若无则写「无」；列出 R1–R6 fail、`projectState` 异常、缺失文件等）。
3. **推荐下一步 Skill**（库内相对路径，例如 `skills/bootstrap/guide-project-onboarding.md`；若为人类动作则写「人类动作」并说明）。
4. **为什么推荐**（一至三句，引用读到的事实）。
5. **用户应该如何回复**（可复制模板）。

本 Skill **不得**修改 `{projectRoot}` 或 `{skillLibraryRoot}` 下任何文件。

## 5. 前置条件

- Agent 能读取 `{projectRoot}/.ai/state/` 下（若存在）各状态文件。

## 6. 执行步骤

1. **解析 `{skillLibraryRoot}`**（同其它 Bootstrap Skill）。
2. **依次尝试读取**（路径均相对 `{projectRoot}`；**不存在**则在本轮输出「未提供 / 文件不存在」，不得推断其内容）：
   - `.ai/state/onboarding-status.md`
   - `.ai/state/onboarding-checklist.md`
   - `.ai/state/readiness.md`
   - `.ai/state/skillkit-status.md`（用于 `projectState`、`libraryVersion` 等）
   - `.ai/state/agent-entry-status.md`
   - `.ai/state/scan-status.md`
   - `.ai/state/pattern-status.md`（若仓库未采用此文件名，记为未提供）
   - `.ai/state/localization-level.md`
3. **解析要点**：
   - `readiness.md`：表格或正文中 R1–R6 的 `pass` / `fail`。
   - `skillkit-status.md`：YAML front matter 中 `projectState`（如 `blocked`、`configuring`、`constraints_ready`）。
   - `onboarding-status.md`：`currentPhase` 与各布尔完成标志；若缺省 `agentDefaultEntryDone` 则视为 `false`。
   - `onboarding-checklist.md`：首个未勾选且与 Bootstrap 相关的项。
   - `agent-entry-status.md`：最近一次 Agent 入口适配结果（若存在）。
   - `scan-status.md`、`pattern-status.md`、`localization-level.md`：仅作**上下文**；本 Skill **不**强制跳转至 Scan / Pattern / Develop（与 `guide-project-onboarding` 边界一致）。
4. **按下列优先级**决定「推荐下一步 Skill」（自 **P0** 起命中即选，可辅以人类动作）：

| 优先级 | 条件（摘要） | 推荐 |
|--------|----------------|------|
| P0 | `onboarding-status.md` 存在且 `currentPhase` 为 `phase_ensure_agent_default_entry`，且 `agentDefaultEntryDone` 不为 `true` | `skills/bootstrap/ensure-agent-default-entry.md`（须显式传入 `targetAgent: all`，或由人类在会话中给出其它合法枚举）；亦可选择继续由 `guide-project-onboarding.md` 编排 |
| P1 | `onboarding-status.md` 不存在，或 `currentPhase` 未到 `phase_handoff_next` 且存在未完成的 Bootstrap 事实（**且**未命中 P0） | `skills/bootstrap/guide-project-onboarding.md` |
| P2 | `readiness.md` 中任一 R1–R6 为 `fail`，或 `skillkit-status.md` 中 `projectState` 为 `blocked` / `configuring` | 优先人类按 `readiness.md`「人类待办」处理；若须重跑自动化检查则 `skills/bootstrap/check-project-readiness.md`；路径或链接错误时 `skills/bootstrap/load-skill-library.md` 或 `skills/bootstrap/initialize-project-ai-context.md`（结合失败项说明） |
| P3 | R1–R6 均 `pass` 且 `projectState` 为 `constraints_ready`，且 `handoffDone` 不为 `true` 或清单仍显示引导未完成 | `skills/bootstrap/guide-project-onboarding.md`（从 `onboarding-status.md` 所示 `currentPhase` 继续） |
| P4 | 引导清单与 `phase_handoff_next` 均显示接入引导已完成，且 Bootstrap 就绪 | **无强制 Skill**；客观说明可进入**第二阶段**扫盘（例如 `skills/scan/scan-project-by-ai.md`）属可选后续，**非本 Skill 的必须命令** |

5. **组装五小节输出**（见第 4 节）。若 P4 且人类仅关心 Bootstrap：明确写「Bootstrap 与接入引导已完成」。

## 7. 写入位置

| 允许 |
|------|
| 无（本 Skill 为只读） |

| 禁止 |
|------|
| 修改 `{projectRoot}/**` 或 `{skillLibraryRoot}/**` |
| 建议 Agent 伪造 `language-policy.md` 的 `reviewedByHuman: true` |

## 8. 禁止事项

- **禁止**因 `scan-status.md` 或 `pattern-status.md` 存在就推荐 Pattern / Develop 流程；仅可在 P4 后作为**可选**后续列出扫盘类 Skill。
- **禁止**在缺少 `readiness.md` 时声称 Bootstrap 已通过。

## 9. 完成标准

- [ ] 已尝试读取第 6 节所列路径并如实声明缺失。
- [ ] 输出含全部五个小节。
- [ ] 推荐路径使用 `{skillLibraryRoot}` 下相对路径字符串（正斜杠）。

## 10. 失败处理

| 情况 | 处理 |
|------|------|
| `{projectRoot}` 无法读取 | 中止；输出五个小节并说明需要有效项目根。 |
| 全部状态文件缺失 | 推荐 `skills/bootstrap/guide-project-onboarding.md`；阻塞项写「尚未初始化 `.ai/state`」 |
