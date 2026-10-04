# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-10-04 14:13 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的中文摘要：

### 🚀 Pull Request (PR) 动态
* **PR #157**: feat(poise): add POISE recipe (NeurIPS 2026)
  * **作者**: choiyunho1129
  * **新特性**: 引入了来自 NeurIPS 2026 的 **POISE** 算法配方。该方法通过 PCA 和岭回归（ridge regression），利用 Actor 模型的隐藏状态和 Token 熵来估计跨 rollout 基线。此外，该配方还包含了 16 步的 RLOO boot（引导）机制。

### 🐛 Issue 动态
* 近期暂无 Issue 动态。

### 📦 Release 动态
* 近期暂无 Release 发版。

---

## 🔀 Pull Requests

### #157 — [feat(poise): add POISE recipe (NeurIPS 2026)](https://github.com/verl-project/verl-recipe/pull/157)
- **作者**: choiyunho1129  **时间**: 2026-10-03 20:32 CST
- **摘要**: Adds [POISE](https://arxiv.org/abs/2605.07579) (NeurIPS 2026): cross-rollout baselines estimated from actor hidden states and token entropies using PCA and ridge regression. Includes 16-step RLOO bootstrap, Qwen3-4B/OLMo3-7B profiles, probe checkpoint/resume, and tests.  Validation against veRL `871…
