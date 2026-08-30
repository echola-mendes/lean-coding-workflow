# Document Contracts — 文件职责 / 生命周期 / 更新规则

读取时机：Phase 2 架构写入判断、Phase 4 同步与清理，
或对任何文件的职责边界有疑问时读取。

何时创建哪个文件：以 SKILL.md 各 Phase 动作为准，本文件不重复。

## 职责速查

| 文件 | 职责 | 生命周期 |
|---|---|---|
| PRD.md | 项目当前最终需求状态 | Permanent（可选：存在才在 Phase 4 同步） |
| @xxx.md / PRD-Vx.md | 用户指定的需求来源 | Permanent（用户所有，只读） |
| AGENTS.md | AI Coding 行为规范（约定文件；Claude Code 回退 CLAUDE.md） | Permanent（必读；缺失时 Phase 4 按实际约定创建） |
| artifacts/architecture.md | 长期架构约束 | Permanent（有长期架构变化时创建 / 更新） |
| artifacts/prd-sub.md | 当前子需求 | Temporary（Phase 4 归档后保留；Phase 1 启动核对后清空） |
| artifacts/execution-plan.md | 当前子需求全部 Step 与验收勾选 | Temporary（Phase 4 归档后保留；Phase 1 启动核对后清空） |
| artifacts/memory.md | 项目长期经验记忆 | Permanent（不清空；禁止流水；必要时压缩） |

禁止让多个文档承担相同职责。

## 各文件契约

### PRD.md

- 职责：表示"当前项目最终需要实现什么"。**可选文件**：不存在则不创建、不同步
  （用户明确要求时才创建），不阻塞流程。
- 更新：每个子需求完成后（Phase 4）同步最终需求结果——仅当文件存在。
- 禁止：记录 Step 执行日志、完整测试日志、每个 Step 都修改、复制 execution-plan 内容。

### @xxx.md / PRD-Vx.md（需求来源文档）

- 任意路径、任意命名的 .md 文件；由用户 @ 指定。
- 职责由"用户当前指定"决定，不按文件名猜测。
- **只读**：本工作流永不修改用户指定的需求文档；成果沉淀只进长期文档。
- Phase 1 用户指定时必读；Phase 2 / Phase 3 默认禁止重读（除非当前任务确实无法完成）。
- 禁止自动搜索并读取所有 PRD-Vx.md。

### AGENTS.md — 约定文件

- 职责：技术栈、编码规范、命名规范、项目约定、测试规则、AI 修改代码必须遵守的规则、开发限制。
- **必读**：Phase 1 / 2 / 3 每次读取；缺失视为无附加规则，不阻塞执行、不臆造规则。
- **宿主回退**：项目无 AGENTS.md 而 CLAUDE.md 存在（Claude Code 项目）→ 读 CLAUDE.md，等同约定文件；
  两者都缺视为无附加规则。
- 创建：缺失时 Phase 4 依据本轮**实际执行中采用的约定**创建（测试命令、命名、目录、
  框架版本等），创建前告知用户；已有 CLAUDE.md 且无 AGENTS.md → 追加进 CLAUDE.md
  （宿主可自动加载），禁止覆盖；新项目的栈约束在 Phase 2 先记录于 execution-plan.md 头部。
- 已存在时禁止覆盖，只追加 / 修正。

### artifacts/architecture.md

- 职责：只记录长期有效的结构性信息 —— 模块边界、目录结构原则、系统分层、
  技术选型、数据架构、API 架构、Agent / Tool 架构、长期组件、长期设计约束。
- **更新判断（写之前必答）**："这个变化是否会成为未来开发必须遵守的长期架构规则？"
  - NO → 不创建 / 不更新。
  - YES → 创建 / 更新。
- 例外：Phase 2 做出长期性技术选型时立即写入（不等 Phase 4）；
  **新项目（无约定文件 AGENTS/CLAUDE.md、无 architecture.md）首次做结构性决策时，Phase 2 必须产出架构设计并写入**。
  除上述两类，Phase 2 不写本文件，其余架构内容仍按 Phase 4 判断。
- 需要更新的例子：整体目录结构变化、新增长期模块边界、新增接口架构规范、
  新增统一技术组件、技术栈变化、数据存储架构变化、Agent / Tool 架构变化、新增长期开发规则。
- 不需要更新的例子：新增普通 Class / Controller / Service / DTO、普通业务逻辑修改、
  Bug 修复、普通字段增加、普通 SQL 修改。
- 禁止：记录执行历史、Step 日志、测试日志。

### artifacts/prd-sub.md — 当前子需求

- 临时，只服务当前子需求（生命周期见 SKILL.md §2），禁止删除文件。
- 是 AI 梳理后的需求，不是需求文档的复制。

### artifacts/execution-plan.md — 当前子需求全部 Step 与验收（唯一计划文件）

- Phase 2 生成：全部 Step，每个 Step 三节（目标 / 方案 / 验收，格式见 file-formats.md）。
- 验收勾选即唯一执行状态（四态定义与执行规则见 file-formats.md）；无独立恢复流程。
- 生命周期（归档 / 清空）见 SKILL.md §2；长期经验进 memory.md。

### artifacts/memory.md — 项目长期经验记忆

- **Permanent，不清空**：项目级长期记忆——完成记录表（一行一子需求）+ 经验条目（踩坑与解法、
  "为什么这么做"的决策理由、可复现问题）；只收 PRD / architecture / AGENTS 放不下的内容；
  不记录执行流水。
- 容量：全文 <300 行（结构、写入时机与压缩规则见 file-formats.md）。

## 子需求完成清理规则（Phase 4）

清理动作以 SKILL.md Phase 4 为准，生命周期见职责速查表；此处只保留历史溯源约定——
溯源不依赖临时文件：PRD.md（若存在）记录最终需求结果，architecture.md 记录长期架构变化，
memory.md 记录长期经验与决策理由，git 历史 + 用户原始需求文档（只读）保留完整过程与需求来源。

## 禁止的设计

- 验收勾选（`[ ]` / `[✅]` / `[❌]` / `[⚠️]`）即执行状态。
- 巨大的 project-context.md；无限增长的 memory.md；memory.md 记录执行流水。
- 新建 memory.md 之外的任何记忆文件。
- 修改用户提供的原始需求文档（@xxx.md 只读）。
- 在多个临时文件重复写入同一份状态指针 / IDLE 占位内容。
- 自动创建 AGENTS.md / PRD.md / 任何工作文件（仅用户明确要求时创建）。
- 每个 Step 自动读取所有文档 / 整个 architecture.md / 整个 memory.md。
- 每个 Step 自动运行完整测试。
- 自动扫描整个项目源码；自动读取所有 PRD-Vx.md。
- 每个 Step 重新分析完整需求；每个 Step 都要求用户确认。
- 用 Token 数字或固定 Context 百分比决定是否继续读取上下文。
- 把所有执行历史永久写入项目；为了"完整性"创建大量辅助文档。
