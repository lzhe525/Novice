# hacf-upgrade-no-overwrite-rule

## 适用主体

执行 `skills/bootstrap/apply-hacf-local-upgrade.md` 的 Agent。

## 规则陈述

### 禁止覆盖（已存在则跳过写入，并记入冲突报告）

下列路径（相对 `{projectRoot}`）**无论是否为空壳**，均**不得**被 `apply-hacf-local-upgrade` **覆盖或删除**：

- `.ai/config/hard-constraints.md`
- `.ai/config/danger-zones.md`
- `.ai/docs/**`
- `.ai/pattern-packs/**`
- `.ai/skills/project-local/**`

### 框架补缺原则

1. **缺失创建**：对计划中允许自动创建且不在上表内的路径，仅当文件**不存在**时，可从 `{skillLibraryRoot}/templates/` 等公共库模板实例化创建。
2. **已存在则冲突记录**：目标路径已存在且与「从公共模板全新实例化」不一致时，**不得覆盖**；须在 `.ai/reports/hacf-version-upgrade-conflicts.md` 记录路径、原因与建议人工动作。
3. **目录**：可创建 `.ai/reports/` 等框架所需父目录，但不得借此向禁止树内批量灌入内容。

## 须同时遵守的其它 Rule

- `rules/base/public-skill-library-purity-rule.md`
- `rules/base/project-local-output-rule.md`

## 禁止事项

- 以「升级」为由合并或改写 `hard-constraints.md`、`danger-zones.md` 正文。
- 向 `.ai/docs/**`、`.ai/pattern-packs/**`、`.ai/skills/project-local/**` 写入公共库模板或覆盖项目文档。
