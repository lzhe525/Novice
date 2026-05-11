# Skill：initialize-project-ai-context

## 1. 目的

在 `{projectRoot}` 下创建 Bootstrap MVP 所需的 **`.ai/entry/`**、**`.ai/state/`**、**`.ai/config/`** 目录，并从 `{skillLibraryRoot}/templates/` 下模板实例化四个项目侧文件，填入占位符，使后续 Agent 可通过 `AI_ENTRY.md` 与 `SKILLKIT_LINK.md` 解析公共库路径。

## 2. 适用场景

- 首次执行 `load-skill-library` 时初始化项目 AI 上下文目录。
- `.ai/` 部分缺失时补齐（不删除人类已改动的文件内容；缺失文件则创建）。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{skillLibraryRoot}` | 公共库根；模板位于 `{skillLibraryRoot}/templates/...`。 |
| `{projectRoot}` | 业务项目根。 |
| `{skillLibraryRelativePath}` | **必填**。从 `{projectRoot}` 到 `{skillLibraryRoot}` 的相对路径，正斜杠，无盘符，例如 `../HACF`。由人类在触发 load 时提供，或由 Agent 根据两路径计算后**经用户确认**。 |

可选：

| 变量 | 说明 |
|------|------|
| `{projectNameOrSlug}` | 项目简称；缺省时用 `{projectRoot}` 目录名（最后一级）。 |

## 4. 输出

| 路径 | 说明 |
|------|------|
| `{projectRoot}/.ai/entry/` | 目录；若不存在则创建。 |
| `{projectRoot}/.ai/state/` | 目录；若不存在则创建。 |
| `{projectRoot}/.ai/config/` | 目录；若不存在则创建。 |
| `{projectRoot}/.ai/entry/AI_ENTRY.md` | 自模板实例化。 |
| `{projectRoot}/.ai/entry/SKILLKIT_LINK.md` | 自模板实例化。 |
| `{projectRoot}/.ai/state/skillkit-status.md` | 自模板实例化或更新首行 metadata。 |
| `{projectRoot}/.ai/config/language-policy.md` | 自模板实例化；若已存在且 `reviewedByHuman: true`，**跳过覆盖**（仅当文件不存在时创建）。 |

## 5. 前置条件

- `{skillLibraryRoot}/VERSION.md` 存在（用于 `{{SKILLKIT_VERSION}}`）。
- 下列模板文件存在：
  - `{skillLibraryRoot}/templates/project-ai-context/AI_ENTRY.template.md`
  - `{skillLibraryRoot}/templates/project-ai-context/SKILLKIT_LINK.template.md`
  - `{skillLibraryRoot}/templates/project-ai-context/skillkit-status.template.md`
  - `{skillLibraryRoot}/templates/config/language-policy.template.md`
- Agent 对 `{projectRoot}/.ai/**` 有创建与写入权限。

## 6. 执行步骤

1. **计算占位符**（在内存中完成，不单独写文件）：
   - `{{GENERATED_AT_ISO}}`：当前 UTC 时间的 ISO8601 字符串，例如 `2026-05-11T12:00:00Z`。
   - `{{SKILLKIT_RELATIVE_PATH}}`：等于输入 `{skillLibraryRelativePath}`。
   - `{{SKILLKIT_VERSION}}`：读取 `{skillLibraryRoot}/VERSION.md` 全文，取**文件首行**（去除行尾换行）；若首行为空或读取失败则向后扫描至第一行非空的纯文本版本号（例如 `0.1.0-mvp`）；仍失败则填 `unknown`。
   - `{{PROJECT_NAME_OR_SLUG}}`：输入 `{projectNameOrSlug}`，若缺省则为 `{projectRoot}` 路径最后一级目录名。
2. **创建目录**（若已存在则跳过）：
   - `{projectRoot}/.ai/entry/`
   - `{projectRoot}/.ai/state/`
   - `{projectRoot}/.ai/config/`
3. **实例化 `SKILLKIT_LINK.md`**：
   - 读取 `{skillLibraryRoot}/templates/project-ai-context/SKILLKIT_LINK.template.md`。
   - 将模板中所有 `{{GENERATED_AT_ISO}}`、`{{SKILLKIT_RELATIVE_PATH}}`、`{{PROJECT_NAME_OR_SLUG}}` 替换为步骤 1 的值。
   - 写入 `{projectRoot}/.ai/entry/SKILLKIT_LINK.md`（若文件已存在：仅当人类明确要求「重置链接」时覆盖；否则跳过写入并在回复中说明）。
4. **实例化 `AI_ENTRY.md`**：
   - 读取 `{skillLibraryRoot}/templates/project-ai-context/AI_ENTRY.template.md`，同样替换三占位符。
   - 写入 `{projectRoot}/.ai/entry/AI_ENTRY.md`（若已存在且无「强制重置」指令，则跳过覆盖）。
5. **实例化 `skillkit-status.md`**：
   - 读取 `{skillLibraryRoot}/templates/project-ai-context/skillkit-status.template.md`，替换 `{{SKILLKIT_VERSION}}`、`{{GENERATED_AT_ISO}}`（所有出现处）。
   - 若 `{projectRoot}/.ai/state/skillkit-status.md` 不存在：写入新文件。
   - 若已存在：由调用方（如 `load-skill-library`）决定是否覆盖；本 Skill 默认**更新** YAML front matter 中 `libraryVersion` 与 `linkedAt` 字段为当前值，保留其它行人类编辑（若解析失败则整体覆盖为模板实例，并在回复声明）。
6. **实例化 `language-policy.md`**：
   - 若 `{projectRoot}/.ai/config/language-policy.md` **不存在**：读取 `{skillLibraryRoot}/templates/config/language-policy.template.md`，**不**替换正文内说明性占位；直接写入，确保 front matter 含 `reviewedByHuman: false`、`status: draft`。
   - 若已存在：读取 front matter；若 `reviewedByHuman: true`，**不得覆盖**；若为 `false` 或无法解析，且用户未要求保留草稿，可将缺失字段补全但**不得**将 `reviewedByHuman` 改为 `true`（须由人类确认）。

## 7. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/entry/AI_ENTRY.md` |
| `{projectRoot}/.ai/entry/SKILLKIT_LINK.md` |
| `{projectRoot}/.ai/state/skillkit-status.md` |
| `{projectRoot}/.ai/config/language-policy.md` |

| 禁止（MVP） |
|-------------|
| `{projectRoot}/.ai/docs/`、`{projectRoot}/.ai/indexes/`、`{projectRoot}/.ai/reports/` 等未在本 Skill 列出的子树 |
| `{skillLibraryRoot}/**` 任意写入 |
| `{projectRoot}/AGENTS.md`（由 `create-or-update-agents-md` 负责） |

## 8. 禁止事项

- **禁止**将公共库整树复制进 `{projectRoot}/.ai/`。
- **禁止**把业务项目源码或扫描结果写入 `{skillLibraryRoot}`。
- **禁止**在未经人类确认的情况下将 `language-policy.md` 的 `reviewedByHuman` 设为 `true`。

## 9. 完成标准

- [ ] 三个子目录 `.ai/entry`、`.ai/state`、`.ai/config` 均存在。
- [ ] 四个目标文件均存在（语言策略在「已人工确认」情况下允许已预先存在且未被覆盖）。
- [ ] `SKILLKIT_LINK.md` 中表格或正文中出现的 `skillLibraryRootRelative` 值与 `{skillLibraryRelativePath}` 一致。
- [ ] `SKILLKIT_LINK.md` 中可解析出指向真实存在的 `{skillLibraryRoot}/ENTRY.md`（由下一步 `check-project-readiness` 校验亦可）。

## 10. 失败处理

| 情况 | 处理 |
|------|------|
| `{skillLibraryRelativePath}` 未提供 | 中止；要求人类给出例如 `../HACF` 的相对路径。 |
| 拼接后 `{skillLibraryRoot}/ENTRY.md` 不存在 | 中止；提示修正相对路径；**不**写入任何 `.ai` 文件或回滚已写入文件（若已部分写入，在回复列出已写入路径供人类删除）。 |
| 模板缺失 | 中止；提示公共库损坏或不完整。 |
| 无写权限 | 中止；报告具体路径。 |
