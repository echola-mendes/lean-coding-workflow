# Changelog

本文件记录 lean-coding-workflow 协议版本演进。格式参考 [Keep a Changelog](https://keepachangelog.com/)，版本号遵循 [Semantic Versioning](https://semver.org/)。

## [1.0.0] - 2026-09-10

### 新增 (Added)

- 冻结文档驱动工作流协议：Phase 1–4、确认点①②、按需上下文、单 Step 执行
- 临时文件生命周期：Phase 4 归档保留，下一子需求 Phase 1 重建清空
- 验收勾选协议：`[ ]` / `[✅]` / `[❌]` / `[⚠️]`，验证范围与修改范围匹配
- 文档职责隔离：`prd-sub` / `execution-plan` / `architecture` / `memory` / 约定文件
- Skill 仅显式调用（`disable-model-invocation: true`）
- 在 `plugin.json` 与 `SKILL.md` 标明 `version: 1.0.0`
