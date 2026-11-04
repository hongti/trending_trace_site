# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-11-04 14:55 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的中文摘要：

### 🚀 Pull Requests (PR)
*   **PR #168: 为 TPU 新增 Qwen3-30B-A3B ReTool 训练配方**
    *   **作者**: askhat-g | **时间**: 2026-11-03
    *   **核心变更与新特性**：针对 TPU 硬件环境引入了 Qwen3-30B-A3B 模型的完整训练配方。该方案基于最新的 `verl` 主干分支，同时支持**冷启动 SFT（监督微调）**以及**多轮 DAPO 强化学习**训练。

### 🐞 Issues
*   近期暂无公开的 Issue 动态。

### 📦 Releases
*   近期暂无新版本发布。

---

## 🔀 Pull Requests

### #168 — [[hardware,recipe] Add Qwen3-30B-A3B ReTool recipe for TPU](https://github.com/verl-project/verl-recipe/pull/168)
- **作者**: askhat-g  **时间**: 2026-11-04 07:16 CST
- **摘要**: ## Background & Motivation  This PR adds a TPU training recipe for Qwen3-30B-A3B covering both cold-start SFT and multi-turn DAPO reinforcement learning on the latest verl main branch. Because upstream verl replaced SandboxFusionTool with function_tool, we also needed a clean way to support the new …
