# pattern-pack-structure-rule

## 适用主体

执行 `extract-code-pattern`、编辑 `{projectRoot}/.ai/pattern-packs/{codeTypeId}/**` 下 Markdown 的 Agent 与人类维护者。

## 规则陈述

### 通用 front matter（四个 Pack 文件均需）

1. 所有 Pattern Pack 内的 Markdown（`pattern.md`、`validator.md`、`examples.md`、`docs.md`）**必须**在文件开头包含合法 YAML front matter（`frontmatter-format-rule.md`）。
2. **必备键**（键名与取值须可被稳定解析）：  
   - `status: draft`（本 Skill 产出初始值；人类晋升流程不在本公共库 Skill 范围内定义）  
   - `reviewedByHuman: false`（Agent **不得**改为 `true`）  
   - `confidence: medium` **或** `confidence: low`（须与 `pattern-extraction-rule.md` 中证据口径一致）  
3. **推荐键**（便于索引与审计，Agent 生成时应尽量补齐）：  
   - `docType`：建议取值 `pattern-pack-pattern` / `pattern-pack-validator` / `pattern-pack-examples` / `pattern-pack-docs`（四文件各选其一）  
   - `moduleId`、`codeTypeId`、`patternName`  
   - `extractSkill: extract-code-pattern`  
   - `extractedAt`：ISO8601 UTC 字符串  
   - `sourceOfTruth: repository`  
4. 当 `confidence: low` 时，front matter **须**包含 `needsHumanConfirmation: true`（与 `pattern-extraction-rule.md` 一致）。
5. 当 `confidence: medium` 且存在合并冲突、`code-types` 条目要求确认、或条目为推断时，front matter **须**包含 `needsHumanConfirmation: true`；否则可为 `needsHumanConfirmation: false`。

### `pattern.md` 必备正文结构

6. 正文须使读者能定位下列概念（可用二级标题或表格，顺序可调整）：  
   - `codeTypeId`、`moduleId`、`patternName`  
   - 适用场景、不适用场景  
   - 固定结构、可变参数  
   - 命名规则、文件位置规则、多文件联动规则  
   - 生成步骤草案（**非**可执行代码生成脚本，仅为步骤草案）  
   - 禁止事项、待人工确认项  

### `validator.md` 必备正文结构

7. 须包含下列清单或等价二级标题（每条下可为 checklist 或表）：  
   - 结构检查清单  
   - 命名检查清单  
   - 文件联动检查清单  
   - 注册 / 声明 / 导出检查清单  
   - 配置检查清单  
   - 测试建议检查清单  
   - 风险检查清单  
   - 完成标准  

### `examples.md` 必备正文结构

8. 须包含：  
   - 样例文件列表（仓库相对路径，原文）  
   - 每个样例的角色  
   - 哪些部分是固定结构、哪些部分是可变实现  
   - 哪些样例证据较强、哪些样例不应作为模板  

### `docs.md` 必备正文结构

9. 须包含：  
   - 给人类阅读的代码类型说明  
   - 给 Agent 执行的简明规则  
   - 新增此类代码的开发说明  
   - 常见坑点  
   - 待人工确认项  

### 状态文件

10. `{projectRoot}/.ai/state/pattern-extraction-status.md` 须有合法 YAML front matter，并包含本轮 `moduleId`、`codeTypeId`、时间与深读文件列表等元数据；结构见 `templates/state/pattern-extraction-status.template.md`。

## 禁止事项

- 在 front matter 中使用非法 YAML（如 `key:value` 无空格、Tab 缩进）。
- 将 Pack `status` 写为 `active` 作为本 Skill 的初始产出（与 `pattern-human-confirmation-rule.md` 一致）。

## 与其它 Rule 的关系

- 落实 `frontmatter-format-rule` 与 `pattern-extraction-rule` 的证据与合并策略。
- 不改变 `source-of-truth-rule`：Pack 正文从属于仓库源码与真实配置。
