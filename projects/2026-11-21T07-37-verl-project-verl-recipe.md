# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-11-21 15:37 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的简洁摘要：

### 🛠️ Pull Request (拉取请求)
*   **#176 新增 TPU DeepSWE 配方的 GKE 沙盒预热器**
    *   **作者:** askhat-g
    *   **重要变更:** 为 TPU DeepSWE recipe 引入了 GKE Sandbox Lookahead 预热机制。主要重构是将沙盒提前预热（warm-ahead）的逻辑从原先的 uni-agent GKE provider 中抽离出来，直接下沉集成到 recipe 自身当中。同时，该 PR 还打包发布了 recipe 专属的任务配置（task config）。

### 🐛 Issue (议题)
*   近期暂无重要 Issue 动态。

### 🚀 Release (版本发布)
*   近期无新版本发布。

---

## 🔀 Pull Requests

### #176 — [[recipe] feat: Add GKE Sandbox Lookahead pre-warmer for TPU DeepSWE recipe](https://github.com/verl-project/verl-recipe/pull/176)
- **作者**: askhat-g  **时间**: 2026-11-20 15:50 CST
- **摘要**: ## Summary  Moves the sandbox warm-ahead logic of the `deepswe_tpu/` recipe out of the uni-agent GKE provider and into the recipe, and ships the recipe's own task config.  The uni-agent provider is now a plain one (start, exec, files, stop) with two optional calls, `prewarm()` and `unwarm()`. Decidi…
