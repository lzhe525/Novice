---
status: active
routeEnabled: false
description: 项目级 AI 扫盘并写入 overview、索引与状态。
triggerWhen:
  - 项目扫盘
  - scan project
---
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
| `{projectRoot}/.ai/indexes/directory-index.md` | [templates/indexes/directory-index.template.md](../../templates/indexes/directory-index.template.md)（记录本轮受控扫描下的**重要目录**、`relatedModuleIds`、触发原因、扫描状态、`truncated` 标记） |
| `{projectRoot}/.ai/indexes/file-index.md` | [templates/indexes/file-index.template.md](../../templates/indexes/file-index.template.md)（文件相对路径与 `moduleId` 归属；不存在则自模板新建，已存在则**合并**） |
| `{projectRoot}/.ai/state/scan-status.md` | [templates/state/scan-status.template.md](../../templates/state/scan-status.template.md) |

## 5. 前置条件

- 已解析 `{skillLibraryRoot}` 与 `{projectRoot}`。
- 可读上述六个模板文件（含 `file-index`）。
- 建议先只读加载：`{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`、`{skillLibraryRoot}/rules/base/source-of-truth-rule.md`、`{skillLibraryRoot}/rules/documentation/structured-doc-writing-rule.md`、`{skillLibraryRoot}/rules/documentation/human-readable-doc-rule.md`、`{skillLibraryRoot}/rules/documentation/agent-context-budget-rule.md`、`{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、`{skillLibraryRoot}/rules/project/module-is-logical-boundary-rule.md`、`{skillLibraryRoot}/rules/project/scan-readonly-and-artifacts-rule.md`。
- `{projectRoot}/.ai/config/language-policy.md` 存在。

## 6. 执行步骤

1. 确认 `{projectRoot}/.ai/` 已存在；若不存在则**中止**并提示先执行 `load-skill-library`。
2. 创建目录（若不存在）：`{projectRoot}/.ai/docs/project/`、`{projectRoot}/.ai/indexes/`、`{projectRoot}/.ai/state/`。
3. 计算 `{{SCANNED_AT_ISO}}`（UTC ISO8601）、`{{PROJECT_NAME_OR_SLUG}}`（输入或目录名）。
4. 按第 **7**（受控扫描策略与基础预算）、**8**（重要目录预算重置机制）、**9**（读取范围）节完成清单与深读，在内存中保留**可追溯笔记**（路径 + 结论），供 overview 与 deep-dive 共用，避免对同一文件重复完整读取；同步维护**重要目录清单**（路径、触发原因、是否已扫到叶子、是否 `truncated`、局部预算是否耗尽），供 `directory-index.md` 写入。
5. 实例化并写入 `overview.md`：不得删除模板内「事实源声明」；校验 YAML front matter。
6. 实例化并写入 `deep-dive.md`：内容须基于步骤 4 笔记；若证据不足，优先写「待确认项」与「推断」措辞，**禁止**虚构架构史。若仍缺关键材料，**追加**深读至多 **2** 个额外源文件（见读取范围表「补读」行），仍不足则 deep-dive 明示数据缺口。
7. 实例化或合并 `module-index.md`：若文件不存在，从模板新建；若已存在，更新「项目级文档快捷链接」小节与 `generatedAt`，并在「模块条目」表中保留已有行。**若**存在 `.ai/config/module-map.md`：优先解析其中已声明的 **`moduleId`**（及可选 `moduleDisplayName`、`moduleType`），为每个在表中找行或追加一行；**不得**为未执行 `scan-module-by-ai` 的模块伪造五件套链接（无则填「—」）。若不存在 `module-map.md` 或其中无表格化模块列表：仍可根据清单阶段观察到的**一级**源码目录名（如 `src`、`apps` 下的直接子目录）用 **slug 化目录名**作为候选 `moduleId` 预填占位行，`moduleType` 可写 `directory-module` 或「—」，文档列无则填「—」。
8. 实例化或合并写入 `directory-index.md`：**必须**列出本轮识别的重要目录；每行含 **`relatedModuleIds`**（来自 `module-map` 或推断，无则写「—」）、触发原因、扫描状态（如 `completed` / `partial` / `skipped`）、`truncated`（是/否）。若某重要目录在局部预算内未扫到叶子或仍有大子树未展开，状态须反映为 `partial` 或 `truncated`，并在回复与 `scan-status.md` 中建议后续执行 `scan-module-by-ai`，**优先给出 `moduleId`**（必填），可选附带 `directoryPathRelative` 供人类对照。
9. 实例化或合并 `file-index.md`：若不存在则从模板新建；若已存在则**合并**保留人类手工行。本轮若无可靠文件级归属证据，保留表头并写一行「无」或仅更新 `generatedAt`，**禁止**编造 `moduleIds`。
10. 实例化或覆盖写入 `scan-status.md`：更新 `lastProjectScanAt`、`lastProjectScanSkill`、`projectDocsStatus`、`moduleIndexPath`、`directoryIndexPath`、`fileIndexPath`；正文须含**预算耗尽目录**（若无则写「无」）、**推荐下一步 Skill**（如 `scan-module-by-ai` 或「无」）、**是否需要人工确认**（是/否及理由）。
11. 全部写入完成后，在回复中列出**六个**输出路径与相对 `.ai/` 的引用方式。

## 7. 受控扫描策略与基础预算

**禁止**对全项目做**无条件**深度递归（例如通读 `node_modules/`、整棵 `.git/`、或不经预算与终止条件判断的无限 `list_dir` 下钻）。

基础扫描预算（在未触发 §8 重置前生效）：

| 范围 | 策略 |
|------|------|
| `{projectRoot}` **顶层** | 非隐藏条目**全量列举**（仍遵守忽略目录与「清单」条数上限，见 §9） |
| **疑似源码目录**（如 `src/`、`lib/`、`packages/`、`apps/` 等，以清单观察为准） | 自该根起默认向下展开 **3** 层（层数指目录深度增量；同一逻辑根下不重复累计跨根的层） |
| **测试 / 配置 / 文档**类目录（名称或用途命中如 `test`、`tests`、`__tests__`、`spec`、`e2e`、`config`、`configs`、`docs`、`doc` 等） | 自该根起默认向下展开 **2** 层 |

更广、更深的结构覆盖通过 §8 的**重要目录局部预算重置**、以及后续 `scan-module-by-ai` / `scan-file-by-ai` 分轮完成。

## 8. 重要目录预算重置机制

在遵守 §7 的前提下，若扫描过程中**发现重要目录**，应对该目录执行**局部扫描预算重置**：将该目录视为新的 `scanRoot`，从该目录**重新计算**相对深度与局部条目预算，继续向下扫描，直到满足 **§8.3** 所列任一终止条件或局部预算耗尽。

### 8.1 重要目录判定

命中以下**任一**条件即可视为重要目录（Agent 须在 `directory-index.md` 中写明对应触发原因标签，可多选）：

| 条件 | 说明 |
|------|------|
| 核心业务词匹配 | 目录名与项目核心业务词（来自 `README`、`package.json` name、人类任务描述、或 `.ai/config` 中已声明的产品名/域名词）显著匹配 |
| 大量源码文件 | 该目录下（当前已枚举范围内）源码文件密度或数量明显高于兄弟目录 |
| 关键文件 | 目录内存在入口、注册、声明、配置等关键文件（如 `main`、`index`、`Program`、`Startup`、`App`、`router`、`routes`、`plugin` 注册文件、`*.{yaml,yml,json,toml}` 中与运行强相关的配置等，以只读观察为准） |
| 固定代码类型目录/文件 | 路径或命名体现 `Action`、`Handler`、`Controller`、`Service`、`Repository`、`Command`、`Job` 等（大小写不敏感，含常见复数/后缀变体） |
| module-map 标记 | `{projectRoot}/.ai/config/module-map.md` 将该路径或前缀标记为**核心模块** |
| 用户任务显式提及 | 当前会话任务文字中明确提到的目录或模块路径 |
| 高风险区域疑似 | 如初始化、配置中心、设备控制、权限/鉴权、数据库访问、密钥加载等（仅基于路径名、文件名与已读片段推断时，须在索引中标注为推断并倾向 `partial` / 待 `scan-module-by-ai`） |

### 8.2 局部预算与重置规则

- 将该重要目录作为新的 `scanRoot`：自该点起**重新计算**向下相对深度与本轮局部「清单/列举」配额，目标为**尽可能扫描到叶子目录**（叶子：无未忽略的再下钻子目录，或仅剩文件）。
- 若该子树的局部预算耗尽，则**停止**该 `scanRoot` 下的进一步列举，并在 `directory-index.md` 将该行 `truncated` 标为**是**，扫描状态为 `partial`（或等价表述）。
- **限制重要目录预算重置次数**（全局计数，建议默认 **≤5** 次/本轮 `scan-project-by-ai`）：超过上限后，不再对新发现的重要目录执行重置，仅记录为「待后续模块扫盘」，并在 `scan-status.md` 中说明已达重置上限。

### 8.3 终止条件

对任一当前 `scanRoot`（含初始 `{projectRoot}` 与每次重置后的重要目录），满足以下**任一**即停止该支向下的递归扫描：

1. **到达叶子目录**（在当前忽略规则下无可继续下钻的子目录）。
2. **局部预算耗尽**（该 `scanRoot` 下清单深度、子条目数或本轮为该根分配的读盘配额用尽）。
3. **命中忽略目录**（如 `.git`、`node_modules`、构建输出目录等，以项目 `.gitignore` 与公共 Rule 为准）。
4. **单目录子项过多并被截断**：子项枚举超过 §9 表中单层上限且标记为 `truncated`，不再继续展开该层以下（除非人类或后续 Skill 显式扩大预算）。
5. **Agent 判断继续扫描收益低**：与当前任务及项目级文档目标相比，再下钻重复度高或信息量边际收益极低；须在 `directory-index.md` 或 `scan-status.md` 备注中简要说明依据（不得虚构未读路径内容）。

### 8.4 与后续 Skill 的衔接

- 若重要目录**未扫完**（`partial`、`truncated`、或已达重置次数上限而仍有待扫子树）：**必须**在回复与 `scan-status.md` 的「推荐下一步」中建议执行 **`scan-module-by-ai`**，并给出 **`moduleId`**（从 `module-map` 或目录 slug 推断；推断须标注需人工确认）。**可选**补充 `directoryPathRelative` 作为人类提示；**不得**仅用 `targetPath` 替代 `moduleId` 作为唯一输入说明。
- 所有扫描产物**仅**写入 `{projectRoot}/.ai/`；**不得**修改任何项目源码；**不得**将扫描结果写入公共 `{skillLibraryRoot}`。

## 9. 读取范围

| 阶段 | 路径 / 动作 | 上限 |
|------|----------------|------|
| 清单 | `{projectRoot}` 顶层非隐藏条目（不进入 `.git`、不枚举 `node_modules` 内文件） | 最多 **50** 个名称；与 §7「顶层全扫」一致 |
| 清单 | 若存在 `src/`、`lib/`、`apps/`、`packages/` 之一，对其**一层子目录**列表；对 §8 中每个当前 `scanRoot` 同样适用「一层子目录」清单规则 | 每目录最多 **40** 个子条目；若超限则该层 `truncated=true` |
| 深读 | `README.md` 或 `README.rst`（存在则其一） | 各最多 **1** 个文件 |
| 深读 | `package.json` 或 `*.sln`（优先 `package.json`） | 最多 **2** 个清单文件 |
| 深读 | `AGENTS.md`、`.ai/entry/AI_ENTRY.md`、`.ai/config/language-policy.md`、`.ai/config/module-map.md`（存在则读，服务 §8 判定） | 各 **1** 次 |
| 深读 | 由 `package.json` 的 `main`/`bin` 或 `*.sln` 指向的**单个**入口源文件（若可解析） | **1** 个文件，≤ **400** 行或 **32KiB**（可先读头尾各约 **120** 行再决定是否读中段） |
| 补读（仅服务于 deep-dive 缺口） | `{projectRoot}` 内任意源文件，须与步骤 4 已识别模块相关 | 额外 **≤2** 文件，每文件 ≤ **220** 行 |

**禁止**默认递归通读全仓库；§7、§8 的预算与终止条件优先于上表中的「尽可能」表述。

## 10. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/docs/project/overview.md` |
| `{projectRoot}/.ai/docs/project/deep-dive.md` |
| `{projectRoot}/.ai/indexes/module-index.md` |
| `{projectRoot}/.ai/indexes/directory-index.md` |
| `{projectRoot}/.ai/indexes/file-index.md` |
| `{projectRoot}/.ai/state/scan-status.md` |

| 禁止 |
|------|
| `{skillLibraryRoot}/**` 任意写入 |
| `{projectRoot}/.ai/` 外任何路径作为本 Skill 输出 |
| 在缺少读取依据时，将具体业务密钥、token 写入上述文件 |

## 11. 禁止事项

- 将扫描结果、项目私有路径或密钥写入公共库。
- 在文档中断言未读文件中的行为。
- 删除或弱化各模板中关于「源码为事实源」的声明。
- 大段粘贴 `node_modules/`、`.git/objects` 或二进制构建产物内容。
- 修改任何项目业务源码或非 `.ai/` 下的文件以「配合」文档生成。

## 12. 完成标准

- [ ] 六个输出路径文件均存在且含合法 YAML front matter（`frontmatter-format-rule`，其中 `directory-index.md`、`module-index.md`、`file-index.md` 按各自模板要求）。
- [ ] `overview.md` 与 `deep-dive.md` 均含「事实源声明」与「待确认项」节（或 deep-dive 明示数据缺口）。
- [ ] `module-index.md` 含「使用方式」节，指导 Agent **经索引按需**打开文档；表头以 `moduleId` 为主键。
- [ ] `directory-index.md` 含本轮**重要目录**表：`relatedModuleIds`、触发原因、扫描状态、`truncated`。
- [ ] `file-index.md` 已创建或合并，表结构允许**多模块归属**同一文件。
- [ ] `scan-status.md` 中 `lastProjectScanAt` 与本轮时间一致，且含**预算耗尽目录**、**推荐下一步 Skill**（若建议 `scan-module-by-ai` 则须含 **`moduleId`**）、**是否需要人工确认**。
- [ ] 所有产物均在 `{projectRoot}/.ai/` 下。

## 13. 失败处理

| 情况 | 处理 |
|------|------|
| 任一模板缺失 | 中止；提示公共库不完整。 |
| 无写权限 | 中止；报告路径。 |
| 无法识别技术栈 | overview / deep-dive 仍生成，表格与正文使用「未知」与待确认项。 |
| `module-index.md` 合并解析失败 | 可整体重写为模板实例化版本，但须在 `scan-status.md` 备注「索引已重建」，并提醒人类检查丢失的手工行。 |
| `directory-index.md` 合并解析失败 | 可整体重写为模板实例化版本，并在 `scan-status.md` 备注「目录索引已重建」。 |
| `file-index.md` 合并解析失败 | 可整体重写为模板实例化版本，并在 `scan-status.md` 备注「文件索引已重建」。 |
