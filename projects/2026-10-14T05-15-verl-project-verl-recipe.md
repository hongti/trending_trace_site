# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-10-14 13:15 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的简洁摘要：

### 🚀 Pull Request (PR) 动态
* **#161 [recipe] feat: enable recipes on Intel GPUs (char_count, open_math_reasoning)**
  * **作者**：cheehook | **时间**：2026-10-13
  * **重要变更**：迈出了支持 **Intel GPU** 的第一步。该 PR 的最终目标是实现跨平台兼容，让所有 recipe 都能通过**同一套脚本**在 NVIDIA、AMD 和 Intel GPU 上运行。目前已在 `char_count` 和 `open_math_reasoning` 两个 recipe 中初步应用了该方案。

### 🐛 Issue 动态
* 近期暂无公开的 Issue 动态。

### 📦 Release 动态
* 近期暂无新的版本发布。

---

## 🔀 Pull Requests

### #161 — [[recipe] feat: enable recipes on Intel GPUs (char_count, open_math_reasoning)](https://github.com/verl-project/verl-recipe/pull/161)
- **作者**: cheehook  **时间**: 2026-10-13 17:18 CST
- **摘要**: This is the first step toward running verl-recipe on Intel GPUs. The goal is for every recipe to run on NVIDIA, AMD and Intel GPUs from the same scripts. This PR sets up the approach and applies it to two recipes: char_count, and open_math_reasoning (SFT, generation and eval), so the mechanics can b…
