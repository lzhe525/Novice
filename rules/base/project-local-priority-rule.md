# project-local-priority-rule

## 适用主体

在 `{projectRoot}` 中选择读取哪一份 Skill、Rule 或上下文的 Agent。

## 规则陈述

当同一主题同时存在「项目本地」与「公共库」两份材料时，**优先使用项目本地**更具体、更贴近当前仓库的版本。

## 优先级链（从高到低）

1. `{projectRoot}/.ai/skills/project-local/` 下的项目本地 Skill（若存在；MVP 可能尚未创建该目录）。
2. `{projectRoot}/.ai/rules/project-local/` 下的项目本地 Rule（若存在）。
3. `{projectRoot}/.ai/config/` 下的项目配置。
4. `{projectRoot}/.ai/docs/`、`{projectRoot}/.ai/indexes/`（后续阶段；MVP 不要求存在）。
5. `{skillLibraryRoot}/skills/` 下的公共 Skill。
6. `{skillLibraryRoot}/rules/` 下的公共 Rule。

## 禁止事项

- **不得**虚构「项目本地已存在」的 Skill 或 Rule 路径；若目录不存在，应回退到公共库并如实说明。
- **不得**在公共库中写入仅适用于当前项目的 overrides 来「冒充」项目本地优先。

## 与其它 Rule 的关系

- 与 `public-skill-library-purity-rule` 共同保证：项目细节沉淀在项目侧，公共库保持通用。
