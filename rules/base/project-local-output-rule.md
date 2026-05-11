# project-local-output-rule

## 适用主体

在 `{projectRoot}` 中生成或更新 AI 协作文档、状态、配置草稿的 Agent。

## 规则陈述

凡属于「AI 协作产物」（说明性文档、协作状态、为 Agent 准备的配置草稿、未来扫描生成的索引与报告等），**默认**只能写入：

`{projectRoot}/.ai/`

Bootstrap MVP 阶段**已定义**、允许创建或更新的项目侧文件限于：

- `{projectRoot}/AGENTS.md`（极薄入口，见 `create-or-update-agents-md` Skill）
- `{projectRoot}/.ai/entry/AI_ENTRY.md`
- `{projectRoot}/.ai/entry/SKILLKIT_LINK.md`
- `{projectRoot}/.ai/state/skillkit-status.md`
- `{projectRoot}/.ai/state/readiness.md`
- `{projectRoot}/.ai/config/language-policy.md`

不在上列且非用户显式要求的源码修改，**不得**新建。

## 禁止事项

- 以「方便 Agent」为由在 `{projectRoot}` 根或 `src/` 下新增与上表同类的协作文档。
- 在 MVP 阶段创建 `.ai/docs/`、`.ai/indexes/` 等目录（除非后续版本 Skill 明确引入）。

## 与其它 Rule 的关系

- 依赖 `single-ai-context-directory-rule` 的单根目录原则。
- 与 `read-source-before-edit-rule` 区分：后者针对**源码**读取；本 Rule 针对**协作文档写入位置**。
