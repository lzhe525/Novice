# module-is-logical-boundary-rule

## 适用主体

执行 `scan-module-by-ai`、维护 `.ai/config/module-map.md`、或根据模块边界规划扫盘的 Agent 与人类维护者。

## 规则陈述

1. **模块是逻辑边界**：模块表示真实的业务或能力边界，**不要求**在 `{projectRoot}` 内对应单一源码目录。
2. **单目录是特例**：若模块恰好集中在一个目录树下，可视为 `directory-module`；不得将「一个文件夹」当作模块的默认定义。
3. **分散式模块须显式声明**：跨多个目录、注册表、配置与入口文件的模块，**必须**在 `{projectRoot}/.ai/config/module-map.md` 中写明同一 `moduleId` 下的 `primaryPaths`、`relatedPaths`、`entryFiles`、`registryFiles`、`configFiles` 等（最小字段集合以 `scan-module-by-ai` Skill 中的「module-map 最小契约」为准）。
4. **扫盘输入优先**：执行 `scan-module-by-ai` 时，**必须优先**依据 `moduleId` 与 `module-map`（及其中列出的路径与关键文件）确定阅读与归纳范围；**不得**默认「模块只存在于一个文件夹」。

## 扫盘执行顺序（Agent，强制）

执行模块级扫描或归纳时，须按下述顺序，**不得**用「当前打开的单一目录」替代完整模块边界：

1. **先确定 `moduleId`**：明确本轮归属的逻辑模块标识；不得在尚未确定 `moduleId` 时，把任意单一路径当作模块 ID 或把未命名边界写进索引。
2. **再根据 `module-map` 收拢范围**：读取 `{projectRoot}/.ai/config/module-map.md`（若存在），定位该 `moduleId`，据此汇总声明中的 **`primaryPaths`、`relatedPaths`、`entryFiles`、`registryFiles`、`configFiles`** 等，作为该模块相关源码与配置的**权威候选集**（实际打开文件仍受对应 Scan Skill 的预算与终止条件约束）。
3. **禁止单目录等同完整模块**：**不得**在仅浏览或列举单一目录后，推断该目录即该 `moduleId` 的全部事实范围；单目录仅作为 `directory-module` 的特例，且仍须在 map 或索引中有据可依（缺 map 时的推断须遵守各 Skill 对 `status` / `needsHumanConfirmation` 的约定）。

## 允许路径

- 读取：`{projectRoot}/.ai/config/module-map.md`、`.ai/indexes/*.md`、源码与配置（只读，受 Scan Skill 预算约束）。

## 禁止事项

- 在缺少 `module-map` 与索引依据时，将某一目录树武断等同于完整模块边界并写为确定事实（应使用「待确认」「推断」并按规定标记 `status` / `needsHumanConfirmation`）。

## 与其它 Rule 的关系

- 与 `scan-readonly-and-artifacts-rule` 互补：前者定义「模块是什么」，后者定义「扫盘写哪里、索引如何记」。
- 落实 `source-of-truth-rule`：逻辑模块文档永远从属于仓库内真实源码与配置。
