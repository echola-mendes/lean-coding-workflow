# Lean Coding Workflow

> **Plan enough. Read only what you need. Execute one step at a time. Remember what matters.**
> 计划足够即可，按需读取上下文，逐步执行，只记住真正重要的事情。

一个轻量、Token 高效、文档驱动、可恢复执行的 AI Coding Workflow Skill。

## 核心特性

- **Context-on-Demand（按需读取上下文）**：核心原则 "Never read context speculatively"（禁止猜测性读取上下文）。只有"不读取就无法可靠完成当前任务"时才读取，上下文足够后立即停止。
- **文档驱动开发**：需求 → 子需求 → 方案 → 执行计划 → 逐步执行 → 验证 → 记忆 → 收尾。
- **严格 Phase 状态机**：Phase 0 初始化 → Phase 1 需求分析 → 用户确认① → Phase 2 方案规划 → 用户确认② → Phase 3 逐步执行 → Phase 4 子需求完成；任意阶段可进入 Phase 5 问题排查 / 恢复 / 回退。
- **一次只执行一个 Step**：execution-plan.md 只保存剩余工作（Future Work），完成即删、失败保留，天然支持中断恢复。
- **Minimal Verification**：验证范围与修改影响匹配，禁止每个 Step 都跑完整测试套件。
- **语义化 Context Sufficiency**：不计算 Token / Context Window 数字，只判断"当前上下文是否足以可靠完成任务"。

## 项目目录约定

```
project/
├── PRD.md                  # 长期：项目当前最终需求状态
├── PRD-V1.md …             # 长期：版本化需求（可选，命名不限，可用任意 @xxx.md 作为需求入口）
├── AGENTS.md                # 长期：AI Coding 规则
└── artifacts/
    ├── architecture.md     # 长期：架构约束（只有架构真正变化时才更新）
    ├── prd-sub.md          # 临时：当前子需求
    ├── plan.md             # 临时：当前子需求整体方案
    ├── execution-plan.md   # 临时：剩余待执行 Step
    └── memory.md           # 临时：当前子需求执行记忆
```

临时文件在子需求完成后**清空内容、绝不删除**，长期知识选择性沉淀到 PRD.md / architecture.md。

plan.md 定位为**跨 Step 方案决策记录**（短文档，不复述 Step 细节）；长期性技术选型在 Phase 2 即时沉淀进 architecture.md，编码层栈约束（框架版本、目录约定、测试规则）进 AGENTS.md。

## 安装

复制到 ZCode 的用户级 skill 目录：

```bash
git clone https://gitee.com/echola/lean-coding-workflow ~/.agents/skills/lean-coding-workflow
```

或项目级：`<project>/.agents/skills/lean-coding-workflow/`。

## 文件结构

```
lean-coding-workflow/
├── SKILL.md                        # 核心执行规则（触发条件、Phase 状态机、读取矩阵、Token 规则）
└── references/
    ├── file-formats.md             # prd-sub / plan / execution-plan / memory 格式模板
    └── document-contracts.md       # 文件职责、生命周期、architecture 更新判断、清理规则
```

## 使用

在 ZCode 中直接提出开发需求即可触发，例如：

```
@docs/travel.md 实现差旅审批
```

工作流会在两个关键点暂停等待确认（子需求确认、执行计划确认），其余时间自动逐步执行。

## Context Loading Matrix（各 Phase 读取矩阵）

> 供人查阅的汇总视图；执行时的规则以 SKILL.md 各 Phase 小节为准，模型执行时无需读取本表。

| Phase | 需求文档(@xxx.md / PRD-Vx) | PRD.md | AGENTS.md | architecture.md | prd-sub.md | plan.md | execution-plan.md | memory.md | 源码 |
|---|---|---|---|---|---|---|---|---|---|
| Phase 0 初始化（仅检查存在性） | ❌ | ⚠️ | ⚠️ | ⚠️ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Phase 1 需求分析 | ✅（用户 @ 指定） | ⚠️（仅当用户未指定文档时按需） | ✅ | ⚠️ | ❌（本阶段生成） | ❌ | ❌ | ❌ | ⚠️ |
| Phase 2 方案规划 | ❌ | ❌ | ✅ | ⚠️（长期选型可写入） | ✅ | ❌（本阶段生成） | ❌（本阶段生成） | ⚠️ | ⚠️ |
| Phase 3 Step 执行 | ❌ | ❌ | ✅ | ⚠️ | ❌ | ❌ | ✅ | ❌ | ✅ |
| Phase 4 子需求完成 | ❌ | ✅（写） | ✅ | ⚠️（判断后写） | ✅（读后清空） | ✅（清空） | ✅（读最终状态后清空） | ✅（提炼后清空） | ⚠️ |
| Phase 5 问题排查 | ⚠️（必要历史需求文档） | ⚠️ | ⚠️ | ⚠️ | ⚠️ | ❌ | ⚠️（必要时更新） | ⚠️ | ✅ |

✅ 必须读取　⚠️ 只有必要时读取（默认禁止）　❌ 默认禁止读取
