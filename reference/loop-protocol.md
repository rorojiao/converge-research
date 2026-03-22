# 无状态 Ralph Loop 协议

## 核心原则

每次触发（无论手动或 /loop）都是一个全新上下文。所有状态通过文件传递，绝不依赖对话历史。

## 轮次启动流程

1. 读取 `converge-research-session/STATE.md` → 获取 round 号、mode、status
2. 若 `status: converged` → 输出提示并终止
3. 读取 `STRATEGY.md` → 了解当前策略和方向
4. 读取 `EXHAUSTED.md` → 了解已耗尽方向（避免重复）
5. 读取 `GOAL_TREE.md` → 了解子目标和完成度
6. 若 round > 1 → 读取 `rounds/round-{N-1}/eval.md` → 了解上轮评估
7. 进入 Phase 1

## 文件即记忆

| 文件 | 作用 | 更新时机 |
|------|------|---------|
| STATE.md | 循环状态机 | 每轮结束 |
| STRATEGY.md | 当前调研/优化策略 | Phase 4 |
| EXHAUSTED.md | 已失败方向注册表 | Phase 4 |
| KNOWLEDGE.md | 累积知识库 | Phase 3 |
| GOAL_TREE.md | 目标分解 + 进度 | Phase 4 |

## 防止重复探索

EXHAUSTED.md 格式：
```markdown
## 已耗尽方向

### [方向名]
- 尝试轮次：round-2, round-3
- 失败原因：[具体原因]
- 结论：不再重试

### [方向名]
- 尝试轮次：round-4
- 失败原因：[具体原因]
- 结论：除非有新信息，否则不重试
```

读取 EXHAUSTED.md 后，Phase 1 规划时必须避开这些方向。
