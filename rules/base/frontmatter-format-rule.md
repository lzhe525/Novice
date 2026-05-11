# frontmatter-format-rule

## 适用主体

在 `{projectRoot}/.ai/**` 或公共库内编写、生成 Markdown 文档的 Agent。

## 规则陈述

凡由 Agent 生成或更新的项目侧 Markdown 文档，若包含 **YAML front matter**，必须符合合法 YAML 语法，并满足下列格式约定，以便人类与工具稳定解析。

## 格式约定

1. **分隔符**：front matter 必须位于文件开头，由第一行 `---` 与第二段闭合行 `---` 包裹；闭合后空一行再接正文标题。
2. **冒号后空格**：键值使用 `key: value` 形式，`:` 后必须至少一个空格，例如 `lastReviewedAt: 2026-05-11`，禁止 `key:value`。
3. **布尔与时间**：布尔使用 JSON 风格小写 `true` / `false`；日期使用 `YYYY-MM-DD` 或 ISO8601 字符串，值前仍须空格。
4. **字符串**：含 `:`、`#`、`[` 等 YAML 特殊字符时，使用双引号包裹整段字符串。
5. **缩进**：YAML 块内使用空格缩进，**禁止**使用 Tab 作为缩进字符。
6. **编码**：文件使用 UTF-8；避免 BOM。

## 允许路径

- `{projectRoot}/.ai/**/*.md` 中由 Scan 或 Bootstrap Skill 允许写入的文件。
- `{skillLibraryRoot}/**/*.md` 中由人类维护的公共文档。

## 禁止路径

- 在 `{skillLibraryRoot}` 写入任何**具体业务项目**的扫描 metadata 或项目私有字段（违反 `public-skill-library-purity-rule`）。

## 与其它 Rule 的关系

- 与 `structured-doc-writing-rule` 共同保证文档可解析、可审查。
- Scan 产出须同时遵守 `source-of-truth-rule`：front matter 不得声称与源码不一致的「事实」。
