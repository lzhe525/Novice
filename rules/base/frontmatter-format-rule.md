# frontmatter-format-rule

## 适用主体

所有在 `{skillLibraryRoot}` 或 `{projectRoot}/.ai/` 下**创建或编辑**带 YAML front matter 的 Markdown 文件的 Agent；以及维护 HACF 模板、Skill 产出物的人类维护者。

「HACF 管理的 Markdown」在本 MVP 中指：公共库内由本仓库版本控制的 Markdown，以及 Bootstrap / 就绪检查等 Skill **按模板写入**的项目侧 `.ai/**/*.md`（含 `readiness.md`、`skillkit-status.md`、`language-policy.md`、`SKILLKIT_LINK.md`、`AI_ENTRY.md` 等）。

## 规则陈述

### 1. 合法 YAML front matter

凡包含 metadata 的上述 Markdown 文件，**必须**在正文首段使用标准 YAML front matter：以第一行 `---` 开始、以单独一行的 `---` 结束，且中间内容可被常见 YAML 1.2 解析器（如 `yaml.safe_load`）**无报错**解析为映射（对象）。

### 2. 冒号与空格

每个键值对必须使用 **`:` + 单个空格** 分隔键与值，即键名后写 `: `（冒号后**必须**有空格）。  
**禁止**使用 `key:value` 粘连写法（易导致扫描器报错或歧义）。

### 3. 日期与时间

- **建议**：日期类字段使用**引号包裹的字符串**，例如 `lastReviewedAt: "2026-05-11"`，避免实现差异将未加引号的日期解析为日期类型后又序列化不一致。
- UTC 时间戳（如 `generatedAt`、`linkedAt`、`lastCheck`）在模板中可为 ISO8601 字符串；若存在解析风险，同样建议使用引号：`"2026-05-11T12:00:00Z"`。

### 4. 布尔字段

- `reviewedByHuman` 等必须为 YAML **原生布尔** `true` 或 `false`。
- **禁止**写成字符串形式的 `"true"` / `"false"` / `"yes"` / `"no"`，以免就绪检查或脚本误判。

### 5. 状态类字段的约定枚举

下列字段取值须与当前 HACF MVP 模板及 Skill 语义一致；**不得**发明未在框架中定义的状态值（若需扩展，应先改模板与本 Rule 再落盘）。

| 字段 | 允许取值 | 说明 |
|------|----------|------|
| `language-policy.md`：`status` | `draft`、`active` | 见 `templates/config/language-policy.template.md` 与「Human confirmation」节 |
| `language-policy.md`：`confidence` | `medium`、`confirmed` | 模板草稿为 `medium`；人工确认后为 `confirmed` |
| `language-policy.md`：`generatedBy` | `ai`、`ai-assisted` | 见模板与「Human confirmation」节 |
| `skillkit-status.md`、`readiness.md`：`projectState` | `loaded`、`configuring`、`blocked`、`constraints_ready` | 含义见 `skills/bootstrap/check-project-readiness.md` 与 `skillkit-status` 模板 |

其它文件中的同类字段，以对应模板或维护该文件的 Skill 正文为准；若未定义枚举，使用小写英文 slug 或模板占位符，且仍须满足本节 1–4 条。

### 6. 修改后的校验义务

Agent 在**任意**修改上述文件的 YAML front matter 后，**必须**重新验证该 front matter 片段可被解析（例如用项目或环境内可用的 YAML 加载器对 `---` 之间内容做一次 `safe_load`），确认无报错后再提交或宣称检查通过。

## 禁止事项

- 在 front matter 中使用 Tab 缩进混用不当、未闭合引号、或依赖解析器专有扩展导致可移植性变差。
- 在明知 YAML 无效的情况下仍写入并依赖后续人工修复（除非 Skill 明确允许「草稿待人类补全」且已在回复中说明风险）。

## 与其它 Rule 的关系

- 与 `language-policy-rule` 互补：后者管自然语言与术语，本条管 **metadata 机器可读性**。
- 与 `skills/bootstrap/check-project-readiness.md` 中 R6 一致：front matter 无法解析时，视为未满足人工门控或格式未就绪，须按本条修正后再检。
