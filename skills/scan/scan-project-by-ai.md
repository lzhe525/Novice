# Skill：scan-project-by-ai

## 1. 目的

在**不修改业务源码**的前提下，由 Agent 基于**有预算的**只读浏览，一次性生成项目级文档与索引、扫盘状态：**源码与真实配置文件始终是事实源**；下列 Markdown **仅为辅助上下文**，不得视为权威实现说明。

## 2. 适用场景

- Bootstrap 已完成且 `check-project-readiness` 中 R1–R6 均为 `pass`（或人类显式授权在 `configuring` 下仍生成草稿，须在相关文档 front matter 或 `scan-status.md` 备注中注明）。
- 首次为仓库建立项目级 `.ai/docs` 与**模块文档索引**骨架。
- 重大架构变更后需同步刷新 overview / deep-dive / 索引 / 状态。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根目录。 |
| `{skillLibraryRoot}` | 公共库根；用于读取模板与 Rule。 |
| 可选 `{projectNameOrSlug}` | 写入模板 `{{PROJECT_NAME_OR_SLUG}}`；缺省为 `{projectRoot}` 目录名。 |

## 4. 输出

| 路径 | 说明 |
|------|------|
| `{projectRoot}/.ai/docs/project/overview.md` | [templates/docs/project-overview.template.md](../../templates/docs/project-overview.template.md) |
| `{projectRoot}/.ai/docs/project/deep-dive.md` | [templates/docs/project-deep-dive.template.md](../../templates/docs/project-deep-dive.template.md) |
| `{projectRoot}/.ai/indexes/module-index.md` | [templates/indexes/module-index.template.md](../../templates/indexes/module-index.template.md)（初始可为空表或仅含表头；已存在则**合并更新**项目级链接与已有模块行，勿删除人类手工行） |
| `{projectRoot}/.ai/state/scan-status.md` | [templates/state/scan-status.template.md](../../templates/state/scan-status.template.md) |

## 5. 前置条件

- 已解析 `{skillLibraryRoot}` 与 `{projectRoot}`。
- 可读上述四个模板文件。
- 建议先只读加载：`{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`、`{skillLibraryRoot}/rules/base/source-of-truth-rule.md`、`{skillLibraryRoot}/rules/documentation/structured-doc-writing-rule.md`、`{skillLibraryRoot}/rules/documentation/human-readable-doc-rule.md`、`{skillLibraryRoot}/rules/documentation/agent-context-budget-rule.md`、`{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`。
- `{projectRoot}/.ai/config/language-policy.md` 存在。

## 6. 执行步骤

1. 确认 `{projectRoot}/.ai/` 已存在；若不存在则**中止**并提示先执行 `load-skill-library`。
2. 创建目录（若不存在）：`{projectRoot}/.ai/docs/project/`、`{projectRoot}/.ai/indexes/`、`{projectRoot}/.ai/state/`。
3. 计算 `{{SCANNED_AT_ISO}}`（UTC ISO8601）、`{{PROJECT_NAME_OR_SLUG}}`（输入或目录名）。
4. 按「读取范围」完成清单与深读，在内存中保留**可追溯笔记**（路径 + 结论），供 overview 与 deep-dive 共用，避免对同一文件重复完整读取。
5. 实例化并写入 `overview.md`：不得删除模板内「事实源声明」；校验 YAML front matter。
6. 实例化并写入 `deep-dive.md`：内容须基于步骤 4 笔记；若证据不足，优先写「待确认项」与「推断」措辞，**禁止**虚构架构史。若仍缺关键材料，**追加**深读至多 **2** 个额外源文件（见读取范围表「补读」行），仍不足则 deep-dive 明示数据缺口。
7. 实例化 `module-index.md`：若文件不存在，从模板新建；若已存在，更新「项目级文档快捷链接」小节与 `generatedAt`，并在「模块条目」表中保留已有行；为本轮若已知的模块根路径（来自清单阶段观察到的**一级**源码目录名，如 `src`、`apps` 下的直接子目录）可预填占位行，**不得**为未执行 `scan-module-by-ai` 的模块伪造子文档链接（无则填「—」）。
8. 实例化或覆盖写入 `scan-status.md`：更新 `lastProjectScanAt`、`lastProjectScanSkill`、`projectDocsStatus`、`moduleIndexPath`；正文简述本轮产出相对路径。
9. 全部写入完成后，在回复中列出四个输出路径与相对 `.ai/` 的引用方式。

## 7. 读取范围

| 阶段 | 路径 / 动作 | 上限 |
|------|----------------|------|
| 清单 | `{projectRoot}` 顶层非隐藏条目（不进入 `.git`、不枚举 `node_modules` 内文件） | 最多 **50** 个名称 |
| 清单 | 若存在 `src/`、`lib/`、`apps/`、`packages/` 之一，对其**一层子目录**列表 | 每目录最多 **40** 个子条目 |
| 深读 | `README.md` 或 `README.rst`（存在则其一） | 各最多 **1** 个文件 |
| 深读 | `package.json` 或 `*.sln`（优先 `package.json`） | 最多 **2** 个清单文件 |
| 深读 | `AGENTS.md`、`.ai/entry/AI_ENTRY.md`、`.ai/config/language-policy.md` | 各 **1** 次 |
| 深读 | 由 `package.json` 的 `main`/`bin` 或 `*.sln` 指向的**单个**入口源文件（若可解析） | **1** 个文件，≤ **400** 行或 **32KiB**（可先读头尾各约 **120** 行再决定是否读中段） |
| 补读（仅服务于 deep-dive 缺口） | `{projectRoot}` 内任意源文件，须与步骤 4 已识别模块相关 | 额外 **≤2** 文件，每文件 ≤ **220** 行 |

**禁止**默认递归通读全仓库；更广覆盖通过多次 `scan-module-by-ai` / `scan-file-by-ai` 完成。

## 8. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/docs/project/overview.md` |
| `{projectRoot}/.ai/docs/project/deep-dive.md` |
| `{projectRoot}/.ai/indexes/module-index.md` |
| `{projectRoot}/.ai/state/scan-status.md` |

| 禁止 |
|------|
| `{skillLibraryRoot}/**` 任意写入 |
| `{projectRoot}/.ai/` 外任何路径作为本 Skill 输出 |
| 在缺少读取依据时，将具体业务密钥、token 写入上述文件 |

## 9. 禁止事项

- 将扫描结果、项目私有路径或密钥写入公共库。
- 在文档中断言未读文件中的行为。
- 删除或弱化各模板中关于「源码为事实源」的声明。
- 大段粘贴 `node_modules/`、`.git/objects` 或二进制构建产物内容。

## 10. 完成标准

- [ ] 四个输出路径文件均存在且含合法 YAML front matter（`frontmatter-format-rule`）。
- [ ] `overview.md` 与 `deep-dive.md` 均含「事实源声明」与「待确认项」节（或 deep-dive 明示数据缺口）。
- [ ] `module-index.md` 含「使用方式」节，指导 Agent **经索引按需**打开文档。
- [ ] `scan-status.md` 中 `lastProjectScanAt` 与本轮时间一致。
- [ ] 所有产物均在 `{projectRoot}/.ai/` 下。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| 任一模板缺失 | 中止；提示公共库不完整。 |
| 无写权限 | 中止；报告路径。 |
| 无法识别技术栈 | overview / deep-dive 仍生成，表格与正文使用「未知」与待确认项。 |
| `module-index.md` 合并解析失败 | 可整体重写为模板实例化版本，但须在 `scan-status.md` 备注「索引已重建」，并提醒人类检查丢失的手工行。 |
