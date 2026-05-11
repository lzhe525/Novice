# HACF 公共 Skill 文档库入口

HACF（Human-AI Collaboration Framework）是一个 **Agent-first** 的公共 Skill 文档库，用于帮助任意业务项目在本地建立「Agent — 项目文档 — 项目源码 — 项目本地 Skill」的协作约定。

本仓库**不是**业务代码仓库，而是可被 **Git 拉取后与业务项目平行放置**的只读参考库。业务项目侧所有 AI 协作产物必须落在项目根下的 `.ai/` 目录；项目根目录仅保留极薄的 `AGENTS.md` 作为统一入口指针。

## 推荐目录布局

将本仓库克隆到与业务项目**同级**的目录（文件夹名可自定义，例如 `HACF` 或 `skillkit`）：

```text
workspace/
  HACF/                 # 本公共库（{skillLibraryRoot}）
    ENTRY.md
    skills/
    ...
  my-project/           # 业务项目（{projectRoot}）
    AGENTS.md
    .ai/
    src/
    ...
```

路径变量约定：

- `{skillLibraryRoot}`：包含本文件 `ENTRY.md` 的目录，即公共库根目录。
- `{projectRoot}`：业务项目根目录（含 `.git`、`package.json`、`.sln` 等之一，或由开发者显式指定）。

## Agent 第一步

在需要接入框架的业务项目中，由开发者指示 Agent **只读**打开并执行：

`{skillLibraryRoot}/skills/bootstrap/load-skill-library.md`

该 Skill 会编排创建或更新 `{projectRoot}/AGENTS.md`，并在 `{projectRoot}/.ai/` 下生成入口、状态与配置的基础结构，然后引导执行就绪检查。

## 第二阶段：AI 扫盘（项目文档）

在 Bootstrap 就绪后，可由 Agent 读取 `{skillLibraryRoot}/routers/SKILL_ROUTER.md` 中 **Scan** 表，按需执行 `skills/scan/` 下 Skill。所有扫盘产出必须写入业务项目的 `.ai/docs/`，**不得**写回公共库。写入前建议阅读 `rules/base/frontmatter-format-rule.md` 与 `rules/documentation/` 下文档类 Rule。

## 版本信息

公共库版本见同目录下的 [VERSION.md](VERSION.md)，变更记录见 [CHANGELOG.md](CHANGELOG.md)。
