# Skill：activate-project-local-skill

## 1. 目的

在开发者**已显式确认**且目标文件为 **`draft`** 项目本地 Skill 的前提下，将 `.ai/skills/project-local/create-{codeTypeId}.md` 的 front matter **激活为 `active`**（并标记人类已审阅），并 **upsert** `.ai/indexes/skill-index.md`。**不**修改 Pattern Pack 四文件（`pattern-human-confirmation-rule` 的适用对象仍为 Pack，本 Skill 的确认对象仅为**项目本地 Skill**）。**不**修改业务源码。

## 2. 适用场景

- 已由 `promote-project-local-skill` 生成 `create-{codeTypeId}.md`，且人类已阅读其正文与相关 Pattern 上下文。
- 需将项目本地 Skill 纳入索引，供 Agent 按 `project-local-priority-rule` 优先加载。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；**仅读取** Rule，**不得**写入。 |
| `{codeTypeId}` | 与 `create-{codeTypeId}.md` 文件名一致的安全 slug；**不得**含 `/`、`\\`、`..`、盘符或空字符串。 |
| `{humanConfirmation}` | 开发者给出的**确认原文字符串**（非空）；判定口径见 §6 与 §11。 |

## 4. 输出

| 路径（相对 `{projectRoot}/.ai/`） | 说明 |
|----------------------------------|------|
| `skills/project-local/create-{codeTypeId}.md` | **仅合并** YAML front matter：`status: active`、`reviewedByHuman: true`、`confidence: confirmed`、`lastReviewedAt`（当日 `YYYY-MM-DD`）；**不得**改正文。 |
| `indexes/skill-index.md` | 创建或更新；维护 `## 项目本地 Skill` 表，对目标行 **upsert**。 |

表列固定为：`skillRelativePath` | `codeTypeId` | `status` | `lastActivatedAt` | `activatedBySkill`  

- `skillRelativePath` 填 `skills/project-local/create-{codeTypeId}.md`  
- `lastActivatedAt` 填 UTC ISO8601  
- `activatedBySkill` 填 `activate-project-local-skill`

## 5. 前置条件

- `{projectRoot}/.ai/` 存在。
- 目标文件 `.ai/skills/project-local/create-{codeTypeId}.md` **存在**。
- 目标文件当前 front matter 须为 **`status: draft`**；若为 `active` 且无人类指令要求「幂等刷新索引」，可仅 upsert `skill-index` 而不重复改 front matter（见 §6）。
- 建议只读：`{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`、`{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、`{skillLibraryRoot}/rules/base/project-local-output-rule.md`。

## 6. 执行步骤

1. **校验 `{codeTypeId}`**：非法则中止（§11），不写任何文件。
2. **校验 `{humanConfirmation}`**（`trim` 后非空，长度 ≥ `12`）：  
   - **必须**包含与参数完全一致的 **`{codeTypeId}`** 字面量子串；  
   - **必须**包含与参数一致的相对路径字面量 **`skills/project-local/create-{codeTypeId}.md`**（将 `{codeTypeId}` 换为实际值，例如 `skills/project-local/create-foo.md`）；  
   - **句式**满足以下**至少一条**：包含「同意激活」；或包含「已审阅并授权激活」。  
   若不满足：**拒绝**，不写任何文件。
3. 打开 `skills/project-local/create-{codeTypeId}.md`：若 `status` 已为 `active` 且 `reviewedByHuman: true`：跳过 front matter 升级，直接进入步骤 5。
4. 否则：若 `status` **不是** `draft`（例如缺失或异常值）：**拒绝**，不写任何文件（§11）。  
   若为 `draft`：仅合并 front matter 键：`status: active`、`reviewedByHuman: true`、`confidence: confirmed`、`lastReviewedAt`（执行当日 `YYYY-MM-DD`）；遵守 `frontmatter-format-rule`；**不**改正文。
5. 打开或创建 `indexes/skill-index.md`：若不存在二级标题 `## 项目本地 Skill`，在文件适当位置创建该节与表头；对 `{codeTypeId}` 对应行 **upsert**（主键为 `skillRelativePath` 或 `codeTypeId`，**本 Skill 写死**以 `skillRelativePath` 为唯一键）。
6. 回复列出已写入路径。

## 7. 读取范围

| 文件 | 说明 |
|------|------|
| `.ai/skills/project-local/create-{codeTypeId}.md` | 读并合并 front matter。 |
| `.ai/indexes/skill-index.md` | 读并更新。 |
| `.ai/pattern-packs/{codeTypeId}/pattern.md` | 可选；交叉核对 `codeTypeId` 是否存在 Pack。 |
| `{skillLibraryRoot}/rules/**` | 按需只读。 |

## 8. 写入位置

| 允许 |
|------|
| `.ai/skills/project-local/create-{codeTypeId}.md`（仅 front matter 合并） |
| `.ai/indexes/skill-index.md` |

| 禁止 |
|------|
| 修改 `{skillLibraryRoot}/**` |
| 修改 `{projectRoot}` 下 `.ai/` **以外**业务源码 |
| 修改 `pattern-packs/{codeTypeId}/` 下四篇 Pack 的 front matter 或正文（本 Skill 职责外） |
| 改正文段落以「代替人类审阅」 |

## 9. 禁止事项

1. **不得**在 `{humanConfirmation}` 未通过 §6 校验时写入 `active` 或 `reviewedByHuman: true`。  
2. **不得**从 `draft` 以外的不明状态强行激活。  
3. **不得**删除 `skill-index.md` 中无关项目的行。  
4. **不得**将 `pattern-human-confirmation-rule` 曲解为允许 Agent 代写 Pack 的 `reviewedByHuman: true`（本 Skill **不**碰 Pack）。  
5. **不得**实现 Placement、Develop、`skills/review/**`、框架升级类能力。

## 10. 完成标准

- [ ] `{humanConfirmation}` 已校验；未通过时零写入。  
- [ ] 目标项目本地 Skill 已为 `active` 协作态（或已为 `active` 且仅刷新了索引）。  
- [ ] `skill-index.md` 中 `## 项目本地 Skill` 表已 upsert。  
- [ ] 未修改业务源码与 Pack 四文件。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| `codeTypeId` 非法 | 中止；无写入。 |
| `humanConfirmation` 不满足 §6 | **拒绝**；无写入。 |
| 目标 `create-{codeTypeId}.md` 不存在 | 中止；提示先 `promote-project-local-skill`。 |
| 目标 `status` 非 `draft` 且非已合规 `active` | 中止；提示人类检查文件状态。 |
| `skill-index.md` 无法解析 | 在备份人类段落前提下尽力追加表；若无法安全写入则中止并说明。 |

**说明**：本 Skill 的「人类确认」**仅**约束项目本地 Skill 文件；Pattern Pack 的人类审阅门控仍以 [`{skillLibraryRoot}/rules/pattern/pattern-human-confirmation-rule.md`](../../rules/pattern/pattern-human-confirmation-rule.md) 为准，但 Agent **不得**借本 Skill 修改 Pack。
