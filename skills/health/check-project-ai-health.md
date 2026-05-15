# Skill：check-project-ai-health

## 1. 目的

在**不修改业务源码、不修改 `AGENTS.md`、`CLAUDE.md`、`.cursor/rules/` 下 Project Rule 文件、不自动修复、不修改 Pattern Pack、不修改项目本地 Skill 正文、不修改公共 HACF 库**的前提下，对当前项目 `{projectRoot}` 下 **HACF 协作框架**（入口、单目录约定、配置、状态、文档、索引、Pattern Pack、项目本地 Skill、风险与闭环）做一次**只读健康检查**，将结论写入 `{projectRoot}/.ai/reports/project-ai-health-report.md` 与 `{projectRoot}/.ai/state/project-ai-health-status.md`。

## 2. 适用场景

- 项目已运行过一段时间，需确认 `.ai/` 内框架仍**可用、可信、可执行**。
- 重大变更（合并分支、批量改配置、晋升/激活 Skill 或 Pattern）后的门禁复查。
- 人类希望获得按严重等级分类的问题清单与**修复优先级**，而非自动改库。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根目录。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；用于读取 Rule 与模板；**不得**写入。若缺省则从 `{projectRoot}/.ai/entry/SKILLKIT_LINK.md` 解析（与 `check-project-readiness` 相同算法：`rel` → `normalize({projectRoot}/rel)`）。 |

## 4. 输出

| 路径（相对 `{projectRoot}/.ai/`） | 模板 |
|----------------------------------|------|
| `reports/project-ai-health-report.md` | [`templates/reports/project-ai-health-report.template.md`](../../templates/reports/project-ai-health-report.template.md) |
| `state/project-ai-health-status.md` | [`templates/state/project-ai-health-status.template.md`](../../templates/state/project-ai-health-status.template.md) |

执行完成后须在回复中列出上述**两个**输出路径（相对 `.ai/`）。

### 模板占位符

| 占位符 | 含义 |
|--------|------|
| `{{CHECKED_AT_ISO}}` | 本次检查 UTC ISO8601 时间戳。 |
| `{{OVERALL_HEALTH}}` | `blocked` / `degraded` / `healthy`，语义见 `project-ai-health-rule`。 |
| `{{CRITICAL_COUNT}}` | 非负整数，`critical` 条数。 |
| `{{HIGH_COUNT}}` | 非负整数。 |
| `{{MEDIUM_COUNT}}` | 非负整数。 |
| `{{LOW_COUNT}}` | 非负整数。 |
| `{{BLOCKING_ISSUES_COUNT}}` | 阻塞项条数（所有 `critical` + 规则定义的阻塞类 `high`）。 |
| `{{HUMAN_INTERVENTION_REQUIRED}}` | 小写 `true` 或 `false`，与 `project-ai-health-rule` 一致。 |

## 5. 前置条件

- `{projectRoot}/.ai/` 目录存在；否则按 §11 中止。
- 可读权威 Rule：`{skillLibraryRoot}/rules/health/project-ai-health-rule.md`、`{skillLibraryRoot}/rules/health/local-skill-capability-boundary-rule.md`。
- 建议只读：`{skillLibraryRoot}/rules/pattern/pattern-validation-rule.md`、`{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`、`{skillLibraryRoot}/rules/base/single-ai-context-directory-rule.md`、`{skillLibraryRoot}/rules/base/project-local-output-rule.md`、`{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、`{skillLibraryRoot}/rules/base/source-of-truth-rule.md`、`{skillLibraryRoot}/rules/documentation/agent-context-budget-rule.md`、`{skillLibraryRoot}/rules/bootstrap/agent-default-entry-rule.md`、`{skillLibraryRoot}/rules/bootstrap/agent-entry-thin-adapter-rule.md`、`{skillLibraryRoot}/rules/bootstrap/hacf-version-compatibility-rule.md`、`{skillLibraryRoot}/rules/bootstrap/hacf-local-upgrade-rule.md`、`{skillLibraryRoot}/rules/bootstrap/hacf-upgrade-no-overwrite-rule.md`。
- 两模板文件存在；否则中止（§11）。

## 6. 执行步骤

按下列 **H1–H11** 领域逐项检查，每条发现须标注 **严重等级**（`critical` / `high` / `medium` / `low`）、**领域编号**、**证据路径**（相对 `{projectRoot}`，保持原文）。最后汇总计数、判定 `overallHealth` 与 `humanInterventionRequired`，**覆盖写入**两输出文件。

### H1 入口健康

1. 验证 `{projectRoot}/AGENTS.md` 存在；否则 **`critical`**（H1）。
2. 验证 `{projectRoot}/.ai/entry/AI_ENTRY.md` 存在；否则 **`critical`**（H1）。
3. 验证 `{projectRoot}/.ai/entry/SKILLKIT_LINK.md` 存在；否则 **`critical`**（H1）。
4. 解析 `{skillLibraryRoot}`（见 `check-project-readiness` §6 步骤 1）；验证 `{skillLibraryRoot}/ENTRY.md` 与 `{skillLibraryRoot}/VERSION.md` 存在。任一缺失：**阻塞类 `high`** 或 **`critical`**（若完全无法继续解析公共库根则 **`critical`**）（H1）。
5. `AGENTS.md` 须含子串 `.ai/entry/AI_ENTRY.md` 或 `AI_ENTRY.md`；否则 **`high`**（H1）。
6. **Agent 默认极薄入口（`agent-entry-policy.md` 驱动）**：
   - 若 `.ai/config/agent-entry-policy.md` **不存在**：记 **`low`**（H1）一条，客观说明可运行 `skills/bootstrap/ensure-agent-default-entry.md` 生成 policy 与多 Agent 入口；**不得**因缺失 `CLAUDE.md` 或 Cursor Rule 单独升级至 `high`/`critical`。
   - 若文件存在：解析 YAML；非法 YAML 记 **`high`**（H1）。
   - 读取 `mandatoryAgents`（缺失或空数组表示**无**强制 Agent 专属入口）。
   - 若 `mandatoryAgents` 含 `claude-code`：验证 `CLAUDE.md` 存在，且含成对 `<!-- HACF_AGENT_DEFAULT_ENTRY:BEGIN -->` / `<!-- HACF_AGENT_DEFAULT_ENTRY:END -->`，且其间或全文含 `AI_ENTRY.md` 或 `.ai/entry/AI_ENTRY.md`；任一不满足：**阻塞类 `high`**（H1）。若仅一侧标记或不成对：**阻塞类 `high`**（H1）（须人工整理，本 Skill 不修复）。
   - 若 `mandatoryAgents` 含 `cursor`：取 `cursorRulePath`（缺省 `.cursor/rules/hacf.mdc`），验证该相对路径文件存在且满足与上条等价的标记与 `AI_ENTRY` 指针；不满足：**阻塞类 `high`**（H1）。
   - **极薄启发式**：`CLAUDE.md` 或上述 Cursor Rule 若全文行数 **> 80** 或非受控区疑似整篇 Skill，记 **`medium`**（H1），提示人工对照 `agent-entry-thin-adapter-rule`。

### H2 单目录规范与 AGENTS 极薄

1. 对照 `single-ai-context-directory-rule`：除允许的 `AGENTS.md`、符合 HACF 极薄的 `CLAUDE.md`、`.cursor/rules/` 下单文件 Cursor Rule 外，AI 协作文档应集中在 `.ai/`；若在 `{projectRoot}` 根发现明显协作文档（如 `AI_NOTES.md`、`CODEX.md` 等，**仅报告不删除**），记 **`medium`** 或 **`high`**（依体积与是否重复 `.ai/` 内容）（H2）。若存在 `CLAUDE.md` 且已满足 H1 中 HACF 受控区块与指针要求，**不得**仅因「根目录存在 CLAUDE.md」记为违规。
2. `AGENTS.md` **极薄**启发式（保守）：若全文行数明显过多（例如 **> 200 行**）或「非受控区」含大量二级标题/ fenced 代码块疑似整篇 Skill，记 **`medium`**，并说明须**人工**对照 Rule 判断是否违规；**不得**擅自修改 `AGENTS.md`（H2）。

### H3 配置约束健康

1. 枚举 `{projectRoot}/.ai/config/*.md`；若目录不存在或为空，记 **`medium`**（H3）（Bootstrap 外项目可能尚未扩展配置）。
2. 对每个文件：解析 YAML front matter；非法 YAML：**`high`**（H3）。
3. 若某文件 `status: active`，则须 `reviewedByHuman: true` 且 `confidence: confirmed`（与 `check-project-readiness` R6 语义一致）；否则 **`high`**（若该文件宣称约束全局且未确认，可升 **`critical`**，须在报告中写明理由）（H3）。
4. 扫描正文是否含 `needsHumanConfirmation`、`待人工确认`、`需人工确认` 等；若 `active` 且存在未关闭的待确认项，记 **`medium`** 或 **`high`**（H3）。

### H4 状态一致性

1. 只读尝试读取：`.ai/state/readiness.md`、`.ai/state/skillkit-status.md`、`.ai/state/onboarding-status.md`、`.ai/state/localization-level.md`。缺失不一律阻塞，但须在报告中列入对应领域（**`low`**–**`medium`**）（H4）。
2. 若 `skillkit-status.md` 存在且 front matter 中 `projectState` 与 `readiness.md` 摘要明显矛盾，记 **`medium`**（H4）。
3. 解析失败记 **`medium`** 并列出文件路径（H4）。

### H4.1 框架版本与能力兼容（`hacf-version-status`）

1. 读取 `{skillLibraryRoot}/capabilities/hacf-capabilities.yml` 是否存在；缺失记 **`high`**（H4.1）并提示公共库不完整。
2. 读取 `.ai/state/hacf-version-status.md`：不存在记 **`medium`**（H4.1）（建议先运行 `check-hacf-version-compatibility`）。
3. 若存在：解析 front matter `compatibilityStatus`。值为 `outdated` 记 **`high`（阻塞类 `high`）**（H4.1）：公共 Skill 与项目本地结构可能不同步，须先 `plan-hacf-local-upgrade` / `apply-hacf-local-upgrade`。值为 `unknown` 记 **`high`**（H4.1）。值为 `incompatible` 记 **`critical`**（H4.1）。
4. 读取 `.ai/state/skillkit-status.md`：`localFrameworkVersion` 与 `libraryVersion` 若同时存在且不等，记 **`medium`**（H4.1）（须在检查或升级流程中收敛）。
5. `{skillLibraryRoot}/VERSION.md` 首行与 `hacf-capabilities.yml` 中 `framework.currentVersion` 不一致，记 **`medium`**（H4.1）（公共库维护问题）。

### H5 文档健康（project / modules）

1. 对 `.ai/docs/project/**/*.md`、`.ai/docs/modules/**/*.md`（若目录不存在则记 **H5 `low`** 一条汇总）抽取正文中的仓库相对路径（链接目标、反引号路径等），排除 `http(s)://`、`#` 锚点、`{{` 占位、`..` 段；**最多校验 40 条**路径在 `{projectRoot}` 下是否存在；不存在记 **`high`** 或 **`medium`**（依是否核心模块文档）；超出 40 条记 **`medium`**「截断」（H5）（遵守 `agent-context-budget-rule`）。
2. 对 `docs/modules/<moduleId>/` 路径，核对目录名与文中声明的 `moduleId`（若有）是否一致；不一致记 **`medium`**（H5）。
3. 扫描上述文档是否含大小写不敏感子串 `stale`、`unknown`、`needs_human_confirmation`；命中记 **`medium`**（H5）（提示人工跟进）。

### H6 索引健康

对下列文件（相对 `{projectRoot}/.ai/`）逐项检查存在性与内容是否为空壳占位：

- `indexes/module-index.md`
- `indexes/file-index.md`
- `indexes/directory-index.md`
- `indexes/code-type-index.md`
- `indexes/pattern-index.md`（若项目未使用该文件，缺失可记 **`low`**，但须在报告说明）
- `indexes/skill-index.md`

若文件存在：抽样核对其中出现的 `codeTypeId`、Pattern 路径、`skills/project-local/` 路径是否在磁盘存在；不存在记 **`high`**（H6）。索引整体缺失且项目声称已完成扫盘，记 **`medium`**（H6）。

### H7 Pattern Pack 健康

1. 枚举 `{projectRoot}/.ai/pattern-packs/*/` 下每个子目录（每个视为一个 `codeTypeId`）。
2. 对每个 Pack：四文件 `pattern.md`、`validator.md`、`examples.md`、`docs.md` 须存在；缺失 **`high`**（H7）。
3. 解析四文件 front matter：合法 YAML；`status` 为 `draft`|`active`|`deprecated`；`reviewedByHuman` 为布尔；**`draft` 且 `reviewedByHuman: true`** 或 **`active` 且 `reviewedByHuman` 非 `true`** 记 **`high`**（active 未审阅可记**阻塞类 `high`**，见 `project-ai-health-rule`）（H7）。
4. **`validator.md` 可执行**：按 `pattern-validation-rule` 检查项 9（不少于 3 条 `- [ ]`/`- [x]` 或编号步骤或带语言标记的可复制命令围栏）判定；不满足 **`high`**（H7）。
5. **`examples.md` 路径**：从 Pack 四文件正文收集候选路径，**本 Pack 本轮上限 40 条**（计入 H5 时单独计数或共享预算由 Agent 在报告中说明；**推荐每 Pack 最多 40 条**以免爆炸）；不存在记 **`high`**（H7）。
6. 可选：若存在 `state/pattern-validation-status.md`，在报告「证据与备注」引用其 `codeTypeId` 与结论，**不替代**本轮 H7 判定。

### H8 项目本地 Skill 健康

1. 枚举 `.ai/skills/project-local/*.md`（若无目录或文件，记 **H8 `low`** 一条）。
2. 对每个文件按 `local-skill-capability-boundary-rule` 检查：`status`/`reviewedByHuman` 组合、`moduleId`、`codeTypeId`、`dependsOn` 存在性与解析、`allowedWriteScopes`、`forbiddenWriteScopes`、`requiresPlanBeforeApply`、validator 关联、失败处理小节；违规项按该 Rule 标注等级并归 **H8**。

### H9 风险控制

1. 在 H8 结果上叠加：是否存在可改高风险区而无计划/人工确认、是否允许写公共库表述、是否缺少失败处理（若 H8 未覆盖则此处 **`medium`**/**`high`**）。
2. 与 `.ai/config/danger-zones.md`、`.ai/config/hard-constraints.md`（若存在）做路径交集启发式检查（见 `local-skill-capability-boundary-rule`）；记 **H9**。

### H10 闭环状态与 localization-level

1. 依据 `evaluate-localization-level` Skill §6 的证据链，核对 `state/localization-level.md` 中声明的 `localizationLevel`（`L0`–`L3`）是否与当前仓库事实一致；不一致记 **`medium`** 或 **`high`**（H10）。
2. 列出各阶段产物缺口：`load`（链接）、`readiness`、`hacf-version-status`（框架版本比对）、`agent-entry`（`agent-entry-status.md` / policy 与极薄多入口）、`scan`（索引/scan-status）、`pattern`（active Pack）、`generation`（如 `pattern-code-generation-status.md` 可选）、`local skill`（`create-*.md` + `skill-index`）；缺口记 **`low`**–**`medium`**，若导致「宣称可执行但证据不足」则 **`high`**（H10）。

### 汇总与写入

1. 统计 `criticalCount`、`highCount`、`mediumCount`、`lowCount`。
2. 计算 **`BLOCKING_ISSUES_COUNT`**：所有 `critical` 条数 + `project-ai-health-rule` 定义的阻塞类 `high` 条数。
3. 按 `project-ai-health-rule` 判定 **`{{OVERALL_HEALTH}}`** 与 **`{{HUMAN_INTERVENTION_REQUIRED}}`**。
4. 填充模板，**覆盖写入** `reports/project-ai-health-report.md` 与 `state/project-ai-health-status.md`；报告须含 Rule 要求的六个中文小节且表格/列表可执行。

## 7. 读取范围

| 路径（相对 `{projectRoot}/`，除非注明） | 说明 |
|------------------------------------------|------|
| `AGENTS.md`、`CLAUDE.md`（若存在） | 入口与极薄启发式 |
| `.ai/entry/AI_ENTRY.md`、`.ai/entry/SKILLKIT_LINK.md` | 入口 |
| `.ai/config/agent-entry-policy.md`、`.ai/state/agent-entry-status.md` | Agent 默认入口策略与执行记录 |
| `.ai/config/**/*.md` | 配置约束 |
| `.ai/state/readiness.md`、`skillkit-status.md`、`onboarding-status.md`、`localization-level.md`、`hacf-version-status.md` | 状态；及其它存在的 `state/*.md` 供 H10 可选引用 |
| `.ai/docs/project/**`、`.ai/docs/modules/**` | 文档健康 |
| `.ai/indexes/*.md` | 索引 |
| `.ai/pattern-packs/**` | Pack 四文件 |
| `.ai/skills/project-local/*.md` | 项目本地 Skill |
| `.ai/state/pattern-validation-status.md` | 可选交叉引用 |
| `{skillLibraryRoot}/ENTRY.md`、`VERSION.md`、`capabilities/hacf-capabilities.yml` | 公共库解析与能力声明 |
| `{skillLibraryRoot}/rules/**` | 按需只读 |

**禁止**为检查目的批量深读业务源码树；路径存在性检查以文件系统 `exists` 为准。

## 8. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/reports/project-ai-health-report.md` |
| `{projectRoot}/.ai/state/project-ai-health-status.md` |

| 禁止 |
|------|
| 修改 `{projectRoot}` 下业务源码 |
| 修改 `{projectRoot}/AGENTS.md`、`CLAUDE.md`、`.cursor/rules/` 下由 policy 声明之 Cursor Rule 文件 |
| 修改 `{projectRoot}/.ai/` 下除上表两文件外的任意文件 |
| 写入或修改 `{skillLibraryRoot}/**` |
| 自动修复（包括但不限于 front matter、索引、Pack、Skill） |

## 9. 禁止事项

1. **不得**修改业务源码或 `AGENTS.md`。  
2. **不得**将任何 `reviewedByHuman` 改为 `true` 或伪造人工确认。  
3. **不得**修改公共 HACF 库。  
4. **不得**因检查失败而删除人类已有文档。  
5. **不得**实现 Placement、`skills/develop/**`、新 Pattern 能力或自动修复逻辑。

## 10. 完成标准

- [ ] `project-ai-health-report.md` 已写入，含 YAML front matter，且正文含：**总体结论**、**阻塞项**、**按严重等级分类的问题清单**、**可用能力判断**、**建议修复顺序**、**是否需要人工处理**。  
- [ ] `project-ai-health-status.md` 已写入，front matter 含 `lastProjectAiHealthCheckAt`、`overallHealth`、各计数、`humanInterventionRequired`、`blockingIssuesCount`。  
- [ ] `overallHealth` 与 `project-ai-health-rule` 定义一致。  
- [ ] 未修改源码、未修改 `AGENTS.md` / `CLAUDE.md` / Cursor Project Rule、未修改公共库。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| `.ai/` 不存在 | 中止；回复提示先执行 `load-skill-library` / `initialize-project-ai-context`；不写报告。 |
| `project-ai-health-rule.md` 或模板缺失 | 中止；列出缺失的 `{skillLibraryRoot}` 路径。 |
| `SKILLKIT_LINK.md` 缺失 | H1 **`critical`**；仍尽力写入两产出（若可创建目录），`overallHealth=blocked`。 |
| 部分路径无读权限 | 在报告中如实记录；该检查子项标 **`high`** 或 **`medium`**；不声称已读。 |
| 路径校验达到上限 | 按 Rule 记 **`medium`** 并备注截断。 |

## 权威 Rule

严重等级、阻塞项、`overallHealth`、`humanInterventionRequired`、报告必备小节及写入边界以 [`{skillLibraryRoot}/rules/health/project-ai-health-rule.md`](../../rules/health/project-ai-health-rule.md) 为准；项目本地 Skill 边界以 [`{skillLibraryRoot}/rules/health/local-skill-capability-boundary-rule.md`](../../rules/health/local-skill-capability-boundary-rule.md) 为准。
