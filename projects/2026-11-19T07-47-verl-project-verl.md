# verl-project/verl — 动态追踪

> 生成时间: 2026-11-19 15:47 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🛠️ Pull Requests (PR)

**重要新特性与功能增强：**
*   **支持跳过训练阶段 (#8747)：** 允许 V1 PPO 在选定的全局步骤中跳过 actor 和 critic 更新，或跳过整个训练计算。这极大地便利了 rollout 和奖励评估的调试工作。

**关键 Bug 修复：**
*   **修复 Torchtitan 兼容性问题 (#8756)：** 修复了新版 torchtitan（如 TPU fork 版本）中检查点和数据加载模块路径重构导致的导入错误。
*   **修复动态批处理中的 Dense Padding 问题 (#8755)：** 在 Megatron 动态批处理中加入了估算的 dense-token 预算，解决了 BSHD 执行和 VLM 前向传播中仅计算有效 token 而忽略 dense padding 的问题。
*   **修复多轨迹优势估计 Bug (#8754)：** 修复了多轨迹 Agent 训练中的两个优势估计缺陷，确保正确处理 session 级估计器和 `norm_adv_by_std_in_grpo` 设置。

**依赖更新 (Dependabot)：**
*   升级 `transferqueue` 至 0.1.12 (#8752)
*   升级 `transformer-engine` 至 2.21.0 (#8751)
*   放宽并更新 `peft` 依赖至 >=0.21.3 (#8750)
*   放宽并更新 `tensordict` 依赖至 <=0.15.6 (#8749)
*   升级 `spmd-types` 至 0.3.2 (#8748)

**其他：**
*   **Codex/pd 注释更新 (#8753)：** 补充和更新了相关代码注释与文档说明。

### 🐛 Issues
*   近期暂无公开的 Issue 动态。

### 🚀 Release
*   近期暂无新版本发布。

---

## 🔀 Pull Requests

### #8756 — [[worker] fix: torchtitan checkpointer/data loader import compat for refactored torchtitan](https://github.com/verl-project/verl/pull/8756)
- **作者**: jialei777  **时间**: 2026-11-19 15:18 CST
- **摘要**: ### What does this PR do?  Newer torchtitan (e.g. the TPU fork at google-pytorch/torchtitan#201) moved two modules that the TorchTitan engine imports:  - `torchtitan.components.checkpoint.CheckpointManager` → `torchtitan.components.checkpointer.dcp.CheckpointManager` - `torchtitan.components.dataloa…

### #8755 — [[megatron, training_utils] fix: account for dense padding in dynamic batching](https://github.com/verl-project/verl/pull/8755)
- **作者**: qiangyupei  **时间**: 2026-11-19 14:59 CST
- **摘要**: <!-- Suggested title: [megatron, training_utils] fix: account for dense padding in dynamic batching -->  ### What does this PR do?  This PR adds an estimated dense-token budget to Megatron dynamic batching for BSHD execution and VLM forwards that still materialize dense inputs. Counting only valid t…

### #8754 — [fix: honor session-level estimators and norm_adv_by_std_in_grpo in multi-trajectory advantages](https://github.com/verl-project/verl/pull/8754)
- **作者**: tx-queyu  **时间**: 2026-11-19 13:49 CST
- **摘要**: ## Summary  Fixes two related advantage-estimation bugs in multi-trajectory agent training:  1. `compute_advantage_for_multi_trajectories()` only reduced session-final outputs for `GRPO`. For `grpo_vectorized`, `rloo`, `rloo_vectorized`, `grpo_passk` and `gpg`, intermediate turns with zero reward we…

### #8753 — [Codex/pd commentary 20261119](https://github.com/verl-project/verl/pull/8753)
- **作者**: Mengyuyang  **时间**: 2026-11-19 11:26 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #8752 — [build(deps): bump transferqueue from 0.1.11 to 0.1.12](https://github.com/verl-project/verl/pull/8752)
- **作者**: dependabot[bot]  **时间**: 2026-11-19 01:35 CST
- **标签**: dependencies, python
- **摘要**: Bumps [transferqueue](https://github.com/Ascend/TransferQueue) from 0.1.11 to 0.1.12. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/Ascend/TransferQueue/releases">transferqueue's releases</a>.</em></p> <blockquote> <h2>v0.1.12</h2> <h2>Highlight</h2> <h3>…

### #8751 — [build(deps-dev): bump transformer-engine from 2.20.2 to 2.21.0](https://github.com/verl-project/verl/pull/8751)
- **作者**: dependabot[bot]  **时间**: 2026-11-19 01:35 CST
- **标签**: dependencies, python
- **摘要**: Bumps [transformer-engine](https://github.com/NVIDIA/TransformerEngine) from 2.20.2 to 2.21.0. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/NVIDIA/TransformerEngine/releases">transformer-engine's releases</a>.</em></p> <blockquote> <h2>v2.21.0</h2> <h1>T…

### #8750 — [build(deps): update peft requirement from >=0.15.2 to >=0.21.3](https://github.com/verl-project/verl/pull/8750)
- **作者**: dependabot[bot]  **时间**: 2026-11-19 01:34 CST
- **标签**: dependencies, python
- **摘要**: Updates the requirements on [peft](https://github.com/huggingface/peft) to permit the latest version. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/peft/releases">peft's releases</a>.</em></p> <blockquote> <h2>v0.21.3</h2> <p>Release 0.21.3 co…

### #8749 — [build(deps): update tensordict requirement from !=0.9.0,<=0.10.0,>=0.8.0 to >=0.8.0,!=0.9.0,<=0.15.6](https://github.com/verl-project/verl/pull/8749)
- **作者**: dependabot[bot]  **时间**: 2026-11-19 01:34 CST
- **标签**: dependencies, python
- **摘要**: Updates the requirements on [tensordict](https://github.com/pytorch/tensordict) to permit the latest version. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/pytorch/tensordict/releases">tensordict's releases</a>.</em></p> <blockquote> <h2>TensorDict v0.15.…

### #8748 — [build(deps-dev): bump spmd-types from 0.2.3 to 0.3.2](https://github.com/verl-project/verl/pull/8748)
- **作者**: dependabot[bot]  **时间**: 2026-11-19 01:33 CST
- **标签**: dependencies, python
- **摘要**: Bumps [spmd-types](https://github.com/meta-pytorch/spmd_types) from 0.2.3 to 0.3.2. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/meta-pytorch/spmd_types/releases">spmd-types's releases</a>.</em></p> <blockquote> <h2>v0.3.2</h2> <h2>What's Changed</h2> <u…

### #8747 — [[trainer, cfg] feat: skip training stages on selected steps](https://github.com/verl-project/verl/pull/8747)
- **作者**: dengwenjun1  **时间**: 2026-11-19 01:09 CST
- **摘要**: ### What does this PR do?  Allow V1 PPO runs to skip actor and critic updates, or the full training computation, on selected global steps. This makes it possible to debug rollout and reward evaluation while keeping the trainer's normal sampling, synchronization, validation, and checkpoint schedule. …
