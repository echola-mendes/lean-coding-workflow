---
name: lean-coding-workflow
description: "Document-driven coding workflow with on-demand context, one-step execution, minimal verification, and recoverable state."
---

# Lean Coding Workflow

## 0. 最高原则

1. **Minimum Reliable Context（按需读取，够即停）。** 每次读取前必须能回答："不读它，我无法可靠完成当前任务"——
   "可能有帮助"不是理由；足够后禁止任何"再确认一下"式读取；只做语义判断，禁止用 token / 百分比数字刹车。
2. **一次只执行一个 Step。** 禁止一次性实现多个 Step，禁止顺手重构无关代码。
3. **验证与修改范围匹配。** 禁止每个 Step 都运行完整测试套件。
4. **文档职责严格隔离。** 每个文件只有一个职责，禁止多个文档记录相同内容。

## 1. 进入与恢复

新开发需求（可带 @xxx.md，如 "@travel.md 实现差旅审批"）→ Phase 1。
说"继续"时按文件内容判定（只读判定所需的那一个文件；"已清空" = 仅含一行职责标题、无实际内容）：

| 判定 | 进入 |
|---|---|
| `artifacts/execution-plan.md` 有内容且含非 DONE Step | Phase 3（尚未过确认点②则先停在确认点②） |
| `artifacts/prd-sub.md` 有内容，plan / execution-plan 已清空 | 停在确认点①，或用户确认后进入 Phase 2 |
| 三个临时文件均已清空 | Phase 1 |

## 2. 项目目录约定

`AGENTS.md` / `PRD.md` 在项目根，workflow 文件在 `artifacts/`：
architecture / prd-sub / plan / execution-plan / memory。
AGENTS.md 必读（缺失视为无附加规则）；PRD.md 可选，不存在不阻塞流程。
各文件职责 / 生命周期 / 创建时机以 `references/document-contracts.md` 职责速查表为准。铁律：
临时文件完成后清空内容、保留文件；memory.md 长期不清空；用户 @ 的需求文档只读，永不修改。
需求入口：用户 @ 的任意 .md = 当前需求来源（不限命名与路径，禁止自动搜索其他需求文档）；
未指定则以当前对话为来源。

参考文件（按需读取，不要现在读）：

- `references/file-formats.md` — 生成 / 更新各 workflow 文件、清空临时文档之前读取（格式模板）。
- `references/document-contracts.md` — 职责边界有疑问，或 Phase 2 / 4 需要判断依据时读取。

## 3. Phase 状态机

```
Phase 1 需求分析 →【确认①】→ Phase 2 方案规划 →【确认②】→ Phase 3 Step 执行 → Phase 4 完成
```

禁止无理由跨 Phase 跳转。

### Phase 1：需求分析

- 读取：当前对话 + 用户 @ 的需求文档 + AGENTS.md（缺失视为无附加规则，不臆造）；按需 architecture / 相关源码。
  禁止读 PRD.md、其他需求文档、plan / execution-plan / memory。
- 若 execution-plan.md 仍含非 DONE Step（上一子需求未完成）：暂停请用户决定——先完成旧子需求，
  或放弃（清空旧 prd-sub / plan / execution-plan，只留职责标题）。
- 写入 `artifacts/prd-sub.md`（artifacts/ 不存在则先创建）：AI 梳理后的可执行子需求，不是需求复制；
  不添加无依据的业务规则；信息不足列入"待确认问题"。
- 禁止：编码、改业务源码、写其他 workflow 文件、改用户需求文档、执行测试。
- **【确认点①】** 展示：需求理解 / 功能范围 / 非目标 / 业务规则 / 验收标准 / 待确认问题；确认后 → Phase 2。

### Phase 2：方案规划

- 读取：prd-sub.md + AGENTS.md；按需 architecture / 相关源码 / memory 相关经验条目。
  禁止读原始需求文档、PRD.md、整个源码目录。
- 动作 1：有跨 Step 决策才写 `artifacts/plan.md`（只记跨 Step 决策与理由，≤100 行，格式见 file-formats.md）；无则不建。
- 动作 2：写 `artifacts/execution-plan.md` — **全部 Step 队列 + 文末验收清单**（三节格式见 file-formats.md，
  状态初始 PENDING；Step 完成后不删除，Phase 4 整体清空；清单从 prd-sub 验收标准转写）。
- 动作 3：按判断标准（document-contracts.md）写 architecture.md——新项目首次结构性决策必须产出架构设计并写入。
  编码栈约束属 AGENTS.md：存在则同步；不存在先记 plan.md（未建则记 execution-plan 头部），Phase 4 转入。
- 禁止：改业务源码、执行任何 Step、提前标 DONE、写 memory、改 PRD.md。
- **【确认点②】** 展示：技术方案 / Step 划分与依赖 / 验证策略 / 风险；确认后 → Phase 3。

### Phase 3：Step 执行

进入：确认②之后，或按 §1 从中断处恢复。核心原则：**一次只执行一个 Step。**

执行循环：

1. 读 execution-plan.md + AGENTS.md → 当前 Step = 计划中**第一个非 DONE 的 Step**。
   IN_PROGRESS / VERIFYING（上次中断）：先用 git diff / 相关源码核对实际完成度再续做或重验，
   禁止不加核对从头重做；FAILED / BLOCKED：先读其"进度 / 下一动作"记录再决定修复路径。
2. 开始实现前把该 Step 状态行写为 IN_PROGRESS。
3. 按需读取当前 Step 相关源码（§4）。Context Sufficient = YES → 实现；
   禁止实现后续 Step、改无关模块、顺手重构；允许当前 Step 必需的最小依赖修改。
4. 执行 Minimal Verification（见下表）。
5. 成功 → 该 Step 标 DONE 并压缩为一行验证结果，高价值信息（决策 / 踩坑 / 复用）追加 memory.md
   （禁止流水、完整源码、完整测试日志）→ 还有 Step 则执行下一个，否则 Phase 4。
   失败 → 状态标 FAILED，更新"进度 / 下一动作"两行，分析修复后重新验证；
   无法可靠解决 → BLOCKED，暂停并向用户说明（禁止隐藏失败）。

状态每次变更立即写回 execution-plan.md（Step 进度的唯一持久载体，不依赖对话历史）；
状态机与检查点格式见 file-formats.md。检查点是尽力而为的记录，恢复时以 git diff / 实际代码为准核对。

**Minimal Verification（验证范围与修改影响匹配）**：

| 修改类型 | 最小验证 |
|---|---|
| 类型 / 声明 / 纯结构 | type check / compile |
| 局部逻辑 | 相关 unit test |
| 对外契约（HTTP / RPC / CLI / 事件） | 相关契约测试 |
| 持久化 / schema | migration 或 schema 检查 + 相关测试 |
| UI 可见行为 | 相关组件测试或该路径浏览器验证 |
| 跨模块 / 破坏性契约 | 按风险扩大到调用方 |

子需求级完整验证只在 Phase 4 做；已有测试规范优先遵守 AGENTS.md。

Phase 3 默认禁止读取：原始需求文档、PRD.md、prd-sub.md、plan.md、memory.md
（当前 Step 缺信息时先找更小范围的信息源）；architecture.md 仅当 Step 涉及模块边界 / 目录结构 /
技术栈 / 数据 / API 架构 / 长期约束时读相关部分。

计划冲突：小范围调整可自行处理；严重冲突或需重大改变技术方案时，禁止默默改整个计划，暂停请用户确认。

**暂停请示仅限**：需求歧义无法自行判断 / 重大架构或方案变化 / 计划与实际严重冲突 /
失败无法可靠解决 / 高风险操作 / 需要用户提供信息。除此之外不要每步询问用户。

退出：所有 Step 均 DONE → Phase 4。

### Phase 4：子需求完成

1. **最终验证**：按 execution-plan.md 文末验收清单逐项验证、通过一项勾选一项（不重复每个 Step 的测试）。
2. **同步 PRD.md（若存在）**：写本次最终需求结果；不存在则跳过、不创建；禁止复制执行细节与日志，
   禁止修改用户需求文档。
3. **architecture.md**：按 document-contracts.md 判断标准决定是否更新；Phase 2 已写入的不重复。
4. **沉淀经验**：架构类 → architecture；规则 / 约定类 → AGENTS.md（缺失则按本轮实际执行的约定创建，
   创建前告知用户，已存在只追加修正）；需求结果 → PRD（若存在）；其余长期经验 → memory.md
   （容量与压缩规则见 file-formats.md）。memory 是唯一记忆文件，禁止新建其他记忆文件。
5. **清空** prd-sub / plan / execution-plan：清空内容、保留文件、只留一行职责标题（见 file-formats.md）；
   未创建过的不补建。
6. **完成报告**：验证结果；PRD / architecture / AGENTS / memory 各文件处理情况
   （未处理注明原因，architecture 未更新 ≠ 异常）。

## 4. Context-on-Demand

加载顺序：当前请求 → 当前 Phase 文档 → 必要的 architecture / memory → 当前 Step 相关源码 → 必要的历史信息；
每进入下一级前重新判断，足够立即停止。
充分判断："现在停止读取，是否已拥有可靠完成当前 Step 的全部信息？"——Step 目标与验证明确 +
AGENTS 规则与架构约束明确 + 源码与依赖已定位理解 → YES 立即开始实现；
NO → 说出缺什么，只读补足缺口的最小文件。
源码：先读 Step 明确涉及的文件及直接依赖，仍缺再读必要依赖 / 同类实现；
禁止读取整个源码目录、任一完整分层目录、全部测试 / 配置、整个项目。
