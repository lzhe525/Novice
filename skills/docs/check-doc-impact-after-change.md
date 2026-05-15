---
status: active
routeEnabled: true
description: 改码后只读判断对 .ai/ 文档体系的影响。
triggerWhen:
  - 改码后文档影响
  - doc impact
  - 文档是否需同步
---
# Skill：check-doc-impact-after-change

## 1. 目的

在**不修改业务源码、不直接更新** `.ai/docs/**`、`.ai/indexes/**`、`.ai/pattern-packs/**`、`.ai/skills/project-local/**`（除本 Skill 明示的两份产出外）、**不修改公共 HACF Skill 库**的前提下，基于**代码变更报告与只读证据**，判断本轮变更对项目内 `.ai/` 文档体系的影响，并将结论写入 `.ai/reports/doc-impact-report.md` 与 `.ai/state/doc-impact-status.md`。

## 2. 适用场景

- 已完成或并行存在 `create-code-by-pattern` 落盘产物（计划 / 结果 / 状态三件套），需在改码后评估文档与索引是否跟进。
- 任意业务代码变更后，人类或编排希望在进入 `update-docs-after-change` 之前获得**结构化、可追溯**的影响判断。
- 需区分 **P1 强证据**（Pattern 三件套等）与 **P3 弱证据**（仅 git 路径），并显式列出**不确定项**。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根目录。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；**仅用于读取** Rule 与模板，**不得**写入。缺省解析算法同 `check-project-readiness`（`SKILLKIT_LINK.md` 的 `rel` → `normalize({projectRoot}/rel)`）。 |
| `{optionalChangeReportPath}` | 可选。相对 `{projectRoot}` 的额外变更报告路径（Markdown）；须位于项目根之下；若缺失则在报告中记为缺口，**不得虚构**。 |
| `{codeTypeId}` | 可选。与 Pattern 流程对齐时的代码类型标识；安全 slug，**不得**含 `/`、`\\`、`..`、盘符或空字符串。 |
| `{moduleId}` | 可选。逻辑模块标识；安全 slug，同上。 |

## 4. 输出

| 路径（相对 `{projectRoot}/.ai/`） | 模板 |
|----------------------------------|------|
| `reports/doc-impact-report.md` | [`templates/reports/doc-impact-report.template.md`](../../templates/reports/doc-impact-report.template.md) |
| `state/doc-impact-status.md` | [`templates/state/doc-impact-status.template.md`](../../templates/state/doc-impact-status.template.md) |

执行完成后须在回复中列出上述**两个**输出路径（相对 `.ai/`）。

### 模板占位符（报告 `doc-impact-report.md`）

| 占位符 | 含义 |
|--------|------|
| `{{IMPACT_EVALUATED_AT_ISO}}` | 本次评估 UTC ISO8601 时间戳。 |
| `{{NEEDS_UPDATE_PROJECT_DOCS}}` | `yes` / `no` / `uncertain`。 |
| `{{NEEDS_UPDATE_MODULE_DOCS}}` | 同上。 |
| `{{NEEDS_UPDATE_FILE_DOCS}}` | 同上。 |
| `{{NEEDS_UPDATE_INDEXES}}` | 同上。 |
| `{{NEEDS_UPDATE_PATTERN_PACK}}` | 同上。 |
| `{{NEEDS_UPDATE_PROJECT_LOCAL_SKILL}}` | 同上。 |
| `{{UNCERTAINTY_COUNT}}` | 非负整数，「不确定项」小节条目数（无则 `0`）。 |

### 模板占位符（状态 `doc-impact-status.md`）

| 占位符 | 含义 |
|--------|------|
| `{{LAST_IMPACT_CHECK_AT_ISO}}` | 与 `{{IMPACT_EVALUATED_AT_ISO}}` 一致。 |
| `{{LAST_DOC_UPDATE_AT_ISO}}` | 若已存在旧 `doc-impact-status.md` 且可解析上一 `lastDocUpdateAt`，保留；否则填 `未执行` 或沿用模板提示「尚未由 update-docs-after-change 写入」。 |
| `{{LAST_REPORT_FINGERPRINT}}` | 短指纹，例如 `impact-{ISO分钟}-{变更路径条数}`，用于人类区分轮次。 |
| `{{LAST_UPDATE_OVERALL}}` | 本轮为仅检查：填 `n/a`；若旧状态存在且本轮未跑 update，保留旧值或填 `n/a`（须在 §6 步骤中说明规则）。 |
| `{{LAST_REPORT_SUMMARY}}` | 一至两句中文，概括七类判断中是否为 `yes` 及不确定项数量。 |

## 5. 前置条件

- `{projectRoot}/.ai/` 存在；否则按 §11 中止。
- 可读权威 Rule：  
  `{skillLibraryRoot}/rules/documentation/doc-impact-after-code-change-rule.md`、  
  `{skillLibraryRoot}/rules/documentation/doc-sync-scope-rule.md`、  
  `{skillLibraryRoot}/rules/documentation/doc-staleness-rule.md`。  
- 建议只读：`{skillLibraryRoot}/rules/documentation/doc-update-policy-rule.md`、`{skillLibraryRoot}/rules/documentation/human-readable-doc-rule.md`、`{skillLibraryRoot}/rules/documentation/agent-context-budget-rule.md`、`{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、`{skillLibraryRoot}/rules/base/project-local-output-rule.md`、`{skillLibraryRoot}/rules/base/source-of-truth-rule.md`、`{skillLibraryRoot}/rules/base/read-source-before-edit-rule.md`（对解析出的业务源码路径**仅允许**按需抽样只读打开以核对与文档描述是否明显冲突，**禁止**编辑）。
- §4 所列两模板文件存在于 `{skillLibraryRoot}`；否则中止（§11）。

## 6. 执行步骤

1. **解析 `{skillLibraryRoot}`**；规范化 `{codeTypeId}` / `{moduleId}` / `{optionalChangeReportPath}`：非法 slug 或路径越界（含 `..`、绝对路径指向项目外）时，在报告「不确定项」首条记录原因，七类判断中相关项倾向 `uncertain`，仍尽量写入两产出（若可写）。
2. **收集变更事实（按 `doc-impact-after-code-change-rule` 优先级）**：  
   - **P1**：只读 `.ai/reports/pattern-code-generation-result.md`、`.ai/reports/pattern-code-generation-plan.md`、`.ai/state/pattern-code-generation-status.md`（存在则记「已读」，缺失记缺口）。  
   - **P2**：若提供了 `{optionalChangeReportPath}`，只读该文件；不存在则记缺口。  
   - **P3**：若仍无法得到「已修改文件」列表，可只读执行 git 命令或等价方式获取路径列表（弱证据），须在报告 P3 行与「不确定项」中明确标注。
3. **解析变更路径列表**：从 P1/P2 正文提取「将修改 / 已修改 / 联动」等业务源码路径（去重，**上限 40 条**，超出则截断并在报告注明）。无法解析时依赖 P3 或空列表，并升高 `uncertain`。
4. **映射模块与类型**：结合可选 `{moduleId}`、`{codeTypeId}`、索引与路径启发式，列出涉及的 `moduleId` / `codeTypeId` 候选（**上限 10 个**模块、**5 个** codeType，超出截断说明）。
5. **只读扫视 `.ai/` 相关产物**（存在则读，缺失记 `low` 级别缺口即可，不阻塞写入本 Skill 产出）：  
   - `.ai/docs/project/**`（抽样：至少关注 `overview` / `deep-dive` / `adapter` 若存在）；  
   - 对每个候选 `moduleId`：`.ai/docs/modules/<moduleId>/` 下五件套与 `code-types.md`（若存在）；  
   - 与变更路径可能对应的 `.ai/docs/files/**`（可借助 `indexes/file-index.md` 映射，**最多打开 15 个**文件文档）；  
   - `.ai/indexes/module-index.md`、`file-index.md`、`directory-index.md`、`code-type-index.md`、`skill-index.md`、`pattern-index.md`（存在则读）；  
   - 若 `codeTypeId` 已知或可从索引解析：`.ai/pattern-packs/{codeTypeId}/` 四文件；  
   - `.ai/skills/project-local/*.md`（枚举文件名，与变更 `codeTypeId` 强相关者优先读，**上限 10 个**文件）。
6. **可选只读核对源码**：对变更路径列表中文件（**最多 15 个**）在 `{projectRoot}` 下只读打开或确认存在；**禁止**编辑。仅用于与文档明显矛盾时的证据，**禁止**大段抄写源码入报告。
7. **填写七类判断**：每项 `yes` / `no` / `uncertain`，须各有一句理由及至少一条证据路径（相对 `{projectRoot}`）。判定须遵守 `doc-impact-after-code-change-rule`，不得全部为同一取值而无解释。
8. **编制「建议更新清单」**：仅为 `yes` 或明确且路径可写的项生成表格行；`uncertain` 项**不**放入可执行清单（可放入「不确定项」建议人工后再跑 update）。清单中目标路径必须位于 `.ai/` 下且属于 `doc-sync-scope-rule` 允许子树。
9. **统计** `{{UNCERTAINTY_COUNT}}`（「不确定项」小节条目数）。
10. **读取已有** `.ai/state/doc-impact-status.md`（若存在）：合并保留「最近一次文档更新执行」中的 `lastDocUpdateAt`、已更新路径等历史，供新状态文件使用；若不存在，`{{LAST_DOC_UPDATE_AT_ISO}}` 填 `未执行`，`{{LAST_UPDATE_OVERALL}}` 填 `n/a`。
11. **覆盖写入** `reports/doc-impact-report.md` 与 `state/doc-impact-status.md`，实例化两模板；状态文件中「最近一次影响检查」须更新，「最近一次文档更新执行」若本轮未执行 update 则保留旧内容或占位「未执行」。

## 7. 读取范围

| 路径（相对 `{projectRoot}/`，除非注明） | 说明 |
|------------------------------------------|------|
| `.ai/reports/pattern-code-generation-*.md`、`.ai/state/pattern-code-generation-status.md` | P1 证据 |
| `{optionalChangeReportPath}` | P2 |
| `.ai/docs/project/**`、`.ai/docs/modules/**`、`.ai/docs/files/**` | 文档事实 |
| `.ai/indexes/*.md` | 索引 |
| `.ai/pattern-packs/**`、`.ai/skills/project-local/**` | Pack 与本地 Skill |
| `.ai/state/doc-impact-status.md` | 合并历史用 |
| 解析出的业务源码路径 | **只读**、有上限 |
| `{skillLibraryRoot}/rules/**` | 按需只读 |

## 8. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/reports/doc-impact-report.md`（可自动创建 `.ai/reports/`） |
| `{projectRoot}/.ai/state/doc-impact-status.md` |

| 禁止 |
|------|
| 修改 `{projectRoot}` 下业务源码 |
| 修改 `.ai/` 下除上表两文件外的任意文件 |
| 写入或修改 `{skillLibraryRoot}/**` |

## 9. 禁止事项

1. **不得**在本 Skill 中调用 `update-docs-after-change` 或手写更新其它 `.ai/` 文档。  
2. **不得**修改公共 HACF Skill 库。  
3. **不得**虚构变更路径或报告内容。  
4. **不得**实现 Placement、`skills/develop/**`、`skills/review/**` 或自动修复全库文档。

## 10. 完成标准

- [ ] `doc-impact-report.md` 已写入，含 YAML front matter，且正文含：**总体结论**、**输入证据**、**已解析的变更路径**、**七类判断**表、**建议更新清单**、**不确定项**、**附录：读取范围摘要**。  
- [ ] `doc-impact-status.md` 已写入，且「最近一次影响检查」与本轮时间一致。  
- [ ] 未修改业务源码（除本 Skill 两文件外未写 `.ai/`）、未修改公共库。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| `.ai/` 不存在 | 中止；提示先 Bootstrap；不写产出。 |
| 模板或核心 Rule 缺失 | 中止；列出缺失的 `{skillLibraryRoot}` 路径。 |
| 全部证据缺失 | 仍可写入报告：七类判断多标 `uncertain`，清单可为空，**不得**编造路径。 |
| 路径超出上限 | 截断并注明。 |

## 权威 Rule

影响判断优先级、七类取值语义、弱证据与不确定项以 [`{skillLibraryRoot}/rules/documentation/doc-impact-after-code-change-rule.md`](../../rules/documentation/doc-impact-after-code-change-rule.md) 为准；可写子树边界以 [`{skillLibraryRoot}/rules/documentation/doc-sync-scope-rule.md`](../../rules/documentation/doc-sync-scope-rule.md) 为准。
