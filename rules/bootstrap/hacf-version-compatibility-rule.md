# hacf-version-compatibility-rule

## 适用主体

执行或编排 `skills/bootstrap/check-hacf-version-compatibility.md` 的 Agent，以及阅读 `{projectRoot}/.ai/state/hacf-version-status.md` 的人类维护者。

## 规则陈述

1. **事实源**：公共框架版本以 `{skillLibraryRoot}/VERSION.md` 首行为准；能力矩阵以 `{skillLibraryRoot}/capabilities/hacf-capabilities.yml` 为准；项目本地记录的框架同步版本以 `{projectRoot}/.ai/state/skillkit-status.md` 的 YAML 键 `localFrameworkVersion` 为准（若缺失则按 `check-hacf-version-compatibility` Skill 文档定义回退，不得臆造）。
2. **前置**：在声称「已做兼容检查」前，须已读取 `VERSION.md`、`hacf-capabilities.yml`、`.ai/entry/SKILLKIT_LINK.md`；若 `hacf-capabilities.yml` 缺失，兼容状态须为 `unknown`，且不得继续 `plan-hacf-local-upgrade` / `apply-hacf-local-upgrade`。
3. **只读公共库**：检查过程**不得**写入或修改 `{skillLibraryRoot}/**`。
4. **状态语义**（`compatibilityStatus`）以 `check-hacf-version-compatibility` Skill 第 6 节定义为准；本 Rule 不重复枚举，以免漂移。
5. **与 load 的关系**：`load-skill-library` 编排完成后应写入或刷新 `hacf-version-status.md`（由检查 Skill 执行），使新项目具备版本事实。

## 须同时遵守的其它 Rule

- `rules/base/public-skill-library-purity-rule.md`
- `rules/base/project-local-output-rule.md`
- `rules/base/frontmatter-format-rule.md`

## 禁止事项

- 伪造 `localFrameworkVersion` 与公共版本一致而未实际读取文件。
- 在 `incompatible` 或 `unknown` 时静默进入自动升级执行阶段（仍须先计划与人类确认，见 `hacf-local-upgrade-rule.md`）。
