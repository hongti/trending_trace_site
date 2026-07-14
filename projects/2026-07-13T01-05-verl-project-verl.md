# verl-project/verl — 动态追踪

> 生成时间: 2026-07-13 09:05 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 🔀 Pull Requests (PR)
1. **新增 UP-GRPO 算法策略损失** (#7022，作者: a-F1)
   - **重要变更**：引入了 UP-GRPO（Unbounded Positive Asymmetric Optimization，无界正不对称优化，基于 arXiv:2607.06987）作为一种全新的、即插即用的策略损失模式（模式名为 `up`）。这为算法模块扩展了新的优化选项。
2. **优化 TransferQueue Agent 批次处理机制** (#7021，作者: luzy99)
   - **重要变更**：在 `AgentLoopWorkerTQ` 中为 TransferQueue 的 agent-loop 批次提交引入了排队机制。此改动使得每个 worker 能够串行处理排队的批次块，同时在内部保留了 prompt 级别的并发性，提升了处理逻辑的严谨性与效率。

### 🐛 Issues
- 本次统计周期内无新增 Issue 动态。

### 🚀 Releases
- 本次统计周期内无新版本发布动态。

---

## 🔀 Pull Requests

### #7022 — [[algo, doc] feat: add UP-GRPO unbounded positive asymmetric policy loss](https://github.com/verl-project/verl/pull/7022)
- **作者**: a-F1  **时间**: 2026-07-13 08:53 CST
- **摘要**: ### What does this PR do?  Adds **UP-GRPO** (Unbounded Positive Asymmetric Optimization, [arXiv:2607.06987](https://arxiv.org/pdf/2607.06987)) as a new, plug-and-play policy-loss mode `up`.  **Motivation.** Standard GRPO/PPO combines a symmetric clip (single `ε`) with importance sampling against `π_…

### #7021 — [[trainer] feat: queue TransferQueue agent batches](https://github.com/verl-project/verl/pull/7021)
- **作者**: luzy99  **时间**: 2026-07-12 17:34 CST
- **摘要**: ### What does this PR do?  Queues TransferQueue agent-loop batch submissions in `AgentLoopWorkerTQ` so each worker processes queued batch chunks serially while preserving prompt-level concurrency within a batch.  Previously, repeated `generate_sequences()` calls spawned fully fire-and-forget prompt …
