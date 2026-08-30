# lean-coding-workflow

> **Plan enough. Read only what you need. Execute one step at a time. Remember what matters.**
> **计划足够即可，按需读取上下文，逐步执行，只记住真正重要的事情。**
 
## 为什么写这个 Skill

直接目的：**减少 `context_usage`。** 少读、读准、读完就停，把窗口留给真正要改的代码。

传统 AI 编码高度依赖对话上下文：会话越长，上下文越膨胀，Token 浪费越多；会话一旦中断或丢失，任务进度与中间决策也随之消失。

默认的 AI 编码过程往往是：把能读的都读一遍 → 一口气改很多文件 → 跑一轮测试（或干脆不测）→ 对话一长或中断，上下文和决策就丢了。`context_usage` 被噪声堆高，真正有用的信息反而被挤掉。

这会带来三个实际问题：

1. **上下文被噪声填满。** Agent 因为「这个文件可能有用」就读它。Token 花在无关代码上，真正该改的地方反而看不清——又贵、又慢、还更容易改错。
2. **进度与结果只活在对话里。** 一次实现一串需求，做到哪一步、验收过没有全凭对话记忆；会话一换全部归零，出了问题也不知道该修哪一步。
3. **知识和日志混在一起。** 把执行过程、架构决策、需求原文、测试输出塞进同一个无限增长的 memory，下次还得整份再读一遍——等于没有记忆，还继续推高 `context_usage`。

`lean‑coding‑workflow` 是一套文档驱动的 Skill：只加载最小必要上下文，计划、执行与验收结果全部落在文件里——新开对话照着文件就能继续做，不依赖历史对话。

这个 Skill 不是又一个「请仔细思考再动手」的提示词。它把上述失败模式写成**可执行约束**：什么时候读、读到什么程度停、何时问人、一次改多大、验证改多大、什么进长期文档、临时文档何时清。

目标是压低 `context_usage`；手段是语义判断「这一步够不够」，而不是盯着百分比刹车。

## 设计理念

### 四句总纲

| 理念 | 含义 |
|---|---|
| **Plan enough** | 对齐到位就动手：不欠计划，也不过度设计。 |
| **Read only what you need** | 按需读取上下文，禁止猜测性读取。问的是「不读这个文件，当前任务是否无法可靠完成」，不是「读了会不会更有把握」。够了立刻停。 |
| **Execute one step at a time** | 按执行计划一次执行一个 Step；完成后执行对应验收。验证范围跟着修改范围走，不每个 Step 跑全量测试。 |
| **Remember what matters** | 每个文件只有一个职责。完成后总结进 memory 并保留，下一子需求启动核对后清空重建；memory.md 是长期经验记忆，不记录执行流水；只有会约束未来开发的决策才进长期文档。 |

两条刻意不做的选择：

- **不拿 Token 数字当刹车。** 要减少的是实际读进去的内容，从而压低 `context_usage`；但不计算 `remaining_tokens`，不设「context 用了 60% 就别读了」。只做语义判断：当前上下文是否已经足够可靠完成这一步。
- **不追求「知道整个项目」。** 目标是 Minimum Reliable Context：带着完成当前任务所需的最小可靠信息把事做对，而不是把仓库装进脑子。

### 落地形态

- **文档驱动开发**：需求 → 子需求 → 执行计划 → 逐步执行 → 验收 → 沉淀 → 收尾。
- **严格 Phase 流程**：Phase 1 需求分析 → 确认① → Phase 2 方案规划 → 确认② → Phase 3 逐步执行 → Phase 4 子需求完成。
- **长期 / 临时两层文档**：
  - 长期：`PRD.md`（需求状态，可选）、`AGENTS.md`（编码规则，必读）、`architecture.md`（架构约束，按需创建）、`memory.md`（长期经验）
  - 临时：`prd-sub.md` / `execution-plan.md`（归档后保留，下个子需求启动时清空）

## 工作流

```
Phase 1 需求分析 ──→ 【用户确认①】prd-sub.md
   ↓
Phase 2 方案规划 ──→ 【用户确认②】execution-plan.md
   ↓
Phase 3 Step 执行（一次一个 Step：实现 → 验收 → 标 ✅ / ❌ / ⚠️，全部 ✅ 进入 Phase 4）
   ↓
Phase 4 子需求完成（同步 PRD（若存在）/ 更新 architecture / 归档 memory（完成记录+经验）与 AGENTS → 临时文件保留，下一子需求启动时清空重建）
```

正常流程只有两个强制确认点。Step 执行过程中不逐步请示；仅在需求歧义、重大架构或方案变化、计划与实际冲突、验收失败无法自行解决、高风险操作时暂停。

## 项目目录约定

```
project/
├── PRD.md                  # 长期：项目当前最终需求状态（可选：存在才同步）
├── PRD-V1.md …             # 长期：版本化需求（可选；也可用任意 @xxx.md 作为需求入口，只读）
├── AGENTS.md               # 长期：AI 编码规则（必读；Claude Code 项目回退 CLAUDE.md；缺失时 Phase 4 创建）
└── artifacts/
    ├── architecture.md     # 长期：架构约束（有长期架构变化时创建 / 更新）
    ├── prd-sub.md          # 临时：当前子需求（归档后保留，下个子需求启动时清空）
    ├── execution-plan.md   # 临时：全部 Step（目标 / 方案 / 验收，勾选即状态）
    └── memory.md           # 长期：项目经验记忆（不清空，禁止流水，必要时压缩）
```

各文件的职责、生命周期与创建时机详见 `skills/lean-coding-workflow/references/document-contracts.md`。

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
│           ├── file-formats.md     # prd-sub /  execution-plan / memory 格式模板
│           └── document-contracts.md
├── .gitignore
├── LICENSE
└── README.md
```
