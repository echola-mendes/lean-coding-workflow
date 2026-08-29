---
name: lean-coding-workflow
description: "Document-driven coding workflow with on-demand context, one-step execution, minimal verification, and recoverable state."
---

# Lean Coding Workflow

## 0. 最高原则

1. **Never read context speculatively（禁止猜测性读取上下文）。**
   每次读取文件前必须能回答："如果不读取这个文件，我无法可靠完成当前任务。"
   如果答案只是"这个文件可能有帮助"，禁止读取。
2. **Context 一旦足够，立即停止读取。** Context Sufficient = YES 之后，任何"再确认一下"式的读取都违反本 Skill。
3. **Minimum Reliable Context，不是 Maximum Available Context。**
   目标是让 AI 在拥有完成当前任务所需最小可靠上下文的情况下正确完成任务，不是"知道整个项目"。
4. **一次只执行一个 Step。** 禁止一次性实现多个 Step，禁止顺手重构无关代码。
5. **验证与修改范围匹配。** 禁止每个 Step 都运行完整测试套件。
6. **不计算 Token。** 禁止用 `remaining_tokens` / `context_usage > X%` 之类的数字决定是否读取；
   只做语义判断："当前上下文是否已经足够可靠完成任务？"
7. **文档职责严格隔离。** 每个文件只有一个职责，禁止让多个文档记录相同内容。

## 1. 触发条件

满足任一即进入本工作流：

- 用户提出新的开发需求（可带 @xxx.md 需求文档），如 "@travel.md 实现差旅审批"
- 用户要求初始化项目工作流，或项目首次使用本 Skill → Phase 0
- 用户要求继续执行未完成的 execution-plan.md → Phase 3
- 用户报告 Bug、询问历史实现原因、要求回退/恢复 → Phase 5

**会话恢复判定**：用户说"继续"时，只读取 `artifacts/execution-plan.md`（它只含剩余 Step，很小）
即可判断当前进度，再进入对应 Phase。禁止为了恢复状态读取全部文档。

## 2. 项目目录约定

`PRD.md` / `AGENTS.md` 在项目根；其余 workflow 文件在 `artifacts/`。

| 路径 | 语义 | 生命周期 |
|---|---|---|
| PRD.md | 项目当前需求 | 长期 |
| @xxx.md / PRD-Vx.md | 用户指定的需求来源 | 用户所有（不清空） |
| AGENTS.md | AI 编码规则 | 长期 |
| artifacts/architecture.md | 长期架构约束 | 长期 |
| artifacts/prd-sub.md | 当前子需求 | 临时 |
| artifacts/plan.md | 当前子需求整体方案 | 临时 |
| artifacts/execution-plan.md | 剩余待执行 Step | 临时 |
| artifacts/memory.md | 当前子需求执行记忆 | 临时 |

需求入口规则：

- 用户 @ 指定的**任意** .md 文件 = 当前需求来源（travel.md、feature-a.md、docs/xxx.md 均可）。
- 不要要求文件必须叫 PRD-Vx.md；不要自动搜索并读取所有 PRD-Vx.md。
- 用户未指定文档时，以当前对话为需求来源；PRD.md 只在必要时按需使用。

生命周期：临时文件在子需求完成后**清空内容、绝不删除文件**。

**参考文件（按需读取，不要现在读）**：

- `references/file-formats.md` — 生成 prd-sub.md / plan.md / execution-plan.md / memory.md，或清空临时文档之前读取。
- `references/document-contracts.md` — Phase 0 创建基础文件、Phase 4 同步与清理、Phase 5 回退，或对任何文件的职责边界有疑问时读取

## 3. Phase 状态机

```
Phase 0 项目初始化
   ↓
Phase 1 需求分析 ──→ 【用户确认①】
   ↓
Phase 2 方案规划 ──→ 【用户确认②】
   ↓
Phase 3 Step 执行（循环：一个 Step 接一个）
   ↓
Phase 4 子需求完成

任意阶段发生问题 → Phase 5 问题排查 / 恢复 / 回退
```

禁止 Agent 无理由跨 Phase 跳转。

### Phase 0：项目初始化

- 进入：项目首次使用本 Skill，或用户明确要求初始化工作流。
- 读取：项目目录结构；仅**检查** AGENTS.md / PRD.md / artifacts/ 是否存在，不读内容。
- 动作：创建缺失的 AGENTS.md、PRD.md、artifacts/ 及五个工作文件（空骨架）；已存在的**禁止覆盖**。
- 禁止：业务需求分析、生成子需求/Plan/Execution Plan、编码、运行测试、扫描源码目录。
- 退出：Workflow 所需基础文件齐备。

### Phase 1：需求分析

- 进入：用户提出新的开发需求。
- 必须读取：用户当前对话 + 用户 @ 指定的需求文档 + AGENTS.md。
- 按需读取：artifacts/architecture.md、相关源码（仅当缺少该信息会阻碍需求正确分析时）。
- 默认禁止读取：PRD.md、其他需求文档、plan.md、execution-plan.md、memory.md。
- 动作：生成 `artifacts/prd-sub.md` — AI 梳理后的**可执行子需求**，不是需求文档的复制；
  不添加用户没要求且无依据的业务规则；信息不足时列入"待确认问题"。
- 禁止：编码、修改业务源码、生成 plan / execution-plan、修改 architecture.md、执行测试。
- 退出：prd-sub.md 生成且用户确认。

**【用户确认点①】** 生成 prd-sub.md 后必须暂停，向用户展示：需求理解 / 功能范围 / 非目标 /
核心业务规则 / 验收标准 / 待确认问题。没有用户确认，禁止进入 Phase 2。

### Phase 2：方案规划

- 进入：用户确认 prd-sub.md。
- 必须读取：artifacts/prd-sub.md + AGENTS.md。
- 按需读取：artifacts/architecture.md、相关源码、artifacts/memory.md
  （仅当本子需求此前已有执行记录，或历史执行信息会影响当前方案时）。
- 默认禁止读取：原始需求文档、PRD-Vx.md、PRD.md、整个源码目录。
- 动作 1：生成 `artifacts/plan.md` — **跨 Step 方案决策记录**：只记横跨多个 Step 的决策与理由
  （技术方案选型、数据模型、API 契约、模块划分、依赖、风险、整体验证策略），
  不复述 Step 级实现细节（那属于 execution-plan.md），保持短（建议 ≤100 行）。
- 动作 2：生成 `artifacts/execution-plan.md` — Step 队列，**只含未完成工作**。
  每个 Step 必须：目标 / 范围 / 涉及文件 / 实现方式 / 验证方式 / 完成标准明确；
  独立可执行、可验证、可恢复、不依赖完整历史上下文；不过度细碎。
- 动作 3：若本次做出**长期性技术选型**（新项目首次定栈、更换存储 / 框架等），
  立即写入 artifacts/architecture.md 的技术选型部分（不等 Phase 4）；
  Phase 2 对 architecture.md 的写入仅限长期技术选型，其余架构内容仍按 Phase 4 判断。
  编码层面的栈约束（框架版本、目录约定、测试规则）同步 AGENTS.md。
- 禁止：修改业务源码、执行任何 Step、提前标记 DONE、写执行结果到 memory.md、修改 PRD.md。
- 退出：plan.md + execution-plan.md 生成且用户确认。

**【用户确认点②】** 生成后必须暂停，向用户展示：技术方案 / Step 划分与依赖 / 验证策略 / 风险。
没有用户确认，禁止进入 Phase 3。

### Phase 3：Step 执行

进入：用户确认 execution-plan.md。核心原则：**一次只执行一个 Step。**

每个 Step 的执行循环：

1. 读取 artifacts/execution-plan.md + AGENTS.md → 第一个未完成 Step = 当前 Step。
2. 按需定位并读取当前 Step 相关源码（Progressive Context Loading，见 §4）。
3. Context Sufficient = YES → 实现当前 Step。
   禁止：实现后续 Step、修改无关模块、重构与当前 Step 无关的代码。
   允许：当前 Step 所必须的最小依赖修改。
4. 执行 Minimal Verification（见下）。
5. 按结果处理：

```
成功：实现完成 + 验证通过
  → 追加写入 artifacts/memory.md（禁止粘贴完整源码、完整测试日志、完整执行历史）
  → 从 artifacts/execution-plan.md 删除当前 Step
  → 还有 Step？执行下一个 : 进入 Phase 4

失败：验证不通过
  → 保留 Step（不删除、不标 DONE）
  → 失败信息追加到 memory.md
  → 分析、修复、再次验证
  → 无法可靠解决 → 状态置 BLOCKED，暂停并向用户说明（禁止隐藏失败）
```

**Step 状态机**：

```
PENDING → IN_PROGRESS → VERIFYING → DONE      （写 memory.md → 从计划删除）
                      → FAILED     （写 memory.md → 诊断/修复 → 回到 VERIFYING）
无法继续 → BLOCKED    （保留在计划中、记录原因、必要时请求用户处理）
```

只有"代码完成 + 验证成功"才能标记 DONE。

**Minimal Verification（验证范围与修改影响匹配）**：

| 修改类型 | 最小验证 |
|---|---|
| 类型 / 声明 / 纯结构 | type check / compile |
| 局部逻辑 | 相关 unit test |
| 对外契约（HTTP / RPC / CLI / 事件） | 相关契约测试 |
| 持久化 / schema | migration 或 schema 检查 + 相关测试 |
| UI 可见行为 | 相关组件测，或该路径的浏览器验证 |
| 跨模块 / 破坏性契约 | 按风险扩大到调用方 |

完整测试套件只在必要时执行；子需求级完整验证在 Phase 4 做。
项目已有测试规范时优先遵守 AGENTS.md。

**Phase 3 默认禁止读取**：原始需求文档、PRD-Vx.md、PRD.md、prd-sub.md、plan.md、memory.md。
禁止为了"确认一下需求"而重读这些文档；当前 Step 缺关键信息时，先找更小范围的信息源。
architecture.md 仅当 Step 涉及新模块、模块边界、目录结构、技术栈、数据 / API / Agent / Tool 架构
或长期约束时，读取**相关部分**。

**计划冲突处理**：实际项目结构与 execution-plan.md 严重冲突、或当前 Step 需要重大改变技术方案时，
禁止默默修改整个计划 —— 识别冲突 → 说明原因 → 小范围调整可自行处理 → 重大变化暂停请求用户确认。

退出：execution-plan.md 中无剩余 Step → Phase 4。

### Phase 4：子需求完成

进入：execution-plan.md 中没有未完成 Step。

1. **最终验证**：按 prd-sub.md 验收标准做一次子需求级验证（不重复每个 Step 的测试；
   按风险执行 type check / unit / integration / 契约或路径验证 / 验收标准检查，只执行与本子需求相关的验证）。
2. **同步 PRD.md**：写入本次子需求最终需求结果。禁止复制 execution-plan、memory 内容、测试日志。
3. **判断是否更新 architecture.md**：核心问题 ——"本次是否改变了未来开发必须遵守的长期架构规则？"
   NO 不更新；YES 才更新（判断标准见 document-contracts.md）。Phase 2 已写入的长期选型不重复写入。
4. **提炼 memory.md**：只提炼长期价值信息（重要技术 / 架构决策、可复现问题与解决方案），
   同步到 PRD.md / architecture.md / 项目已有长期文档。**禁止新建任何长期记忆文件。**
5. **清空临时文档**：清空 prd-sub.md / plan.md / execution-plan.md / memory.md 的内容，
   保留极简状态指针（格式见 file-formats.md）。**清空而不是删除，文件必须继续存在。**

退出：临时文档已清空、PRD.md 已同步 → 等待下一个子需求。

### Phase 5：问题排查 / 恢复 / 回退

触发：用户报告 Bug、历史问题、"为什么这样实现"、某个 Step 出错、需要回退或恢复。

**排查读取顺序**（只读与问题有关的信息，禁止一开始读取所有文档）：

当前用户问题 → 当前错误信息 → 相关源码 → 必要时 memory.md → 必要时 architecture.md
→ 必要时 PRD.md → 必要时相关历史需求文档

**回退流程**：

1. 定位问题发生在哪个 Step
2. 查看 memory.md 中该 Step 的记录
3. 定位相关代码修改
4. 判断回退范围
5. 执行回退
6. 验证回退结果
7. 记录问题和回退结果
8. 必要时更新 execution-plan.md（如需重做该 Step，将其加回）

禁止无依据地恢复整个项目；禁止依赖完整历史上下文才能回退。
退出：问题解决 / 回退完成并记录后，回到被中断的 Phase。

## 4. Context-on-Demand（按需读取策略）

### Context Loading Levels（渐进加载）

```
Level 0  当前用户请求
Level 1  当前 Phase 要求的文档
Level 2  必要的 architecture / memory 信息
Level 3  当前 Step 相关源码
Level 4  必要的历史信息 / 类似实现
```

每进入下一级之前重新判断上下文是否已经足够；足够立即停止，不为了"保险"继续扩大。

### Context Sufficiency（语义判断）

判断问题：**"如果现在停止读取，我是否已经拥有可靠完成当前 Step 所需要的全部信息？"**

YES 的条件（全部满足）：

1. 当前 Step 明确
2. AGENTS.md 相关规则明确
3. 必要架构约束明确
4. 相关源码已定位
5. 必要依赖已理解
6. 可以可靠开始实现

YES → 立即停止读取，开始实现。
NO → 明确说出"当前到底缺少什么信息"，只读取能补足该缺口的最小文件，然后重新判断。

### 源码读取规则

先读当前 Step 明确涉及的文件及其直接依赖；仍缺信息再读必要依赖 / 同类实现；
只有被当前实现真正阻塞时才扩大范围。

禁止：读取整个源码目录、任一完整分层目录、全部测试、全部配置、整个项目。

## 5. 用户确认规则

正常流程只有两个强制确认点：① prd-sub.md 之后；② plan.md + execution-plan.md 之后。

Step 执行过程中不要每一步询问用户。仅以下情况暂停：

1. 需求存在无法自行判断的歧义
2. 发现重大架构变化
3. 实际项目结构与计划严重冲突
4. 测试失败且无法可靠解决
5. 即将执行高风险操作
6. 需要用户提供信息
7. 技术方案发生重大变化
