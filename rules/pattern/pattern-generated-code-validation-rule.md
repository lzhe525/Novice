# pattern-generated-code-validation-rule

## 适用主体

在 `create-code-by-pattern` 于 **`apply-after-confirmation`** 下完成源码修改后，执行收尾校验与报告填写的 Agent。

## 规则陈述

### 校验来源

1. **唯一权威清单**：以 `{projectRoot}/.ai/pattern-packs/{codeTypeId}/validator.md` **正文**（不含 front matter）中的可执行检查项为准，包括：  
   - Markdown 任务列表项 `- [ ]` / `- [x]`；  
   - 编号步骤；  
   - 带语言标记的可复制命令围栏（如 `bash`、`powershell`、`sh`）。  
2. **不得**跳过 `validator.md` 中可解析项而声称「已校验」；**不得**伪造命令未执行的通过结果。

### 执行与记录

3. 对每一项：在实际环境执行或人工核对后，在 `.ai/reports/pattern-code-generation-result.md`（或 Skill 规定的等价附录小节）中记录 **pass** / **fail** / **skipped**。  
4. **`skipped`** 仅当该项明确依赖的资源不存在、或人类事先声明跳过、或该项仅适用于未发生的变更类型时可用；须在结果中写明 **skipped 原因**。  
5. 任一项 **fail**：结果报告中 `validatorOverall` 语义为 **failed**（或等价字段）；**不强制要求 Agent 自动回滚**已写入源码；应列出失败项与建议修复动作，由人类决定回滚或修复。

### 与构建/测试命令

6. 若 `validator.md` 要求运行构建或测试命令：Agent 应在**项目约定**的 shell 中执行；若环境无法执行（无依赖、无网络等），该项记 **skipped** 或 **fail**（按该项文字是否将「可运行」作为硬条件而定），**不得**虚构退出码。

### 与 Pack 状态

7. **不得**为通过本校验而修改 `validator.md` 正文以削弱检查项；发现清单与实现不一致时，应记在结果报告中建议人类更新 Pack。

## 须同时遵守的其它 Rule

- `rules/pattern/pattern-application-rule.md`
- `rules/base/source-of-truth-rule.md`

## 与其它 Rule 的关系

- 与 `pattern-validation-rule` 检查项 9 精神一致：本规则约束**生成后**对同一 `validator.md` 的消费方式。
