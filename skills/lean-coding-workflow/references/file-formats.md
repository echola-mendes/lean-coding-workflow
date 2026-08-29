# File Formats — Workflow 文件格式模板

读取时机：Phase 1 生成 prd-sub.md、Phase 2 生成 plan.md / execution-plan.md、
Phase 3 写 memory.md、Phase 4 清空临时文档之前。

## artifacts/prd-sub.md（Phase 1 生成）

```markdown
# 当前子需求

## 1. 目标
## 2. 背景
## 3. 功能范围
## 4. 非目标
## 5. 业务规则
## 6. 输入与输出
## 7. 涉及模块
## 8. 验收标准
## 9. 待确认问题
```

要求：

- 是 AI 梳理后的**可执行子需求**，不是用户需求文档的简单复制。
- 不添加用户没有要求且没有依据的业务规则。
- 信息不足时明确列入"待确认问题"，不要自行臆测补全。

## artifacts/plan.md（Phase 2 生成）

```markdown
# Plan

## 1. 技术方案
## 2. 模块设计
## 3. 数据设计
## 4. API 设计
## 5. 核心流程
## 6. 依赖
## 7. 风险
## 8. 验证策略
```

回答"这个子需求整体准备怎么实现？"——只记**跨 Step 的方案决策与理由**：

- 不复述 Step 级实现细节（那属于 execution-plan.md 的每个 Step）。
- 不复制需求原文（那属于 prd-sub.md）。
- 保持短（建议 ≤100 行）：每节只写决策与理由，不写教程式展开。

## artifacts/execution-plan.md（Phase 2 生成）

```markdown
# Execution Plan

## Step 1：<标题>

### 目标
<这个 Step 要达成什么>

### 涉及文件 / 模块
<明确列出>

### 实现方式
<怎么做，写到足够执行即可>

### 验证方式
<最小必要验证，与修改范围匹配>

### 完成标准
<怎样才算完成>

### 依赖
<仅存在依赖时保留此节>

---

## Step 2：<标题>
（同上结构）
```

要求：

- 每个 Step 的目标、范围、涉及文件、实现方式、验证方式、完成标准必须明确。
- Step 独立可执行、可验证、可恢复，不依赖完整历史上下文。
- 不过度细碎：Step 大小以"能在较小上下文中可靠完成"为准。
- **只保存未完成 Step**：完成并验证成功即删除；失败 / BLOCKED 必须保留。
- 它是 Future Work，不是历史记录，禁止保留 DONE / 历史 Step。

## artifacts/memory.md（Phase 3 每个 Step 后追加）

Step 成功条目：

```markdown
## Step N

状态：DONE

完成：
+ <做了什么>

修改：
+ <文件列表>

验证：
+ <验证方式与结果，如 xxx test PASS>

决策：
+ <重要技术决策，如：复用现有 xxx 机制>

备注：
+ <后续 Step 需要知道的信息>
```

Step 失败条目：

```markdown
## Step N

状态：FAILED

问题：
+ <现象>

原因：
+ <原因分析>

尝试：
+ <已尝试的修复>

下一步：
+ <打算怎么做>
```

写入前必须判断："这条信息未来是否有助于当前子需求继续执行、问题排查、状态恢复或理解重要决策？"
NO 不记录，YES 才记录。

禁止记录：Agent 思维过程、完整源码、完整测试输出、所有命令、无意义操作、重复信息。

Memory = 短、精确、高价值、可恢复。

## 子需求完成后的清空格式（Phase 4）

四个临时文件（prd-sub.md / plan.md / execution-plan.md / memory.md）清空为极简状态：

```markdown
# Current Workflow State

状态：IDLE

上一个子需求：<名称>

完成时间：<日期>

结果：已完成并同步至 PRD.md

架构：architecture.md 已更新 / 无架构变化
```

要求：

- **清空而不是删除**：文件必须继续存在，固定结构保持稳定。
- 状态信息必须非常短，禁止把整个历史重新写回来。
