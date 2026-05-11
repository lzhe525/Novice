# Skill：scan-module-by-ai

## 1. 目的

针对 `{projectRoot}` 内**单一模块根目录** `{moduleRootRelative}`，在只读源码前提下，于 `.ai/docs/modules/` 镜像路径下一次性生成五类文档：**模块真实行为以该路径下源码为准**；下列文件均为辅助上下文。

## 2. 适用场景

- 已存在项目级 `overview.md` 或索引，需要对子树补充模块级文档套系。
- 单体仓库按功能目录（如 `src/features/checkout`）分批扫盘。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根。 |
| `{skillLibraryRoot}` | 公共库根。 |
| `{moduleRootRelative}` | **必填**。模块根相对 `{projectRoot}` 的路径，正斜杠，无盘符，例如 `src/services/payment`。不得包含 `..`。 |

## 4. 输出

以下路径均在 `{projectRoot}/.ai/docs/modules/{moduleRootRelative}/` 下镜像创建（与仓库模块目录结构一致）：

| 文件 | 模板 |
|------|------|
| `overview.md` | [templates/docs/module-overview.template.md](../../templates/docs/module-overview.template.md) |
| `deep-dive.md` | [templates/docs/module-deep-dive.template.md](../../templates/docs/module-deep-dive.template.md) |
| `constraints.md` | [templates/docs/module-constraints.template.md](../../templates/docs/module-constraints.template.md) |
| `change-guide.md` | [templates/docs/module-change-guide.template.md](../../templates/docs/module-change-guide.template.md) |
| `risk-points.md` | [templates/docs/module-risk-points.template.md](../../templates/docs/module-risk-points.template.md) |

## 5. 前置条件

- `{projectRoot}/{moduleRootRelative}` 为已存在目录。
- 可读上述五个模板与 `scan-project-by-ai` 第 5 节所列 Rule 路径。

## 6. 执行步骤

1. 规范化 `{moduleRootRelative}`，拒绝 `..`。
2. 验证 `{projectRoot}/{moduleRootRelative}` 为目录；否则中止。
3. 创建目录链至 `{projectRoot}/.ai/docs/modules/{moduleRootRelative}/`。
4. 设 `{{MODULE_ROOT_RELATIVE}}`、`{{MODULE_SLUG}}`（路径最后一级）、`{{SCANNED_AT_ISO}}`。
5. **清单**：列出 `{projectRoot}/{moduleRootRelative}` 下一层条目，最多 **60** 名称。
6. **深读**：选择最多 **8** 个与对外接口相关的源文件（启发式同前版），每文件 ≤ **260** 行或 **24KiB**（超限则头 **160** + 尾 **80** 行）。在内存中保留笔记，**五份产出共用同一批阅读结果**，禁止为每个模板分别再通读全模块树。
7. 依次实例化五个模板，填充正文；`constraints` / `change-guide` / `risk-points` 中凡无依据的条目必须以「待确认」或「推断」表达。
8. 依次写入五个 `.md` 文件，分别校验 front matter。
9. **更新索引**：若 `{projectRoot}/.ai/indexes/module-index.md` 存在，读取之；在「模块条目」表中找到或追加 `moduleRootRelative` 对应行，填入指向以下五个文件的 `.ai` 相对链接：`docs/modules/{moduleRootRelative}/overview.md` 等同理五列；若索引不存在，**不**由本 Skill 强制创建完整项目索引（可写一条备注到本轮最后一个写入文件的正文末尾或提醒人类先跑 `scan-project-by-ai`）。若索引存在且表格更新失败，须在回复中说明。

## 7. 读取范围

| 类型 | 范围 | 上限 |
|------|------|------|
| 目录清单 | `{projectRoot}/{moduleRootRelative}` 一层 | 60 条目 |
| 源文件深读 | 该目录下一层内选中文件 | 8 文件；单文件行/字节见步骤 6 |
| 配置引用 | 模块内 `package.json` 或 `project.json` | ≤1 个文件，≤200 行 |

**不**默认递归读取子包全部深层；子目录需另一次本 Skill 或 `scan-file-by-ai`。

## 8. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/docs/modules/{moduleRootRelative}/overview.md` |
| `{projectRoot}/.ai/docs/modules/{moduleRootRelative}/deep-dive.md` |
| `{projectRoot}/.ai/docs/modules/{moduleRootRelative}/constraints.md` |
| `{projectRoot}/.ai/docs/modules/{moduleRootRelative}/change-guide.md` |
| `{projectRoot}/.ai/docs/modules/{moduleRootRelative}/risk-points.md` |
| `{projectRoot}/.ai/indexes/module-index.md`（仅允许**合并更新**模块行，禁止整体替换为人类未备份的大段手工内容） |

| 禁止 |
|------|
| 写入 `{skillLibraryRoot}/**` |
| 修改 `{projectRoot}/{moduleRootRelative}` 下源码 |
| 在 `.ai/docs/modules/` 下写入与 `{moduleRootRelative}` 镜像不一致的旁路路径 |

## 9. 禁止事项

- 虚构对外 API（未出现于已读文件则标待确认）。
- 将本 Skill 产出写入公共库。
- 单次深读文件数超过上表（违反 `agent-context-budget-rule`）。

## 10. 完成标准

- [ ] 五个 Markdown 均已写入且 front matter 中 `moduleRootRelative`、`scanSkill: scan-module-by-ai`、`scannedAt` 正确。
- [ ] 每份文档含模板所载「事实源声明」或等价表述。
- [ ] `overview.md` 含「对外接口」节且列表带来源路径。
- [ ] `risk-points.md` 含风险表模板结构（允许行为「无已知风险」但须说明依据不足）。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| `{moduleRootRelative}` 非法 | 中止。 |
| 选中 0 个可读源文件 | 仍生成五文件，正文说明数据不足；待确认项列出需人类指定入口文件。 |
| 某一模板缺失 | 中止本轮；已写入文件在回复中列出（人类可删除半成品）。 |
