---
status: active
routeEnabled: false
description: 依据 doc-impact 报告定点更新 .ai/ 内文档。
triggerWhen:
  - 更新文档
  - update docs after change
---
# Skill：update-docs-after-change

## 1. 目的

在**不修改业务源码、不修改公共 HACF Skill 库**的前提下，**仅**依据 `.ai/reports/doc-impact-report.md` 中的「建议更新清单」，对 `{projectRoot}/.ai/` 内相关文档做**定点**合并或补丁式更新，并在 `.ai/state/doc-impact-status.md` 中记录执行结果。

## 2. 适用场景

- 已存在由 `check-doc-impact-after-change` 生成的 `doc-impact-report.md`，且人类或编排授权进入文档写入阶段。
- 需将报告中的 `yes` 项落地为轻量文档更新，**不**希望触发全量扫盘或全库重写。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根目录。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；**仅用于读取** Rule 与模板，**不得**写入。 |

## 4. 输出

| 路径（相对 `{projectRoot}/.ai/`） | 说明 |
|----------------------------------|------|
| `state/doc-impact-status.md` | **必须**更新：合并写入本轮「最近一次文档更新执行」摘要、已更新路径、跳过项、阻塞原因。 |
| `docs/**`、`indexes/**`、`pattern-packs/**`、`skills/project-local/**` 下**仅**出现在 `doc-impact-report.md`「建议更新清单」表中且目标路径合法的文件 | 按清单「变更类型」定点更新；**不得**写入未列出的路径。 |

执行完成后须在回复中列出**实际写入**的相对 `.ai/` 路径列表（含 `state/doc-impact-status.md`）。

## 5. 前置条件

- `{projectRoot}/.ai/` 存在；`.ai/reports/doc-impact-report.md` 存在且可解析出「建议更新清单」表格（或等价结构化列表）；否则按 §11 将状态标为 `blocked` 并中止正文更新。
- 可读权威 Rule：  
  `{skillLibraryRoot}/rules/documentation/doc-update-policy-rule.md`、  
  `{skillLibraryRoot}/rules/documentation/doc-sync-scope-rule.md`、  
  `{skillLibraryRoot}/rules/documentation/doc-staleness-rule.md`。  
- 建议只读：`{skillLibraryRoot}/rules/documentation/human-readable-doc-rule.md`、`{skillLibraryRoot}/rules/documentation/structured-doc-writing-rule.md`、`{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`、`{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、`{skillLibraryRoot}/rules/base/project-local-output-rule.md`、`{skillLibraryRoot}/rules/documentation/agent-context-budget-rule.md`。
- 状态模板 [`templates/state/doc-impact-status.template.md`](../../templates/state/doc-impact-status.template.md) 存在于 `{skillLibraryRoot}`（用于对齐 front matter 字段；允许在已存在状态文件上合并而非清空人类备注）。

## 6. 执行步骤

1. **只读打开** `.ai/reports/doc-impact-report.md`；解析 front matter 中七类判断（可选用于校验）与正文「建议更新清单」表。若文件缺失、表格缺失或无法解析出至少「目标路径」列，则**仅**写入 `blocked` 态的 `doc-impact-status.md`（说明原因）并中止。
2. **校验每一行清单**：  
   - `目标路径` 必须相对 `{projectRoot}` 且以 `.ai/` 前缀（规范化正斜杠）；须落在 `doc-sync-scope-rule` 允许的子树（`docs`、`indexes`、`pattern-packs`、`skills/project-local`）内。  
   - 若该行在七类判断或备注中被标为 `uncertain`、或路径为空、或指向 `.ai/` 外：**跳过**，记入「跳过或未执行项」。
3. **按行执行变更类型**（须在清单「变更类型」列声明；未声明则默认 `patch_section`）：  
   - `merge_frontmatter`：仅合并 YAML front matter 中允许字段，不删除人类注释性正文。  
   - `patch_section`：仅更新指定标题下的段落或表格行。  
   - `append_changelog`：在约定「变更同步记录」或等价小节末尾追加一条中文记录（含日期与关联变更路径）。  
   - `mark_stale_notice`：按 `doc-staleness-rule` 插入或更新「可能过时」提示，**不**杜撰新架构事实。  
4. **每文件写入前**只读打开当前版本；**禁止**整文件替换为模板空壳；**禁止**删除未在清单依据中说明的整节内容。
5. **上限**：本轮实际执行写入的文件数**建议不超过 25 个**；超出部分跳过并在状态中注明「截断，建议拆分报告轮次」。
6. **更新** `.ai/state/doc-impact-status.md`：`lastDocUpdateAt` 为本次 UTC ISO8601；`lastUpdateOverall` 为 `completed` / `partial` / `skipped`（清单全跳过且无写入）/ `blocked`；列出已更新路径与跳过原因；保留或合并 `lastImpactCheckAt` / `lastReportFingerprint`（可从报告 front matter 同步）。
7. **不得**修改 `doc-impact-report.md` 正文（报告为检查阶段事实冻结；若需更正应重新运行 `check-doc-impact-after-change`）。

## 7. 读取范围

| 路径 | 说明 |
|------|------|
| `.ai/reports/doc-impact-report.md` | **强制** |
| `.ai/state/doc-impact-status.md` | 合并历史 |
| 清单列出的每个目标文件 | 写入前只读当前内容 |
| `{skillLibraryRoot}/rules/**` | 按需只读 |

## 8. 写入位置

| 允许 |
|------|
| `.ai/state/doc-impact-status.md` |
| `.ai/docs/**`、`.ai/indexes/**`、`.ai/pattern-packs/**`、`.ai/skills/project-local/**` 中**已列入** `doc-impact-report.md`「建议更新清单」且通过 §6 校验的路径 |

| 禁止 |
|------|
| 修改业务源码 |
| 修改 `.ai/reports/doc-impact-report.md` |
| 修改 `{skillLibraryRoot}/**` |
| 修改 `AGENTS.md`、`.ai/entry/**`、未列入清单的任意 `.ai/` 文件 |

## 9. 禁止事项

1. **不得**全量重写 `module-index.md` / `file-index.md` 等索引为空白模板式内容。  
2. **不得**在清单外「顺手」更新其它文档。  
3. **不得**将项目私有内容写回公共库。  
4. **`uncertain`** 对应项默认不执行（见 `doc-update-policy-rule`）。

## 10. 完成标准

- [ ] 已读取并解析 `doc-impact-report.md`（或已写 `blocked` 状态并中止）。  
- [ ] 所有写入均可追溯到清单行与依据列。  
- [ ] `doc-impact-status.md` 已反映本轮执行结果。  
- [ ] 未修改业务源码与公共库。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| 报告缺失 | `doc-impact-status.md` 标 `blocked`，列原因；不改其它文件。 |
| 清单为空且七类均为 `no` | 可仅更新状态为 `skipped`；不强制改其它文档。 |
| 目标文件无写权限 | 跳过该条，`partial`，状态中记录。 |

## 权威 Rule

定点更新与禁止全量重生成以 [`{skillLibraryRoot}/rules/documentation/doc-update-policy-rule.md`](../../rules/documentation/doc-update-policy-rule.md) 为准；可写子树与排除项以 [`{skillLibraryRoot}/rules/documentation/doc-sync-scope-rule.md`](../../rules/documentation/doc-sync-scope-rule.md) 为准；过时标注以 [`{skillLibraryRoot}/rules/documentation/doc-staleness-rule.md`](../../rules/documentation/doc-staleness-rule.md) 为准。
