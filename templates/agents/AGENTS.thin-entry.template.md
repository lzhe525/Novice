# AGENTS.md

本项目使用 HACF（Human-AI Collaboration Framework）公共 Skill 文档库做 Agent-first 协作约定。

## 必读入口

所有 Agent 在执行任务前必须先阅读：

- `.ai/entry/AI_ENTRY.md`

## Skill 路由预检

涉及代码理解/修改、调试、规划、测试、文档、Pattern 或项目本地 Skill 的任务，开始前须完成 Skill 路由预检（见 `.ai/entry/AI_ENTRY.md` 或公共库 `skills/router/select-skill-for-task.md`）。仅对 `available` 且文件存在的 Skill 必须先读再执行；项目本地 Skill 优先。

## 写入范围

所有 AI 协作产物只能写入项目根下的 `.ai/` 目录；不得在仓库其它路径散落协作文档、扫描结果或项目私有 Skill 正文。

## 公共库位置

公共库与项目平行放置时，链接信息见 `.ai/entry/SKILLKIT_LINK.md`。
