# verl-project/verl — 动态追踪

> 生成时间: 2026-08-04 09:03 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🚀 Release 动态
近期无新版本发布。

---

### 📝 Issue 动态
本期有两个重要的算法与架构设计讨论（RFC/RPC）：
*   **多轮 RL 的每轮奖励支持 (#7244)**：提出为多轮强化学习引入无偏的 turn-level baseline。当前 verl 将整条轨迹的奖励作为一个标量计算梯度，这成为了工具调用和搜索智能体信用分配的瓶颈，该 RFC 旨在解决此问题。
*   **全词表多教师在线策略蒸馏 (#7239)**：讨论 verl OPD（在线策略蒸馏）损失的改进，涉及 `forward_kl_topk` 等现有实现，探索全词表多教师蒸馏方案。

---

### 🔀 PR 动态
近期 PR 主要集中在**算法目标函数扩展、新模型/硬件适配及关键 Bug 修复**：

**1. 算法与训练特性（核心亮点）**
*   **新增标准 PPO 选项 (#7246)**：允许设置 `clip_ratio_c=null` 以使用不带双重裁剪的标准 PPO，历史默认值 `3.0` 保持不变，兼容原有配置。
*   **新增重要性采样与 DRO 损失 (#7245)**：增加两种新的策略目标函数——无裁剪的 token 级重要性采样，以及带可配置二次对数比惩罚的直接奖励优化（DRO）。
*   **新增无偏 pass@k 验证指标 (#7240)**：引入 Codex/HumanEval 风格的无偏 pass@k 评估指标，与现有的 mean@k/best@k 互补，提供更准确的验证标准。

**2. 模型与硬件支持**
*   **支持 DeepSeek V4 (#7242)**：VeOmni 引入对 DeepSeek V4 模型的训练支持，并验证了 rollout 概率差的有效性。
*   **ROCm (AMD 显卡) 兼容性修复 (#7241)**：为 DeepSeek 稀疏注意力 (DSA) 提供了纯 PyTorch 的 `fast_hadamard_transform` 回退方案，解决 AMD ROCm 环境下的硬依赖报错。
*   **多模态 RoPE 扩展 (#7236)**：为多模态旋转位置编码增加了处理器钩子。

**3. 关键 Bug 修复**
*   **修复 PPO 融合算子梯度丢失 (#7235)**：修复了 `FusedLinearForPPO` 反向传播中静默丢弃梯度的严重 Bug，现根据 `ctx.needs_input_grad` 正确控制梯度分配。
*   **修复 GPT-OSS 权重同步 (#7243)**：修复了在未开启专家并行时，GPT-OSS 融合检查点张量全权重同步失败的问题。

**4. 重构与文档**
*   **PPO Trainer 重构 (#7237)**：将共享的 PPO 辅助函数从已弃用的 `RayPPOTrainer` 移至独立的 `verl.trainer.ppo.trainer_utils` 模块，提升代码复用性。
*   **NPU 环境文档更新 (#7238)**：更新了 vllm/vllm-ascend、torch 及 transformer 版本说明（针对 NPU 昇腾安装文档和 Dockerfile）。

---

## 🐛 Issues

### #7244 — [[RFC] Per-turn reward support for multi-turn RL: a turn-level baseline that stays unbiased under variable turn counts](https://github.com/verl-project/verl/issues/7244)
- **作者**: lijuliana  **时间**: 2026-08-04 05:49 CST
- **摘要**: ### Summary  verl has no per-turn advantage estimator today: rewards reach the gradient as one scalar per trajectory. For tool-use and search agents this is the credit-assignment bottleneck, and past requests for it (#2540, #3683) were closed without an answer. We implemented per-turn rewards on a f…

### #7239 — [[RPC] Full-Vocabulary Multi-Teacher On-Policy Distillation](https://github.com/verl-project/verl/issues/7239)
- **作者**: Moocharr  **时间**: 2026-08-03 21:13 CST
- **摘要**: ### 1. Motivation  verl's OPD losses split into two families :  - `forward_kl_topk` (`use_topk=True`): approximates forward KL using the teacher's top-k log-probs (vLLM `prompt_logprobs=k`). Truncated on large vocabs → tail-mass bias, and needs `clamp_min(0)` since top-k isn't normalized.  - Estimat…

## 🔀 Pull Requests

### #7246 — [[algo, cfg] feat: allow standard PPO without dual clipping](https://github.com/verl-project/verl/pull/7246)
- **作者**: wyettzeng  **时间**: 2026-08-04 08:35 CST
- **摘要**: ### What does this PR do?  Allows `clip_ratio_c=null` to select standard PPO without VERL's additional dual-clipping branch. The historical default remains `3.0`, so existing configurations retain dual-clip PPO behavior.  This is the third part of the former mixed #7197 scope. It is intentionally in…

### #7245 — [[algo, cfg, doc] feat: add importance-sampling and DRO losses](https://github.com/verl-project/verl/pull/7245)
- **作者**: wyettzeng  **时间**: 2026-08-04 08:35 CST
- **摘要**: ### What does this PR do?  Adds two registered policy objectives: unclipped token-level importance sampling and Direct Reward Optimization (DRO) with a configurable quadratic log-ratio penalty.  This PR is intentionally independent of #7197 so the token-sum aggregation and policy-loss changes can be…

### #7243 — [[veomni] fix: preserve GPT-OSS weights without expert parallelism](https://github.com/verl-project/verl/pull/7243)
- **作者**: Luosuu  **时间**: 2026-08-04 04:48 CST
- **摘要**: ## What does this PR do?  Fix VeOmni full-weight synchronization for GPT-OSS when expert parallelism is disabled.  GPT-OSS stores all experts in fused checkpoint tensors, with gate and up values interleaved along the last dimension. The generic VeOmni MoE exporter treated those tensors like Qwen-sty…

### #7242 — [[veomni] feat: add DeepSeek V4 support](https://github.com/verl-project/verl/pull/7242)
- **作者**: wuxibin89  **时间**: 2026-08-04 01:20 CST
- **摘要**: ### What does this PR do?  VeOmni counterpart PR https://github.com/ByteDance-Seed/VeOmni/pull/962  ``` training/rollout_probs_diff_valid:1.0 training/rollout_probs_diff_max:0.6111682653427124 training/rollout_probs_diff_mean:0.014773058705031872 training/rollout_probs_diff_std:0.029917456209659576 …

### #7241 — [[megatron, hardware] pure-torch fast_hadamard_transform fallback for DSA on ROCm](https://github.com/verl-project/verl/pull/7241)
- **作者**: PeterYang12  **时间**: 2026-08-03 23:05 CST
- **摘要**: DeepSeek sparse-attention (DSA) in Megatron-LM binds `from fast_hadamard_transform import hadamard_transform` at import time and asserts it is not None inside the indexer's `rotate_activation`. The Dao-AILab package only ships nvcc kernels and its setup.py hard-requires nvcc, so it cannot be built o…

### #7240 — [[trainer] feat: add unbiased pass@k validation metric](https://github.com/verl-project/verl/pull/7240)
- **作者**: emmericp  **时间**: 2026-08-03 22:24 CST
- **摘要**: ### What does this PR do?  Adds an **unbiased pass@k** validation metric (the Codex/HumanEval estimator) and logs it next to the existing `mean@k` / `best@k` / `worst@k` metrics.  Today, `process_validation_metrics` reports `best@k` via `bootstrap_metric`, which subsamples `k` items **with replaceme…

### #7238 — [[doc] feat: Update vllm/vllm-ascend and torch version](https://github.com/verl-project/verl/pull/7238)
- **作者**: LeoYao123  **时间**: 2026-08-03 20:33 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  update vllm/vllm-ascend torch and transformer verion in npu install docs and dockerfile  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be chec…

### #7237 — [[trainer] refactor: move shared PPO helpers out of RayPPOTrainer](https://github.com/verl-project/verl/pull/7237)
- **作者**: zzzzzzzxh  **时间**: 2026-08-03 17:41 CST
- **摘要**: ### What does this PR do?  Part of #6985.  This PR moves shared PPO trainer helper functions out of the deprecated `RayPPOTrainer` module and into a dedicated `verl.trainer.ppo.trainer_utils` module.  The refactor is intentionally behavior-preserving:  - moves `apply_kl_penalty`, `compute_response_m…

### #7236 — [[data] feat: Add processor hook for multimodal RoPE kwargs](https://github.com/verl-project/verl/pull/7236)
- **作者**: ZihaoW123  **时间**: 2026-08-03 17:17 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `…

### #7235 — [[training_utils] fix: gate FusedLinearForPPO backward grads on ctx.needs_input_grad](https://github.com/verl-project/verl/pull/7235)
- **作者**: thuwyh  **时间**: 2026-08-03 16:47 CST
- **摘要**: ### What does this PR do?  Fixes a silent gradient-dropping bug in `FusedLinearForPPOFunction.backward` (`use_fused_kernels=True`): the allocation of `dhidden_states` / `dvocab_weights` was gated on the **saved tensors'** `requires_grad`, which is not reliable. `autograd.Function.forward` runs with …
