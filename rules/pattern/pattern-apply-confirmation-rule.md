# pattern-apply-confirmation-rule

## 适用主体

在 `create-code-by-pattern` 的 **`apply-after-confirmation`** 模式下，准备修改业务源码前收集并校验人类意图的 Agent 与人类确认者。

## 规则陈述

### 门控

1. **无确认则不得改源码**：未收到符合本条第 2–4 款的 `{planApplyConfirmation}` 原文字符串时，Agent **禁止**对 `{projectRoot}` 下业务源码做任何写入。

### 确认原句格式（须全部满足）

2. **非空**：对 `{planApplyConfirmation}` 做 `trim` 后长度 **≥ 24** 字符。  
3. **标识符字面量**：确认原句中须**同时**包含**与本轮调用参数完全一致**的下列子串（区分大小写，逐字匹配）：  
   - `{codeTypeId}` 的完整字面量；  
   - `{moduleId}` 的完整字面量。  
4. **计划同意语义**：确认原句须表达已阅读并同意执行 **`.ai/reports/pattern-code-generation-plan.md`** 中列出的变更，满足下列**之一**即可：  
   - 含文件名 `pattern-code-generation-plan.md` 或相对路径 `` `.ai/reports/pattern-code-generation-plan.md` ``；  
   - 含「已审阅并同意执行」且同句或紧邻句指代「本轮代码生成计划」或「计划中所列变更」（允许中文同义表述，但须能排除其它模糊任务）。  
5. **禁止**：Agent **不得**自拟或拼接人类未提供的确认句；**不得**在人类仅回复「好的」「继续」而无标识符与计划指代时视为通过。

### 与激活确认的区别

6. 本规则针对 **源码变更计划** 的确认，**不**替代 `activate-pattern-pack` 所需的 `humanConfirmation`（Pattern 激活仍按该项目 Skill 与 `pattern-pack-activation-rule` 执行）。

## 须同时遵守的其它 Rule

- `rules/pattern/pattern-application-rule.md`
- `rules/base/source-of-truth-rule.md`

## 与其它 Rule 的关系

- 与 `pattern-human-confirmation-rule.md` 区分：后者约束 Pack 的 `reviewedByHuman` / `status` 晋升，不约束本 Skill 的计划执行确认。
