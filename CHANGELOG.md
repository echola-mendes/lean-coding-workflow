# Changelog

本文件记录 lean-coding-workflow 协议版本演进。格式参考 [Keep a Changelog](https://keepachangelog.com/)，版本号遵循 [Semantic Versioning](https://semver.org/)。

## [1.1.0] - 2026-11-17

### 新增 (Added)

- Java Project Discovery 与调用级 Dynamic Project Context，项目实际版本优先且禁止普通需求隐式升级
- 可选 Lanhu MCP 需求源，以及 backend-skill 的需求、设计、生成、审查、测试和变更能力编排
- Phase 3 优先使用匹配的项目级实现 Skill；仅在缺失时回退 `proj-gen`，避免职责重叠和规则冲突
- Phase 3 Change Flow：CR ID、局部恢复受影响验收项、保留未受影响完成状态

### 变更 (Changed)

- Java 代码 Step 必须完成 Review、必要测试更新、Lean 实际执行测试和最终验收闭环
- 恢复位置统一为 `execution-plan.md` 中第一个存在非 `[✅]` 验收项的 Step

## [1.0.0] - 2026-09-10

### 新增 (Added)

- 冻结文档驱动工作流协议：Phase 1–4、确认点①②、按需上下文、单 Step 执行
- 临时文件生命周期：Phase 4 归档保留，下一子需求 Phase 1 重建清空
- 验收勾选协议：`[ ]` / `[✅]` / `[❌]` / `[⚠️]`，验证范围与修改范围匹配
- 文档职责隔离：`prd-sub` / `execution-plan` / `architecture` / `memory` / 约定文件
- Skill 仅显式调用（`disable-model-invocation: true`）
- 在 `plugin.json` 与 `SKILL.md` 标明 `version: 1.0.0`
