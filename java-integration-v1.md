# Lean Coding Workflow — External Skills & MCP Integration

## 1. 背景

当前 `lean-coding-workflow` 负责 Coding Agent 的开发生命周期管理：

```text
Phase 1 Requirement
    ↓
Confirm ①
    ↓
Phase 2 Planning
    ↓
Confirm ②
    ↓
Phase 3 Execution
    ↓
Phase 4 Finish
```

实际 Java 后端开发过程中，目前存在两个可以复用的外部能力：

### Lanhu MCP

外部仓库：[dsphper/lanhu-mcp](https://github.com/dsphper/lanhu-mcp)

负责从蓝湖获取：

- 产品原型
- 页面信息
- 页面标注
- 交互说明

用于解决当前实际工作中：

> 产品经理没有完整 PRD，需求主要存在于蓝湖原型及标注中。

### backend-skill

外部仓库：[zhangloveyan/backend-skill](https://github.com/zhangloveyan/backend-skill)

提供 Java 后端研发相关 Skills，包括：

- `proj-analyze-req`
- `proj-analyze-design`
- `proj-gen`
- `proj-review`
- `proj-gen-test`
- `proj-change`
- 其他 Java Backend Skills

这些能力与 Lean Coding Workflow 并不冲突。

Lean Coding Workflow 负责：

> Workflow / Lifecycle / Context / Progress

backend-skill 负责：

> Java Backend 专业能力

Lanhu MCP 负责：

> Requirement Source

因此计划在长期维护的 `java-integration` 分支中，让 Lean Coding Workflow 在适当阶段调用以上外部仓库提供的能力，而不是复制、修改或重新实现这些能力。仓库地址用于能力发现和集成定位，不代表将其源码纳入 Lean Coding Workflow。

---

# 2. 目标

实现一个面向 Java Backend、支持外部 MCP / Skill 协作的 Lean Coding Workflow 集成分支。

最终希望用户可以通过类似：

```text
开始这个需求：

<蓝湖 URL>
```

触发完整开发流程。

Workflow 根据当前 Phase 自动选择需要的外部能力，而无需用户重复编写：

```text
调用 Lanhu MCP 获取需求

使用 proj-analyze-req

使用 proj-analyze-design

使用 proj-gen

使用 proj-review

使用 proj-change
```

等提示词。

---

# 3. 核心原则

## 3.1 Lean Coding Workflow 仍然是主 Workflow

Lean 负责：

- 当前处于哪个 Phase
- 当前应该执行什么
- 是否允许修改代码
- 是否需要用户确认
- 当前 Step
- 开发进度
- 上下文恢复
- 需求变更后的恢复

外部 Skill 不接管 Workflow 生命周期。

---

## 3.2 External Skill 保持独立

不得要求修改：

```text
backend-skill
```

才能使用 Lean。

同时不得将 backend-skill 的所有内容直接复制到 Lean Coding Workflow。

应该允许：

```text
backend-skill
```

继续独立使用。

Lean 只在适当阶段：

```text
discover
→ select
→ invoke/reference
```

对应 Skill。

GitHub 仓库地址只用于定位来源、安装和版本追溯，不能直接作为运行时 Skill 调用入口。`backend-skill` 应独立安装在当前 Agent 或目标项目可发现的 Skill 目录中，不得下载、复制或内嵌到 `lean-coding-workflow` 仓库。Workflow 运行时先发现已安装的 Skill，再按当前 Phase 调用；未发现时使用 Lean 自身等价能力继续执行。

---

## 3.3 Lanhu MCP 保持独立

不得将 Lanhu MCP 实现代码复制进入 Lean。

Lean 只负责：

```text
识别 Lanhu Requirement Source
        ↓
调用 Lanhu MCP
        ↓
获取 Requirement Facts
```

Lanhu MCP 仍然可以脱离 Lean 独立使用。

---

## 3.4 Lanhu 不是唯一 Requirement Source

Requirement Source 应保持开放。

支持：

```text
Requirement Source

├── User Prompt
├── PRD.md
├── Lanhu
├── Issue
└── Future Sources
```

Lanhu 只是其中一种 Source Adapter。

---

## 3.5 项目技术栈优先原则

Java Integration 在调用任何外部 Java Skill 前，必须优先识别并遵循当前项目已有技术栈、依赖版本、代码规范和架构约束。约束优先级固定为：

```text
实际项目配置 / 现有代码约束
    >
项目 AGENTS.md / architecture.md
    >
Lean Workflow
    >
backend-skill 默认技术栈
```

backend-skill 中声明的以下技术栈仅是其默认参考环境：

```text
Java 17
Spring Boot 3.x
MyBatis-Plus 3.5.x
Redis 7.x
Knife4j 4.x
JUnit 5 + Mockito + AssertJ
```

这些默认版本不得覆盖当前项目实际配置，也不得因为调用 Skill 自动触发框架、依赖或测试体系升级。

调用 `proj-analyze-req`、`proj-analyze-design`、`proj-gen`、`proj-review`、`proj-gen-test`、`proj-change` 时，都必须服从当前项目识别出的实际技术栈。

---

## 3.6 Project Discovery / Dynamic Project Context

在调用外部 Java Skill 前，Workflow 必须针对当前打开的项目动态执行 Project Discovery，不得在 `java-integration` 中预设或写死任何项目的版本。至少读取：

```text
pom.xml
application.yml
application-*.yml
已有代码
测试目录
AGENTS.md
architecture.md
```

并据此识别：

- Java 版本
- Spring Boot 版本
- ORM 及其版本
- 数据库及其版本
- Redis 版本
- API 文档方案及其版本
- 测试框架及其版本
- 现有代码规范和架构约束

识别结果形成当前调用所需的 Dynamic Project Context，再作为外部 Java Skill 的输入约束。其结构可以表示为：

```yaml
project_context:
  java: <discovered-from-current-project>
  spring_boot: <discovered-from-current-project>
  orm: <discovered-from-current-project>
  database: <discovered-from-current-project>
  redis: <discovered-from-current-project>
  api_doc: <discovered-from-current-project>
  test:
    framework: <discovered-from-current-project>
```

Dynamic Project Context 不是新的状态或持久化文件。每次调用外部 Java Skill 前，Workflow 根据当前 Step 按需读取项目配置、约定文件和相关源码，动态形成此次调用所需的技术栈约束；每个版本结论必须可追溯到当前项目文件或现有代码，不得复用其他项目或历史聊天中的版本值。

恢复任务时继续沿用 Lean 原有方式：读取 `execution-plan.md`，找到第一个存在非 `[✅]` 验收项的 Step，再按当前 Step 需要重新读取项目事实。不得新增 Project Context 文件、checkpoint 文件或独立恢复状态。

---

## 3.7 禁止普通需求隐式升级

普通需求开发中禁止隐式升级：

- Java
- Spring Boot
- Redis
- 核心依赖
- 测试框架

如果 Skill 推荐能力与当前项目版本不兼容，应优先采用当前项目兼容实现。若确实需要升级，必须作为独立技术变更或架构变更提出，明确影响范围、迁移方案和验证方式，经确认后执行，不得夹带在普通需求开发中完成。

---

# 4. 外部能力职责

## Lanhu MCP

职责：

```text
Requirement Source
```

提供：

- 原型
- 页面
- 标注
- 交互信息

不得：

- 自行生成技术方案
- 管理 execution-plan
- 控制 Workflow Phase

---

## backend-skill

职责：

```text
Domain Capability
```

backend-skill 提供 Workflow Capability，不拥有当前项目技术栈的最终决定权；其默认参考环境不得覆盖 Dynamic Project Context。

优先使用：

### Requirement

```text
proj-analyze-req
```

### Design

```text
proj-analyze-design
```

### Implementation

```text
proj-gen
```

### Review

```text
proj-review
```

### Test

```text
proj-gen-test
```

`proj-gen-test` 只负责新增或更新测试代码。实际测试由 Lean 按当前项目的 Maven / Gradle 配置执行。

### Requirement Change

```text
proj-change
```

---

# 5. Workflow

在进入任一 Java 专业能力前执行：

```text
Project Discovery
    ↓
Dynamic Project Context
    ↓
Lean Workflow
    ↓
backend-skill
```

每次调用所形成的 Dynamic Project Context 对该次调用生效；后续调用按当前 Step 重新按需识别。

## Phase 1 — Requirement

输入可能来自：

```text
PRD
Lanhu
User Prompt
Other Sources
```

如果检测到 Lanhu URL：

```text
Lanhu URL
    ↓
Lanhu MCP
    ↓
原型 + 标注
    ↓
Requirement Facts
```

然后可以使用：

```text
proj-analyze-req
```

辅助需求分析。

最终仍然生成 Lean 的：

```text
prd-sub.md
```

其中必须区分：

```text
Confirmed Requirement
Code-derived Fact
Technical Inference
Unknown / Need Confirmation
```

不得把 AI 推断直接视为产品需求。

完成后：

```text
Confirm ①
```

---

# 6. Phase 2 — Planning

输入：

```text
prd-sub.md
+
Existing Codebase
+
Dynamic Project Context
```

可使用：

```text
proj-analyze-design
```

分析：

- API
- DB
- Java
- Spring
- Redis
- MQ
- Transaction
- Permission
- Compatibility
- Existing Architecture

所有设计必须与当前项目动态识别出的版本兼容。

最终生成 Lean 的：

```text
execution-plan.md
```

必要时复杂需求可以增加：

```text
design.md
```

完成后：

```text
Confirm ②
```

---

# 7. Phase 3 — Execution

按照：

```text
execution-plan.md
```

逐 Step 执行。

所有涉及代码修改的 Step 必须完成以下闭环：

```text
proj-gen
    ↓
Implementation
    ↓
proj-review
    ↓
必要问题修复
    ↓
需要新增或更新测试时调用 proj-gen-test
    ↓
Lean 按当前项目的 Maven / Gradle 配置执行测试
    ↓
Acceptance
```

backend-skill 可用时，优先调用 `proj-review` 完成 Java 专项审查；需要新增或更新测试时调用 `proj-gen-test`。实际测试始终由 Lean 按当前项目的 Maven / Gradle 配置执行，Lean 负责根据 Review、测试结果和原验收标准进行最终验收，并更新 `execution-plan.md` 的验收勾选。backend-skill 不可用时，Lean 使用自身等价能力继续完成 Review 和测试代码工作，并明确记录本次未调用对应外部 Skill。所有能力均必须服从当前项目动态识别出的技术栈约束。

`proj-review` 必须按照当前项目实际版本、既有代码规范和架构约束审查，重点检查当前 Java、Spring Boot、ORM、Redis、API 文档与测试体系的兼容性。不得把 backend-skill 的默认参考环境作为升级依据，也不得仅因为默认环境不同而建议升级 Java、Spring Boot、Redis、核心依赖或测试框架。

全部验收通过后，将当前 Step 的验收项标记为 `[✅]`，再继续下一 Step。

---

# 8. execution-plan 是唯一进度 Source of Truth

backend-skill 中类似：

```text
proj-task
proj-resume
```

的能力不得产生第二套开发状态。

Lean 的：

```text
execution-plan.md
```

仍然是唯一开发进度 Source of Truth。

它记录：

- Step
- Acceptance
- Change Impact
- 验收勾选所表达的开发进度

Workflow 恢复时沿用 Lean 原有方式，优先读取：

```text
AGENTS.md
+
execution-plan.md
+
当前 Step 所需 Context
```

找到第一个存在非 `[✅]` 验收项的 Step 后，再按需读取当前 Step 涉及的项目配置、约定文件和相关源码。不得依赖历史聊天恢复开发状态，也不得新增独立状态或恢复文件。

---

# 9. Phase 3 Requirement Change

这是本版本重点能力。

当前 Lean Coding Workflow 在 Phase 3 发生需求变更时，不应重新生成 execution-plan 并覆盖原有开发状态。

需求变更必须进入：

```text
Change Flow
```

---

# 10. Change Flow

Phase 3 出现需求变更时，沿用 backend-skill 的简洁流程：

```text
1. proj-change：输入变更内容，输出影响分析和变更计划
2. 根据变更计划修改相关代码
3. proj-review：重新审查受影响代码
4. 需要时调用 proj-gen-test 更新测试；Lean 执行相关测试并完成验收
```

backend-skill 不可用时，Lean 使用自身等价能力完成以上步骤，并明确说明未调用对应外部 Skill。

Lean 只负责维护原有 Workflow 和 `execution-plan.md`：

- 不重新生成或覆盖整个 `execution-plan.md`。
- 对受影响的原验收项，将 `[✅]` 改回 `[ ]` 并注明对应 Change ID。
- 根据影响分析调整受影响 Step；确需新增工作时，只追加必要的 Change Step。
- 已完成且不受影响的 `[✅]` 保持不变。
- Review、测试和验收通过后继续原 Phase 3。

每次需求变更使用 `CR-001`、`CR-002` 等 Change ID，最少记录变更内容、影响范围、变更计划和受影响 Step。Change ID 只是变更追溯标记，不构成第二套状态。

不新增 `TODO / IN_PROGRESS / PAUSED / DONE / SUPERSEDED` 等状态，也不新增 checkpoint 文件。执行进度仍只由 `execution-plan.md` 中的 `[ ] / [✅] / [❌] / [⚠️]` 验收勾选表达。

需求歧义、重大架构或方案变化、计划与实际严重冲突、高风险操作或需要用户提供信息时，沿用 Lean 原有暂停请示规则；不额外引入固定 Confirm③。

---

# 11. Phase 4 — Finish

保持 Lean 原有 Finish 逻辑。

根据实际开发结果更新：

```text
architecture
memory
AGENTS
```

必要时同步相关项目文档。

Change History 应保留，不得在 Finish 时删除。

---

# 12. Skill 与 Workflow 边界

遵循：

```text
Workflow
=
When to do

Skill
=
How to do

Document
=
What we know / Where we are

MCP
=
External capability / information
```

例如：

```text
Lean:
现在需要分析需求变化。

proj-change:
需求变化应该如何分析，但实现必须服从 Dynamic Project Context。

Lanhu MCP:
产品原型实际上写了什么。

execution-plan:
当前已经开发到哪里。
```

不得混淆职责。

---

# 13. 非目标

本版本暂不：

- 修改 backend-skill
- 修改 Lanhu MCP
- Fork backend-skill
- 复制全部 backend-skill
- 重写 Lean Workflow
- 引入复杂 Multi-Agent Graph
- 引入独立 Gate Engine、任务系统或第二套进度状态
- 引入新的任务状态数据库
- 替换 execution-plan
- 在第一版集成 SonarQube、覆盖率阈值或复杂缺陷分级策略
- 强制项目必须使用 Lanhu
- 将 backend-skill 默认技术栈强加给现有项目
- 在普通需求开发中隐式升级项目技术栈

`java-integration` 是长期维护的 Java Backend 集成分支，会持续服务 Java 项目并整合 Lanhu MCP 与 backend-skill。

---

# 14. 兼容性

没有 Lanhu MCP 时：

```text
Lean Workflow
```

仍然可以正常使用。

没有 backend-skill 时：

```text
Lean Workflow
```

仍然可以执行完整 Workflow。需要 Java 专业能力时由 Lean 使用自身等价能力完成，并明确说明未调用对应外部 Skill。

安装 backend-skill 后：

```text
Java Capability Enhanced
```

安装 Lanhu MCP 后：

```text
Lanhu Requirement Source Enabled
```

两个同时存在：

```text
Lean
 +
Lanhu MCP
 +
backend-skill
```

获得完整 Java Backend Workflow。

技术栈兼容规则：

```text
backend-skill 默认环境
    ↓ 仅作参考
Project Discovery
    ↓
Dynamic Project Context
    ↓
当前项目兼容实现
```

所有分析、设计、生成、Review、测试和变更能力均按照 Project Discovery 从当前项目动态识别出的版本执行，不因 backend-skill 的默认参考环境而改变。

---

# 15. 目标使用体验

用户不需要：

```text
请调用 proj-analyze-req

现在进入 Phase 2

调用 proj-analyze-design

请调用 proj-gen

现在 proj-review

产品改需求了，请 proj-change
```

用户只需要正常表达：

```text
开发这个蓝湖需求：
<URL>
```

或者：

```text
继续开发
```

或者：

```text
产品改需求了：

预约成功后不要直接通知，
改成审批通过以后通知。
```

Workflow 自动判断：

```text
当前 Phase
+
当前 Context
+
Dynamic Project Context
+
Available Skills
+
Available MCP
```

选择正确能力，并确保外部 Java Skill 的输出与当前项目版本兼容。

---

# 16. 验收标准

## AC-01

给出 Lanhu URL 后，可以使用 Lanhu MCP 获取需求信息并进入 Phase 1。

## AC-02

Phase 1 可以使用 `proj-analyze-req`，但最终产物仍遵循 Lean Workflow。

## AC-03

Phase 2 可以使用 `proj-analyze-design` 辅助技术方案。

## AC-04

Phase 3 的代码修改 Step 在全部验收项标记 `[✅]` 前必须完成 Review、必要修复、测试和验收；backend-skill 可用时优先调用 `proj-review`，需要新增或更新测试时调用 `proj-gen-test`；实际测试由 Lean 按当前项目的 Maven / Gradle 配置执行，最终由 Lean 验收并更新勾选。

## AC-05

不安装 backend-skill 时 Lean 原流程仍可运行。

## AC-06

不安装 Lanhu MCP 时 Lean 原流程仍可运行。

## AC-07

backend-skill 可以脱离 Lean 独立使用。

## AC-08

Lanhu MCP 可以脱离 Lean 独立使用。

## AC-09

Phase 3 发生需求变更时不得覆盖原 execution-plan。

## AC-10

已完成 Step 的历史状态必须保留。

## AC-11

需求变更后，受影响 Step 的验收勾选和 Change ID 能够直接反映继续执行的位置，不新增独立 checkpoint 或状态。

## AC-12

需求变更通过 CR ID 记录。

## AC-13

使用 `proj-change` 或等价能力完成 Impact Analysis。

## AC-14

根据 `proj-change` 或等价能力输出的变更计划，只调整受影响的验收项和 Step，不覆盖整个执行计划。

## AC-15

Change 完成后，可以按 `execution-plan.md` 中第一个存在非 `[✅]` 验收项的 Step 继续执行。

## AC-16

调用外部 Java Skill 前，能够按当前 Step 需要读取 `pom.xml`、`application*.yml`、已有代码、测试目录、`AGENTS.md`、`architecture.md` 等项目事实，并动态形成可追溯的技术栈约束；不得新增独立 Project Context 文件。

## AC-17

针对技术栈版本不同的多个项目执行相同 Workflow 时，能够分别识别各项目版本；所有 `proj-analyze/design/gen/review/test/change` 能力均服从各次调用动态形成的技术栈约束，不复用或写死其他项目的版本，也不生成仅适用于 backend-skill 默认环境的实现。

## AC-18

普通需求开发不得隐式升级 Java、Spring Boot、Redis、核心依赖或测试框架；必要升级必须作为独立技术或架构变更提出并确认。

## AC-19

`proj-review` 按当前项目版本、现有代码规范和架构约束审查，不得因 backend-skill 默认环境而建议升级。

## AC-20

`main` 保持通用 Lean Workflow；`java-integration` 作为长期维护分支持续整合 Lanhu MCP 与 backend-skill，并能够定期同步 `main` 的通用能力。

---

# 17. 实施策略

`main` 与 `java-integration` 的定位如下：

```text
lean-coding-workflow
│
├── main
│   └── 通用、轻量、技术栈无关的 Lean Coding Workflow
│
└── java-integration
    ├── 长期维护
    ├── Java Backend Workflow
    ├── Lanhu MCP Requirement Source
    ├── backend-skill 编排
    └── Project Discovery / Dynamic Project Context
```

`java-integration` 不是开发完成后合并并删除的临时 feature 分支，而是公司真实 Java 项目长期使用和持续优化的 Java Backend 集成分支。

分支维护策略：

```text
main
  │
  │ 通用 Lean Workflow 改进
  ↓
java-integration
  │
  ├─ 长期整合 Lanhu MCP + backend-skill
  ├─ 在真实 Java 项目中持续验证
  └─ 定期同步 main 的通用能力
```

`java-integration` 中产生的技术栈无关改进可以提炼回 `main`；Java、Lanhu 默认入口、backend-skill 编排及 Dynamic Project Context 兼容机制继续保留在 `java-integration`。

第一阶段重点验证：

```text
Lean
  +
Project Discovery / Dynamic Project Context
  +
backend-skill
  +
Lanhu MCP
  +
Phase 3 Change Flow
```

建议至少使用 2～3 个真实需求验证。

重点观察：

- Skill 是否存在规则冲突
- Project Discovery 是否准确识别实际技术栈
- Skill 输出是否严格服从 Dynamic Project Context
- Lanhu Requirement 是否可靠
- Context 是否明显膨胀
- execution-plan 是否仍然足够 Lean
- Change Flow 是否能够正确恢复
- 是否存在重复状态文件
- 是否发生隐式依赖或框架升级

---

# 18. 第一版本优先级

P0：

```text
项目技术栈优先原则
Project Discovery / Dynamic Project Context
禁止普通需求隐式升级技术栈
所有 proj-* 能力服从当前项目技术栈
proj-review 按当前项目版本和既有规范审查
External Skill 调用机制
Lanhu Requirement Source
proj-analyze-req
proj-analyze-design
proj-change
Change Request
Patch execution-plan
沿用 execution-plan 验收勾选恢复任务
proj-gen
proj-review
proj-gen-test
长期维护 java-integration 分支并同步 main
```

P1：

```text
更多 Java 专业能力编排
```

P2：

```text
自动 Skill Discovery
更多 Requirement Source
更多语言 Profile
```

第一版优先解决：

> 在不污染当前项目技术栈的前提下，完成从需求来源、规划、编码、Review、测试、验收到需求变更的完整闭环，并确保开发过程中需求改变后不丢失进度。
