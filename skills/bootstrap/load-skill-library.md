# Skill：load-skill-library

## 1. 目的

将 HACF 公共 Skill 库以**只读引用**方式接入 `{projectRoot}`：依次执行 `create-or-update-agents-md`、`initialize-project-ai-context`、`check-project-readiness`，完成极薄 `AGENTS.md`、`.ai/entry|state|config` 基础结构与就绪检查，**不**复制整个公共库到项目内。

## 2. 适用场景

- 开发者已将公共库与业务项目**平行克隆**，希望 Agent 一次性完成 Bootstrap MVP。
- 升级机器或重克隆后重新链接 `SKILLKIT_LINK.md` 与状态文件（在提供正确相对路径前提下）。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{skillLibraryRoot}` | 含 `ENTRY.md` 的公共库根目录绝对路径；若 Agent 从本 Skill 文件路径反推，则本 Skill 磁盘路径必为 `{skillLibraryRoot}/skills/bootstrap/load-skill-library.md`，对文件路径连续执行三次父目录解析（去掉文件名、`bootstrap`、`skills`）即得到 `{skillLibraryRoot}`。 |
| `{projectRoot}` | 业务项目根目录绝对路径；须由用户显式给出或从工作区根推断并经用户确认。 |
| `{skillLibraryRelativePath}` | **必填**。从 `{projectRoot}` 指向 `{skillLibraryRoot}` 的相对路径（正斜杠），例如 `../HACF`。用于 `initialize-project-ai-context`。 |

可选：

| 变量 | 说明 |
|------|------|
| `{projectNameOrSlug}` | 传入 `initialize-project-ai-context`；缺省则使用该 Skill 文档定义的默认（项目文件夹名）。 |

## 4. 输出

| 路径 | 说明 |
|------|------|
| `{projectRoot}/AGENTS.md` | 由子 Skill 创建或更新。 |
| `{projectRoot}/.ai/entry/AI_ENTRY.md` | 由子 Skill 创建或更新（在允许覆盖策略下）。 |
| `{projectRoot}/.ai/entry/SKILLKIT_LINK.md` | 由子 Skill 创建或更新。 |
| `{projectRoot}/.ai/state/skillkit-status.md` | 由子 Skill 与就绪检查更新。 |
| `{projectRoot}/.ai/config/language-policy.md` | 由子 Skill 创建（若不存在）。 |
| `{projectRoot}/.ai/state/readiness.md` | 由 `check-project-readiness` 写入。 |
| Agent 回复 | 人类可读摘要：公共库版本、`projectState`、待办链接到 `readiness.md`。 |

## 5. 前置条件

- `{skillLibraryRoot}/ENTRY.md` 与 `{skillLibraryRoot}/VERSION.md` 存在。
- 下列子 Skill 文件存在：
  - `{skillLibraryRoot}/skills/bootstrap/create-or-update-agents-md.md`
  - `{skillLibraryRoot}/skills/bootstrap/initialize-project-ai-context.md`
  - `{skillLibraryRoot}/skills/bootstrap/check-project-readiness.md`
- 已提供 `{skillLibraryRelativePath}`，且拼接后 `{skillLibraryRoot}` 校验通过（见步骤 1）。

## 6. 执行步骤

1. **解析并校验 `{skillLibraryRoot}`**：
   - 若本 Skill 从磁盘打开，路径为 `{skillLibraryRoot}/skills/bootstrap/load-skill-library.md`。
   - 验证 `{skillLibraryRoot}/ENTRY.md`、`{skillLibraryRoot}/VERSION.md` 可读；失败则**停止整个流程**，不修改 `{projectRoot}`。
2. **解析 `{projectRoot}`**：
   - 与用户确认其为目标业务仓库根（含 `.git` 或构建清单之一为佳）。
3. **（建议）阅读约束 Rule**（只读，不写入）：
   - `{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`
   - `{skillLibraryRoot}/rules/base/single-ai-context-directory-rule.md`
   - `{skillLibraryRoot}/rules/base/project-local-output-rule.md`
   - `{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`
4. **执行子 Skill `create-or-update-agents-md`**：
   - 打开 `{skillLibraryRoot}/skills/bootstrap/create-or-update-agents-md.md`，严格按其中 **第 6 节「执行步骤」** 对 `{projectRoot}/AGENTS.md` 操作。
5. **执行子 Skill `initialize-project-ai-context`**：
   - 打开 `{skillLibraryRoot}/skills/bootstrap/initialize-project-ai-context.md`，严格按其中 **第 6 节「执行步骤」** 操作；传入 `{skillLibraryRelativePath}` 与可选 `{projectNameOrSlug}`。
6. **执行子 Skill `check-project-readiness`**：
   - 打开 `{skillLibraryRoot}/skills/bootstrap/check-project-readiness.md`，严格按其中 **第 6 节「执行步骤」** 操作。
7. **汇总输出给人类**：
   - 读取 `{projectRoot}/.ai/state/skillkit-status.md` front matter 的 `libraryVersion`、`projectState`。
   - 读取 `{projectRoot}/.ai/state/readiness.md` 中是否存在 `fail` 行。
   - 在回复中列出下一步：若 `projectState` 为 `blocked` 或 `configuring`，指导人类打开 `readiness.md`「人类待办」与 `.ai/config/language-policy.md` 完成确认。

子 Skill 的**十节全文**不在本文件中重复；任何与本文冲突之处，以子 Skill 文件为准。

## 7. 写入位置

| 允许（汇总） |
|--------------|
| `{projectRoot}/AGENTS.md` |
| `{projectRoot}/.ai/entry/AI_ENTRY.md` |
| `{projectRoot}/.ai/entry/SKILLKIT_LINK.md` |
| `{projectRoot}/.ai/state/skillkit-status.md` |
| `{projectRoot}/.ai/state/readiness.md` |
| `{projectRoot}/.ai/config/language-policy.md` |

| 禁止 |
|------|
| `{skillLibraryRoot}/**` 任意写入或删除 |
| 向 `{projectRoot}` 除 `AGENTS.md` 与 `.ai/**` 外写入 AI 协作文档 |
| 复制整个 `{skillLibraryRoot}` 目录树到 `{projectRoot}` |

## 8. 禁止事项

- **禁止**修改公共库内任何文件以「记录项目状态」。
- **禁止**跳过 `check-project-readiness` 仍声称接入已完成。
- **禁止**在未成功完成步骤 1 校验的情况下继续步骤 4–6。

## 9. 完成标准

- [ ] 步骤 1 校验通过；步骤 4–6 已按子 Skill 执行完毕（或因子 Skill 中止条件而整流程中止）。
- [ ] `{projectRoot}/.ai/state/readiness.md` 存在且 R1–R6 已填充。
- [ ] Agent 回复包含 `libraryVersion` 与 `projectState`。
- [ ] 公共库目录仍为只读意图（无对 `{skillLibraryRoot}` 的写入）。

## 10. 失败处理

| 情况 | 处理 |
|------|------|
| `{skillLibraryRoot}` 校验失败 | 不触碰 `{projectRoot}`；提示检查克隆路径与本 Skill 是否位于公共库内。 |
| `{skillLibraryRelativePath}` 错误导致 ENTRY 不存在 | 由 `initialize` 或 `check` 子 Skill 中止；人类修正相对路径后从步骤 4 或 5 重试。 |
| 子 Skill 中途失败 | 在回复中标注失败子 Skill 文件名与原因；列出已写入路径；**不**伪造就绪为通过。 |
| 用户撤销写权限 | 立即中止；说明哪个路径无法写入。 |
