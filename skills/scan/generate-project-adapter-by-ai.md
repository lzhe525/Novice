# Skill：generate-project-adapter-by-ai

## 1. 目的

生成两类项目侧文档：（1）紧凑 `adapter.md` 草稿；（2）与之配套的 `adapter-evidence.md`，用表格把 Adapter 中的断言绑定到**已读**仓库路径，支撑审计。**源码与配置仍是事实源**；Adapter 与 Evidence 均不得包含未经验证的实现细节。

## 2. 适用场景

- 与 `scan-project-by-ai` 配套，作为 Agent 会话第一站小上下文。
- 需要向人类展示「结论—证据路径」对照时。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根。 |
| `{skillLibraryRoot}` | 公共库根。 |
| 可选 `{projectNameOrSlug}` | 填入模板。 |

## 4. 输出

| 路径 | 说明 |
|------|------|
| `{projectRoot}/.ai/docs/project/adapter.md` | [templates/docs/project-adapter.template.md](../../templates/docs/project-adapter.template.md) |
| `{projectRoot}/.ai/docs/project/adapter-evidence.md` | [templates/docs/project-adapter-evidence.template.md](../../templates/docs/project-adapter-evidence.template.md) |

## 5. 前置条件

- `{projectRoot}/.ai/` 存在。
- 可读两模板与 `scan-project-by-ai` 第 5 节所列 Rule。

## 6. 执行步骤

1. 创建 `{projectRoot}/.ai/docs/project/`（若不存在）。
2. 设 `{{GENERATED_AT_ISO}}`、`{{PROJECT_NAME_OR_SLUG}}`。
3. 按「读取范围」完成清单与深读，在内存中维护**断言列表**（每条含：结论短语、依据路径、读取行范围说明）。
4. 实例化并写入 `adapter.md`：「技术栈标签」≤ **10**；「主入口与关键路径」表格中每行须在步骤 3 中有依据，否则标「未知」并进入 evidence 的「未验证项」。
5. 实例化 `adapter-evidence.md`：将步骤 3 中**每一条**已写入 adapter 的确定性结论映射到「结论与证据」表；无法在已读文件中印证的条目列入「未验证项」节，**禁止**编造路径。
6. 「建议后续 Scan」两节（adapter 模板内）列出 **3–10** 条已观察存在的 `{moduleRootRelative}` 或 `{filePathRelative}`。
7. 分别校验两份 front matter 后写入；若人类曾将 adapter `reviewedByHuman` 置 `true`，覆盖前须提醒备份。

## 7. 读取范围

| 阶段 | 路径 | 上限 |
|------|------|------|
| 清单 | `{projectRoot}` 顶层 | 40 名称 |
| 深读 | `README*`（0–1） | ≤120 行/文件 |
| 深读 | `package.json` 或首个 `*.sln`（0–1） | ≤160 行 |
| 深读 | `.ai/docs/project/overview.md`（若存在） | ≤200 行 |
| 深读 | `.ai/docs/project/deep-dive.md`（若存在） | ≤200 行 |
| 清单 | `src/`、`lib/`、`apps/` 之一的一层子目录 | ≤30 名称 |

**禁止**超表读取；不读 `node_modules/`。

## 8. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/docs/project/adapter.md` |
| `{projectRoot}/.ai/docs/project/adapter-evidence.md` |

| 禁止 |
|------|
| `{skillLibraryRoot}/**` 写入 |
| 将 evidence 仅放在 adapter 正文内而不生成独立 `adapter-evidence.md` |

## 9. 禁止事项

- 在 evidence 表中引用未在步骤 3 读取的文件路径。
- 大段复制 `overview.md` / `deep-dive.md`（应路径引用 + 短摘要）。

## 10. 完成标准

- [ ] `adapter.md` 与 `adapter-evidence.md` 均存在且 `docType`、`scanSkill: generate-project-adapter-by-ai` 正确。
- [ ] evidence 表行数 ≥ adapter 中「已断言非未知」条目数（或「未验证项」解释缺口）。
- [ ] 两文件均含事实源声明。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| `project-adapter-evidence.template.md` 缺失 | 中止；不单独写 adapter。 |
| 空仓库 | 两文件仍生成，大量「未知」与未验证项。 |
