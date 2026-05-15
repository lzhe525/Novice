# pattern-application-rule

## 适用主体

执行 `create-code-by-pattern`、审阅 `{projectRoot}/.ai/reports/pattern-code-generation-plan.md` 与生成结果的 Agent 与人类维护者。

## 规则陈述

### 总原则

1. **业务源码为事实源**：对任何将修改或新建的源码路径，Agent **必须**在编辑前读取 `{projectRoot}` 下实际文件内容（或确认路径不存在以新建）；**不得**仅凭 Pattern Pack 正文或协作文档推断当前实现。
2. **仅依赖 `active` Pack**：当且仅当 `.ai/pattern-packs/{codeTypeId}/` 下 `pattern.md`、`validator.md`、`examples.md`、`docs.md` 四文件 YAML front matter 均可解析，且 **`status` 均为 `active`**、**`reviewedByHuman` 均为 `true`** 时，方允许将本 Pattern 作为**受控代码生成**的主要依据之一。任一为 `draft` / `deprecated` 或与上述组合不一致：视为**不满足应用前置**（由 Skill 失败处理表落盘）。
3. **索引交叉核对（可选增强）**：若 `.ai/indexes/code-type-index.md` 存在小节 `## Pattern Pack 状态` 且含目标 `codeTypeId` 行，则 `patternPackStatus` 应为 `active`；与 Pack front matter 冲突时记 **warning**，在计划/结果中说明，**不**自动修改索引或 Pack。

### 变更范围（允许改动的源码）

4. **允许集合**仅包括：  
   - Pattern 文档中**显式声明**的新增文件路径或目录约定下将要创建的文件；  
   - Pattern 文档与 `examples.md` 中**显式列出**的联动、注册、导出、配置等**已有**仓库相对路径（以项目根为根的类 Unix 相对路径）。  
5. **禁止**修改未在四文件正文中作为「须联动」「须修改」「注册点」等语义出现的源码文件；**禁止**以「顺便重构」为由扩大范围。
6. **联动路径收集上限**：从四文件正文收集候选路径时，**总数上限 40**（与 `pattern-validation-rule` 检查项 14 同量级）；超出部分不得用于授权新编辑目标，须在计划/结果中备注「截断」。

### 执行模式边界

7. **`plan-only`**：只生成并写入 `.ai/reports/pattern-code-generation-plan.md` 及状态/结果报告；**不得**创建或修改 `{projectRoot}` 下业务源码。
8. **`apply-after-confirmation`**：须先完成第 7 条意义上的计划落盘；再取得符合 `pattern-apply-confirmation-rule.md` 的人类确认原句；**未通过确认校验则不得改源码**。
9. **`projectState: blocked`**（见 `skillkit-status.md` front matter）：允许生成计划与报告，但 **apply-after-confirmation 下禁止写入业务源码**；须在状态与结果中标注阻塞原因。

### 高风险与例外

10. **默认拒绝**编辑下列语义类型的路径，**除非**人类在对话中对该路径（或 glob）**显式列外**且确认原句中可核对：  
    - 凭经验可识别的密钥、凭证、许可证私钥、生产环境仅配置文件名模式；  
    - 数据库迁移、破坏性 schema 变更脚本；  
    - 仅含生成物且通常由工具覆盖的目录（若 Pattern 未声明为合法落点）。  
    具体文件名由项目而定；Rule 不枚举业务路径。

### 与公共库及协作文档

11. **不得**向 `{skillLibraryRoot}/**` 写入项目私有内容或生成结果。  
12. **不得**在 `.ai/` 外创建 AI 协作文档（落实 `project-local-output-rule`）。  
13. **文档同步影响预判**：在生成计划后、实际应用源码前，Agent 必须按 `doc-impact-after-code-change-rule` 做轻量预判，并写入计划或回复；预判至少覆盖 `.ai/docs/project/**`、`.ai/docs/modules/**`、`.ai/docs/files/**`、`.ai/indexes/**`、`.ai/pattern-packs/**`、`.ai/skills/project-local/**` 六类对象。
14. 若预判或实际改动表明 `.ai/docs/**` 等协作文档可能与源码不同步：Agent 可在结果报告与回复中建议轻量同步、标记过期，或运行 `check-doc-impact-after-change` 生成正式报告；**不得**在无单独任务授权时自动大范围改写模块/项目文档。

## 须同时遵守的其它 Rule

- `rules/base/read-source-before-edit-rule.md`
- `rules/base/source-of-truth-rule.md`
- `rules/base/public-skill-library-purity-rule.md`
- `rules/base/project-local-output-rule.md`
- `rules/documentation/doc-impact-after-code-change-rule.md`
- `rules/documentation/agent-context-budget-rule.md`

## 与其它 Rule 的关系

- 与 `pattern-validation-rule` 互补：后者校验 Pack 草稿结构；本规则约束**已激活 Pack** 如何映射到可编辑源码集合。
- 与 `pattern-apply-confirmation-rule` 衔接：后者定义人类确认计划的格式门控。
