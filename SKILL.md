---
name: converge-research
description: 目标收敛式深度调研与优化引擎。支持两种模式：调研模式（深度搜索+报告生成）和优化模式（代码/配置自主优化循环）。无状态Ralph Loop驱动，每轮自进化策略，多维收敛终止。当用户说"深度调研"、"converge research"、"收敛调研"、"/converge-research"时触发。
---

# ConvergeResearch — 目标收敛式深度调研引擎

## 子命令

| 命令 | 用途 |
|------|------|
| `/converge-research "目标"` | 启动新调研会话 |
| `/converge-research --optimize "目标" --metric "命令" --guard "测试"` | 启动优化会话 |
| `/converge-research:next` | 执行下一轮（配合 /loop 使用） |
| `/converge-research:status` | 查看当前进度 |
| `/converge-research:report` | 强制生成当前阶段报告 |
| `/converge-research:reset` | 清除会话重新开始 |

## 配合 /loop 自动推进

```
/loop 3m /converge-research:next
```

## 核心协议

### 入口路由

1. 解析用户输入，判断模式：
   - 含 `--optimize` → 优化模式
   - 否则 → 调研模式
2. 检查工作目录下 `converge-research-session/STATE.md` 是否存在：
   - **不存在** → 新会话：创建目录 → Phase 0（目标分解）→ Phase 1-5（首轮）
   - **存在** → 读取 STATE.md：
     - `status: converged` → 告知已完成，询问是否 reset
     - `status: running` → 执行 Phase 1-5（下一轮）

### 新会话初始化

创建 `converge-research-session/` 目录及所有子文件。读取 `reference/goal-decomposer.md` 执行目标分解。

### 每轮执行（Phase 1-5）

**严格按顺序执行，每个 Phase 完成后写入对应文件再进入下一个。**

**Phase 1 — 自适应规划**
读取 reference/loop-protocol.md。读取 STRATEGY.md + EXHAUSTED.md + 上轮 eval.md → 生成 `rounds/round-N/plan.md`。

**Phase 2 — 并行探索**
读取 reference/parallel-agents.md。
- 调研模式：启动 2-4 个 Agent tool 并行调研，遵循写后搜协议
- 优化模式：生成原子修改 → 运行 metric → 运行 guard → commit/revert

**Phase 3 — 合成验证**
合并所有 Agent 输出 → 交叉验证 → 更新 KNOWLEDGE.md（调研）或 RESULTS.tsv（优化）

**Phase 4 — 自评与进化**
读取 reference/strategy-evolution.md。对每个子目标打分（**严格遵守校准规则：首轮≤60%、每轮增量≤30%、必须列出未回答问题**） → 计算加权总完成度 → 评估策略有效性 → 更新 STRATEGY.md → 无效方向移入 EXHAUSTED.md

**Phase 5 — 收敛判断**
读取 reference/convergence.md。**前 3 轮（min_rounds=3）跳过收敛检查，直接进入下一轮。** 第 3 轮起检查终止条件：
- 加权完成度 ≥ 85%
- 连续 2 轮信息增量 < 10%
- 达到最大轮次
- 优化模式：连续 3 轮无改进

收敛 → 生成 FINAL_REPORT.md（读取 reference/output-format.md）→ STATE.md status=converged
未收敛 → 更新 STATE.md round++ → 等待下次触发

### :next 子命令

读取 STATE.md → 若 status=converged 则输出"已收敛，使用 :reset 重新开始" → 否则执行 Phase 1-5

### :status 子命令

读取并展示 STATE.md 中的进度表 + 收敛信号

### :report 子命令

基于当前 KNOWLEDGE.md 强制生成报告，不等待收敛

### :reset 子命令

确认后删除 converge-research-session/ 目录
