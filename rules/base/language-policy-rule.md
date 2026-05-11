# language-policy-rule

## 适用主体

所有阅读或编写 `{skillLibraryRoot}` 与 `{projectRoot}/.ai/` 下 Markdown 的 Agent。

## 规则陈述

- **自然语言**以简体中文为主，便于人类审查与维护。
- **代码标识符、文件路径、仓库路径、类名、方法名、API 字段名、Skill 文件名、Rule 文件名**保持英文或与源码一致，**不翻译**。
- 技术术语可采用「中文说明 + 英文术语」混写（例如「执行 `load-skill-library` Skill」）。

## 权威来源

项目侧具体约定以 `{projectRoot}/.ai/config/language-policy.md` 为准。若该文件为 `draft` 且 `reviewedByHuman: false`，Agent 仍须遵守本 Rule 的上述底线，并在就绪检查中引导人工确认该文件。

## 与其它 Rule 的关系

- 不改变 `source-of-truth-rule`：自然语言文档永远从属于源码事实。
- `.ai/config/language-policy.md` 的 YAML front matter 须同时满足 `frontmatter-format-rule`（合法 YAML、`reviewedByHuman` 为布尔值等）。
