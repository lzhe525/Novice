---
status: active
routeEnabled: true
description: 检查公共库与项目本地 Skill 的路由配置、metadata 与磁盘路径是否一致。
triggerWhen:
  - 路由校验
  - validate skill routing
  - 检查 Router Matrix 一致性
---

# Skill：validate-skill-routing

## 1. 目的

对公共 HACF 库与（可选）业务项目 `.ai/` 下的 Skill 路由体系做**只读**一致性检查：验证 `SKILL_ROUTER.md`、`TASK_TRIGGER_MATRIX.md`、Skill 文件 front matter、`routeEnabled` 与路径存在性，产出结构化校验报告，供维护者修复 routing error 后再参与 `select-skill-for-task` 自动路由。

## 2. 适用场景

- 新增或修改公共 Skill 后，在合并前确认 Router / Matrix / metadata 一致。
- 激活项目本地 Skill 或更新 `skill-index.md` 后，确认登记与文件一致。
- 定期健康检查或 `register-new-skill` 完成后的必然后续步骤。
- 用户或维护者显式要求「路由校验」「validate skill routing」。

**不适用**：

- 需要自动修复 Router / Matrix 或批量改写 metadata（本 Skill 仅报告，不修复）。
- 纯业务源码变更且未触及 Skill 路由配置。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根目录；检查项目本地时必填。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；可由 `SKILLKIT_LINK.md` 解析或从本 Skill 路径反推。 |
| `{scope}` | 可选。`public`（默认）\| `project-local` \| `all`。 |

## 4. 输出

| scope | 输出路径 |
|-------|----------|
| `public` | `{skillLibraryRoot}/reports/skill-routing-validation-report.md` |
| `project-local` | `{projectRoot}/.ai/reports/skill-routing-validation-report.md` |
| `all` | 上述**两份**报告；公共报告**不得**含项目私有路径全文、密钥或未脱敏内容 |

报告须包含下列章节（顺序固定）：

1. **总体状态**：`passed` \| `warning` \| `failed`（存在任一 error ⇒ `failed`；无 error 有 warning ⇒ `warning`；否则 `passed`）
2. **Router 路径存在性检查结果**
3. **Trigger Matrix 状态检查结果**（Available / Planned 分区分别汇总）
4. **Skill metadata 检查结果**（`status`、`description`、`triggerWhen`）
5. **`routeEnabled` 检查结果**
6. **planned / available 状态一致性检查**
7. **项目本地 Skill 索引检查**（scope 含 project-local 或 all 时；公共报告仅写统计，不写私有路径明细）
8. **错误列表**（error 级，含路径与规则编号）
9. **警告列表**（warning 级）
10. **修复建议**（每条 error/warning 对应可执行建议）

## 5. 前置条件

- `{skillLibraryRoot}/ENTRY.md` 可读。
- `{skillLibraryRoot}/routers/SKILL_ROUTER.md` 与 `{skillLibraryRoot}/routers/TASK_TRIGGER_MATRIX.md` 可读。
- 只读关联 Rule：`rules/routing/skill-routing-validation-rule.md`、`rules/routing/skill-route-enabled-rule.md`、`rules/routing/new-skill-registration-rule.md`。
- scope 为 `project-local` 或 `all` 时：`{projectRoot}/.ai/` 可读（缺失则项目本地检查记为 skipped 并在报告中说明）。
- 写入前若 `reports/` 目录不存在，可创建 `{skillLibraryRoot}/reports/` 或 `{projectRoot}/.ai/reports/`（**仅**报告目录，不创建其它 `.ai/` 子树）。

## 6. 执行步骤

1. **解析范围**：根据 `{scope}` 确定读取与写入目标（见 §4、§7）。
2. **抽取 Router 路径**：只读 `SKILL_ROUTER.md`，用正则提取所有 `` `skills/...` `` 路径，去重得集合 `R`。
3. **抽取 Matrix 路径**：只读 `TASK_TRIGGER_MATRIX.md`，按 **Available Skill Triggers** 与 **Planned Skill Triggers** 分区分别提取路径，得 `A`（available）与 `P`（planned）。
4. **路径存在性探测**（公共库）：
   - 对 `R` 中每条 `p`：探测 `{skillLibraryRoot}/{p}`，记录 `exists`。
   - 对 `A` 中每条 `p`：探测存在性；不存在记 **error**（幽灵 available）。
   - 对 `P` 中每条 `p`：探测存在性；若存在记 **warning**（拟升级须走注册流程，不得自动改 Matrix）。
5. **Planned 强制措辞检查**：对 Matrix Planned 分区各行，若含 `must_invoke` 或「必须使用」记 **error**。
6. **解析 Skill metadata**：对 `R ∪ A` 中 `exists: true` 的公共 Skill 文件，读取**首块** YAML（行首 `---` 至下一 `---`），解析 `status`、`routeEnabled`、`description`、`triggerWhen`。
7. **metadata 规则**：
   - `status: active` 且缺 `description` 或 `triggerWhen` ⇒ **error**
   - `routeEnabled: true` 且路径不在 `R ∪ A ∪ P` ⇒ **error**（未登记）
   - `routeEnabled: true` 且 `status` 非 `active` ⇒ **error**
   - Matrix `A` 中 `must_invoke` 对应路径若 `routeEnabled: false` 或非 `active` ⇒ **error**
8. **交叉一致性**：
   - `A ∩ P` 非空 ⇒ **error**（同一路径不得同时在 Available 与 Planned）
   - Available 指向缺失文件 ⇒ **error**（已在步骤 4）
9. **项目本地**（scope 为 `project-local` 或 `all`）：
   - 解析 `{projectRoot}/.ai/indexes/skill-index.md` 中 `## 项目本地 Skill` 表各行 `skillRelativePath` 与 `status`。
   - 探测 `{projectRoot}/.ai/{skillRelativePath}` 存在性；缺失记 **error**。
   - 对 `status: active` 行读取文件 front matter，校验与 `skill-route-enabled-rule` 一致。
   - glob 列出 `{projectRoot}/.ai/skills/project-local/**/*.md`，未出现在 skill-index 的文件记 **warning**。
10. **汇总总体状态**并写入报告（§4 结构）；公共 scope 报告中项目本地小节仅写「已检查 / 跳过 / error 计数」，不写业务路径明细。
11. **对话输出**：报告路径、总体状态、error/warning 计数、首要修复项（至多 5 条）。

## 7. 检查范围

### 公共库（scope `public` 或 `all`）

| 路径 | 用途 |
|------|------|
| `{skillLibraryRoot}/routers/SKILL_ROUTER.md` | Router 登记 |
| `{skillLibraryRoot}/routers/TASK_TRIGGER_MATRIX.md` | 信号触发与 planned |
| `{skillLibraryRoot}/skills/**/*.md` | 仅校验被 `R ∪ A ∪ P` 引用且存在的文件 metadata（**不得**为发现新 Skill 而全库扫描后自由挑选） |

### 项目本地（scope `project-local` 或 `all`）

| 路径 | 用途 |
|------|------|
| `{projectRoot}/.ai/indexes/skill-index.md` | 项目本地登记 |
| `{projectRoot}/.ai/skills/project-local/**/*.md` | 存在性与未登记 warning |

## 8. 检查规则

与 `{skillLibraryRoot}/rules/routing/skill-routing-validation-rule.md` 一致；本 Skill 为可执行实现。Front matter 规范见 `new-skill-registration-rule.md`。

**error 与 warning 分级摘要**：

| 级别 | 示例 |
|------|------|
| error | Router/Matrix 指向不存在文件；Available + must_invoke 但 metadata 不满足；planned 行含 must_invoke；active 缺 description/triggerWhen；routeEnabled true 未登记 |
| warning | Planned 路径文件已存在（拟升级）；skill-index 未登记的项目本地文件；Router 登记但 routeEnabled false（仅 info，可合并入 warning 节） |

## 9. 写入位置

| 允许 |
|------|
| `{skillLibraryRoot}/reports/skill-routing-validation-report.md`（scope 含 public） |
| `{projectRoot}/.ai/reports/skill-routing-validation-report.md`（scope 含 project-local） |

| 禁止 |
|------|
| 修改 `{skillLibraryRoot}/routers/**`、`skills/**` metadata |
| 修改业务源码 |
| 将项目私有信息写入公共库报告 |

## 10. 禁止事项

1. **不修改**业务源码。
2. **不自动修改** Router 或 Trigger Matrix。
3. **不自动修改** Skill metadata。
4. **不把**项目私有信息写入公共库报告。
5. **不将** planned Skill 自动改成 available。

## 11. 完成标准

- [ ] 已按 `{scope}` 读取约定范围内的 Router、Matrix 与（若适用）skill-index。
- [ ] 报告含 §4 所列 10 个小节且总体状态已判定。
- [ ] 所有 error 已列入错误列表并附修复建议。
- [ ] 对话中已给出报告路径与总体状态。

## 12. 失败处理

| 情况 | 处理 |
|------|------|
| `{skillLibraryRoot}` 不可解析 | 中止；不写入报告；提示 Bootstrap。 |
| Router 或 Matrix 缺失 | 记 `failed`；报告中 error 说明文件缺失。 |
| `.ai/` 不可读且 scope 含 project-local | 项目本地检查 skipped；公共部分仍可完成。 |
| 报告目录不可写 | 对话输出完整摘要；注明写盘失败。 |

## 13. 与其它 Skill 的关系

- 由 `register-new-skill` 在第 7 步要求后续执行本 Skill。
- 与 `select-skill-for-task` 互补：本 Skill 供维护者校验配置；后者供任务运行时路由。
- 遵守 `skill-routing-validation-rule`、`skill-route-enabled-rule`。
