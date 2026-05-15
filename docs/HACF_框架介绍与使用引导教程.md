# HACF 框架介绍与使用引导教程

> 项目名称：HACF — Human-AI Collaboration Framework  
> 中文名：人类与 AI 协作框架  
> 当前版本：0.2.7（`bootstrap-mvp+scan+pattern+health-onboarding+productivity-grill+version-compat-local-upgrade+skill-routing+doc-sync-precheck`）
> 当前定位：Agent-first 公共 Skill 文档库  
> 推荐使用环境：Cursor、Codex、Claude Code 或其他可读取项目文件的 Agent  
> 推荐文档语言：中文为主，代码路径、类名、方法名、配置名、Skill 名称保持原文

---

## 1. HACF 是什么

HACF 是一套面向 Agent 编程协作的公共 Skill 文档库。

它不是业务代码仓库，不是 CLI 工具，也不是单纯的提示词集合。它通过一组公共 Skill、Rule、Router 和 Template，帮助任意业务项目在本地建立自己的 AI 协作空间，让 Agent 能在明确边界下理解项目、生成项目文档、抽象代码模式、受控生成代码，并逐步沉淀项目本地 Skill。

一句话定义：

> HACF 是一个公共 Skill 孵化框架，用来帮助项目建立自己的 Agent — 项目文档 — 项目源码 — 项目本地 Skill 协作体系。

---

## 2. 核心理念

HACF 的目标不是让业务项目长期依赖公共 Skill，而是让公共 Skill 帮助项目形成自己的本地协作能力。

推荐演进路径：

```text
公共 HACF Skill 库
  ↓
接入业务项目
  ↓
建立项目本地 .ai 协作空间
  ↓
确认项目约束和语言策略
  ↓
扫描项目、模块和关键文件
  ↓
识别固定代码类型
  ↓
抽象 Pattern Pack
  ↓
校验并人工激活 Pattern Pack
  ↓
基于 Pattern 受控生成代码
  ↓
复盘生成结果
  ↓
沉淀项目本地 Skill
  ↓
项目本地 Skill 接管高频任务
  ↓
定期健康检查和本地化等级评估
```

最终目标是：

```text
让 Agent 在项目规则、项目文档、项目 Pattern 和项目本地 Skill 的约束下自主协作。
```

---

## 3. 核心原则

### 3.1 公共库保持纯净

公共 HACF 库只保存通用能力：

- 通用 Skill；
- 通用 Rule；
- 通用 Router；
- 通用 Template；
- 公共库版本和变更记录。

公共库不保存：

- 业务项目扫描结果；
- 业务项目文档；
- 业务项目 Pattern Pack；
- 业务项目本地 Skill；
- 业务项目状态、报告或私有配置。

除非用户明确要求维护 HACF 本身，否则 Agent 不应修改公共 HACF 库。

### 3.2 项目本地单目录原则

目标项目中的 AI 协作产物必须集中放在：

```text
.ai/
```

项目根目录只允许保留一个极薄入口文件：

```text
AGENTS.md
```

`AGENTS.md` 只负责指向：

```text
.ai/entry/AI_ENTRY.md
```

不应在项目根目录散落大量 AI 文档、Skill、Rule、Pattern 或报告。

### 3.3 项目本地优先

当同一主题同时存在项目本地材料和公共库材料时，Agent 应优先读取项目本地内容。

当前规则中的优先级链是：

```text
1. .ai/skills/project-local/
2. .ai/rules/project-local/
3. .ai/config/
4. .ai/docs/ 与 .ai/indexes/
5. HACF 公共库 skills/
6. HACF 公共库 rules/
```

也就是说：

> 项目本地 Skill 优先，公共 Skill 兜底。

### 3.4 源码永远是事实源

`.ai/docs/` 中的文档是辅助上下文，不是最终事实源。

规则：

- 修改代码前必须读取真实源码；
- 如果文档和源码冲突，以源码为准；
- 如果发现文档过期，需要标记或更新；
- 不允许仅根据 AI 文档直接修改代码。

### 3.5 中文描述，代码原文保留

HACF 推荐：

- Skill、Rule、文档、Pattern 说明使用中文；
- 文件路径、类名、方法名、枚举名、配置名、API 字段名保持原文；
- 技术术语可以中英混写，例如 Agent、Skill、Pattern、Router、Context、Validator。

---

## 4. 当前仓库结构

### 4.1 推荐工作区布局

推荐把 HACF 公共库和业务项目放在同一工作区下的平行目录中：

```text
workspace/
  HACF/                 # 公共 HACF Skill 库，即 {skillLibraryRoot}
    ENTRY.md
    VERSION.md
    CHANGELOG.md
    routers/
    skills/
    rules/
    templates/

  target-project/       # 业务项目，即 {projectRoot}
    AGENTS.md
    .ai/
    src/
    ...
```

路径变量：

- `{skillLibraryRoot}`：包含 `ENTRY.md` 的 HACF 公共库根目录。
- `{projectRoot}`：业务项目根目录，通常包含 `.git`、`package.json`、`.sln` 等，或由开发者显式指定。

### 4.2 HACF 公共库目录

当前 0.2.7 仓库结构：

```text
HACF/
  ENTRY.md
  VERSION.md
  CHANGELOG.md

  routers/
    SKILL_ROUTER.md

  skills/
    bootstrap/
    scan/
    pattern/
    state/
    health/

  rules/
    base/
    bootstrap/
    documentation/
    project/
    pattern/
    health/

  templates/
    agents/
    project-ai-context/
    config/
    docs/
    indexes/
    pattern-packs/
    reports/
    state/
```

注意：当前仓库没有 `workflows/` 目录，任务编排以 `routers/SKILL_ROUTER.md` 和各 Skill 文档为准。

### 4.3 项目本地 `.ai/` 结构

项目侧目录不是一次性全部创建。Bootstrap 只创建最小闭环，Scan、Pattern、Health 等阶段会按需补齐目录和文件。

典型完整结构如下：

```text
.ai/
  entry/
    AI_ENTRY.md
    SKILLKIT_LINK.md

  state/
    skillkit-status.md
    readiness.md
    onboarding-status.md
    onboarding-checklist.md
    onboarding-questions.md
    scan-status.md
    code-type-detection-status.md
    pattern-extraction-status.md
    pattern-validation-status.md
    pattern-activation-status.md
    pattern-code-generation-status.md
    project-local-skill-promotion-status.md
    localization-level.md
    project-ai-health-status.md

  config/
    language-policy.md
    project-profile.md
    hard-constraints.md
    danger-zones.md
    architecture-rules.md
    coding-style.md
    testing-style.md
    domain-glossary.md
    module-map.md
    code-type-registry.md
    lifecycle-map.md
    code-placement.md

  docs/
    project/
      overview.md
      deep-dive.md
      adapter.md
      adapter-evidence.md
    modules/
      <moduleId>/
        overview.md
        deep-dive.md
        constraints.md
        change-guide.md
        risk-points.md
        code-types.md
    files/

  indexes/
    directory-index.md
    module-index.md
    file-index.md
    code-type-index.md
    pattern-index.md
    skill-index.md

  pattern-packs/
    <codeTypeId>/
      pattern.md
      validator.md
      examples.md
      docs.md

  skills/
    project-local/
      create-<codeTypeId>.md

  reports/
    pattern-validation-report.md
    pattern-code-generation-plan.md
    pattern-code-generation-result.md
    pattern-code-generation-review.md
    project-ai-health-report.md
```

---

## 5. 当前主干能力

当前 HACF 0.2.7 主干能力分为六类：Bootstrap、Scan、Pattern、Docs、State / 本地化、Health。

### 5.1 Bootstrap：项目接入

用于把公共 HACF 库接入目标项目，建立最小可用 `.ai/` 协作空间。

核心 Skill：

```text
skills/bootstrap/load-skill-library.md
skills/bootstrap/create-or-update-agents-md.md
skills/bootstrap/initialize-project-ai-context.md
skills/bootstrap/check-project-readiness.md
skills/bootstrap/guide-project-onboarding.md
skills/bootstrap/recommend-next-onboarding-step.md
```

主要产出：

```text
AGENTS.md
.ai/entry/AI_ENTRY.md
.ai/entry/SKILLKIT_LINK.md
.ai/state/skillkit-status.md
.ai/state/readiness.md
.ai/state/onboarding-status.md
.ai/state/onboarding-checklist.md
.ai/state/onboarding-questions.md
.ai/config/language-policy.md
```

说明：

- `load-skill-library` 适合一次性完成 Bootstrap 最小闭环。
- `guide-project-onboarding` 适合分阶段、带人工确认地完成接入流程。
- `recommend-next-onboarding-step` 是只读推荐，不写文件。

### 5.2 Scan：项目、模块、文件扫描

用于让 Agent 建立项目理解，生成项目文档和索引。Scan 阶段必须只读源码，产物只能写入 `.ai/`。

核心 Skill：

```text
skills/scan/scan-project-by-ai.md
skills/scan/generate-project-adapter-by-ai.md
skills/scan/scan-module-by-ai.md
skills/scan/scan-file-by-ai.md
```

主要产出：

```text
.ai/docs/project/overview.md
.ai/docs/project/deep-dive.md
.ai/docs/project/adapter.md
.ai/docs/project/adapter-evidence.md
.ai/docs/modules/<moduleId>/overview.md
.ai/docs/modules/<moduleId>/deep-dive.md
.ai/docs/modules/<moduleId>/constraints.md
.ai/docs/modules/<moduleId>/change-guide.md
.ai/docs/modules/<moduleId>/risk-points.md
.ai/docs/files/...
.ai/indexes/directory-index.md
.ai/indexes/module-index.md
.ai/indexes/file-index.md
.ai/state/scan-status.md
```

注意：HACF 中“模块”是逻辑边界，不一定是单一目录。

### 5.3 Pattern：代码类型、Pattern Pack 和受控生成

用于识别固定代码类型，抽象 Pattern Pack，校验并激活 Pattern，然后基于 active Pattern 受控生成代码。

核心 Skill：

```text
skills/pattern/detect-code-types-by-ai.md
skills/pattern/create-pattern-pack-draft.md
skills/pattern/extract-code-pattern.md
skills/pattern/validate-pattern-pack.md
skills/pattern/activate-pattern-pack.md
skills/pattern/create-code-by-pattern.md
skills/pattern/review-pattern-code-generation.md
skills/pattern/promote-project-local-skill.md
skills/pattern/activate-project-local-skill.md
```

主要产出：

```text
.ai/docs/modules/<moduleId>/code-types.md
.ai/indexes/code-type-index.md
.ai/state/code-type-detection-status.md

.ai/pattern-packs/<codeTypeId>/pattern.md
.ai/pattern-packs/<codeTypeId>/validator.md
.ai/pattern-packs/<codeTypeId>/examples.md
.ai/pattern-packs/<codeTypeId>/docs.md
.ai/state/pattern-extraction-status.md

.ai/state/pattern-validation-status.md
.ai/reports/pattern-validation-report.md

.ai/indexes/pattern-index.md
.ai/state/pattern-activation-status.md

.ai/reports/pattern-code-generation-plan.md
.ai/reports/pattern-code-generation-result.md
.ai/reports/pattern-code-generation-review.md
.ai/state/pattern-code-generation-status.md

.ai/skills/project-local/create-<codeTypeId>.md
.ai/state/project-local-skill-promotion-status.md
.ai/indexes/skill-index.md
```

关键门禁：

- Pattern Pack 初始必须是 `draft`。
- Pattern Pack 必须经过 `validate-pattern-pack` 校验。
- Pattern Pack 必须由人类确认后才能 `active`。
- `create-code-by-pattern` 必须先生成计划。
- 高风险或实际改源码必须等待明确人工确认。
- 修改后必须执行 `validator.md` checklist 并记录结果。

### 5.4 State / 本地化

用于评估项目 AI 协作能力是否已从公共库迁移到项目本地。

核心 Skill：

```text
skills/state/evaluate-localization-level.md
```

主要产出：

```text
.ai/state/localization-level.md
.ai/state/skillkit-status.md
```

`skillkit-status.md` 会合并更新：

```text
localizationLevel
localizationEvaluatedAt
```

常见等级语义：

```text
L0：仅完成公共库接入
L1：已有项目文档和索引
L2：已有 active Pattern Pack
L3：已有 active 项目本地 Skill，部分高频任务由本地 Skill 接管
```

### 5.5 Health：项目 AI 协作框架健康检查

用于检查项目本地 `.ai/` 是否健康、索引是否断链、状态是否一致、Pattern Pack 和项目本地 Skill 是否可用。

核心 Skill：

```text
skills/health/check-project-ai-health.md
```

主要产出：

```text
.ai/reports/project-ai-health-report.md
.ai/state/project-ai-health-status.md
```

健康检查默认只报告，不自动修复，不改源码，不改公共库。

---

## 6. 推荐落地流程

### 阶段 0：准备 HACF 公共库

从 Git 拉取 HACF 公共库，并放在业务项目旁边：

```text
workspace/
  HACF/
  target-project/
```

业务项目中由 Agent 只读打开：

```text
{skillLibraryRoot}/ENTRY.md
```

入口文件会提示第一步读取：

```text
{skillLibraryRoot}/skills/bootstrap/load-skill-library.md
```

### 阶段 1：加载 HACF 到项目

在业务项目根目录对 Agent 说：

```text
请读取 ../HACF/skills/bootstrap/load-skill-library.md，
将 HACF 公共 Skill 库加载到当前项目。

要求：
1. 当前项目为 projectRoot。
2. 公共库路径为 ../HACF。
3. 生成或更新极薄 AGENTS.md。
4. 所有项目 AI 协作产物只能写入 .ai/。
5. 不得修改 HACF 公共库。
```

完成后检查：

```text
AGENTS.md
.ai/entry/AI_ENTRY.md
.ai/entry/SKILLKIT_LINK.md
.ai/state/skillkit-status.md
.ai/config/language-policy.md
```

如果希望更稳地分步接入，可以改用：

```text
请执行 HACF 的 guide-project-onboarding。
要求分阶段推进，每个需要人工确认的步骤都先问我，不进入 Scan / Pattern / Develop。
```

### 阶段 2：执行 readiness 检查

```text
请执行 HACF 的 check-project-readiness。

要求：
1. 检查 .ai/config/ 下的必要配置。
2. 引导我确认项目基础信息、硬约束、高风险区域、语言策略。
3. 未经我确认的配置保持 draft。
4. 已确认配置更新为 reviewedByHuman: true。
5. 更新 .ai/state/readiness.md 和 .ai/state/skillkit-status.md。
```

通常需要确认：

```text
项目类型
主要语言和框架
主要源码目录
忽略目录
只读文件
高风险区域
架构边界
代码风格
业务术语
模块候选
文档语言策略
```

项目至少达到 `constraints_ready` 后，再进入 Scan。

### 阶段 3：项目级扫盘

```text
请执行 HACF 的 scan-project-by-ai，对当前项目进行项目级扫盘。

要求：
1. 先读取 AGENTS.md 和 .ai/entry/AI_ENTRY.md。
2. 确认 readiness 至少为 constraints_ready。
3. 使用受控递归扫描策略。
4. 忽略 bin、obj、.git、.vs、node_modules、packages 等目录。
5. 重要目录可以重置局部扫描预算。
6. 所有产物只能写入 .ai/。
7. 不得修改源码。
8. 不得修改 HACF 公共库。
```

预期产出：

```text
.ai/docs/project/overview.md
.ai/docs/project/deep-dive.md
.ai/indexes/directory-index.md
.ai/indexes/module-index.md
.ai/indexes/file-index.md
.ai/state/scan-status.md
```

### 阶段 4：生成项目 Adapter

Adapter 是更紧凑的项目画像，适合给后续 Agent 快速建立上下文。

```text
请执行 HACF 的 generate-project-adapter-by-ai。

要求：
1. 读取项目级扫盘结果和关键索引。
2. 生成紧凑项目画像和证据表。
3. 所有产物只能写入 .ai/docs/project/。
4. 不修改源码。
```

预期产出：

```text
.ai/docs/project/adapter.md
.ai/docs/project/adapter-evidence.md
```

### 阶段 5：模块级扫盘

选择一个核心逻辑模块，例如：

```text
auth
device
config
ui
workflow
```

然后对 Agent 说：

```text
请执行 HACF 的 scan-module-by-ai。

输入：
- moduleId: <moduleId>

要求：
1. 模块是逻辑边界，不等同于单一目录。
2. 优先读取 .ai/config/module-map.md。
3. 同时参考 module-index、file-index、directory-index。
4. 扫描 primaryPaths、relatedPaths、entryFiles、registryFiles、configFiles。
5. 文档输出到 .ai/docs/modules/<moduleId>/。
6. 不得使用源码目录名直接替代模块 ID，除非它确实是业务逻辑边界。
7. 不修改源码。
```

预期产出：

```text
.ai/docs/modules/<moduleId>/overview.md
.ai/docs/modules/<moduleId>/deep-dive.md
.ai/docs/modules/<moduleId>/constraints.md
.ai/docs/modules/<moduleId>/change-guide.md
.ai/docs/modules/<moduleId>/risk-points.md
```

### 阶段 6：单文件扫盘

当某个文件风险高、入口关键或频繁被修改时，可以单独建档。

```text
请执行 HACF 的 scan-file-by-ai。

输入：
- filePath: <相对项目根路径>
- fileImportance: critical

要求：
1. 读取真实源码文件。
2. 参考已有 project / module 文档和索引。
3. 输出到 .ai/docs/files/ 下。
4. 不修改源码。
```

`fileImportance` 可取：

```text
normal
critical
```

### 阶段 7：识别代码类型

```text
请执行 HACF 的 detect-code-types-by-ai。

输入：
- moduleId: <moduleId>

目标：
识别该模块中是否存在固定代码类型。

要求：
1. 读取模块文档和真实源码样例。
2. 输出候选 codeType。
3. 标记证据强度。
4. 不创建 Pattern Pack。
5. 不修改源码。
```

预期产出：

```text
.ai/docs/modules/<moduleId>/code-types.md
.ai/indexes/code-type-index.md
.ai/state/code-type-detection-status.md
```

### 阶段 8：创建或抽象 Pattern Pack 草稿

如果只需要先放一个结构骨架：

```text
请执行 HACF 的 create-pattern-pack-draft。

输入：
- moduleId: <moduleId>
- codeTypeId: <codeTypeId>

要求：
1. 基于 code-types.md 和 code-type-index.md 创建四文件草稿。
2. 不深读源码。
3. 不覆盖已有主文件；如冲突，写入冲突说明。
4. 不修改源码。
```

如果要基于真实样例抽象可用 Pattern：

```text
请执行 HACF 的 extract-code-pattern。

输入：
- moduleId: <moduleId>
- codeTypeId: <codeTypeId>

要求：
1. 读取 code-types.md。
2. 读取模块文档。
3. 深读真实源码样例。
4. 抽象 Pattern Pack 草稿。
5. status 必须为 draft。
6. reviewedByHuman 必须为 false。
7. 不修改源码。
```

预期产出：

```text
.ai/pattern-packs/<codeTypeId>/pattern.md
.ai/pattern-packs/<codeTypeId>/validator.md
.ai/pattern-packs/<codeTypeId>/examples.md
.ai/pattern-packs/<codeTypeId>/docs.md
.ai/state/pattern-extraction-status.md
```

### 阶段 9：校验并激活 Pattern Pack

先校验：

```text
请执行 HACF 的 validate-pattern-pack。

输入：
- codeTypeId: <codeTypeId>

要求：
1. 只读 Pattern Pack 四文件。
2. 检查结构完整性。
3. 检查 validator 是否可执行。
4. 输出校验报告。
5. 不激活 Pattern。
```

预期产出：

```text
.ai/state/pattern-validation-status.md
.ai/reports/pattern-validation-report.md
```

校验通过后，人工确认：

```text
我确认 <codeTypeId> Pattern Pack 可以激活。

请执行 activate-pattern-pack，只更新 metadata：
- status: active
- reviewedByHuman: true
- confidence: confirmed
- lastReviewedAt: 今天日期

不要修改正文。
不要修改源码。
不要修改 HACF 公共库。
```

预期更新：

```text
.ai/pattern-packs/<codeTypeId>/*.md
.ai/indexes/code-type-index.md
.ai/indexes/pattern-index.md
.ai/state/pattern-activation-status.md
```

### 阶段 10：基于 Pattern 生成计划

```text
请基于已激活的 <codeTypeId> Pattern Pack，模拟一次代码生成计划。

要求：
1. 使用 create-code-by-pattern。
2. 模式为 plan-only。
3. 不修改源码。
4. 必须读取 Pattern Pack 四文件。
5. 必须读取 validator.md。
6. 必须读取真实源码和联动文件。
7. 在计划中加入执行前“文档同步预判”，覆盖项目文档、模块文档、文件文档、索引、Pattern Pack、项目本地 Skill 六类对象。
8. 如果信息不足，列出缺失项，不要猜。
```

预期产出：

```text
.ai/reports/pattern-code-generation-plan.md
.ai/state/pattern-code-generation-status.md
```

### 阶段 11：小范围真实代码生成

确认计划稳定后，执行：

```text
请执行 HACF 的 create-code-by-pattern。

模式：apply-after-confirmation

输入：
- moduleId: <moduleId>
- codeTypeId: <codeTypeId>
- userRequirement: <你的需求>

要求：
1. 先展示最终变更计划。
2. 等我明确回复“确认执行”后再修改源码。
3. 修改范围必须严格限制在计划内。
4. 修改后执行 validator.md checklist。
5. 若实际改动范围超出计划中的文档同步预判，重新说明是否建议同步文档或运行 doc-impact 正式检查。
6. 输出结果报告。
```

预期产出：

```text
.ai/reports/pattern-code-generation-plan.md
.ai/reports/pattern-code-generation-result.md
.ai/state/pattern-code-generation-status.md
```

### 阶段 12：生成结果复盘

```text
请执行 HACF 的 review-pattern-code-generation，对刚才的 create-code-by-pattern 结果做一次复盘。

要求：
1. 读取生成计划、生成结果、Pattern Pack、validator 和实际修改文件。
2. 检查是否符合 Pattern。
3. 检查是否漏掉联动文件。
4. 检查是否需要更新 Pattern。
5. 判断是否适合沉淀项目本地 Skill。
6. 输出复盘报告。
7. 不修改源码。
```

预期产出：

```text
.ai/reports/pattern-code-generation-review.md
```

### 阶段 13：沉淀项目本地 Skill

如果复盘结论是稳定、可重复、适合本地化：

```text
请执行 HACF 的 promote-project-local-skill。

输入：
- moduleId: <moduleId>
- codeTypeId: <codeTypeId>

输出：
- .ai/skills/project-local/create-<codeTypeId>.md

要求：
1. 初始 status 为 draft。
2. reviewedByHuman 为 false。
3. 引用 Pattern Pack，不复制 Pattern 全文。
4. 必须声明读取范围、写入范围、禁止事项、validator checklist。
5. 不修改源码。
6. 不修改 HACF 公共库。
```

预期产出：

```text
.ai/skills/project-local/create-<codeTypeId>.md
.ai/state/project-local-skill-promotion-status.md
```

### 阶段 14：激活项目本地 Skill

人工确认后：

```text
我确认 .ai/skills/project-local/create-<codeTypeId>.md 可以激活。

请执行 HACF 的 activate-project-local-skill，只更新 metadata：
- status: active
- reviewedByHuman: true
- confidence: confirmed
- lastReviewedAt: 今天日期

并更新：
- .ai/indexes/skill-index.md

不要修改源码。
不要修改 HACF 公共库。
```

预期更新：

```text
.ai/skills/project-local/create-<codeTypeId>.md
.ai/indexes/skill-index.md
```

### 阶段 15：评估本地化等级

项目本地 Skill 激活后，建议执行：

```text
请执行 HACF 的 evaluate-localization-level。

要求：
1. 读取 active Pattern Pack。
2. 读取 .ai/skills/project-local/ 和 .ai/indexes/skill-index.md。
3. 更新 .ai/state/localization-level.md。
4. 合并更新 .ai/state/skillkit-status.md 中的 localizationLevel / localizationEvaluatedAt。
5. 不修改源码。
```

### 阶段 16：健康检查

项目运行一段时间后，建议执行：

```text
请执行 HACF 的 check-project-ai-health。

要求：
1. 检查入口健康、单目录规范、配置、状态、文档、索引、Pattern Pack、项目本地 Skill 和风险控制。
2. 只输出报告和状态。
3. 不自动修复。
4. 不修改源码。
5. 不修改 HACF 公共库。
```

预期产出：

```text
.ai/reports/project-ai-health-report.md
.ai/state/project-ai-health-status.md
```

---

## 7. 常用用户回复模板

### 7.1 确认配置

```text
针对本次待确认项，我的确认如下：

- 项目类型：确认，是 <项目类型>。
- 主要源码目录：确认，<路径>。
- 忽略目录：<路径列表>。
- 只读文件：<规则列表>。
- 高风险区域：<模块或路径>。
- 测试目录：未知，标记为 unknown。

请根据以上内容更新 .ai/config/、.ai/state/ 和相关索引。
不要修改源码。
不要修改 HACF 公共库。
```

### 7.2 激活 Pattern Pack

```text
我确认 <codeTypeId> Pattern Pack 可以激活。

请只更新 Pattern Pack 四文件 metadata：
- status: active
- reviewedByHuman: true
- confidence: confirmed
- lastReviewedAt: 今天日期

不要修改正文。
不要修改源码。
不要修改 HACF 公共库。
```

### 7.3 确认执行代码生成

```text
我确认执行本次 create-code-by-pattern 计划。

要求：
1. 只修改计划中列出的文件。
2. 不做无关重构。
3. 修改后执行 validator checklist。
4. 输出生成结果报告。
```

### 7.4 激活项目本地 Skill

```text
我确认 .ai/skills/project-local/create-<codeTypeId>.md 可以激活。

请只更新 metadata：
- status: active
- reviewedByHuman: true
- confidence: confirmed
- lastReviewedAt: 今天日期

并更新 .ai/indexes/skill-index.md。
不要修改源码。
不要修改 HACF 公共库。
```

### 7.5 询问下一步

```text
请执行 HACF 的 recommend-next-onboarding-step。

要求：
1. 读取当前 .ai/state/ 和 .ai/entry/。
2. 判断下一步最适合执行哪个 Skill。
3. 只给建议，不写文件。
```

---

## 8. 推荐运行节奏

第一次接入项目时，不要急于生成代码。推荐顺序：

```text
1. load-skill-library 或 guide-project-onboarding
2. check-project-readiness
3. scan-project-by-ai
4. generate-project-adapter-by-ai
5. scan-module-by-ai
6. scan-file-by-ai（按需）
7. detect-code-types-by-ai
8. create-pattern-pack-draft（可选）
9. extract-code-pattern
10. validate-pattern-pack
11. activate-pattern-pack
12. create-code-by-pattern（plan-only）
13. create-code-by-pattern（apply-after-confirmation）
14. review-pattern-code-generation
15. promote-project-local-skill
16. activate-project-local-skill
17. evaluate-localization-level
18. check-project-ai-health
```

---

## 9. 当前阶段判断

如果一个项目已经完成：

```text
load
readiness
scan-project
scan-module
detect-code-types
extract-pattern
validate-pattern
activate-pattern
create-code-by-pattern
review-pattern-code-generation
promote-project-local-skill
activate-project-local-skill
evaluate-localization-level
```

则项目可以认为进入：

```text
skill_localized
```

如果多个高频任务都已由项目本地 Skill 接管，并且健康检查持续通过，则可以认为进入：

```text
self_governed
```

---

## 10. HACF 的价值总结

HACF 解决的是 Agent 开发中的长期问题：

```text
Agent 不理解项目
Agent 找不到关键文件
Agent 不知道团队约束
Agent 生成代码风格不一致
Agent 漏掉联动文件
Agent 修改范围失控
Agent 每次都从零开始读项目
AI 协作产物散落且难维护
```

HACF 通过以下方式解决：

```text
公共 Skill 提供通用方法
项目 .ai 沉淀本地上下文
项目文档帮助 Agent 和人类理解代码
Pattern Pack 固化固定代码类型
Validator 限制生成质量
项目本地 Skill 接管高频任务
本地化等级评估展示成熟度
健康检查保证框架可用
```

最终形成：

> 人类确认关键约束，Agent 在约束内自主执行，项目不断沉淀自己的 AI 协作能力。
