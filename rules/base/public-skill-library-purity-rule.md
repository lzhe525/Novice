# public-skill-library-purity-rule

## 适用主体

- 在 `{skillLibraryRoot}` 或 `{projectRoot}` 中工作的 Agent。
- 人类维护者将本公共库作为上游只读参考时。

## 规则陈述

公共 Skill 库（`{skillLibraryRoot}`）**只存放**通用 Skill、Rule、Router、Template 等框架内容。**不得**在其中保存或提交任何业务项目派生内容。

## 禁止写入 `{skillLibraryRoot}` 的内容（示例）

- 某具体项目的扫描结果、索引、报告。
- 项目私有配置、密钥、环境变量值。
- 仅适用于单一业务仓库的 Skill 正文（应放在 `{projectRoot}/.ai/skills/project-local/` 等路径，由后续能力定义）。
- 从业务项目复制粘贴的源码片段作为「事实数据」长期存放。

## 允许路径

- 在 `{projectRoot}` 内按 Bootstrap Skill 写入 `.ai/` 与根目录 `AGENTS.md`（仅限 Skill 明文允许的文件）。
- 在 `{skillLibraryRoot}` 内仅进行**本框架仓库自身**的版本维护提交（人类维护者），Agent 执行 load 流程时**不得修改** `{skillLibraryRoot}` 下任何文件。

## 与其它 Rule 的关系

- 与 `project-local-output-rule` 互补：项目派生物只进 `.ai/`；公共库不进项目派生物。
- Agent 在写文件前应先满足 `single-ai-context-directory-rule`（针对项目侧）。
