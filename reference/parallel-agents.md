# 并行 Agent 调度协议

## 调研模式：并行搜索 Agent

### 启动

根据 plan.md 中的调研流分配，使用 Agent tool 启动 2-4 个子代理。每个 Agent 的 prompt 包含：
1. 调研流的具体任务和要回答的问题
2. 输出文件路径：`rounds/round-N/agent-X.md`
3. 必须遵循的写后搜协议（见下）

### 写后搜协议（Write-After-Search Protocol）

**这是防止 Agent 卡死的关键协议。**

每个 Agent 必须严格交替执行：
```
搜索/获取信息 → 将发现写入文件 → 搜索/获取信息 → 将发现写入文件 → ...
```

禁止连续两次搜索不写入。每次写入必须包含：
- 信息来源 URL
- 关键发现摘要
- 与调研问题的关联

### 卡住检测与恢复

启动 Agent 后，主流程用 TaskOutput 定期检查：
- 30秒后第一次检查
- 之后每 2 分钟检查一次
- 检查方法：读取 agent-X.md 文件行数
- 若连续两次检查行数不变 → Agent 卡住 → TaskStop → 重新启动（prompt 中注入已有数据）

### Agent 输出格式

```markdown
# Agent-X: [调研流名称]
Status: IN_PROGRESS | COMPLETE

## 发现 1: [标题]
**来源**: [URL]
**摘要**: ...
**关键信息**: ...

## 发现 2: [标题]
...
```

Agent 完成所有搜索后将 Status 改为 COMPLETE。

## 优化模式：单线程原子实验

优化模式不使用并行 Agent，而是顺序执行：
1. 基于 STRATEGY.md 选择一个优化方向
2. 生成最小原子修改（一个明确的代码/配置变更）
3. `git add -A && git commit -m "experiment: [描述]"`
4. 运行 metric 命令 → 记录结果
5. 运行 guard 命令 → 检查是否通过
6. 结果判断：
   - 指标改进 + guard 通过 → 保留提交，记录成功
   - 指标改进 + guard 失败 → 尝试 rework（最多2次）→ 仍失败则 `git revert HEAD`
   - 指标无改进 → `git revert HEAD`，记录失败原因
