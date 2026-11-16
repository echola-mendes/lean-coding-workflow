# Lean Coding Workflow — External Skills & MCP Integration

> 主题：Java backend-skill + Lanhu MCP Integration  
> 目标仓库：`echola-mendes/lean-coding-workflow`  
> 文档状态：Draft for implementation

## 1. 背景

Lean Coding Workflow 当前负责从需求澄清到开发完成的主流程。Java 项目已有独立的 `backend-skill`，产品设计信息可由 Lanhu MCP 提供。若 Lean 重复实现这些能力，会产生能力分叉、状态冲突和维护成本。

本需求要求 Lean 保持轻量：只负责发现、选择、调用外部能力，并统一管理流程、确认点和开发进度；不修改或复制外部能力本体。

## 2. 问题

当前需要解决以下问题：

1. Lean 缺少统一的外部 Skill / MCP 发现与调用约定。
2. Lanhu 原型、标注、交互信息尚未稳定进入需求分析上下文。
3. `backend-skill` 与 Lean 的职责边界不明确，可能重复生成任务和进度状态。
4. Phase 3 缺少明确的 Code Review 完成条件。
5. 执行中的需求变更可能覆盖原计划、重置已完成工作或破坏历史可追溯性。

## 3. 目标

1. 保持主流程不变：

   `Phase 1 Requirement → Confirm① → Phase 2 Planning → Confirm② → Phase 3 Execution → Phase 4 Finish`

2. 支持 Lean 发现、选择并调用外部 Skills 与 MCP。
3. 将 Lanhu MCP 作为可选的 Requirement Source，并把读取结果归一化为 Requirement Facts。
4. 在 Java 项目中复用 `backend-skill` 的分析、实现、审核、测试和变更能力。
5. 以 `execution-plan.md` 作为唯一开发进度 Source of Truth。
6. 将 Code Review 设为 Phase 3 中代码变更 Step 的强制完成条件。
7. 以增量 Change Flow 处理 Phase 3 需求变更，保留计划与执行历史。

## 4. 非目标

1. 不修改、复制、内嵌或重新发布 `backend-skill`。
2. 不修改、复制或重新实现 Lanhu MCP。
3. 不让 Lanhu 成为唯一需求来源。
4. 不引入独立的 Gate Engine、任务系统或第二套进度状态。
5. 不长期维护 Java / Lanhu 专用分支。
6. 不在第一版集成 SonarQube、覆盖率阈值或复杂的缺陷分级策略。

## 5. 职责边界

| 能力 | Lean Coding Workflow | backend-skill | Lanhu MCP |
|---|---|---|---|
| 主流程与 Phase 推进 | 负责 | 不负责 | 不负责 |
| Confirm①/②/③ | 负责触发与记录 | 提供分析依据 | 提供事实依据 |
| 外部能力发现、选择、调用 | 负责 | 被调用 | 被调用 |
| Java 需求/设计分析 | 编排并汇总 | `proj-analyze-req`、`proj-analyze-design` | 不负责 |
| Java 实现/审核/测试 | 编排并判断完成条件 | `proj-gen`、`proj-review`、`proj-gen-test`、`proj-run-test` | 不负责 |
| 需求变更影响分析 | 编排、记录、更新计划 | `proj-change` | 可补充变更后的设计事实 |
| 原型、标注、交互读取 | 归一化并消费 | 不负责 | 负责读取 |
| 开发进度 | `execution-plan.md` 唯一维护者 | 不建立第二套状态 | 不负责 |

外部能力不可用时，Lean 应明确报告缺失能力及影响，不得假装已经执行。

## 6. 完整流程

```text
Phase 1 Requirement
  ├─ 收集文本、文件、Issue、Lanhu URL 等 Requirement Sources
  ├─ 检测 Lanhu URL → 调用 Lanhu MCP
  ├─ 归一化 Requirement Facts
  └─ 可调用 proj-analyze-req
        ↓
Confirm①：确认需求
        ↓
Phase 2 Planning
  ├─ 可调用 proj-analyze-design
  ├─ 生成实施方案
  └─ 创建 execution-plan.md
        ↓
Confirm②：确认方案与执行计划
        ↓
Phase 3 Execution
  ├─ 按 execution-plan.md 执行 Step
  ├─ 代码 Step：Implementation → Review → Test → Acceptance → DONE
  └─ 需求变化：进入 Change Flow，完成后恢复执行
        ↓
Phase 4 Finish
  ├─ 验证所有必要 Step 与 Change Step 已完成
  ├─ 汇总实现、审核、测试和变更记录
  └─ 输出完成结果
```

## 7. Phase 1 — Requirement

### 7.1 Requirement Sources

Lean 应支持多个需求来源，包括但不限于：

- 用户对话输入
- 本地需求文档
- Issue / Ticket
- Lanhu URL
- 其他可用 MCP 或外部数据源

Lanhu MCP 只是 Requirement Source 之一。不存在 Lanhu URL 时，流程必须可以正常运行。

### 7.2 Lanhu URL 处理

检测到 Lanhu URL 后，Lean 应：

1. 检查 Lanhu MCP 是否可用。
2. 读取与 URL 对应的原型页面、视觉标注和交互信息。
3. 将结果归一化为 Requirement Facts，而不是直接把原始响应当作最终需求。
4. 标记事实来源，保留页面或节点引用，便于追溯。
5. 对缺失、冲突或无法读取的信息明确列为 Open Questions。

Requirement Facts 至少包含适用项：

- 页面/模块
- 用户角色与场景
- 功能行为
- 页面字段与校验
- 状态与状态转换
- 交互规则
- 接口或数据约束
- 异常与边界情况
- 来源引用
- Open Questions

### 7.3 Java 需求分析

Java 项目中，Lean 可将归一化后的 Requirement Facts 交给 `proj-analyze-req` 分析。Lean 负责整合分析结果并推进 Confirm①；`proj-analyze-req` 不直接改变 Phase 或进度状态。

### 7.4 Confirm①

进入 Phase 2 前，用户需确认需求范围、关键规则和未决问题。未解决且会影响实现方案的问题不得被静默假设。

## 8. Phase 2 — Planning

1. Lean 可调用 `proj-analyze-design` 生成或校验 Java 技术设计。
2. Lean 根据已确认需求和设计生成 `execution-plan.md`。
3. 每个 Step 应可独立执行和验收，并标明是否会产生代码变更。
4. Lean 不调用 `proj-task` 建立另一份任务状态；不依赖 `proj-resume` 恢复另一套进度。

### Confirm②

用户确认技术方案、Step 拆分和验收口径后进入 Phase 3。后续执行进度只写入 `execution-plan.md`。

## 9. Phase 3 — Execution

Lean 逐个执行 `execution-plan.md` 中的 Step，并负责状态推进。

### 9.1 Step 类型

- **Code Step**：新增、修改或删除代码、配置、数据库脚本等会影响运行行为的内容。
- **Non-code Step**：仅文档、调查、分析或无需代码变更的操作。

Code Step 必须经过 Mandatory Code Review；Non-code Step 不强制 Code Review，但仍需满足自身 Acceptance。

### 9.2 Code Step 标准闭环

```text
Implementation（默认 proj-gen）
  → proj-review
  → 存在必须修复的问题？
      ├─ 是：Fix → 再次 proj-review
      └─ 否：Test（proj-gen-test / proj-run-test）
  → Acceptance
  → DONE
```

### 9.3 DONE 条件

Code Step 只有同时满足以下条件才能标记 `DONE`：

- Implementation completed
- Review passed
- Test passed
- Acceptance passed

Non-code Step 的 DONE 条件为工作产物完成且 Acceptance passed。

## 10. Mandatory Code Review

“Mandatory Code Review”是 Workflow 的强制步骤和完成条件，不是新增的审核工具或 Gate Engine。

- Lean 规定所有产生代码变更的 Step 在 DONE 前必须 Review。
- 默认由 `backend-skill` 的 `proj-review` 执行 Java 代码审核。
- Lean 根据 Review 结果决定是否允许进入 Test 和 DONE。
- Review 发现必须修复的问题时，Step 保持 `IN_PROGRESS`，修复后必须重新 Review。
- 若 `proj-review` 不可用，Lean 应暂停该 Code Step 并报告阻塞；不得跳过后直接标记 DONE。

第一版不规定固定的 Critical/Major 数量阈值；“必须修复”优先遵循 `proj-review` 的审核结论和当前项目规范。

`execution-plan.md` 中应记录最小审核结果：

```markdown
Review:
- Skill: proj-review
- Result: PASS
- Required fixes: 0
```

## 11. Phase 3 Change Flow

Phase 3 出现需求变更时，禁止直接修改后继续执行，也禁止重新生成或覆盖整个 `execution-plan.md`。

```text
Pause current execution
  → 保存当前 Step checkpoint
  → proj-change：Impact Analysis
  → Change Proposal
  → 条件式 Confirm③
  → Patch Plan
  → 追加 CR-001 与 Change Steps
  → Implementation
  → proj-review
  → Regression Test
  → Acceptance
  → Resume Phase 3
```

### 11.1 Pause 与 Checkpoint

Lean 暂停当前执行，并在 `execution-plan.md` 记录：

- 当前 Step 与状态
- 已完成内容
- 未完成内容
- 工作区或提交引用（如适用）
- 暂停原因

### 11.2 Impact Analysis 与 Change Proposal

Lean 默认调用 `proj-change` 分析：

- 需求差异
- 受影响模块、接口、数据和测试
- 对已完成 Step 的影响
- 对当前 `IN_PROGRESS` Step 的影响
- 兼容性与回归风险
- 建议的 Patch Plan

Change Proposal 至少应包含变更摘要、影响范围、处置建议和验收变化。

### 11.3 条件式 Confirm③

满足任一条件时必须 Confirm③：

- 改变已在 Confirm① 确认的需求范围或行为
- 改变 Confirm② 确认的技术方案或验收标准
- 影响已完成 Step
- 需要 `ROLLBACK` 或 `SUPERSEDE` 当前工作
- 引入明显的兼容性、数据或交付风险

仅为不改变已确认范围和方案的澄清、错字修正或低风险实现细节调整时，可不单独确认，但必须记录理由。

### 11.4 Patch Plan 与历史保留

- 只对受影响部分生成 Patch Plan。
- 在原文件中追加 `CR-001`、`CR-002` 等 Change Request。
- 在对应 CR 下追加 Change Steps。
- 不删除、不重置已完成 Step。
- 已完成 Step 受影响时，保留 `DONE`，并增加 `Change Impact: CR-xxx`。
- Change Step 与普通 Code Step 使用相同 Review、Test、Acceptance 规则；测试需包含 Regression Test。

### 11.5 IN_PROGRESS Step 处置

| 处置 | 含义 |
|---|---|
| `KEEP` | 当前工作不受影响，完成 Change Flow 后继续 |
| `ADJUST` | 保留已有工作，按 Patch Plan 调整剩余内容 |
| `SUPERSEDE` | 当前 Step 被新的 Change Step 替代，保留历史并指向替代项 |
| `ROLLBACK` | 明确撤销未完成工作；记录原因、范围和恢复点 |

Change Flow 完成后，Lean 从 checkpoint 和更新后的 `execution-plan.md` 恢复 Phase 3。

## 12. Phase 4 — Finish

进入 Finish 前，Lean 应验证：

1. 所有必要的普通 Step 和 Change Step 已完成。
2. 所有 Code Step 均有通过的 Review、Test 和 Acceptance 记录。
3. 所有 Change Request 均有最终状态和回归测试结果。
4. 不存在未处置的 `IN_PROGRESS`、阻塞项或必须确认的 Open Questions。
5. 最终交付摘要与 `execution-plan.md` 一致。

## 13. 状态与文档规则

### 13.1 Source of Truth

`execution-plan.md` 是开发 Step、Change Request、Review、Test、Acceptance 和 checkpoint 的唯一进度 Source of Truth。

- `proj-task` 不创建并行任务账本。
- `proj-resume` 不作为恢复进度的来源。
- 外部 Skill 的临时输出可作为证据，但不得覆盖 Lean 的主状态。

### 13.2 建议状态

Step 使用最小状态集：

- `PENDING`
- `IN_PROGRESS`
- `BLOCKED`
- `DONE`
- `SUPERSEDED`（仅用于被变更替代的未完成 Step）

Change Request 使用：

- `PROPOSED`
- `CONFIRMED`（需要 Confirm③ 时）
- `IN_PROGRESS`
- `DONE`
- `REJECTED`

### 13.3 最小记录示例

```markdown
## Step 3 — Message Service

Status: DONE
Type: CODE

Implementation:
- Files: ...

Review:
- Skill: proj-review
- Result: PASS
- Required fixes: 0

Test:
- Unit: PASS
- Integration: PASS

Acceptance: PASS
Change Impact: CR-001
```

## 14. 外部能力发现与选择规则

1. Lean 在需要调用前检查目标 Skill / MCP 是否可用。
2. 按任务类型选择能力，不因安装了某能力就强制调用。
3. 默认映射：

| 场景 | 默认外部能力 |
|---|---|
| Java 需求分析 | `proj-analyze-req` |
| Java 技术设计 | `proj-analyze-design` |
| Java 代码实现 | `proj-gen` |
| Java 代码审核 | `proj-review` |
| Java 测试生成 | `proj-gen-test` |
| Java 测试执行 | `proj-run-test` |
| Phase 3 需求变更 | `proj-change` |
| Lanhu 设计读取 | Lanhu MCP |

4. 用户明确指定其他可用能力时，Lean 可覆盖默认选择，并在执行记录中注明。
5. 外部调用结果必须回写为 Lean 可判断的结构化结论或记录，不把控制权交给外部能力。

## 15. 兼容性要求

1. 没有安装 `backend-skill` 时，非 Java 或不依赖该 Skill 的既有流程保持可用。
2. 没有配置 Lanhu MCP 时，非 Lanhu 来源的需求流程保持可用。
3. 没有 Lanhu URL 时，不触发 Lanhu MCP。
4. `backend-skill` 和 Lanhu MCP 保持可被其他 Workflow 独立使用。
5. 现有 `execution-plan.md` 应可继续读取；新增字段应采用向后兼容的可选结构。
6. 本集成不得要求长期维护独立的 Java / Lanhu 版本。

## 16. 验收标准

### 16.1 主流程

- [ ] 无外部能力时，既有 Phase 1～4 流程仍可运行。
- [ ] Confirm①、Confirm② 的职责与触发位置保持不变。

### 16.2 Lanhu Integration

- [ ] 输入包含 Lanhu URL 时可检测并调用 Lanhu MCP。
- [ ] 原型、标注、交互信息被归一化为带来源的 Requirement Facts。
- [ ] Lanhu 不可用或信息不完整时，能报告问题并形成 Open Questions。
- [ ] 不含 Lanhu URL 时不调用 Lanhu MCP。

### 16.3 backend-skill Integration

- [ ] Java 场景可按阶段调用七个目标 Skills。
- [ ] 外部 Skill 不直接维护 Phase 或 Step 状态。
- [ ] 不通过 `proj-task` / `proj-resume` 建立第二套进度。

### 16.4 Mandatory Code Review

- [ ] 每个 Code Step 在 DONE 前至少有一次 `proj-review` 记录。
- [ ] Review 存在必须修复的问题时无法进入 DONE。
- [ ] 修复后重新 Review，通过后才进入 Test。
- [ ] Non-code Step 不被错误要求 Code Review。

### 16.5 Change Flow

- [ ] Phase 3 需求变更会暂停当前执行并保存 checkpoint。
- [ ] 生成 Impact Analysis、Change Proposal 和增量 Patch Plan。
- [ ] 满足条件时触发 Confirm③。
- [ ] 原 `execution-plan.md` 不被整体覆盖。
- [ ] 已完成 Step 不删除、不重置，受影响时记录 Change Impact。
- [ ] `IN_PROGRESS` Step 可记录 KEEP/ADJUST/SUPERSEDE/ROLLBACK。
- [ ] Change Step 完成前经过 Review、Regression Test 和 Acceptance。
- [ ] Change Flow 完成后可恢复 Phase 3。

### 16.6 真实需求验证

- [ ] 使用 2～3 个真实需求完成端到端验证。
- [ ] 至少一个需求包含 Lanhu URL。
- [ ] 至少一个需求触发 Phase 3 Change Flow。
- [ ] 至少一个 Code Step 出现 Review → Fix → Re-review 闭环。

## 17. 实施优先级

### P0 — 可用闭环

- 外部 Skill / MCP 的发现、可用性检查与调用约定
- Lanhu URL 检测、读取和 Requirement Facts 归一化
- 七个 `backend-skill` 能力的阶段映射
- `execution-plan.md` 唯一进度源规则
- Code Step Mandatory Code Review
- Phase 3 增量 Change Flow、checkpoint、CR 与 Change Steps
- 基本兼容性和端到端验证

### P1 — 稳定性与可追溯性

- 外部调用结果的统一记录格式
- Requirement Facts 来源引用和冲突提示
- Review / Test / Acceptance 证据链接
- Change Impact 与回归范围记录优化
- 失败恢复和明确的阻塞信息

### P2 — 体验优化

- 更多 Requirement Sources / Skills 的适配模板
- 外部能力选择提示优化
- `execution-plan.md` 展示与汇总优化
- 在不引入第二套状态的前提下提供统计或报告

## 18. 分支与版本策略

1. 在 `feature/skill-mcp-integration` 分支完成首轮实现。
2. 使用 2～3 个真实需求验证 P0 验收项，并记录问题与调整。
3. 验证成熟后合并到 `main`，发布一个包含集成功能的新版本。
4. 不长期维护 Java / Lanhu 专用分支；后续修复和增强回到主线演进。

## 19. 完成定义

本需求完成需同时满足：

- P0 功能实现并通过全部对应验收标准。
- 2～3 个真实需求验证完成，覆盖 Lanhu、Review 闭环和 Change Flow。
- `execution-plan.md` 被验证为唯一进度源，无第二套状态产生。
- `backend-skill` 与 Lanhu MCP 本体未被修改或复制。
- 功能合并 `main` 并发布新版本，或已形成可合并、可发布的候选版本。
