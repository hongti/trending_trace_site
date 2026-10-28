# verl-project/verl — 动态追踪

> 生成时间: 2026-10-28 14:08 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🐛 Issue
*   **[Bug] GMPO 梯度计算错误 (#8406)**
    报告了 `geo_mean` (GMPO) 忽略 IcePop 零 IS 权重的问题。此 Bug 会导致被拒绝的 token 仍然保留梯度，并且序列的其余部分被错误地以 `exp(log(1e-10)/L)` 进行下调缩放，影响训练正确性。

### 🔀 Pull Request
本次合入的 PR 涵盖了新特性、硬件生态支持、训练示例及重要 Bug 修复：

**重要新特性与架构升级**
*   **NeMo AutoModel 引擎支持 (#8402, 草案):** 引入完全异步的 PPO 训练器，支持将训练引擎切换为 NeMo AutoModel，提升了异步 RL 训练的灵活性。
*   **Rollout 运行时隔离 (#8407):** 新增可选的 `actor_rollout_ref.rollout.runtime` 描述符，允许在预配置的 Python 解释器中启动独立的 vLLM HTTP actors，实现了训练环境与推理环境的彻底解耦。
*   **Intel GPU (XPU) 支持 (#8401):** 新增多阶段 Docker 镜像，支持在 Intel Arc Pro B 系列 GPU 上运行 vLLM 0.32.0 推理及 FSDP/FSDP2 训练。
*   **华为昇腾 950 环境升级 (#8404):** 将 Ascend 950 环境升级至 Megatron-Core 0.18，并新增了 Qwen3-30B-A3B 的 MXFP8 GRPO 训练示例。

**新增训练示例与配方**
*   **DeepSeek-V4.1 超长上下文 LoRA RL (#8403):** 基于 Megatron Lite + vLLM 新增示例，支持在 8 张 GPU 上进行 26.2 万 token 上下文的训练，且保证训练与推理的 log-probs 达到位级一致。
*   **单卡 24GB GRPO + LoRA 配方 (#8405):** 针对 Qwen2.5-Math-1.5B-Instruct 新增了单卡（如 RTX 3090/4090）即可跑通的 GRPO + LoRA 示例，大幅降低了尝试门槛。

**关键 Bug 修复**
*   **Profiler 配置修复 (#8409):** 修复了全局 torch profiler 的 `contents` 和 `.discrete` 参数被静默忽略的问题，确保其能正确传递给 Actor、Critic 和 Ref 角色。
*   **Position IDs 构建修复 (#8408):** 修复了 `left_right_2_no_padding()` 在处理 mRoPE 序列时未正确指定 token 轴作为 jagged 维度的问题，确保未填充的 `position_ids` 构建正确。

### 🚀 Release
*   近期无新版本发布。

---

## 🐛 Issues

### #8406 — [[Bug] geo_mean (GMPO) ignores IcePop's zero IS weights: the rejected token keeps its gradient and the rest of the sequence is down-scaled by exp(log(1e-10)/L)](https://github.com/verl-project/verl/issues/8406)
- **作者**: yusufozdil  **时间**: 2026-10-28 05:37 CST
- **标签**: bug
- **摘要**: ### System Info   ```text ----------Python Info---------- Version      : 3.12.3 ----------Pip Info----------- vllm         : not found. sglang       : not found. ray          : 2.59.0 torch        : 2.14.1 ----------verl Info----------- Version      : 0.10.0.dev Commit Hash  : 4d3ed12a7617c6b78ee0ac…

## 🔀 Pull Requests

### #8409 — [[perf, trainer] fix: forward global torch profiler contents/discrete to role tool configs](https://github.com/verl-project/verl/pull/8409)
- **作者**: arvyanh  **时间**: 2026-10-28 12:56 CST
- **摘要**: ### What does this PR do?  `global_profiler.global_tool_config.torch.contents` and `.discrete` were silently ignored. The actor, critic, and ref role configs hardcoded `tool_config.torch.contents: []` and `discrete: false`, instead of interpolating from the global config the way `nsys.discrete` (`ac…

### #8408 — [[trainer] fix: construct unpadded position IDs with explicit sequence axis](https://github.com/verl-project/verl/pull/8408)
- **作者**: yue-zeng-yue  **时间**: 2026-10-28 11:25 CST
- **摘要**: ### What does this PR do?  Fixes the `left_right_2_no_padding()` producer so that unpadded `position_ids` always use the token axis as their jagged dimension.  For an equal-length batch of eight mRoPE samples shaped `(4, 575)`, `as_nested_tensor(..., layout=torch.jagged)` packs the component axis. T…

### #8407 — [[rollout, vllm, ray] feat: isolate standalone rollout Python runtime](https://github.com/verl-project/verl/pull/8407)
- **作者**: Levius-Fubuki  **时间**: 2026-10-28 09:23 CST
- **摘要**: ### What does this PR do?  Add an opt-in `actor_rollout_ref.rollout.runtime` descriptor for a pre-provisioned Python interpreter. A training-only environment can launch standalone vLLM HTTP actors, backend children, and rollout checkpoint workers in a separate environment without importing vLLM on t…

### #8405 — [examples: add single-GPU 24GB GRPO + LoRA recipe (Qwen2.5-1.5B)](https://github.com/verl-project/verl/pull/8405)
- **作者**: Rich-Young98  **时间**: 2026-10-28 00:18 CST
- **摘要**: ## What  Add a single-GPU 24GB GRPO + LoRA example for Qwen2.5-Math-1.5B-Instruct.  Official GRPO recipes in `examples/grpo_trainer/` assume multi-GPU nodes (8x80G class). Users with one 24 GB card (RTX 4090 / L4 / A10) hit OOM in rollout or actor update and have to guess LoRA rank, vLLM memory frac…

### #8404 — [[hardware, megatron, doc] feat: upgrade Ascend 950 to Megatron-Core 0.18 and add MXFP8 GRPO example](https://github.com/verl-project/verl/pull/8404)
- **作者**: Rain6ow  **时间**: 2026-10-27 22:13 CST
- **摘要**: ### What does this PR do?  This PR updates the Huawei Ascend 950 environment to Megatron-Core 0.18 and adds a Qwen3-30B-A3B MXFP8 GRPO training example.  Upgrade both Ascend 950 CANN 9.1.0 Dockerfiles from Megatron-Core 0.12 to 0.18 and update MindSpeed to core_r0.18.0.  Add MegatronAdaptor, Transfo…

### #8403 — [[recipe] feat: DeepSeek-V4.1 LoRA with bitwise train/inference log-probs and 262K context on 8 GPUs](https://github.com/verl-project/verl/pull/8403)
- **作者**: ISEEKYAN  **时间**: 2026-10-27 21:36 CST
- **摘要**: ### What does this PR do?  Adds an example for DeepSeek-V4.1-Flash LoRA RL on verl + Megatron Lite (mLite) + vLLM with **training/inference bitwise-identical log-probs**, including **262,144-token context on 8 GPUs** (2 nodes x 4 GB200).  - `examples/grpo_trainer/run_deepseek_v41_lora_bitwise_megatr…

### #8402 — [(Draft) feat(automodel): NeMo AutoModel engine support for fully-async RL](https://github.com/verl-project/verl/pull/8402)
- **作者**: zpltys  **时间**: 2026-10-27 17:10 CST
- **摘要**: Enable the fully-async PPO trainer with the training engine swapped to NeMo AutoModel (nemo_automodel):  - Actor/critic/ref: AutomodelActorConfig / AutomodelCriticConfig dataclasses plus Hydra config groups (actor/critic/ref/model_engine automodel yaml) so actor.strategy=automodel resolves in the fu…

### #8401 — [[hardware, docs] feat: add multi-stage Intel GPU (XPU) Docker image for vLLM 0.32.0](https://github.com/verl-project/verl/pull/8401)
- **作者**: kahlun  **时间**: 2026-10-27 15:57 CST
- **摘要**: ### What does this PR do?  Add `docker/xpu/Dockerfile.stable.vllm`: a multi-stage Docker image for running verl on Intel GPUs (Arc Pro B-series, XPU), with vLLM 0.32.0+xpu rollout and FSDP/FSDP2 training on torch 2.14.0+xpu. XPU platform support comes from [verl-hardware-plugin](https://github.com/v…
