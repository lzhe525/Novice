# Skill：guide-project-onboarding

## 1. 目的

作为 HACF **项目接入总向导**，在 `{projectRoot}` 内以**分阶段、可人工门控**的方式，引导开发者完成：确认根目录与公共库路径、执行或引导执行 `load-skill-library`、校验 `AGENTS.md` 与 `.ai/entry/`、执行就绪检查、收集项目基础信息与约束类确认，并维护 `.ai/state/onboarding-*.md`。**不**进入 Scan / Pattern / Develop 能力；**不**批量创建 `check-project-readiness` 所列「后续阶段勿自动创建」的全量配置壳文件（如 `project-profile.md`、`hard-constraints.md` 等）。

## 2. 适用场景

- 业务仓库**首次**引入 HACF，人类希望逐步确认而非仅一次性 `load-skill-library`。
- 接入中途卡住（路径、语言策略未确认），需从当前阶段恢复推进。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根绝对路径；须由用户显式给出或从工作区推断并经**用户确认**。 |
| `{skillLibraryRoot}` | 公共库根；若 Agent 从本 Skill 磁盘路径反推，则本文件路径为 `{skillLibraryRoot}/skills/bootstrap/guide-project-onboarding.md`，连续三次父目录解析（去掉文件名、`bootstrap`、`skills`）得到 `{skillLibraryRoot}`。 |
| `{skillLibraryRelativePath}` | 从 `{projectRoot}` 指向 `{skillLibraryRoot}` 的相对路径（正斜杠）；在 `phase_confirm_roots` 阶段由人类确认，**必填**后方可进入 `phase_run_load_skill_library`。 |
| `{projectNameOrSlug}` | 可选；缺省为 `{projectRoot}` 最后一级目录名，用于模板占位 `{{PROJECT_NAME_OR_SLUG}}`。 |

## 4. 输出

| 路径 | 说明 |
|------|------|
| `{projectRoot}/.ai/state/onboarding-status.md` | 阶段、布尔门控、摘要；不存在时从模板创建或按阶段合并更新 front matter 与正文。 |
| `{projectRoot}/.ai/state/onboarding-checklist.md` | 与模板一致的勾选清单；在 `phase_sync_onboarding_artifacts` 及之后按需更新勾选。 |
| `{projectRoot}/.ai/state/onboarding-questions.md` | 待确认问题与人类答复摘要；在 `phase_confirm_project_basics` / `phase_confirm_constraints_bundle` 及同步阶段更新。 |
| 子 Skill 产出 | 当本向导编排 `load-skill-library` 或其子 Skill 时，另见各子 Skill「输出」节（仍限于允许路径）。 |
| Agent 回复 | **每次**须含「执行原则 3」规定的五段结构（中文）。 |

## 5. 前置条件

- Agent 对 `{projectRoot}/.ai/state/` 有创建与写入权限（目录不存在时须先创建）。
- 下列模板存在于公共库（用于实例化或对照合并）：
  - `{skillLibraryRoot}/templates/state/onboarding-status.template.md`
  - `{skillLibraryRoot}/templates/state/onboarding-checklist.template.md`
  - `{skillLibraryRoot}/templates/state/onboarding-questions.template.md`

## 6. 执行原则

1. **一次只推进一个阶段**：本轮仅处理「当前 `currentPhase`」对应步骤；完成后将 `currentPhase` 更新为**下一**阶段枚举并停止，除非该阶段无需人类输入且人类已在上文明确授权「连续执行」（仍不得跨 Pattern/Develop）。
2. **需要人工确认时必须停止**：不得猜测 `{projectRoot}`、相对路径、硬约束、高风险区、目录策略等；不得伪造 `language-policy.md` 的 `reviewedByHuman: true`。
3. **每次面向用户的回复必须包含**（五段，中文）：
   - **当前阶段**（含 `currentPhase` 枚举值与中文说明）；
   - **已完成事项**（事实列表，可引用已存在文件路径）；
   - **当前需要用户确认的问题**（若无则写「无」，并说明为何可直接进入下一阶段）；
   - **推荐回复格式**（可复制示例）；
   - **下一步**（例如：「请回复确认后再次激活本 Skill」或「请让 Agent 只读执行 `skills/bootstrap/recommend-next-onboarding-step.md`」）。
4. **项目产物**仅写入 `{projectRoot}/.ai/**` 及子 Skill 允许的 `{projectRoot}/AGENTS.md`；**不得**写入 `{skillLibraryRoot}/**`。
5. **不得修改业务源码**（`AGENTS.md` 除外，且须通过 `create-or-update-agents-md` 或 `load-skill-library` 编排）。
6. **不得进入** `skills/scan/**`、`skills/pattern/**`、`skills/develop/**`。

### 6.1 建议只读前置 Rule

执行写入前阅读（不写入）：

- `{skillLibraryRoot}/rules/bootstrap/onboarding-stepwise-rule.md`
- `{skillLibraryRoot}/rules/bootstrap/onboarding-human-confirmation-rule.md`
- `{skillLibraryRoot}/rules/base/project-local-output-rule.md`
- `{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`

## 7. 阶段模型（`currentPhase`）

| `currentPhase` 枚举 | 对应向导目标 |
|---------------------|----------------|
| `phase_confirm_roots` | 确认 `{projectRoot}` 与 `{skillLibraryRelativePath}` |
| `phase_run_load_skill_library` | 执行或引导执行 `load-skill-library` |
| `phase_verify_agents` | 校验根目录 `AGENTS.md` 与 `AI_ENTRY.md` 引用关系 |
| `phase_verify_ai_tree` | 校验 `.ai/` 存在 |
| `phase_verify_entry` | 校验 `.ai/entry/` 与 `AI_ENTRY.md`、`SKILLKIT_LINK.md` |
| `phase_run_readiness` | 执行或引导执行 `check-project-readiness` |
| `phase_confirm_project_basics` | 人类确认项目基础信息（写入 `onboarding-questions.md` 第一节摘要） |
| `phase_confirm_constraints_bundle` | 人类确认硬约束、高风险区、源码/忽略/只读目录与语言策略（第二节摘要） |
| `phase_sync_onboarding_artifacts` | 合并更新 `onboarding-status.md`、`onboarding-checklist.md`、`onboarding-questions.md` 与事实一致 |
| `phase_handoff_next` | 告知下一步应执行的 Skill（可指向 `recommend-next-onboarding-step`） |

**阶段顺序**：上表自上而下；仅当上一阶段事实满足或人类已明确确认后，才更新 `currentPhase` 为下一阶段。

## 8. 执行步骤（按当前阶段仅执行一步）

0. **解析 `{skillLibraryRoot}`**：同 `load-skill-library` 的路径反推规则；失败则停止，不写入 `{projectRoot}`。
1. **读取或初始化 `onboarding-status.md`**：
   - 若 `{projectRoot}/.ai/state/onboarding-status.md` 不存在：不得在未由人类确认 `{projectRoot}` 与 `{skillLibraryRelativePath}` 前创建。若文件不存在且人类尚未确认：将「逻辑上的」`currentPhase` 视为 `phase_confirm_roots`，不在此步写入磁盘。
   - 若已存在：解析 YAML front matter 得到 `currentPhase` 与各布尔字段。
2. **`phase_confirm_roots`**：
   - 若 `projectRootConfirmed` 与 `skillLibraryRelativePathConfirmed` 均为 `true`：若 `onboarding-status.md` 尚不存在，则创建目录 `{projectRoot}/.ai/state/`（及必要的 `.ai/` 父目录）后，自模板实例化该文件；占位符：`{{GENERATED_AT_ISO}}`、`{{LAST_UPDATED_AT_ISO}}` 均为当前 UTC ISO8601；`{{PROJECT_NAME_OR_SLUG}}` 来自输入或目录名；写入已确认布尔。将 `currentPhase` 更新为 `phase_run_load_skill_library`，写回；**本轮结束**（五段输出）；**不在同一轮**执行 load。
   - 否则：列出须人类确认的两项；停止等待；**不得**进入 load。
3. **`phase_run_load_skill_library`**：
   - 若 `SKILLKIT_LINK.md` 与 `readiness.md` 已存在且人类声明已手动完成 load：将 `loadSkillLibraryDone` 置 `true`，`currentPhase` 更新为 `phase_verify_agents`，写回状态；**本轮结束**。
   - 否则：打开 `{skillLibraryRoot}/skills/bootstrap/load-skill-library.md`，严格按其**第 6 节**执行；失败则五段输出说明阻塞，**不**强行推进 `currentPhase`。
   - 若 load 成功：将 `loadSkillLibraryDone: true`，`currentPhase` 更新为 `phase_verify_agents`，写回状态；**本轮结束**（**不在同一轮**执行 `phase_verify_agents`）。
4. **`phase_verify_agents`**：验证 `{projectRoot}/AGENTS.md` 存在且正文含 `.ai/entry/AI_ENTRY.md` 或 `AI_ENTRY.md` 子串；通过则 `agentsMdVerified: true`，`currentPhase` → `phase_verify_ai_tree`，写回状态，**本轮结束**；否则五段输出并建议 `create-or-update-agents-md` 或重跑 load，**停止**且不推进 `currentPhase`。
5. **`phase_verify_ai_tree`**：验证 `{projectRoot}/.ai/` 目录存在；通过则 `aiDirectoryVerified: true`，`currentPhase` → `phase_verify_entry`，写回状态，**本轮结束**；否则停止并说明须先完成 initialize/load。
6. **`phase_verify_entry`**：验证 `{projectRoot}/.ai/entry/AI_ENTRY.md` 与 `SKILLKIT_LINK.md` 存在；通过则 `entryDirectoryVerified: true`，`currentPhase` → `phase_run_readiness`，写回状态，**本轮结束**；否则停止。
7. **`phase_run_readiness`**：打开 `{skillLibraryRoot}/skills/bootstrap/check-project-readiness.md`，严格按其第 6 节执行；完成后 `readinessChecked: true`，`currentPhase` → `phase_confirm_project_basics`，写回状态，**本轮结束**。若 R1–R6 存在 `fail`，五段输出须列出人类待办（不得声称语言策略已确认）。
8. **`phase_confirm_project_basics`**：依据 `{skillLibraryRoot}/templates/state/onboarding-questions.template.md` **第一节**，向用户展示问题；若人类**尚未**在本轮提供答复：**停止**（可仅创建或刷新 `onboarding-questions.md` 骨架，不得编造摘要）；收到答复后：将摘要写入 `onboarding-questions.md` 对应节，置 `humanBasicsConfirmed: true`，`currentPhase` → `phase_confirm_constraints_bundle`，写回状态，**本轮结束**。
9. **`phase_confirm_constraints_bundle`**：依据模板**第二节**逐项确认；语言策略须指向 `.ai/config/language-policy.md` 的人类确认流程，**禁止**代写 `reviewedByHuman: true`；若人类尚未答复：**停止**；完成后：`humanConstraintsBundleConfirmed: true`，`currentPhase` → `phase_sync_onboarding_artifacts`，写回状态，**本轮结束**。
10. **`phase_sync_onboarding_artifacts`**：
    - 若 `onboarding-checklist.md` 或 `onboarding-questions.md` 缺失：自模板实例化（替换占位符同步骤 1）。
    - 合并更新三份状态文件：`lastUpdatedAt` 刷新；`onboarding-status.md` 中各布尔与 `readiness.md` / 仓库事实对齐；`onboarding-checklist.md` 仅对**已满足事实**的步骤勾选 `- [x]`。
    - 置 `onboardingArtifactsSynced: true`，`currentPhase` → `phase_handoff_next`，写回状态，**本轮结束**（**不在同一轮**执行交接文案以外的重复 load）。
11. **`phase_handoff_next`**：五段输出中明确下一步：建议人类或 Agent 打开 `{skillLibraryRoot}/skills/bootstrap/recommend-next-onboarding-step.md` 执行只读推荐；置 `handoffDone: true`；`currentPhase` 保持 `phase_handoff_next`；正文说明后续 Scan / Pattern 不在本 Skill 范围。写回状态后**本轮结束**。

**模板占位符约定**（实例化时）：

- `{{GENERATED_AT_ISO}}`、`{{LAST_UPDATED_AT_ISO}}`：当前 UTC ISO8601。
- `{{PROJECT_NAME_OR_SLUG}}`：输入或目录名。

## 9. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/state/onboarding-status.md` |
| `{projectRoot}/.ai/state/onboarding-checklist.md` |
| `{projectRoot}/.ai/state/onboarding-questions.md` |
| 经由子 Skill 允许的 `{projectRoot}/AGENTS.md`、`{projectRoot}/.ai/entry/**`、`{projectRoot}/.ai/state/readiness.md`、`{projectRoot}/.ai/state/skillkit-status.md`、`{projectRoot}/.ai/config/language-policy.md`（仅当编排 `load-skill-library` / `check-project-readiness` 等时） |

| 禁止 |
|------|
| `{skillLibraryRoot}/**` 任意写入 |
| `{projectRoot}` 下除 `AGENTS.md` 外的源码与构建产物修改 |
| 未经专门 Skill 许可创建 `.ai/docs/**`、`.ai/indexes/**`、`.ai/pattern-packs/**` 等 |
| 因本向导而批量创建 `project-profile.md`、`hard-constraints.md` 等（除非未来独立 Skill 明确允许） |

## 10. 禁止事项

- **禁止**单次激活内从 `phase_confirm_roots` 跳跃至 `phase_confirm_constraints_bundle` 等跨阶段行为（人类书面要求「跳到某阶段」时，须在 `onboarding-status.md` 正文「备注」记录例外原因）。
- **禁止**将 checklist 未满足项标为已完成。
- **禁止**进入 Pattern 或 Develop 流程。

## 11. 完成标准

- [ ] `onboarding-status.md` 存在且 `currentPhase` 与布尔字段反映真实进度。
- [ ] `phase_handoff_next` 已完成时，用户被告知可使用 `recommend-next-onboarding-step` 获取下一步建议。
- [ ] 每一次用户可见回复均含五段结构。

## 12. 失败处理

| 情况 | 处理 |
|------|------|
| 模板缺失 | 中止写入；列出缺失模板路径；五段输出。 |
| 无法解析 `onboarding-status.md` YAML | 五段输出说明；建议人类参照 `frontmatter-format-rule` 修复；本轮不推进阶段。 |
| 子 Skill 中止 | 保持 `currentPhase` 在对应阶段或回退至上一稳定阶段；列出已写入路径与待办。 |
