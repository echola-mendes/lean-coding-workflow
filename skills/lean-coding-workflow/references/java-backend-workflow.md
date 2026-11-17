# Java Backend Workflow

## 外部能力边界

- 只使用当前 Agent / 项目已安装且可发现的 Lanhu MCP 与 `proj-analyze-req`、`proj-analyze-design`、
  `proj-gen`、`proj-review`、`proj-gen-test`、`proj-change`；禁止运行时下载、复制或内嵌外部仓库。
- Lanhu 只提供 Requirement Facts，不生成技术方案、不控制 Phase；backend-skill 不拥有确认点、进度或
  项目技术栈决定权，禁止调用会产生第二套状态的 `proj-task` / `proj-resume`。
- 外部能力缺失时由 Lean 完成等价工作并在当前回复说明，不中断原流程。
- 调用时提供当前 Phase / Step、目标、需求事实、Dynamic Project Context、修改边界与验收标准；
  外部输出不能替代 Lean 验收。

## Project Discovery

调用 Java 专业能力前，按当前 Step 形成 Dynamic Project Context，不写文件、不跨项目复用。优先级：

```text
实际项目配置 / 现有代码 > AGENTS.md / architecture.md > Lean Workflow > backend-skill 默认环境
```

1. 从当前模块 `pom.xml` 或 Gradle 构建文件识别 Java、Spring Boot、ORM、数据库驱动、Redis、API 文档、
   测试框架及版本；按需读取直接版本来源。
2. 按需读取 `application.yml`、`application-*.yml`、相关源码与测试、约定文件和 architecture；禁止扫描全项目。
3. 每个结论附来源文件；无法确认标 `unknown`，不得使用外部默认值或历史聊天值。
4. 恢复时从 `execution-plan.md` 找到当前 Step，再重新发现所需事实。

最小调用结构（只填当前 Step 所需字段）：

```yaml
project_context:
  java: { value: <version-or-unknown>, source: <project-file> }
  spring_boot: { value: <version-or-unknown>, source: <project-file> }
  orm: { value: <name-and-version-or-unknown>, source: <project-file> }
  database: { value: <name-and-version-or-unknown>, source: <project-file> }
  redis: { value: <version-or-unknown>, source: <project-file> }
  api_doc: { value: <name-and-version-or-unknown>, source: <project-file> }
  test: { value: <framework-and-version-or-unknown>, source: <project-file> }
```

普通需求禁止隐式升级 Java、Spring Boot、Redis、核心依赖或测试框架。确需升级时，先作为独立技术 / 架构
变更说明影响、迁移和验证方案，确认后执行。

## Phase 规则

### Phase 1

- Lanhu URL：调用可用的 Lanhu MCP 获取原型、标注和交互；不可用时使用其他来源并说明。
  URL 是唯一来源且无法读取时，将缺失信息列为待确认并停在确认①。
- Java 项目：Project Discovery 后调用可用的 `proj-analyze-req`。

### Phase 2

Project Discovery 后调用可用的 `proj-analyze-design`，辅助分析 API、DB、Java、Spring、Redis、MQ、事务、
权限、兼容性和现有架构；所有设计服从当前项目事实。

### Phase 3

Java 代码实现前仅检查项目 `.agents/skills/*/SKILL.md` 的 frontmatter；匹配的实现型 Skill 替代 `proj-gen`，
否则依次回退 `proj-gen`、Lean。禁止加载无关 Skill 或同时调用两个实现型 Skill。随后执行 `proj-review`、
必要修复和 `proj-gen-test`，由 Lean 按 Maven / Gradle 测试并验收；项目 Skill 继续约束后续环节，其自检不替代闭环。

## Change Flow

Phase 3 需求变更时：

1. 分配下一个 `CR-001`、`CR-002`……，形成受影响范围的 Dynamic Project Context。
2. 调用可用的 `proj-change` 获取影响分析和变更计划；不可用时由 Lean 完成。
3. 在原 `execution-plan.md` Change History 记录变更、影响、计划和受影响 Step；仅将受影响 `[✅]` 改回
   `[ ]` 并标注 CR ID。保留无关 `[✅]`；按影响调整原 Step，只在必要时追加 Change Step。
4. 从第一个存在非 `[✅]` 验收项的 Step 继续 Phase 3 闭环；代码实现沿用 Phase 3 的项目级 Skill 优先规则。

CR ID 只用于追溯。禁止新增 TODO / IN_PROGRESS / PAUSED / DONE / SUPERSEDED、checkpoint、change-plan 或
其他状态文件；`execution-plan.md` 始终是唯一进度来源，不增加固定确认点。
