# Document Contracts — 文件职责 / 生命周期 / 更新规则

读取时机：Phase 0 创建基础文件、Phase 4 同步与清理、Phase 5 回退，
或对任何文件的职责边界有疑问时。

## 职责速查

| 文件 | 职责 | 生命周期 |
|---|---|---|
| PRD.md | 项目当前最终需求状态 | Permanent |
| @xxx.md / PRD-Vx.md | 用户指定的需求来源 | Permanent |
| AGENT.md | AI Coding 行为规范 | Permanent |
| artifacts/architecture.md | 长期架构约束 | Permanent（内容长期有效） |
| artifacts/prd-sub.md | 当前子需求 | Temporary（清空不删除） |
| artifacts/plan.md | 当前子需求整体方案 | Temporary（清空不删除） |
| artifacts/execution-plan.md | 剩余待执行 Step | Temporary（清空不删除） |
| artifacts/memory.md | 当前子需求执行记忆 | Temporary（清空不删除） |

禁止让多个文档承担相同职责。

## 各文件契约

### PRD.md

- 职责：表示"当前项目最终需要实现什么"。
- 更新：每个子需求完成后（Phase 4）同步最终需求结果。
- 禁止：记录 Step 执行日志、完整测试日志、每个 Step 都修改、复制 execution-plan 内容。
- 读取：Phase 3 默认不读；Phase 4 必读（写）；Phase 1 默认禁止、确需按需；Phase 5 按需。

### @xxx.md / PRD-Vx.md（需求来源文档）

- 任意路径、任意命名的 .md 文件；由用户 @ 指定。
- 职责由"用户当前指定"决定，不按文件名猜测。
- Phase 1 用户指定时必读；Phase 2 / Phase 3 默认禁止重读（除非当前任务确实无法完成）。
- 禁止自动搜索并读取所有 PRD-Vx.md。

### AGENT.md

- 职责：技术栈、编码规范、命名规范、项目约定、测试规则、AI 修改代码必须遵守的规则、开发限制。
- Phase 1 / 2 / 3 必读；Phase 0 已存在时禁止覆盖。

### artifacts/architecture.md

- 职责：只记录长期有效的结构性信息 —— 模块边界、目录结构原则、系统分层、
  技术选型、数据架构、API 架构、Agent / Tool 架构、长期组件、长期设计约束。
- **更新判断（写之前必答）**："这个变化是否会成为未来开发必须遵守的长期架构规则？"
  - NO → 不更新。
  - YES → 更新。
- 需要更新的例子：整体目录结构变化、新增长期模块边界、新增接口架构规范、
  新增统一技术组件、技术栈变化、数据存储架构变化、Agent / Tool 架构变化、新增长期开发规则。
- 不需要更新的例子：新增普通 Class / Controller / Service / DTO、普通业务逻辑修改、
  Bug 修复、普通字段增加、普通 SQL 修改。
- 禁止：记录执行历史、Step 日志、测试日志。
- 读取：默认不读。仅当任务涉及新模块、模块边界、目录结构、技术栈、数据 / API / Agent / Tool
  架构、项目级设计约束时，读取**相关部分**（不为保险完整读取）。

### artifacts/prd-sub.md — 当前子需求

- 临时，只服务当前子需求；完成后清空内容、禁止删除文件。
- 是 AI 梳理后的需求，不是需求文档的复制。

### artifacts/plan.md — 当前子需求整体方案

- 临时，回答"整体准备怎么实现"；完成后清空内容、禁止删除文件。

### artifacts/execution-plan.md — 剩余待执行工作（Future Work）

- 只保存未完成 Step。
- Step 完成并验证成功 → 必须删除该 Step。
- Step 失败 / BLOCKED → 必须保留。
- 所有 Step 完成 → 清空。
- 它不是历史记录；历史在 memory.md。

### artifacts/memory.md — 当前子需求执行记忆

- 临时，记录"对后续 Step、问题排查、回退、理解当前实现有价值的信息"。
- 每完成一个 Step 追加一条高价值记录。
- 子需求完成后：提炼长期价值信息 → 同步到 PRD.md / architecture.md / 项目已有长期文档 → 清空。
- **写入判断**："这条信息未来是否有助于当前子需求继续执行、问题排查、状态恢复或理解重要决策？"

## 子需求完成清理规则（Phase 4）

1. 清空 prd-sub.md / plan.md / execution-plan.md / memory.md 的内容。
2. "清空而不是删除"：文件必须继续存在，固定文件结构保持稳定。
3. 只保留极简状态指针（格式见 file-formats.md），不要把历史写回来。
4. 历史溯源通过三层保证，不依赖临时文件：
   - PRD.md 记录最终需求结果；
   - architecture.md 记录产生的长期架构变化；
   - 用户原始需求文档（长期保留）保留需求来源。

## 禁止的设计

- 巨大的 project-context.md；无限增长的 memory.md。
- memory/ 目录；memory/recent.md、decisions.md、problems.md、history.md 等任何额外记忆文件。
- 每个 Step 自动读取所有文档 / 整个 architecture.md / 整个 memory.md。
- 每个 Step 自动运行完整测试。
- 自动扫描整个项目源码；自动读取所有 PRD-Vx.md。
- 每个 Step 重新分析完整需求；每个 Step 都要求用户确认。
- 用 Token 数字或固定 Context 百分比决定是否继续读取上下文。
- 把所有执行历史永久写入项目；为了"完整性"创建大量辅助文档。
