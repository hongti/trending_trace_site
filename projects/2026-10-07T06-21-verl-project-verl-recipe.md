# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-10-07 14:21 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期的动态摘要：

### 🛠️ Pull Request 动态

*   **新特性：添加 ObsSpec 联合训练与复现脚本 (#158)**
    *   引入了 ObsSpec 配方，支持联合训练策略模型与世界模型。
    *   **亮点**：世界模型能够基于相同的 rollout 预测工具的观测结果，从而允许策略在工具执行期间利用预测的观测值继续进行生成。
*   **修复与优化：JSD 蒸馏损失 Beta 值退化警告 (#159, #160)**
    *   两个 PR 均修复了 Issue #151。针对 `jsd` 蒸馏损失模式将 `beta` 钳制到 `(1e-6, 1-1e-6)` 区间的问题进行了优化。
    *   **变更说明**：此前若用户配置 `beta: 1.0` 或 `0.0`，损失会静默降为零且无任何信号反馈，导致模型无法正常训练。现在系统会在检测到此类退化（degenerate）配置时向用户发出警告，避免静默失效。

### 🐛 Issue 动态
*   提供的数据中无近期独立的 Issue 动态（注：上述 PR 中提及的 Issue #151 已通过代码警告修复）。

### 🚀 Release 动态
*   提供的数据中近期无 Release 更新。

---

## 🔀 Pull Requests

### #160 — [Warn on endpoint beta in jsd distill loss instead of silently collapsing (fixes #151)](https://github.com/verl-project/verl-recipe/pull/160)
- **作者**: jasonchen505  **时间**: 2026-10-07 09:24 CST
- **摘要**: Fixes #151.  The `jsd` mode in `gkd/megatron/megatron_distill_losses.py` clamps beta into (1e-6, 1-1e-6), so `beta: 1.0` (or `0.0`) silently yields a near-zero loss with no signal to the user. This implements fix option (a) from the issue: emit a `UserWarning` when beta >= 0.99 or beta <= 0.01 in `j…

### #159 — [[gkd] fix: warn when the jsd distill loss is configured with a degenerate beta](https://github.com/verl-project/verl-recipe/pull/159)
- **作者**: lozlrc  **时间**: 2026-10-07 08:28 CST
- **摘要**: Fixes #151.  ### What does this PR do?  The `jsd` distill loss clamps `beta` into `(1e-6, 1-1e-6)`, and the loss goes to zero as `beta` approaches 0 or 1. So `distill_loss: {name: jsd, beta: 1.0}` gives a near-zero loss instead of the pure KL a user might expect, and nothing says so. This PR covers …

### #158 — [feat(recipe): add ObsSpec co-training and reproduction scripts](https://github.com/verl-project/verl-recipe/pull/158)
- **作者**: kylemontgomery1  **时间**: 2026-10-07 03:50 CST
- **摘要**: ## Background & Motivation  ObsSpec co-trains a policy and a world model that predicts tool observations from the same rollouts. While tools execute, the policy generates ahead using predicted observations to accelerate RL rollouts.  ## Key Changes  Add a new `obsspec/` recipe containing:  - Co-trai…
