# lean-coding-workflow

> **Plan enough. Read only what you need. Execute one step at a time. Remember what matters.**
> 计划足够即可，按需读取上下文，逐步执行，只记住真正重要的事情。

一套给 AI Agent 用的编码工作流协议：文档驱动需求与方案，用最少可靠上下文逐步实现，把可恢复状态写在文件里，而不是对话历史里。

## 为什么写这个 Skill

直接目的：**减少 `context_usage`。** 少读、读准、读完就停，把窗口留给真正要改的代码。

默认的 AI 编码过程往往是：把能读的都读一遍 → 一口气改很多文件 → 跑一轮测试（或干脆不测）→ 对话一长或中断，上下文和决策就丢了。`context_usage` 被噪声堆高，真正有用的信息反而被挤掉。

这会带来三个实际问题：

1. **上下文被噪声填满。** Agent 因为「这个文件可能有用」就读它。Token 花在无关代码上，真正该改的地方反而看不清——又贵、又慢、还更容易改错。
2. **执行不可恢复。** 一次实现一串需求，失败了不知道改到哪、该回退什么。换一次会话，计划只活在上一轮对话里。
3. **知识和日志混在一起。** 把执行过程、架构决策、需求原文、测试输出塞进同一个无限增长的 memory，下次还得整份再读一遍——等于没有记忆，还继续推高 `context_usage`。

这个 Skill 不是又一个「请仔细思考再动手」的提示词。它把上述失败模式写成**可执行约束**：什么时候读、读到什么程度停、何时问人、一次改多大、验证改多大、什么进长期文档、什么做完就清掉。

目标是压低 `context_usage`；手段是语义判断「这一步够不够」，而不是盯着百分比刹车。

## 设计理念

### 四句总纲

| 理念 | 含义 |
|---|---|
| **Plan enough** | 对齐到位就动手：不欠计划，也不过度设计。 |
| **Read only what you need** | 按需读取上下文，禁止猜测性读取。问的是「不读这个文件，当前任务是否无法可靠完成」，不是「读了会不会更有把握」。够了立刻停。 |
| **Execute one step at a time** | 执行计划保留全部 Step 并维护状态行，状态变更即落盘；中断后凭状态行 + git diff 恢复。验证范围跟着修改范围走，不每个 Step 跑全量测试。 |
| **Remember what matters** | 每个文件只有一个职责。临时文档服务当前子需求，完成后清空；memory.md 只沉淀长期经验，禁止执行流水；只有会约束未来开发的决策才进长期文档。 |

两条刻意不做的选择：

- **不拿 Token 数字当刹车。** 要减少的是实际读进去的内容，从而压低 `context_usage`；但不计算 `remaining_tokens`，不设「context 用了 60% 就别读了」。只做语义判断：当前上下文是否已经足够可靠完成这一步。
- **不追求「知道整个项目」。** 目标是 Minimum Reliable Context：带着完成当前任务所需的最小可靠信息把事做对，而不是把仓库装进脑子。

### 落地形态

- **文档驱动开发**：需求 → 子需求 → 方案 → 执行计划 → 逐步执行 → 验证 → 沉淀 → 收尾。
- **严格 Phase 状态机**：Phase 0 检查 → Phase 1 需求分析 → 确认① → Phase 2 方案规划 → 确认② → Phase 3 逐步执行 → Phase 4 子需求完成。
- **长期 / 临时两层文档**：
  - 长期：`PRD.md`（需求状态，可选）、`AGENTS.md`（编码规则，可选）、`architecture.md`（架构约束，按需创建）、`memory.md`（长期经验）
  - 临时：`prd-sub.md` / `plan.md`（按需）/ `execution-plan.md`

## 工作流

```
Phase 0 检查（只检查不创建）
   ↓
Phase 1 需求分析 ──→ 【用户确认①】prd-sub.md
   ↓
Phase 2 方案规划 ──→ 【用户确认②】execution-plan.md（plan.md 按需）
   ↓
Phase 3 Step 执行（一次一个 Step：标 IN_PROGRESS → 实现 → 最小验证 → 标 DONE，状态实时落盘）
   ↓
Phase 4 子需求完成（同步 PRD（若存在）/ 更新 architecture / 沉淀 AGENTS 与 memory → 清空临时文档）
```

正常流程只有两个强制确认点。Step 执行过程中不逐步请示；仅在需求歧义、重大架构变化、计划与代码冲突、失败无法自行解决、高风险操作时暂停。

## 项目目录约定

```
project/
├── PRD.md                  # 长期：项目当前最终需求状态（可选：存在才同步）
├── PRD-V1.md …             # 长期：版本化需求（可选；也可用任意 @xxx.md 作为需求入口，只读）
├── AGENTS.md               # 长期：AI 编码规则（可选：存在才读；缺失时 Phase 4 按实际约定创建）
└── artifacts/
    ├── architecture.md     # 长期：架构约束（有长期架构变化时创建 / 更新）
    ├── prd-sub.md          # 临时：当前子需求（完成后清空）
    ├── plan.md             # 临时：跨 Step 方案决策（按需创建，完成后清空）
    ├── execution-plan.md   # 临时：全部 Step 及状态行（完成后清空）
    └── memory.md           # 长期：项目经验记忆（不清空，禁止流水，必要时压缩）
```

临时文件在子需求完成后**清空内容、保留文件**；`memory.md` 是唯一长期记忆文件——只沉淀经验与决策理由，禁止执行流水。长期知识分流沉淀到 `PRD.md`（若有）/ `architecture.md` / `AGENTS.md`。

- `execution-plan.md` 保留全部 Step 并维护状态行（PENDING / IN_PROGRESS / VERIFYING / FAILED / BLOCKED / DONE），中断后凭状态行 + git diff 恢复。
- `plan.md` 只记横跨多个 Step 的决策与理由，按需创建。
- 长期性技术选型在 Phase 2 即时写入 `architecture.md`；新项目首次结构性决策必须产出架构设计。
- 编码层栈约束（框架版本、目录约定、测试规则）进 `AGENTS.md`；项目缺失该文件时，Phase 4 按实际执行约定创建。

## 安装

### Codex / Cursor

**全局**（本机所有项目）：

```bash
npx skills add https://gitee.com/echola/lean-coding-workflow.git \
  --skill lean-coding-workflow -a cursor -a codex -g --copy -y
```

**项目级**（写入当前仓库，可随 git 共享）：

```bash
npx skills add https://gitee.com/echola/lean-coding-workflow.git \
  --skill lean-coding-workflow -a cursor -a codex --copy -y
```

只装其中一个 Agent 时去掉对应 `-a`，例如只装 Cursor：`-a cursor`。GitHub 源把 URL 换成 `echola-mendes/lean-coding-workflow` 即可。

### Claude Code

本仓库即 plugin 根目录（`.claude-plugin/plugin.json` + `skills/`）。启用 plugin 后会自动加载 skill，**不要**再执行 `npx skills add -a claude-code`。

**会话试用**：

```bash
git clone https://gitee.com/echola/lean-coding-workflow.git
claude --plugin-dir ./lean-coding-workflow
```

**持久安装**（本机所有项目，下次启动自动加载为 `lean-coding-workflow@skills-dir`）：

```bash
git clone https://gitee.com/echola/lean-coding-workflow.git ~/.claude/skills/lean-coding-workflow
```

**项目级**：clone 到 `<project>/.claude/skills/lean-coding-workflow/`。

## 使用

必须通过 `/lean-coding-workflow` 显式调用。`@xxx.md` 是当前子需求输入，不是 Skill。

```
/lean-coding-workflow [@需求文档]
```

```
/lean-coding-workflow @PRD-V1.md
/lean-coding-workflow @docs/travel-approval.md
/lean-coding-workflow @requirements/mqtt.md
```

工作流在两个关键点暂停等待确认（子需求、执行计划），其余时间自动逐步执行。

## 仓库结构

```
lean-coding-workflow/
├── .claude-plugin/
│   └── plugin.json                 # Claude plugin 清单
├── skills/
│   └── lean-coding-workflow/
│       ├── SKILL.md                # 核心执行规则
│       ├── agents/
│       │   └── openai.yaml         # UI 元数据（列表展示名 / chips / 默认 prompt）
│       └── references/
│           ├── file-formats.md     # prd-sub / plan / execution-plan / memory 格式模板
│           └── document-contracts.md
├── .gitignore
├── LICENSE
└── README.md
```

## Context Loading Matrix（各 Phase 读取矩阵）

> 供人查阅的汇总视图；执行时的规则以 SKILL.md 各 Phase 小节为准，模型执行时无需读取本表。

| Phase | 需求文档(@xxx.md / PRD-Vx) | PRD.md | AGENTS.md | architecture.md | prd-sub.md | plan.md | execution-plan.md | memory.md | 源码 |
|---|---|---|---|---|---|---|---|---|---|
| Phase 0 检查（只检查不创建） | ❌ | ⚠️ | ⚠️ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Phase 1 需求分析 | ✅（用户 @ 指定） | ⚠️（仅当用户未指定文档时按需） | ✅（存在才读） | ⚠️ | ❌（本阶段生成） | ❌ | ❌ | ❌ | ⚠️ |
| Phase 2 方案规划 | ❌ | ❌ | ✅（存在才读） | ⚠️（长期选型 / 新项目架构必写） | ✅ | ❌（按需生成） | ❌（本阶段生成） | ⚠️（相关经验条目） | ⚠️ |
| Phase 3 Step 执行 | ❌ | ❌ | ✅（存在才读） | ⚠️ | ❌ | ❌ | ✅（状态实时写回） | ❌ | ✅ |
| Phase 4 子需求完成 | ❌ | ⚠️（存在才写） | ✅（缺失时创建） | ⚠️（判断后写） | ✅（读后清空） | ✅（清空） | ✅（读最终状态后清空） | ✅（分流追加） | ⚠️ |

✅ 必须读取　⚠️ 只有必要时读取（默认禁止）　❌ 默认禁止读取

PRD.md / AGENTS.md 为可选文件：不存在时跳过读取与同步，不阻塞流程。
