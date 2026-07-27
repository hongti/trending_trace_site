# verl-project/verl — 动态追踪

> 生成时间: 2026-07-27 09:06 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📋 Issue 动态
- **#7152 请求完善 Agentic RL 文档**
  - 作者 `cqray1990` 提出目前缺少关于 Agentic RL（智能体强化学习）的详细文档，希望官方能补充如何训练 Agentic RL 以及如何实现自定义 Agentic RL 的指南。

### 🛠 PR 动态（拉取请求）
近期 PR 主要集中在**新特性扩展**与**核心算法/逻辑修复**两方面：

**✨ 新特性**
- **#7153 集成 OpenAgora 沙盒智能体循环**
  - 新增 `ArenaAgentLoop`（注册名为 `arena_agent`），将 [OpenAgora](https://github.com/albert-lv/OpenAgora) 沙盒环境整合至 verl 的 agent-loop 框架中，进一步增强了 Agentic RL 的环境交互能力（此 PR 正好回应了上述 Issue 的部分需求）。
- **#7149 支持动态 MXFP4 rollout 权重更新**
  - 将 Megatron QAT 的权重同步路径从 NVFP4 扩展至 MXFP4。在每次 rollout 重同步时，可将 BF16 主权重沿输入维度打包为 OCP E2M1 数值格式，提升了低精度量化训练与部署的支持。

**🐛 Bug 修复**
- **#7151 修复稀疏 reward_extra_info 导致 Reward Loop 批次组装崩溃或丢失的问题**
  - 修复了 `RewardLoopManager.compute_rm_score` 中的批次组装缺陷：原先从样本推断 `reward_extra_info` schema 的逻辑在遇到稀疏键时会引发崩溃或数据消失，现已修正。
- **#7150 修复 group-wise advantage 路径中的索引与向量化一致性问题**
  - 修复了 `as_torch_index` 中密集 group id 处理缺陷以及 RLOO 向量化估计器的奇偶校验问题，确保向量化估计器的行为与原本的循环参考实现保持一致，避免算法精度偏差。

### 🚀 Release 动态
- 近期**无新版本发布**记录。

---

## 🐛 Issues

### #7152 — [there is no detailed docs about Agentic RL ](https://github.com/verl-project/verl/issues/7152)
- **作者**: cqray1990  **时间**: 2026-07-26 19:54 CST
- **摘要**: ### Feature request how to train agentic RL， and how to implement a custom agentic rl  ### Motivation    ### Your contribution  。

## 🔀 Pull Requests

### #7153 — [[rollout, doc] feat: add OpenAgora sandbox agent loop integration](https://github.com/verl-project/verl/pull/7153)
- **作者**: albert-lv  **时间**: 2026-07-27 08:13 CST
- **摘要**: ### What does this PR do?  Adds an [OpenAgora](https://github.com/albert-lv/OpenAgora) integration to the agent-loop framework: a new `ArenaAgentLoop` (registered as `arena_agent`) that executes the agent inside OpenAgora-managed sandboxes (Docker) instead of running tools in the trainer process, an…

### #7151 — [[reward] fix: sparse reward_extra_info keys crash or vanish in reward loop batch assembly](https://github.com/verl-project/verl/pull/7151)
- **作者**: keepkeen  **时间**: 2026-07-26 18:47 CST
- **摘要**: ### What does this PR do?  Fixes a batch-assembly defect in `RewardLoopManager.compute_rm_score` (`verl/experimental/reward_loop/reward_loop.py`): the `reward_extra_info` schema was inferred **from sample 0 only** and then every sample was indexed with `info[key]`:  ```python reward_extra_keys = lis…

### #7150 — [[algo] fix: dense group ids in as_torch_index and RLOO vectorized parity](https://github.com/verl-project/verl/pull/7150)
- **作者**: keepkeen  **时间**: 2026-07-26 18:47 CST
- **摘要**: ### What does this PR do?  Fixes two defects in the group-wise advantage path that make the *vectorized* estimators behave differently from the loop-based references they are supposed to replace.  **1. `as_torch_index` does not return the dense ids it promises**  Its docstring states it returns "a c…

### #7149 — [[megatron, vllm] feat: support dynamic MXFP4 rollout weight updates](https://github.com/verl-project/verl/pull/7149)
- **作者**: ISEEKYAN  **时间**: 2026-07-26 12:55 CST
- **摘要**: ### What does this PR do?  This PR extends the Megatron QAT weight-sync path from NVFP4 to MXFP4. At every rollout resync, BF16 master weights are packed along the input dimension into OCP E2M1 values with one E8M0 scale per 32 values, then loaded by vLLM's compressed-tensors dense and MoE paths. Th…
