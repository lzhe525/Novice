# agent-default-entry-rule

## 适用主体

执行或编排 `skills/bootstrap/ensure-agent-default-entry.md`、阅读 `.ai/state/agent-entry-status.md` 或 `.ai/config/agent-entry-policy.md` 的 Agent 与人类维护者。

## 规则陈述

1. **目的**：在业务项目 `{projectRoot}` 内为不同 Agent 工具链维护**极薄**默认入口（`AGENTS.md`、`CLAUDE.md`、Cursor Project Rule 等），使其统一指向 `.ai/entry/AI_ENTRY.md`（或经 `AGENTS.md` 再指向该路径），避免新对话未显式提 HACF 时绕过框架。
2. **公共库只读**：不得修改 `{skillLibraryRoot}/**`；不得将项目私有扫描结果、密钥或未脱敏内容写回公共库。
3. **项目源码**：不得修改业务源码；允许触及的根文件仅限各 Skill「写入位置」明示的极薄入口与 `.ai/**` 下本规则相关配置/状态。
4. **编排关系**：`ensure-agent-default-entry` 对 `AGENTS.md` 的处理须通过编排 `create-or-update-agents-md` 第 6 节完成，不重复其受控区块逻辑。
5. **入口发现**：执行任意 `{skillLibraryRoot}/skills/**` 下 Skill 前，若工作区为已接入 HACF 的业务项目，建议先只读确认根入口与 `.ai/entry/AI_ENTRY.md` 可达；若缺失则优先 Bootstrap 系列 Skill 而非在随机路径新建长篇说明。
6. **与 Router 一致**：能力登记见 `{skillLibraryRoot}/routers/SKILL_ROUTER.md` 中 Bootstrap 表 `ensure-agent-default-entry` 一行。

## 须遵守的关联 Rule

- `rules/bootstrap/agent-entry-thin-adapter-rule.md`
- `rules/base/project-local-output-rule.md`
- `rules/base/single-ai-context-directory-rule.md`
- `rules/base/public-skill-library-purity-rule.md`

## 禁止事项

- 在 Agent 专属入口文件中复制完整 HACF 规则、Skill 全文或项目扫盘结果。
- 整文件覆盖用户已有 `AGENTS.md`、`CLAUDE.md`、Cursor Rule 中非 HACF 受控区块内容。

## 与其它 Rule 的关系

- 与 `onboarding-stepwise-rule` 互补：引导接入阶段可包含「安装默认入口」一步。
- 与 `project-ai-health-rule` 互补：健康检查 H1 可依据 `agent-entry-policy.md` 校验可选入口。
