---
name: lean-coding-workflow
description: "Document-driven coding workflow with on-demand context, one-step execution, and acceptance checklists."
disable-model-invocation: true
---

# Lean Coding Workflow

## 0. 原则

1. **Minimum Reliable Context（按需读取，够即停）。** 读取前必须能回答："不读它，我无法可靠完成当前任务"；
   足够后禁止"再确认一下"式读取；禁止用 token / 百分比数字刹车。
2. **一次只执行一个 Step。** 禁止批量实现、顺手重构无关代码。
3. **验证与修改范围匹配。** 禁止每个 Step 都跑完整测试套件。
4. **文档职责严格隔离。** 一个文件一个职责，禁止多文件记录相同内容。

## 1. 进入

继续未完成工作时按文件内容判定（只读所需的一个文件）：
`artifacts/execution-plan.md` 存在：
- 存在 `[❌]` 验收项 → Phase 3 续做；
- 存在 `[⚠️]` 验收项 → 根据该项说明判断是否仍需继续；若需继续则 Phase 3，否则视为遗留项；
- 全部 `[✅]` → 当前子需求已完成，不续做。
若 `execution-plan.md` 不存在：
- `artifacts/prd-sub.md` 有内容 → 停在确认点①；
- 否则 → Phase 1。

## 2. 文件

约定文件 `AGENTS.md`（必读；缺失时回退 `CLAUDE.md`，两者都缺视为无附加规则）与 `PRD.md`（可选）在项目根；
workflow 文件在 `artifacts/`：architecture / prd-sub / execution-plan / memory。
职责与生命周期以 `references/document-contracts.md` 为准。临时文件生命周期（prd-sub / execution-plan）：
- 归档点（Phase 4 完成）：总结两文件 → 写入 memory.md 完成记录 → 两文件保留不动。
- 重建点（下一子需求启动，Phase 1）：读旧文件 → 核对 memory 已有本轮记录（缺失则补写）→
  清空（只留一行职责标题）→ 压缩 memory 至 <300 行 → 写入新 prd-sub.md。
- 禁止：Phase 4 刚完成就清空；新需求开始时不读旧文件直接覆盖 prd-sub.md。
memory 长期不清空；用户需求文档只读，永不修改。
需求入口：用户 @ 的任意 .md = 当前需求来源（不限命名与路径，禁止自动搜索其他需求文档）；
未指定则以当前对话为来源。

参考文件（按需读取，不要现在读）：

- `references/file-formats.md` — 写各 workflow 文件、清空临时文档之前读取（格式模板）。
- `references/document-contracts.md` — 职责边界有疑问，或 Phase 2 / 4 需要判断依据时读取。

## 3. 流程

```
Phase 1 需求分析 →【确认①】→ Phase 2 方案规划 →【确认②】→ Phase 3 Step 执行 → Phase 4 完成
```

禁止无理由跨 Phase 跳转。

### Phase 1：需求分析

- 读：当前对话 + 用户 @ 的需求文档 + 约定文件（§2，缺失视为无附加规则）；按需 architecture / 相关源码。
- 存在旧 prd-sub / execution-plan → 按 §2 重建点收尾（核对沉淀 → 清空 → 压缩 memory）；
  禁止跳过收尾直接写入新 prd-sub.md。
- 写 `artifacts/prd-sub.md`（目录不存在则创建）：AI 梳理后的可执行子需求，不是需求复制；
  不添加无依据的业务规则；信息不足列入"待确认问题"。
- 禁止：编码、改业务源码、写其他 workflow 文件、改用户需求文档。
- **【确认①】** 展示：需求理解 / 功能范围 / 非目标 / 业务规则 / 验收标准 / 待确认问题；确认后 → Phase 2。

### Phase 2：方案规划

- 读：prd-sub.md + 约定文件（§2）；按需 architecture / 相关源码 / memory 经验条目。
- 写 `artifacts/execution-plan.md`：全部 Step，每个 Step 三节——目标 / 方案 / 验收（`- [ ]` 清单项）。
  验收与修改范围匹配：类型改动用 type check，局部逻辑用单测，对外契约用契约测试，
  持久化用 schema 检查，UI 用路径验证，跨模块按风险扩大到调用方。格式见 file-formats.md。
- 按判断标准（document-contracts.md）写 architecture.md：新项目首次结构性决策必须产出架构设计并写入；
  已有项目仅存在长期架构变化时更新。
- 编码栈约束属约定文件（§2）：存在则同步；不存在先记 execution-plan.md 头部，Phase 4 转入。
- 禁止：改业务源码、执行 Step、写 memory、改 PRD.md。
- **【确认②】** 展示：技术方案 / Step 划分与依赖 / 验收方式；确认后 → Phase 3。

### Phase 3：Step 执行

按 execution-plan.md 顺序执行，一次一个 Step：

1. 读 execution-plan.md + 约定文件（§2）→ 第一个存在非 `[✅]` 验收项的 Step = 当前 Step；
   读取相关源码并实现。禁止实现后续 Step、改无关模块；允许必需的最小依赖修改。
2. 执行该 Step 的验收：通过 → `[ ]` 改 `[✅]`；该 Step 全部 `[✅]` 后进入下一 Step。
3. 验收失败 → 标 `[❌]`，分析原因：当前 Step 的问题改当前 Step；前置 Step 引起的问题回到该
   前置 Step 修改并重验；重新执行受影响 Step 的验收，通过后改 `[✅]` 继续。
4. 无法自行解决 → 标 `[⚠️]`，暂停向用户说明。暂停请示仅限：需求歧义 / 重大架构或方案变化 /
   计划与实际严重冲突 / 高风险操作 / 需要用户提供信息。

### Phase 4：完成

所有 Step 的验收项均为 `[✅]` → 本次子需求完成：

1. PRD.md 存在则同步本次需求结果（不存在不创建）。
2. architecture.md 仅在存在架构变化时创建或更新（判断标准见 document-contracts.md）。
3. 约定文件缺失时按本轮实际执行的约定创建（AGENTS.md；已有 CLAUDE.md 且无 AGENTS.md 则追加其中），
   创建前告知用户。
4. 按 §2 归档点归档：总结两文件写入 memory 完成记录，并追加值得长期保留的经验（memory 是唯一记忆文件）。
5. 临时文件保留不动（Phase 4 不清空，见 §2 生命周期）。
6. 向用户报告结果与各文件处理情况。

## 4. Context-on-Demand

顺序：当前请求 → 当前 Phase 文档 → 必要的 architecture / memory → 当前 Step 相关源码 → 必要的历史；
每进入下一级前重新判断，足够立即停止。
充分判断："现在停止读取，是否已有可靠完成当前 Step 的全部信息？"——目标与验收明确 +
AGENTS 规则明确 + 源码与依赖已定位理解 → 立即实现；否则说出缺什么，只读补足缺口的最小文件。
源码：先读 Step 明确涉及的文件及直接依赖，仍缺再读必要依赖 / 同类实现；
禁止读取整个源码目录、任一完整分层目录、全部测试 / 配置、整个项目。
