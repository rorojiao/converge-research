# converge-research

> 目标收敛式深度调研与优化引擎 — Claude Code Skill

## 简介

`converge-research` 是一个 Claude Code Skill，实现**多轮自收敛的深度调研与优化循环**。

支持两种模式：
- **调研模式**：并行 Agent 搜索 → LLM 合成 → 自评收敛，自动生成深度调研报告
- **优化模式**：原子修改 → 运行指标 → commit/revert，自动寻找最优配置

## 核心特性

- **无状态设计**：所有状态通过文件传递，每次触发都是独立上下文
- **并行 Agent**：每轮启动 2-4 个子 Agent 并行调研不同方向
- **写后搜协议**：防止 Agent 卡死的关键机制
- **策略自进化**：每轮自动评估、淘汰无效方向、聚焦短板
- **多维收敛**：完成度 ≥ 85% 或信息饱和或达最大轮次

## 安装

```bash
git clone https://github.com/rorojiao/converge-research ~/.claude/skills/converge-research
```

## 使用

```
# 启动新调研
/converge-research "你的调研目标"

# 优化模式
/converge-research --optimize "优化目标" --metric "评估命令" --guard "验证命令"

# 配合 /loop 自动推进（每3分钟一轮）
/loop 3m /converge-research:next

# 查看进度
/converge-research:status

# 强制生成报告
/converge-research:report

# 重置会话
/converge-research:reset
```

## 目录结构

```
converge-research/
├── SKILL.md                    # Skill 主文件（Claude 读取的协议）
└── reference/
    ├── goal-decomposer.md      # 目标分解协议
    ├── loop-protocol.md        # 无状态 Ralph Loop 协议
    ├── parallel-agents.md      # 并行 Agent 调度协议
    ├── strategy-evolution.md   # 策略自进化规则
    ├── convergence.md          # 收敛判断算法
    └── output-format.md        # 输出格式模板
```

## 会话文件结构

运行后在工作目录生成：

```
converge-research-session/
├── STATE.md          # 循环状态机
├── GOAL_TREE.md      # 目标分解树 + 完成度
├── KNOWLEDGE.md      # 累积知识库
├── STRATEGY.md       # 当前调研策略
├── EXHAUSTED.md      # 已耗尽方向注册表
├── FINAL_REPORT.md   # 最终报告（收敛后生成）
└── rounds/
    ├── round-1/
    │   ├── plan.md
    │   ├── agent-A.md
    │   ├── agent-B.md
    │   └── eval.md
    └── round-2/
        └── ...
```

## License

MIT
