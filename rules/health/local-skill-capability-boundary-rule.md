# local-skill-capability-boundary-rule

## 适用主体

编写或审阅 `{projectRoot}/.ai/skills/project-local/*.md` 的人类，以及执行 `check-project-ai-health` 时对项目本地 Skill 做静态边界评估的 Agent。

## 规则陈述

### 文件范围

1. 本 Rule 适用于 glob：`.ai/skills/project-local/**/*.md`（通常文件名为 `create-{codeTypeId}.md`）。

### status 与 reviewedByHuman

2. **`status: active`** 时，front matter 中 **`reviewedByHuman` 必须为布尔 `true`**；否则健康检查记 **阻塞类 `high`** 或 **`critical`**（若正文已指示可自动改源码）。
3. **`status: draft`** 时，**不得** `reviewedByHuman: true`；若违反，记 **`high`**（状态语义不一致）。
4. `status` 取值须为 `draft`、`active`、`deprecated` 之一；非法或缺失记 **`high`**。

### 建议必填元数据（健康检查维度）

5. 下列键**建议**出现在项目本地 Skill 的 YAML front matter 或正文中可被稳定解析的位置；缺失时按严重度记 **`medium`** 或 **`high`**（若 Skill 已 `active` 且缺失多项，倾向 **`high`**）：

   - `moduleId`：默认目标模块标识。
   - `codeTypeId`：与 Pattern Pack 目录名一致；若文件名形如 `create-{codeTypeId}.md`，须与正文/`codeTypeId` 字段一致。
   - `dependsOn`：依赖列表（YAML 数组或正文明确列表）；每项须可解析为：
     - 相对 `{skillLibraryRoot}` 的公共 Skill 路径（如 `skills/pattern/create-code-by-pattern.md`），或
     - 相对 `{projectRoot}` 的项目内路径（如 `.ai/pattern-packs/foo/validator.md`）。
   - 任一项指向不存在的文件：记 **`high`**。

### 写权限与计划门控

6. 须声明 **`allowedWriteScopes`** 与 **`forbiddenWriteScopes`**（YAML 数组、列表或正文等价结构）；**完全缺失**记 **`high`**。
7. 须声明 **`requiresPlanBeforeApply`**（布尔）；缺失记 **`medium`**；若为 `active` 且声明 `false` 同时 `allowedWriteScopes` 含多目录或宽泛通配，记 **`high`**。
8. **过度泛化**（满足任一记 **`high`**，若 combined 可改密钥/CI/根配置且无门控则 **`critical`**）：
   - 单条 scope 覆盖仓库根下「全树」语义（如 `**/*`、`` `src/**` `` 且与「仅新增单类文件」目的明显不符）；
   - 显式包含 `.env`、`secrets`、`credentials`、`.github/workflows` 等敏感路径模式且无并列的人工确认流程说明。

### validator 引用

9. `active` 的 Skill 须在正文或 front matter 中能关联到对应 **`.ai/pattern-packs/{codeTypeId}/validator.md`**（路径字符串或链接），或给出**等价的可执行校验步骤**（不少于 3 条可观察检查项，语义对齐 `pattern-validation-rule` 检查项 9）；否则记 **`high`**。

### 失败处理

10. 正文须包含可被标题或关键词识别的 **「失败处理」「中止条件」「无写权限」** 之一小节或段落；**完全缺失**记 **`medium`**；`active` 且缺失记 **`high`**。

### 公共库与纯洁性

11. 若正文或 front matter 出现下列任一，记 **`critical`**：
    - 要求将项目私有内容**写入、同步、复制**到 `{skillLibraryRoot}` 或「HACF 公共库」「公共 Skill 库」；
    - 将业务绝对路径与公共库 `skills/`、`rules/`、`templates/` 组合为**写入目标**。

### 与危险区配置交叉（若存在）

12. 若 `{projectRoot}/.ai/config/danger-zones.md` 或 `hard-constraints.md` **存在**，且某 Skill 的 `allowedWriteScopes` 与其中声明的高风险路径**相交**，但该 Skill 未 `requiresPlanBeforeApply: true` 或未在正文要求人工确认：记 **`high`** 或 **`critical`**（依危险区严重程度）。

## 须同时遵守的其它 Rule

- `rules/base/project-local-priority-rule.md`
- `rules/base/public-skill-library-purity-rule.md`
- `rules/pattern/pattern-validation-rule.md`（validator 形态参考）
- `rules/base/frontmatter-format-rule.md`

## 禁止事项

- 通过项目本地 Skill 指示绕过 `create-code-by-pattern` 的计划/确认门禁（健康检查发现即记 **`critical`** / **`high`**）。

## 与其它 Rule 的关系

- 与 `project-ai-health-rule` 配合：`check-project-ai-health` 的第 8、9 类检查应落实本 Rule 的可判定项。
