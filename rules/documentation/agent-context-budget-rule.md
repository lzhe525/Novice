# agent-context-budget-rule

## 适用主体

执行 Scan 类 Skill、需要读取多文件的 Agent；以及在业务项目中消费 `.ai/docs/**` 与 `.ai/indexes/**` 的任意 Agent。

## 规则陈述

Agent 必须按 **预算** 读取源码与配置：先清单、后抽样、再深入；禁止默认「通读」整个 `{projectRoot}` 树。

## 预算原则

1. **清单阶段**：仅列出目录名与文件名（浅层 `list_dir` 或等价），不超过 Scan Skill 中「读取范围」规定的深度与条目上限。
2. **深读阶段**：仅打开与当前文档小节直接相关的文件；单 Skill 执行中深读文件数不超过该 Skill「读取范围」表中的上限。
3. **分次执行**：若任务超出预算，拆分为 `scan-module-by-ai` 或 `scan-file-by-ai` 多次执行，并在文档中标注「本节为部分覆盖」。

## 索引与按需加载（强制）

1. **禁止一次性加载** `{projectRoot}/.ai/docs/**` 下全部 Markdown 到上下文（除非人类显式要求全文审查且已接受成本）。
2. 会话开场优先读取：`{projectRoot}/.ai/docs/project/adapter.md`（若存在）与 `{projectRoot}/.ai/indexes/module-index.md`（若存在）。
3. 根据当前任务从 **module-index**（或各 Skill 给出的路径表）中挑选 **1–3** 个与任务直接相关的文档路径，再逐个打开阅读；未入选文件保持不读。
4. 若索引不存在，应先运行 `scan-project-by-ai` 生成 `module-index.md` 骨架，或临时用 `SKILL_ROUTER.md` 与 `AI_ENTRY.md` 中的链接表代替，但仍须遵守「少而准」打开文件。

## 允许路径

- 读取：`{projectRoot}` 下源码与配置（受各 Scan Skill 读取范围约束）。
- 写入：仅限各 Scan Skill「写入位置」表。

## 禁止事项

- 无差别递归读取 `node_modules/`、`.git/`、`dist/`、`build/` 等大型或生成目录（除非人类显式要求且 Skill 已修订范围）。
- 为凑字数复制大段源码进入 `.ai/docs/`（应使用路径引用与短摘录）。
- 在未读索引、未限定任务范围的情况下「扫读」整个 `.ai/docs` 树。

## 与其它 Rule 的关系

- 与 `read-source-before-edit-rule` 区分：本 Rule 约束**读的范围与数量**及**文档消费策略**；后者约束**改代码前须读目标源文件**。
