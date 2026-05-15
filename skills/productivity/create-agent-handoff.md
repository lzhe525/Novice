---
status: active
routeEnabled: true
description: 将会话上下文压缩为交接文档供下一 Agent 续写。
triggerWhen:
  - 交接
  - handoff
  - 切换 Agent
  - 会话总结
---
# Skill：create-agent-handoff

## 1. 目的

将当前会话中的工作目标、已达成结论、未决事项与建议的后续阅读路径，压缩为一份**固定路径**的交接文档，供下一次 Agent 或人类快速续写；**不**依赖 shell 临时文件或 CLI，**不**重复粘贴已在其它产物中的全文。

## 2. 适用场景

- 会话即将结束，需要将上下文交给新的 Agent 会话继续。
- 长任务中途暂停，需留下「下一步焦点」与证据指针。
- 用户明确说「写 handoff」「生成交接」「固化到 agent-handoff」。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根目录。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；**仅用于读取** Rule，**不得**写入。 |
| `{nextFocus}` | 下一会话应优先处理的一句话目标（可由用户在对话中给出；缺省则由 Agent 根据对话归纳，并在写盘后请用户确认是否准确）。 |
| `{optionalRelatedPaths}` | 可选。相对 `{projectRoot}` 的重要路径列表（例如 `.ai/reports/doc-impact-report.md`、`.ai/state/readiness.md`），以引用代替长文复制。 |

## 4. 输出

| 路径（相对 `{projectRoot}/.ai/`） | 说明 |
|----------------------------------|------|
| `state/agent-handoff.md` | **覆盖写入**（每次执行本 Skill 均更新为该会话版本）；含 YAML front matter 与正文结构化小节。 |

执行完成后须在回复中列出上述路径（相对 `.ai/`），并提醒下一会话先读该文件。

### front matter 建议字段

| 字段 | 说明 |
|------|------|
| `handoffVersion` | 固定 `1` 或递增整数；与历史兼容时保留约定。 |
| `createdAt` | UTC ISO8601 时间戳。 |
| `nextFocus` | 与输入 `{nextFocus}` 一致或为其精炼一句。 |
| `relatedReportPaths` | 可选。YAML 字符串列表，元素为相对 `{projectRoot}` 的正斜杠路径。 |

正文须使用中文，路径与标识符保持英文。

## 5. 前置条件

- `{projectRoot}/.ai/` 存在；`.ai/state/` 可随写自动创建（若当前环境允许创建目录）；若无法创建目录则按 §11 中止。
- 建议只读：`{skillLibraryRoot}/rules/base/project-local-output-rule.md`、`{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、`{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`。
- **不得**要求用户执行 `mktemp`、随机临时文件名生成或其它 shell 命令来完成写盘。

## 6. 执行步骤

1. **汇总会话事实**：从当前对话提取：初始目标、已确认结论、未决问题、已读或已写的关键 `.ai/` 路径（不含密钥与大段 diff）。  
2. **去重与引用**：若某内容已存在于 `.ai/reports/`、`.ai/state/readiness.md`、扫盘文档等，**仅**在 `relatedReportPaths` 或正文「引用」小节列出相对路径与一句话摘要，**禁止**整篇复制报告正文。  
3. **建议下一动作**：列出 1–3 条建议执行的公共库 Skill 路径（相对 `{skillLibraryRoot}`，使用正斜杠），例如 `skills/bootstrap/check-project-readiness.md`；若未知公共库根，写明「从 `.ai/entry/SKILLKIT_LINK.md` 解析后再拼接」。  
4. **组装 `agent-handoff.md`**：  
   - 顶部 YAML front matter 含 `createdAt`、`nextFocus`、可选 `relatedReportPaths`。  
   - 正文建议包含：`## 背景与目标`、`## 已完成`、`## 未决 / 风险`、`## 建议下一步`、`## 引用路径`（表格或列表）。  
5. **覆盖写入** `{projectRoot}/.ai/state/agent-handoff.md`（UTF-8，遵守 `frontmatter-format-rule`）。  
6. **回复人类**：给出 `.ai/state/agent-handoff.md` 相对路径、一句「下一会话打开先读」提示；**不得**将 handoff 写入 `{skillLibraryRoot}`。

## 7. 读取范围

| 路径 | 说明 |
|------|------|
| 当前对话上下文 | 必需 |
| `{projectRoot}/.ai/state/agent-handoff.md` | 若存在，可读上一版以合并「长期未决」项（可选） |
| `{projectRoot}/.ai/state/readiness.md`、`{projectRoot}/.ai/reports/*.md` | 按需只读以填写引用 |
| `{skillLibraryRoot}/rules/**` | 按需只读 |

## 8. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/state/agent-handoff.md`（**仅当**执行本 Skill 时覆盖写入） |

| 禁止 |
|------|
| 修改 `{projectRoot}` 下业务源码 |
| 修改 `.ai/` 下除 `state/agent-handoff.md` 外的任意文件（本 Skill 不负责其它状态文件） |
| 写入或修改 `{skillLibraryRoot}/**` |
| 在 handoff 正文中粘贴密钥、token、完整环境变量值或大段未裁剪的 diff |

## 9. 禁止事项

1. **不得**使用 `mktemp`、随机临时路径或要求用户本地执行 shell 以确定输出路径。  
2. **不得**对接 git hooks、外部 issue tracker 或自动开单。  
3. **不得**将 handoff 作为向公共库写入项目私有内容的通道。

## 10. 完成标准

- [ ] `agent-handoff.md` 已写入且含合法 YAML front matter。  
- [ ] 正文含 `nextFocus` 与至少一节结构化事实或引用。  
- [ ] 未修改业务源码与公共库；未使用 CLI 生成路径。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| `.ai/` 不存在 | 中止；提示先 Bootstrap；不写 handoff。 |
| 无写权限 | 中止；说明路径；不伪称已写。 |
| 信息不足 | 仍写入 handoff，但在「未决」中显式列出缺口与需人类补充的问题。 |

## 权威 Rule

协作写入白名单以 [`{skillLibraryRoot}/rules/base/project-local-output-rule.md`](../../rules/base/project-local-output-rule.md) 为准。
