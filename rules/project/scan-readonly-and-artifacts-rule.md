# scan-readonly-and-artifacts-rule

## 适用主体

执行 `scan-project-by-ai`、`scan-module-by-ai`、`scan-file-by-ai` 等第二阶段 Scan 类 Skill 的 Agent。

## 规则陈述

### 索引与多模块归属（对应多路径逻辑模块）

1. **`module-index.md`**：以 **`moduleId`** 为主键维护模块条目；文档链接指向 `.ai/docs/modules/{moduleId}/` 下五件套路径。
2. **`directory-index.md`**：重要目录行须能表达与 **`moduleId`** 的关联（例如 `relatedModuleIds` 列），同一目录可对应多个 `moduleId`。
3. **`file-index.md`**：**必须**允许同一 **`filePathRelative`** 归属多个 **`moduleId`**（同一文件多行，或 `moduleIds` 列内逗号分隔多值，以项目侧模板约定为准）；合并更新时不得静默删除人类手工行。

### 扫盘产物位置

4. 上述索引与模块五件套、项目级扫盘文档等 **AI 协作文档**，在本阶段 **只能** 写入 `{projectRoot}/.ai/` 下各 Skill 明文允许的路径（见 `project-local-output-rule`）。

### 源码只读

5. 执行 Scan 类 Skill 时，对 `{projectRoot}` 下业务源码与配置为 **只读**；**不得**为配合文档生成而修改业务源码或非 `.ai/` 下的文件（与 `scan-project-by-ai` 禁止事项一致）。

### 公共库纯净

6. **不得**将具体项目的扫描结果、模块五件套、索引内容或项目私有扫描 metadata 写入 `{skillLibraryRoot}`（与 `public-skill-library-purity-rule` 一致）。

### 文档语言

7. 产出正文以 **简体中文** 描述为主；**文件路径、仓库相对路径、代码标识符、类名、方法名、API 字段名** 等与源码或仓库一致，**不翻译**（权威细则见 `language-policy-rule` 与项目侧 `language-policy.md`）。

## 允许路径

- 写入：仅各 Scan Skill「写入位置」表中列出的 `{projectRoot}/.ai/**` 路径。

## 禁止事项

- 将 `file-index` 或 `module-index` 简化为「每文件 / 每模块唯一一行且禁止重复路径」以致无法表达多模块共享文件。
- 在扫盘过程中修改 `{skillLibraryRoot}` 或业务源码树。

## 与其它 Rule 的关系

- 依赖：`project-local-output-rule`、`public-skill-library-purity-rule`、`language-policy-rule`、`module-is-logical-boundary-rule`。
- 与 `read-source-before-edit-rule` 区分：本 Rule 强调 Scan 的只读边界与 `.ai/` 产物位置。
