---
status: active
routeEnabled: false
description: 创建或更新项目根极薄 AGENTS.md 或 AI-SKILLKIT 区块。
triggerWhen:
  - 更新 AGENTS.md
  - 极薄入口
---

# Skill：create-or-update-agents-md

## 1. 目的

在 `{projectRoot}` 根目录**创建或增量更新**极薄 `AGENTS.md`，使任意 Agent 能发现项目 AI 协作入口（`.ai/entry/AI_ENTRY.md`），并声明 AI 协作产物仅写入 `.ai/`。**不得**全文覆盖用户已有 `AGENTS.md` 中非受控内容。

## 2. 适用场景

- 业务项目首次接入 HACF Bootstrap MVP。
- 业务项目已有 `AGENTS.md`，需要插入或修正 HACF 的受控区块。
- 由 `load-skill-library` 编排调用，或开发者单独要求仅更新根入口。

## 3. 输入

| 变量 / 输入 | 说明 |
|-------------|------|
| `{skillLibraryRoot}` | 公共库根目录，含 `ENTRY.md`；用于读取模板 `{skillLibraryRoot}/templates/agents/AGENTS.thin-entry.template.md`。 |
| `{projectRoot}` | 业务项目根目录；目标文件为 `{projectRoot}/AGENTS.md`。 |
| 可选：用户指示 | 若用户禁止修改已有 `AGENTS.md`，则中止本 Skill。 |

## 4. 输出

| 路径 | 变更 |
|------|------|
| `{projectRoot}/AGENTS.md` | 新建全文（仅当文件不存在），或在保留全文前提下插入/更新 `<!-- AI-SKILLKIT:BEGIN -->` … `<!-- AI-SKILLKIT:END -->` 区块。 |

本 Skill **不产生** `.ai/` 下任何文件。

## 5. 前置条件

- `{skillLibraryRoot}/templates/agents/AGENTS.thin-entry.template.md` 存在且可读。
- Agent 对 `{projectRoot}/AGENTS.md` 具备创建或写入权限（若文件已存在则为更新权限）。
- 已解析绝对路径或等价的 `{projectRoot}`（避免写到错误目录）。

## 6. 执行步骤

1. **定位目标文件**：构造路径 `{projectRoot}/AGENTS.md`，使用文件系统检查该路径是否存在。
2. **分支 A — 文件不存在**：
   - 读取 `{skillLibraryRoot}/templates/agents/AGENTS.thin-entry.template.md` 的完整文本。
   - 将读取结果**原样**（除必要时统一换行符外不做改写）写入 `{projectRoot}/AGENTS.md`。
   - 确认写入后文件中至少包含字符串 `.ai/entry/AI_ENTRY.md`。
3. **分支 B — 文件已存在**：
   - 读取当前 `{projectRoot}/AGENTS.md` 全文到内存；**禁止**删除 BEGIN 之外的用户内容。
   - 若全文中**不存在**子串 `<!-- AI-SKILLKIT:BEGIN -->` 或**不存在**子串 `<!-- AI-SKILLKIT:END -->`：
     - 在文件**末尾**追加换行后追加以下受控区块（中英文可保持如下写法；须含 `AI_ENTRY.md` 指针）：

```markdown
<!-- AI-SKILLKIT:BEGIN -->
## AI Skill Kit Integration

本项目使用外部公共 Skill 库（HACF）。

请先阅读：

- `.ai/entry/AI_ENTRY.md`

要求：

1. 项目本地所有 AI 协作产物只能写入 `.ai/`。
2. 公共 Skill 库默认只读；不得将项目私有内容写入公共库。
3. 开发前执行公共库内 Skill `skills/bootstrap/check-project-readiness.md`（从 `.ai/entry/SKILLKIT_LINK.md` 解析公共库根后再拼接完整路径）。
4. 若存在项目本地 Skill / Rule，优先于公共 Skill。
5. 修改代码前必须读取真实源码。

<!-- AI-SKILLKIT:END -->
```

     - 注意：上块中第三条的 Skill 路径在落盘时应写为**相对项目根**的可执行说明，例如：「执行公共库内 Skill：`check-project-readiness.md`（路径见 `.ai/entry/SKILLKIT_LINK.md`）」，避免硬编码本机绝对路径。
   - 若 BEGIN 与 END **均存在**：
     - 截取 BEGIN 与 END 之间（不含边界标记行）的文本，检查是否包含子串 `AI_ENTRY.md` 或 `.ai/entry/AI_ENTRY.md`。
     - 若不包含：仅替换 BEGIN 与 END **之间**的段落为与上面「分支 B」相同的区块正文（保留 BEGIN、END 两行本身）。
     - 若已包含：不修改该区块（本 Skill 对该文件无变更），在回复中说明「受控区块已存在且含 AI_ENTRY 引用」。

## 7. 写入位置

| 允许写入的路径 | 内容类型 |
|----------------|----------|
| `{projectRoot}/AGENTS.md` | 极薄入口或受控区块 |

| 禁止写入 |
|----------|
| `{projectRoot}/.ai/**`（本 Skill 不负责） |
| `{skillLibraryRoot}/**` |

## 8. 禁止事项

- **禁止**删除、覆盖或改写 `AGENTS.md` 中 `<!-- AI-SKILLKIT:BEGIN -->` 之前与 `<!-- AI-SKILLKIT:END -->` 之后的用户自有内容。
- **禁止**在 `AGENTS.md` 中粘贴完整公共 Skill 正文、扫描结果、Pattern 或大段项目文档。
- **禁止**在除 `AGENTS.md` 外的 `{projectRoot}` 任意路径新建「第二份」Agent 入口说明来绕过 `.ai/`。

## 9. 完成标准

- [ ] `{projectRoot}/AGENTS.md` 存在。
- [ ] 文件中可通过文本搜索找到 `.ai/entry/AI_ENTRY.md`。
- [ ] 若存在 HACF 受控区块，BEGIN/END 标记成对出现。
- [ ] `{projectRoot}` 根目录下**未**新增除 `AGENTS.md` 以外的 AI 说明文件。

## 10. 失败处理

| 情况 | 处理 |
|------|------|
| 模板文件不存在 | 中止；提示人类检查 `{skillLibraryRoot}` 是否为完整 HACF 仓库。 |
| 无写权限 | 中止；输出权限错误与目标路径。 |
| 用户拒绝修改已有 `AGENTS.md` | 不写入；建议用户手动添加受控区块或自行合并。 |
| BEGIN/END 不成对（仅存在一个标记） | **不**自动修复；中止并请求人类整理 `AGENTS.md` 标记后再执行。 |
