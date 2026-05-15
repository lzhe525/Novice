---
status: active
routeEnabled: false
description: 将 Pattern 生成流程沉淀为项目本地 Skill 草稿。
triggerWhen:
  - 晋升项目本地 Skill
  - promote project local skill
---
# Skill：promote-project-local-skill

## 1. 目的

将已验证稳定、可重复的「基于某 `codeTypeId` 执行 Pattern 受控代码生成」流程，沉淀为**项目本地** Skill 文档 `.ai/skills/project-local/create-{codeTypeId}.md`，便于团队复用编排步骤；新建文件 front matter 中 **`status` 必须为 `draft`**，激活交由 `activate-project-local-skill`。

## 2. 适用场景

- 某 `codeTypeId` 的 Pattern Pack 已为 `active`，且已至少完成一轮 `create-code-by-pattern` 与（建议）`review-pattern-code-generation`。
- 人类希望把固定调用顺序、变量约定、门禁提醒写入**项目内** `.ai/skills/project-local/`，而非扩展公共 HACF 库。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根目录。 |
| `{skillLibraryRoot}` | 公共 HACF 库根；**仅用于读取** Rule 与公共 Skill 路径引用，**不得**写入项目私有内容至公共库。 |
| `{codeTypeId}` | 目标代码类型；安全 slug，**不得**含 `/`、`\\`、`..`、盘符或空字符串。 |
| `{moduleId}` | 默认模块标识，写入项目本地 Skill 正文占位；安全 slug。 |
| `{promotionNotes}` | 可选；人类对「为何可沉淀」的简短说明（非空时写入项目本地 Skill 的「维护说明」小节）。 |

## 4. 输出

| 路径（相对 `{projectRoot}/.ai/`） | 说明 |
|----------------------------------|------|
| `skills/project-local/create-{codeTypeId}.md` | 项目本地 Skill 正文；**仅当该路径尚不存在时**由本 Skill **新建**。 |
| `state/project-local-skill-promotion-status.md` | 每次执行**必须**更新或覆盖为**最新一次**晋升运行摘要（含成功或冲突原因）。 |

目标项目本地 Skill 文件须含 YAML front matter，且**至少**包含：

- `status: draft`（**禁止**本 Skill 写入 `active`）
- `codeTypeId`：与 `{codeTypeId}` 一致
- `kind: project-local-pattern-codegen`
- `promotedAt`：UTC ISO8601
- `sourcePublicSkills`：列表，须含 `skills/pattern/create-code-by-pattern.md` 等字符串（相对 `{skillLibraryRoot}` 的引用路径，使用正斜杠）

正文须为**编排说明**（调用顺序、必备输入 `{moduleId}` / `{userRequirement}` / `{mode}` 等、须阅读的 `.ai/` 路径），**禁止**大段粘贴业务源码。

## 5. 前置条件

- `{projectRoot}/.ai/` 存在。
- 建议只读：`{skillLibraryRoot}/rules/base/public-skill-library-purity-rule.md`、`{skillLibraryRoot}/rules/base/project-local-output-rule.md`、`{skillLibraryRoot}/rules/base/frontmatter-format-rule.md`、`{skillLibraryRoot}/rules/base/project-local-priority-rule.md`。
- **必须**能读取 `.ai/pattern-packs/{codeTypeId}/` 下四文件且均为 `status: active`、`reviewedByHuman: true`；否则本 Skill **不得**新建项目本地 Skill（仅写 `promotion-status` 为 `blocked`，见 §6、§11）。

## 6. 执行步骤

1. **校验 `{codeTypeId}` / `{moduleId}`**：非法则写入 `project-local-skill-promotion-status.md` 为 `outcome: blocked` 并中止创建。
2. **校验 Pattern Pack 协作态**：读取 `pattern-packs/{codeTypeId}/pattern.md`、`validator.md`、`examples.md`、`docs.md`；若缺一或任一 front matter 非 `status: active` 或 `reviewedByHuman: true`：写入 `project-local-skill-promotion-status.md` 为 `outcome: blocked`、`reason: pack_not_active_or_not_reviewed`，**不**创建 `create-{codeTypeId}.md`；**结束**。
3. 确保目录 `.ai/skills/project-local/` 存在（可创建）。
4. 若 `.ai/skills/project-local/create-{codeTypeId}.md` **已存在**：**禁止覆盖**；写入 `state/project-local-skill-promotion-status.md`：`outcome: conflict_existing_file`，并记录已存在路径与建议（人类手动合并或删除后重试）；**结束**。
5. 从本 Skill 内嵌的**骨架**（见下方「正文骨架」）实例化，替换占位符，写入 `create-{codeTypeId}.md`，保证 `status: draft`。
6. 写入 `state/project-local-skill-promotion-status.md`：`outcome: promoted`，含 `codeTypeId`、时间戳、相对路径 `skills/project-local/create-{codeTypeId}.md`。
7. 回复中列出新建或冲突结论及上述路径。

**正文骨架（须本地化到目标文件，保持中文）**：

- 标题：`# 项目本地 Skill：create-{codeTypeId}`
- 节：目的；何时调用；必备输入（`{moduleId}`、`{userRequirement}`、`{mode}`、`{planApplyConfirmation}` 在 apply 模式下）；推荐执行顺序（先读 `AGENTS.md`、`.ai/entry/AI_ENTRY.md`、Pack 四文件、模块文档；再调用公共库 `skills/pattern/create-code-by-pattern.md`）；禁止事项（不得写公共库、不得跳过计划）；指向 `{skillLibraryRoot}/skills/pattern/create-code-by-pattern.md` 的相对链接说明（由人类在运行环境解析 `{skillLibraryRoot}`）。

## 7. 读取范围

| 路径 | 说明 |
|------|------|
| `.ai/pattern-packs/{codeTypeId}/pattern.md` 等四文件 | **必须**读取以核对 `active` / `reviewedByHuman`。 |
| `.ai/reports/pattern-code-generation-review.md` | 可选；若存在可摘要引用一条结论，不得长文复制。 |
| `.ai/indexes/code-type-index.md` | 可选；核对 `codeTypeId` 是否登记。 |
| `{skillLibraryRoot}/rules/**` | 按需只读。 |

## 8. 写入位置

| 允许 |
|------|
| `.ai/skills/project-local/create-{codeTypeId}.md`（仅当不存在时新建） |
| `.ai/state/project-local-skill-promotion-status.md`（每次覆盖为最新一次） |
| 创建目录 `.ai/skills/project-local/` |

| 禁止 |
|------|
| 覆盖已存在的 `create-{codeTypeId}.md`（冲突时仅写 `promotion-status`） |
| 修改 `{skillLibraryRoot}/**` |
| 修改 `{projectRoot}` 下业务源码 |
| 将 `status` 写为 `active` 或将 `reviewedByHuman` 写为 `true`（须由 `activate-project-local-skill` 或人类流程处理） |

## 9. 禁止事项

1. **不得**静默覆盖已有项目本地 Skill 文件。  
2. **不得**把项目私有源码全文写入公共 HACF 库任意文件。  
3. **不得**在本 Skill 中执行业务代码生成或修改 Pack 四文件正文。  
4. **不得**实现 Placement、Develop、`skills/review/**`、框架升级类能力。  
5. **不得**在 front matter 省略 `status: draft` 或使用非 `draft` 的初始 `status`。

## 10. 完成标准

- [ ] `codeTypeId` / `moduleId` 已校验；Pattern Pack 四文件为 `active` / `reviewedByHuman`（若不满足则未创建项目本地文件）。  
- [ ] 若目标文件不存在：已新建且 front matter 含 **`status: draft`**。  
- [ ] 若目标文件已存在：未覆盖，且 `project-local-skill-promotion-status.md` 记录 `conflict_existing_file`。  
- [ ] `project-local-skill-promotion-status.md` 已更新。  
- [ ] 未修改业务源码与公共库。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| `codeTypeId` 非法 | 不写 `create-*.md`；`promotion-status` 标 `blocked`。 |
| `.ai/` 不存在 | 中止；提示 Bootstrap。 |
| 目标文件已存在 | 不创建；`promotion-status` 标 `conflict_existing_file`。 |
| Pack 非 active 或 `reviewedByHuman` 非全 `true` | **不**创建 `create-*.md`；`promotion-status` 标 `blocked` 与原因。 |