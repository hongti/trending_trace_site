# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-07-18 09:03 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的中文简洁摘要：

### 🚀 Pull Request (PR)
*   **PR #121**：新增 `tau2_airline` recipe（作者：yuyu0529nya）
    *   **核心变更**：引入了一个针对**多轮工具调用 Agent** 的强化学习训练方案。
    *   **重要特性**：基于 τ²-bench 的 `airline` 领域实现了 GRPO 算法训练，最大亮点是**支持在单张 GPU 上运行**此多轮双控 Agent 的 RL 训练。

### 🐛 Issue
*   近期无新的 Issue 动态。

### 📦 Release
*   近期无新的 Release 版本发布。

---

## 🔀 Pull Requests

### #121 — [[recipe] feat: add tau2_airline — multi-turn tool-agent RL on τ²-bench](https://github.com/verl-project/verl-recipe/pull/121)
- **作者**: yuyu0529nya  **时间**: 2026-07-17 12:18 CST
- **摘要**: ## What  A recipe for GRPO over a **multi-turn, tool-calling agent** in [τ²-bench](https://github.com/sierra-research/tau2-bench)'s `airline` domain, on a single GPU.  τ²-bench is *dual-control*: the agent and a simulated user both act on a shared, stateful reservations DB. Reward arrives only at th…
