---
generatedAt: {{GENERATED_AT_ISO}}
projectSlug: {{PROJECT_NAME_OR_SLUG}}
---

# SKILLKIT_LINK

本文件描述从 `{projectRoot}` 到公共 Skill 库 `{skillLibraryRoot}` 的链接信息，供 Agent 解析路径。

## 字段（由执行 `initialize-project-ai-context` 的 Agent 填写）

| 字段名 | 值 | 说明 |
|--------|-----|------|
| `skillLibraryRootRelative` | `{{SKILLKIT_RELATIVE_PATH}}` | 从 `{projectRoot}` 到 `{skillLibraryRoot}` 的相对路径，使用正斜杠，例如 `../HACF`。不得包含盘符绝对路径。 |
| `entryMd` | `{skillLibraryRootRelative}/ENTRY.md` | 公共库入口文件。 |
| `versionMd` | `{skillLibraryRootRelative}/VERSION.md` | 公共库版本文件。 |
| `bootstrapLoadSkill` | `{skillLibraryRootRelative}/skills/bootstrap/load-skill-library.md` | 重新执行接入流程时使用。 |

## 解析约定

1. 将 `{projectRoot}` 视为本文件所在目录向上直到项目根（`.ai/entry/` 的父的父）。
2. 将 `{skillLibraryRoot}` 解析为 `{projectRoot}` 与 `skillLibraryRootRelative` 拼接后的规范化路径。
3. 验证 `{skillLibraryRoot}/ENTRY.md` 与 `{skillLibraryRoot}/VERSION.md` 可读；若不存在，本链接无效，须由人类修正 `skillLibraryRootRelative` 后重新执行 Bootstrap。
