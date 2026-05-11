# Skill：scan-file-by-ai

## 1. 目的

为 `{projectRoot}` 内**单个已存在的源文件**生成 `.ai/docs/files/` 下 Markdown 说明，并区分**普通文件**与**重点文件**两类扫盘深度。**该源文件全文是事实源**；说明文档仅摘要符号、依赖与角色，不得替代阅读源码。

## 2. 适用场景

- 普通源文件需要一张「卡片」式说明（`fileImportance: normal`）。
- 入口、路由注册、核心领域逻辑等**重点文件**（`fileImportance: critical`）需在受控预算内增加有限关联阅读。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{projectRoot}` | 业务项目根。 |
| `{skillLibraryRoot}` | 公共库根。 |
| `{filePathRelative}` | **必填**。相对 `{projectRoot}` 的文件路径，正斜杠；不得为目录；不得含 `..`。 |
| `{fileImportance}` | **可选**，默认 `normal`。取值：`normal`（普通文件）或 `critical`（重点文件）。 |

## 4. 输出

| 路径 | 说明 |
|------|------|
| `{projectRoot}/.ai/docs/files/{dirPart}/{base}.md` | 由 [templates/docs/file-doc.template.md](../../templates/docs/file-doc.template.md) 实例化；`{{FILE_IMPORTANCE}}` 与 `{fileImportance}` 一致。 |

路径规则同前版：`src/app/main.ts` → `.ai/docs/files/src/app/main.md`；`Dockerfile` → `.ai/docs/files/Dockerfile.md`。

## 5. 前置条件

- `{projectRoot}/{filePathRelative}` 为普通可读文件。
- 模板与 Rule 可读（同 `scan-project-by-ai` 第 5 节 Rule 列表）。

## 6. 执行步骤

1. 规范化 `{filePathRelative}`；若 `{fileImportance}` 缺省则设为 `normal`。
2. 若路径为目录或不存在，中止。
3. 计算输出路径（`dirname` + 去扩展名 `basename` + `.md`），创建父目录。
4. 读取模板，替换 `{{FILE_PATH_RELATIVE}}`、`{{SCANNED_AT_ISO}}`、`{{FILE_IMPORTANCE}}`（与步骤 1 一致）。
5. **深读主文件**：全文；若大于 **512KiB** 或 **8000** 行，则读前 **400** 行 + 后 **200** 行并在正文顶部注明「部分读取」。
6. **分级阅读**：
   - `normal`：仅允许「读取范围」中「可选辅助文件」一行。
   - `critical`：除主文件与可选辅助文件外，可从主文件内静态 `import` / `require` / `#include` 解析出至多 **3** 个**仓库内**相对路径（解析失败则跳过），每个追加读取 ≤ **120** 行或 ≤ **16KiB**；仍计入 `agent-context-budget-rule`，不得递归再跟链。
7. 填充模板各节；在「扫盘分级」节中显式写出本次为 `normal` 或 `critical` 及实际追加读取的路径列表（若无则写「无」）。
8. 校验 front matter 后写入输出路径。

## 7. 读取范围

| 目标 | `normal` | `critical` |
|------|----------|------------|
| `{projectRoot}/{filePathRelative}` | 全文或步骤 5 截断策略 | 同左 |
| 同目录 `*.d.ts` 或同名 `.h`（体积 < **64KiB**） | 最多 **1** 个可选 | 最多 **1** 个可选 |
| 主文件解析出的直接依赖源文件 | **禁止** | 最多 **3** 个，各 ≤ **120** 行 / **16KiB** |

**禁止**在 `critical` 模式下继续对依赖的依赖做链式深读（禁止二跳以上）。

## 8. 写入位置

| 允许 |
|------|
| `{projectRoot}/.ai/docs/files/` 下按规则生成的单个 `.md` |

| 禁止 |
|------|
| 修改源文件 |
| 向 `{skillLibraryRoot}/**` 写入 |

## 9. 禁止事项

- 输出路径导致覆盖源码（必须 `.md` 且位于 `.ai/docs/files/`）。
- 编造未出现符号名。
- 复制密钥、token；疑似则省略并注明。
- 在 `normal` 模式下执行「依赖三文件」追加读。

## 10. 完成标准

- [ ] 输出文件存在，`filePathRelative`、`fileImportance`、`scanSkill: scan-file-by-ai` 在 front matter 中正确。
- [ ] 含「事实源声明」与「扫盘分级」节。
- [ ] `critical` 时正文列出追加读取路径或无追加的原因。

## 11. 失败处理

| 情况 | 处理 |
|------|------|
| 二进制或无法解码 | 中止。 |
| `critical` 解析依赖全失败 | 按 `normal` 降级完成，正文说明无法解析依赖。 |
| 输出冲突 | 中止或请求人类确认覆盖。 |
