# project-ai-health-rule

## 适用主体

执行 `check-project-ai-health`、阅读 `{projectRoot}/.ai/reports/project-ai-health-report.md` 或 `{projectRoot}/.ai/state/project-ai-health-status.md` 的 Agent 与人类维护者。

## 规则陈述

### 只读与写入边界

1. 健康检查 Skill **默认只读**：可读取 `{projectRoot}` 下 `AGENTS.md`、`CLAUDE.md`、`.cursor/rules/` 下 Project Rule 单文件（若存在）、`.ai/**` 及解析得到的 `{skillLibraryRoot}/**`（仅用于校验 `ENTRY.md`、`VERSION.md`、`capabilities/hacf-capabilities.yml`、公共 Skill 路径存在性等）。
2. **不得**修改业务源码、**不得**修改 `AGENTS.md`、`CLAUDE.md`、`.cursor/rules/` 下 Project Rule 文件、**不得**自动修复任何发现问题、**不得**修改 `{skillLibraryRoot}/**`。
3. **仅允许**写入下列两路径（可自动创建 `.ai/reports/`、`.ai/state/`）：
   - `{projectRoot}/.ai/reports/project-ai-health-report.md`
   - `{projectRoot}/.ai/state/project-ai-health-status.md`
4. **不得**因检查而删除或覆盖人类在其它 `.ai/**` 文件中的既有内容（本 Skill 产出仅限上述两文件）。

### 叙述与命名

5. 报告与状态正文中的**说明性文字**使用**简体中文**；仓库内**路径、文件名、Skill 名称、YAML 键名**保持原文，不擅自翻译或改写。

### 严重等级（severity）

每条发现项必须标注下列之一：

| 等级 | 含义 | 典型情形（非穷尽） |
|------|------|-------------------|
| `critical` | 框架不可信或不可安全执行；须先处理 | 缺失 `SKILLKIT_LINK.md` 或无法解析公共库根；`AGENTS.md` 或入口缺失导致无法发现 `.ai/`；正文明确指示将项目内容写入公共 HACF 库；`status: active` 的配置或 Pattern/项目本地 Skill 在 `reviewedByHuman` 非 `true` 时仍宣称可执行写操作；`.ai/state/hacf-version-status.md` 中 `compatibilityStatus` 为 `incompatible`（见 `check-project-ai-health` H4.1） |
| `high` | 高风险；强烈建议在继续自动化前修复 | 合法 YAML 但 `active` 且缺 `confidence: confirmed` 或缺人工审阅；索引与磁盘上 Pack/Skill 严重不一致；`dependsOn` 声明缺失或指向不存在；`allowedWriteScopes` 过度泛化；高风险区可写但无计划门控与人工确认；`agent-entry-policy.md` 中 `mandatoryAgents` 要求 `claude-code` 或 `cursor` 但对应极薄入口缺失、标记不成对或缺少 `AI_ENTRY` 指针（见 `check-project-ai-health` H1） |
| `medium` | 影响可维护性或一致性；应计划修复 | 文档/索引空壳、截断的路径校验、`stale`/`unknown` 标记；`localization-level` 与证据链不符；缺「失败处理」类小节但边界尚可接受 |
| `low` | 信息类或优化建议 | 可选文件缺失、轻微结构建议、非阻塞的待确认项提示 |

**阻塞项（blocking）**：所有 `critical` 项，以及 Rule 中显式定义为阻塞的 `high` 项（例如：`SKILLKIT_LINK` 可解析但 `ENTRY.md`/`VERSION.md` 缺失；`active` 的 Pattern Pack 或项目本地 Skill 未 `reviewedByHuman: true`；`agent-entry-policy.md` 中 `mandatoryAgents` 含 `claude-code` 或 `cursor` 但对应极薄入口未满足 `check-project-ai-health` H1 子项之**阻塞类 high** 条件；`.ai/state/hacf-version-status.md` 中 `compatibilityStatus` 为 `outdated` 之 **H4.1 阻塞类 high** 项）。

### 总体健康度（overallHealth）

汇总语义（在写入两输出文件的 front matter 与正文中保持一致）：

- **`blocked`**：存在任一条 **阻塞项**。
- **`degraded`**：无阻塞项，但存在至少一条 `high` 或 `medium`。
- **`healthy`**：无 `critical`、无 `high`、无 `medium`（允许仅有 `low` 或无问题）。

### 人工门控

6. **`humanInterventionRequired`**：当 `overallHealth` 为 `blocked` 或 `degraded`，或存在任一 `critical` / 阻塞类 `high` 时，必须为 `true`；否则为 `false`。

### 报告必备小节

`project-ai-health-report.md` 正文须包含且标题建议使用下列中文名称：

1. **总体结论**（含 `overallHealth` 一句话摘要）
2. **阻塞项**（若无则写「无」）
3. **按严重等级分类的问题清单**（建议表格：严重等级、领域编号 H1–H11、说明、证据路径）
4. **可用能力判断**（对 scan、`validate-pattern-pack`、`create-code-by-pattern`、项目本地 Skill、`ensure-agent-default-entry`（Agent 极薄多入口）、`check-hacf-version-compatibility` / `plan-hacf-local-upgrade` / `apply-hacf-local-upgrade`（框架跨版本对齐）等给出 `pass` / `conditional` / `blocked` 及一句理由）
5. **建议修复顺序**（有序列表：先解阻塞，再降 `high`、`medium`）
6. **是否需要人工处理**（与 `humanInterventionRequired` 一致并简述理由）

### Pattern 与路径校验复用

7. **validator 可执行**、**examples/docs 中仓库相对路径存在性**的判定口径，与 `rules/pattern/pattern-validation-rule.md` 中检查项 9、14 的语义对齐；健康检查为**全量框架**扫描，**不得**因曾运行 `validate-pattern-pack` 而跳过对 Pack 的独立核查（可将历史校验状态记入「证据与备注」）。
8. 单轮报告中，从文档正文收集的**待校验路径**总数**上限 40**（与 `pattern-validation-rule` 及 `agent-context-budget-rule` 一致）；超出部分记 `medium` 并注明「未校验截断」。

## 须同时遵守的其它 Rule

- `rules/base/frontmatter-format-rule.md`
- `rules/base/single-ai-context-directory-rule.md`
- `rules/base/project-local-output-rule.md`
- `rules/base/public-skill-library-purity-rule.md`
- `rules/base/source-of-truth-rule.md`
- `rules/documentation/agent-context-budget-rule.md`
- `rules/bootstrap/agent-default-entry-rule.md`
- `rules/bootstrap/agent-entry-thin-adapter-rule.md`
- `rules/bootstrap/hacf-version-compatibility-rule.md`
- `rules/bootstrap/hacf-local-upgrade-rule.md`
- `rules/health/local-skill-capability-boundary-rule.md`（项目本地 Skill 与风险子项）
- `rules/pattern/pattern-validation-rule.md`（Pack 子项校验语义）

## 禁止事项

- 伪造未执行的存在性检查或 front matter 解析结果。
- 将健康检查产出写入 `.ai/` 外路径。

## 与其它 Rule 的关系

- 与 `pattern-validation-rule` 互补：后者针对**单一** `codeTypeId` 的 Pack；本 Rule 针对**整个**项目 `.ai/` 协作框架。
- 与 `project-local-output-rule` 互补：健康检查写入路径须在该 Rule 的明示例外列表中。
