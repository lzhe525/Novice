# skill-routing-preflight-rule

## 适用主体

在已接入或正在接入 HACF 的 `{projectRoot}` 中执行协作任务的 Agent。

## 规则陈述

1. **预检义务**：除下列豁免外，凡涉及**代码理解、代码修改、调试、规划、测试、文档、Pattern、项目本地 Skill** 的任务，在修改业务源码、批量写入 `.ai/` 文档（本规则允许的 `last-skill-routing.md` 除外）或输出长篇实施方案之前，**必须先**完成 Skill 路由预检。
2. **预检执行方式**：打开并严格按 `{skillLibraryRoot}/skills/router/select-skill-for-task.md` **第 6 节「执行步骤」** 完成决策链；或在对话中给出与该节等价的结构化输出（含 `decision`、`selectedSkillPath`、`evidence`）。
3. **invoke 后续**：当预检结论为 `decision: invoke` 且目标 Skill 文件存在时，Agent **必须先全文读取**该 Skill，再进入任务主体；**不得**跳过 Skill 正文直接改码。
4. **产出位置**：预检结论可在对话中给出；可选写入 `{projectRoot}/.ai/state/last-skill-routing.md`（须遵守 `project-local-output-rule`）。
5. **豁免**：
   - 纯 Bootstrap 接入，且 `{projectRoot}/.ai/` 尚未建立，任务仅限于 `skills/bootstrap/**` 中已登记的 Skill。
   - 正在执行 `select-skill-for-task` 自身。
   - 用户明确声明跳过 HACF 协作且任务与仓库无关。

## 须遵守的关联 Rule

- `{skillLibraryRoot}/rules/base/project-local-priority-rule.md`
- `{skillLibraryRoot}/rules/base/read-source-before-edit-rule.md`
- `{skillLibraryRoot}/rules/documentation/agent-context-budget-rule.md`
- `{skillLibraryRoot}/rules/routing/mandatory-skill-trigger-rule.md`
- `{skillLibraryRoot}/rules/routing/skill-route-enabled-rule.md`
- `{skillLibraryRoot}/rules/routing/new-skill-registration-rule.md`
- `{skillLibraryRoot}/rules/routing/skill-routing-validation-rule.md`

## 禁止事项

- 跳过预检对业务源码做大范围修改。
- 对 `TASK_TRIGGER_MATRIX.md` §2 中 `planned` Skill 声称「已按 Skill 执行」。
- 在预检阶段将项目私有内容写入 `{skillLibraryRoot}`。

## 与其它 Rule 的关系

- 与 `agent-default-entry-rule` 互补：极薄入口指向 `AI_ENTRY.md`，本规则约束任务开始后的第一层协作决策。
- 与 `doc-impact-after-code-change-rule` 衔接：改码后文档影响判断仍由 `check-doc-impact-after-change` 执行，但进入该 Skill 前仍须完成预检（若任务类型属于本规则适用范围）。
