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

**会话恢复判定**：用户说"继续"时，按**文件内容**判断进度，只读判定所需的那一个文件。禁止为了恢复状态读取全部文档。
"已清空" = 文件仅含一行职责标题、无实际内容。

| 判定 | 进入 |
|---|---|
| `artifacts/execution-plan.md` 有内容且含非 DONE Step | Phase 3（若尚未过确认点②，先停在确认点②） |
| `artifacts/prd-sub.md` 有内容，`plan.md` / `execution-plan.md` 已清空 | 停在确认点①，或用户确认后进入 Phase 2 |
| 三个临时文件均已清空 | Phase 0（检查项目基础文件）或 Phase 1 |

## 2. 项目目录约定

`PRD.md` / `AGENTS.md` 在项目根；其余 workflow 文件在 `artifacts/`。
PRD.md 与 AGENTS.md 均为**可选**，不存在不阻塞流程（各自创建时机见 document-contracts.md）。

| 路径 | 语义 | 生命周期 |
|---|---|---|
| PRD.md | 项目当前最终需求结果 | 长期（可选：存在才在 Phase 4 同步） |
| @xxx.md / PRD-Vx.md | 用户指定的需求来源 | 用户所有（只读，永不修改） |
| AGENTS.md | AI 编码规则 | 长期（可选：存在才读；缺失时 Phase 4 按实际约定创建） |
| artifacts/architecture.md | 长期架构约束 | 长期（有长期架构变化时创建 / 更新） |
| artifacts/prd-sub.md | 当前子需求 | 临时（完成后清空内容） |
| artifacts/plan.md | 当前子需求整体方案 | 临时（按需创建；完成后清空内容） |
| artifacts/execution-plan.md | 当前子需求全部 Step 及状态行 | 临时（完成后清空内容） |
| artifacts/memory.md | 项目长期经验记忆 | 长期（不清空；禁止执行流水；必要时压缩） |

需求入口规则：

- 用户 @ 指定的**任意** .md 文件 = 当前需求来源（travel.md、feature-a.md、docs/xxx.md 均可）。
- 不要要求文件必须叫 PRD-Vx.md；不要自动搜索并读取所有 PRD-Vx.md。
- 用户未指定文档时，以当前对话为需求来源；PRD.md 只在必要时按需使用。

**参考文件（按需读取，不要现在读）**：

- `references/file-formats.md` — 生成 prd-sub.md / plan.md / execution-plan.md、更新 Step 状态与进度、
  写 memory.md，或清空临时文档之前读取。
- `references/document-contracts.md` — Phase 0 检查、Phase 2 架构写入判断、Phase 4 同步与清理，
  或对任何文件的职责边界有疑问时读取

## 3. Phase 状态机

```
Phase 0 项目检查
   ↓
Phase 1 需求分析 ──→ 【用户确认①】
   ↓
Phase 2 方案规划 ──→ 【用户确认②】
   ↓
Phase 3 Step 执行（循环：一个 Step 接一个，状态实时落盘）
   ↓
Phase 4 子需求完成
```

禁止 Agent 无理由跨 Phase 跳转。

### Phase 0：项目检查（只检查，不创建）

- 进入：项目首次使用本 Skill，或用户明确要求初始化工作流。可重复执行。
- 读取：项目目录结构；仅**检查** AGENTS.md / PRD.md / `artifacts/` 目录是否存在，不读内容。
- 动作：向用户报告缺失项及各自的按需创建时机（以 document-contracts.md 各文件契约为准），
  不创建任何文件（用户明确要求的除外）。
- 禁止：创建项目文件（用户明确要求的除外）、业务需求分析、生成子需求/Plan/Execution Plan、
  编码、运行测试、扫描源码目录。
- 退出：检查结果已报告 → 进入 Phase 1（或等待用户下一步指示）。

### Phase 1：需求分析

- 进入：用户提出新的开发需求。
- 必须读取：用户当前对话 + 用户 @ 指定的需求文档 + AGENTS.md（存在才读，不存在不臆造规则）。
- 按需读取：artifacts/architecture.md、相关源码（仅当缺少该信息会阻碍需求正确分析时）。
- 默认禁止读取：PRD.md、其他需求文档、plan.md、execution-plan.md、memory.md。
- 动作 0：若 execution-plan.md 仍含非 DONE Step（上一子需求未完成），暂停请用户决定——
  先完成/恢复旧子需求，或放弃（放弃则清空旧 prd-sub / plan / execution-plan，只留职责标题）。
- 动作：写入 `artifacts/prd-sub.md`（`artifacts/` 目录不存在则先创建）—
  AI 梳理后的**可执行子需求**，不是需求文档的复制；
  不添加用户没要求且无依据的业务规则；信息不足时列入"待确认问题"。
- 禁止：编码、修改业务源码、写入 plan / execution-plan / memory / architecture、
  修改用户需求文档、执行测试。
- 退出：prd-sub.md 生成且用户确认。

**【用户确认点①】** 生成 prd-sub.md 后必须暂停，向用户展示：需求理解 / 功能范围 / 非目标 /
核心业务规则 / 验收标准 / 待确认问题。没有用户确认，禁止进入 Phase 2。

### Phase 2：方案规划

- 进入：用户确认 prd-sub.md。
- 必须读取：artifacts/prd-sub.md + AGENTS.md（存在才读）。
- 按需读取：artifacts/architecture.md、相关源码、artifacts/memory.md
  （长期经验记忆；仅当本子需求涉及历史模块、以往踩过相关坑，或旧经验会影响当前方案时读相关条目）。
- 默认禁止读取：原始需求文档、PRD-Vx.md、PRD.md、整个源码目录。
- 动作 1：存在跨 Step 的方案决策时，写入 `artifacts/plan.md`（无则跳过、不建文件）—
  **跨 Step 方案决策记录**：只记横跨多个 Step 的决策与理由
  （技术方案选型、数据模型、API 契约、模块划分、依赖、风险、整体验证策略），
  不复述 Step 级实现细节（那属于 execution-plan.md），保持短（建议 ≤100 行）。
- 动作 2：写入 `artifacts/execution-plan.md` — 当前子需求的**全部 Step 队列**，
  每个 Step 带状态行（初始 PENDING；状态机与中断检查点见 Phase 3 与 file-formats.md）。
  Step 完成后**不删除**，保留至 Phase 4 整体清空。
  每个 Step 必须：目标 / 范围 / 涉及文件 / 实现方式 / 验证方式 / 完成标准明确；
  独立可执行、可验证、可恢复、不依赖完整历史上下文；不过度细碎。
- 动作 3：按更新判断写入 artifacts/architecture.md（判断标准与例外见 document-contracts.md；
  新项目首次结构性决策必须产出架构设计并写入）。
  编码层栈约束属 AGENTS.md：存在则同步，不存在则记录在 plan.md
  （未建 plan.md 时记在 execution-plan.md 头部），Phase 4 创建时转入。
- 禁止：修改业务源码、执行任何 Step、提前标记 DONE、写执行结果到 memory.md、修改 PRD.md。
- 退出：execution-plan.md 生成（plan.md 按需）且用户确认。

**【用户确认点②】** 生成后必须暂停，向用户展示：技术方案 / Step 划分与依赖 / 验证策略 / 风险。
没有用户确认，禁止进入 Phase 3。

### Phase 3：Step 执行

进入：用户确认 execution-plan.md，或从会话中断处恢复（按 §1 会话恢复判定）。
核心原则：**一次只执行一个 Step。**

每个 Step 的执行循环：

1. 读取 artifacts/execution-plan.md + AGENTS.md（存在才读）→
   恢复入口 = 计划中**第一个非 DONE 的 Step**。
   正常顺序推进时它就是第一个 PENDING，即当前 Step；
   若其状态为 IN_PROGRESS / VERIFYING（上次会话中断），先用 git diff / 相关源码核对实际完成度，
   再续做或重验，**禁止不加核对从头重做**；
   若为 FAILED / BLOCKED，先读该 Step 的"进度 / 下一动作"记录，再决定修复路径。
2. 开始实现前，把当前 Step 的状态行写为 IN_PROGRESS。
3. 按需定位并读取当前 Step 相关源码（Progressive Context Loading，见 §4）。
4. Context Sufficient = YES → 实现当前 Step。
   禁止：实现后续 Step、修改无关模块、重构与当前 Step 无关的代码。
   允许：当前 Step 所必须的最小依赖修改。
5. 执行 Minimal Verification（见下）。
6. 按结果处理：

```
成功：实现完成 + 验证通过
  → 该 Step 状态标 DONE，正文压缩为一行验证结果（格式见 file-formats.md）
  → 高价值信息（决策、踩坑、复用方式）追加到 artifacts/memory.md
    （禁止执行流水、完整源码、完整测试日志）
  → 还有 Step？执行下一个 : 进入 Phase 4

失败：验证不通过
  → 状态标 FAILED，更新该 Step 的"进度 / 下一动作"两行
  → 分析、修复、再次验证（回到第 5 步）
  → 无法可靠解决 → 状态置 BLOCKED，暂停并向用户说明（禁止隐藏失败）
```

**状态与检查点**：状态每次变更立即写回 execution-plan.md——它是 Step 进度的唯一持久载体，
会话中断后凭状态行 + 实际代码进度恢复，不依赖对话历史；只有"代码完成 + 验证成功"才能标 DONE。
状态机、DONE 压缩格式与检查点写入时机见 file-formats.md。
检查点是尽力而为的记录；恢复时**以 git diff / 实际代码为准核对**，文件记录是提示不是事实。

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

退出：execution-plan.md 中所有 Step 均为 DONE → Phase 4。

### Phase 4：子需求完成

进入：execution-plan.md 中所有 Step 均为 DONE。

1. **最终验证**：按 prd-sub.md 验收标准做一次子需求级验证（不重复每个 Step 的测试；
   按风险执行 type check / unit / integration / 契约或路径验证 / 验收标准检查，只执行与本子需求相关的验证）。
2. **同步 PRD.md（若存在）**：写入本次子需求最终需求结果。不存在则跳过、不创建；
   禁止复制 execution-plan、memory 内容、测试日志；**禁止修改用户的原始需求文档**。
3. **判断是否更新 architecture.md**：判断标准见 document-contracts.md；Phase 2 已写入的长期选型不重复写入。
4. **沉淀经验（分流写入）**：
   - 架构类 → architecture.md；
   - 编码规则 / 约定类 → AGENTS.md：不存在则按本轮**实际执行中采用的约定**创建（创建前告知用户），
     已存在只追加 / 修正；
   - 需求结果 → PRD.md（若存在）；
   - 其余长期经验 → memory.md 追加条目（容量与压缩规则见 file-formats.md）。
   memory.md 是唯一记忆文件，禁止新建任何其他长期记忆文件。
5. **清空临时文档**：对 prd-sub.md / plan.md / execution-plan.md 清空内容、保留文件，
   只留一行职责标题（格式见 file-formats.md）；未创建过的文件不补建。
6. **完成报告**：向用户简要说明——验证结果、PRD.md 是否同步（未同步注明原因）、
   architecture.md 是否更新及原因（未更新 ≠ 异常）、AGENTS.md 是否新建、
   memory.md 追加了哪些经验。

退出：临时文档已清空、长期文档已沉淀 → 等待下一个子需求。

## 4. Context-on-Demand（按需读取策略）

### Context Loading Levels（渐进加载）

当前用户请求 → 当前 Phase 要求的文档 → 必要的 architecture / memory 信息 → 当前 Step 相关源码
→ 必要的历史信息 / 类似实现。
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
7. 当前 Step 的完成标准和验证方式明确

YES → 立即停止读取，开始实现。
NO → 明确说出"当前到底缺少什么信息"，只读取能补足该缺口的最小文件，然后重新判断。

### 源码读取规则

先读当前 Step 明确涉及的文件及其直接依赖；仍缺信息再读必要依赖 / 同类实现；
只有被当前实现真正阻塞时才扩大范围。

禁止：读取整个源码目录、任一完整分层目录、全部测试、全部配置、整个项目。

## 5. 用户确认规则

Step 执行过程中不要每一步询问用户（强制确认点只有① / ②，见 Phase 1 / 2 退出）。仅以下情况暂停：

1. 需求存在无法自行判断的歧义
2. 发现重大架构变化
3. 实际项目结构与计划严重冲突
4. 测试失败且无法可靠解决
5. 即将执行高风险操作
6. 需要用户提供信息
7. 技术方案发生重大变化
