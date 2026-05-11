# read-source-before-edit-rule

## 适用主体

任何准备修改 `{projectRoot}` 内**源码或非 Markdown 配置**的 Agent。

## 规则陈述

在编辑具体源文件（如 `.cs`、`.ts`、`.java`、`.json` 配置等）之前，Agent **必须**使用读取工具打开将要修改的**真实文件**，基于当前内容规划补丁。

## 例外

- 仅创建或更新本 Bootstrap 流程明确允许的 Markdown 路径（`AGENTS.md` 与 `.ai/` 下 entry/state/config）时，按对应 Skill 步骤执行即可。
- 纯文档任务且用户明确禁止读源码时，须在回复中声明风险并避免推断代码细节。

## 禁止事项

- 猜测未读文件的内容结构或符号名并直接写入补丁。

## 与其它 Rule 的关系

- 落实 `source-of-truth-rule` 的操作性要求。
