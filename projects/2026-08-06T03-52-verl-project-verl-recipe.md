# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-08-06 11:52 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的中文摘要：

### 📥 Issue 动态
*   **#130 [RFC] Enc-LLM 解耦多模态 RL 训练** (作者: nekoteai | 2026-08-05)
    *   **核心提议**：建议在仓库中新增一个可选的 `enc_disaggregate` 训练方案。
    *   **重要变更/新特性**：提出一种全新的架构设计，旨在将多模态编码器与语言模型（LLM）进行物理/计算上的解耦分离。此举有望优化多模态强化学习（RL）的训练效率与资源分配。

### 🔀 PR 动态
*   近期无显著 PR 动态。

### 🚀 Release 动态
*   近期无新版本发布。

---

## 🐛 Issues

### #130 — [[RFC] Enc-LLM Disaggregated Multimodal RL Training](https://github.com/verl-project/verl-recipe/issues/130)
- **作者**: nekoteai  **时间**: 2026-08-05 21:31 CST
- **摘要**: ## 1. Summary  This issue proposes adding an opt-in `enc_disaggregate` recipe to `verl-project/verl-recipe`. The recipe defines a complete architecture for separating multimodal encoders from the language model during PPO/GRPO execution, so vision, video, audio, and LLM stages can use independent wo…
