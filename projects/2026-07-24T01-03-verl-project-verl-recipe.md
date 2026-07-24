# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-07-24 09:03 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 最近动态的中文简洁摘要：

### 🚀 Pull Request (PR)
近期共2项合并/提交，主要聚焦于**异步强化学习架构优化**与**模型支持扩展**：

*   **PR #123** `[recipe] feat(async_flow): add FlexFetch CheckpointEngine` (作者: hongti)
    *   **重要新特性**：为 `async_flow` recipe 引入了 **FlexFetch CheckpointEngine**。这是一项关键变更，提供了一种按需、基于拉取的权重同步路径，为全异步强化学习提供了底层抽象和支持，有望显著提升异步训练的效率与灵活性。
*   **PR #124** `Verl tinker teacher model` (作者: wyettzeng)
    *   **新特性**：在 verl Tinker 中增加了对**教师模型**的支持，扩展了框架在模型蒸馏或引导训练方面的能力。

### 🐛 Issue
*   近期暂无公开的 Issue 动态。

### 📦 Release
*   近期暂无新版本发布动态。

---

## 🔀 Pull Requests

### #124 — [Verl tinker teacher model](https://github.com/verl-project/verl-recipe/pull/124)
- **作者**: wyettzeng  **时间**: 2026-07-24 05:11 CST
- **摘要**: Add in support for teacher model into verl Tinker

### #123 — [[recipe] feat(async_flow): add FlexFetch CheckpointEngine for asynchronous weight sync](https://github.com/verl-project/verl-recipe/pull/123)
- **作者**: hongti  **时间**: 2026-07-23 14:23 CST
- **摘要**: ## Summary  This PR introduces the **FlexFetch CheckpointEngine** to the `async_flow` recipe — an on-demand, pull-based weight synchronization path for fully asynchronous RL. The CheckpointEngine abstraction is split into two backends: the existing synchronous **HCCL** path is preserved, and a new *…
