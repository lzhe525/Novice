# onboarding-stepwise-rule

## 适用主体

执行 `guide-project-onboarding` 或 `recommend-next-onboarding-step` 的 Agent，以及按 HACF Bootstrap 引导协作的人类维护者。

## 规则陈述

### 分阶段与范围

1. **一次只推进一个阶段**：每轮对话或单次 Skill 激活中，仅处理 `onboarding-status.md` 所声明的**当前** `currentPhase` 对应工作；不得在同一轮内擅自跨越下一阶段，除非人类明确要求「跳过」且已在状态中记录风险。
2. **不进入 Pattern / Develop**：本组 Skill 仅服务「项目接入与 Bootstrap 落地」；不得启动 `skills/pattern/**`、`skills/develop/**` 或受控改源码流程。
3. **不得修改业务源码**：除既有 Bootstrap 允许的 `{projectRoot}/AGENTS.md` 外，不得对源码树做任何写入。

### 输出形态（`guide-project-onboarding`）

4. 每次面向用户的回复**必须**包含以下五段（顺序可固定为下列顺序），且使用中文：
   - 当前阶段；
   - 已完成事项；
   - 当前需要用户确认的问题；
   - 推荐回复格式；
   - 下一步。

### 写入边界

5. 项目侧 AI 协作产物默认仅写入 `{projectRoot}/.ai/`；Bootstrap 允许的 `AGENTS.md` 与子 Skill 明示路径除外。
6. **禁止**写入或修改 `{skillLibraryRoot}/**`（公共 HACF 库保持纯净）。

### 与其它 Rule 的关系

- 须遵守 `rules/base/project-local-output-rule.md`。
- 须遵守 `rules/base/public-skill-library-purity-rule.md`。
- 须遵守 `rules/base/frontmatter-format-rule.md`（凡生成 YAML front matter）。
- 人工门控与禁止代确认事项见 `rules/bootstrap/onboarding-human-confirmation-rule.md`。
