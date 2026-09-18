# verl-project/verl — 动态追踪

> 生成时间: 2026-09-18 13:04 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### Issue 动态
本期共有 3 个主要 Issue，涉及训练机制限制、算法数值稳定性及多适配器功能需求：
1. **Megatron 序列均值限制问题** (#7914)：在使用 Megatron 引擎时，`seq-mean-token-mean` 保护机制过于严格，导致 CP=1（上下文并行=1）的合法输出被错误拒绝。
2. **GMPO 几何均值损失计算异常** (#7913)：当 minibatch 被拆分为更多 microbatch 时，`geo_mean` 损失值会发生非预期的变化。
3. **多 LoRA 训练需求** (#7909)：用户希望 verl 能支持在同一个基础模型上进行多 LoRA 适配器驻留及混合适配器训练步骤。

### PR 动态
本期共合并/提交 10 个 PR，涵盖重要 Bug 修复、新特性引入及硬件适配：

**重要修复与算法变更：**
- **[破坏性变更] GMPO geo_mean 修复** (#7916)：修复 Issue #7913，要求对 `geo_mean` 进行 minibatch 归一化，并补偿 DP 梯度平均带来的误差。
- **Megatron 损失限制解除** (#7915)：修复 Issue #7914，移除了 Megatron 逐 token 损失回调中的拒绝机制，允许在 CP=1 时使用 sequence-mean loss。
- **全异步请求处理修复** (#7912)：修复全异步模式下请求在关闭的门后到达被异常挂起的问题，改为直接拒绝并立即返回。
- **GRPO 内存泄漏修复** (#7908) 与 **响应截断追踪修复** (#7906)：分别修复了内存泄漏问题，以及因上下文长度限制导致的响应提前截断追踪问题。

**重要新特性：**
- **亚秒级容错架构** (#7910)：为解耦式 RL 训练（Trainer 与 Rollout 跨节点分离）引入了可选的容错机制，支持动态 NCCL 重新 rendezvous 和内存故障转移，防止流水线崩溃。
- **TPU 硬件适配与加速** (#7907, #7905)：#7907 通过最小修复使 Qwen3-0.6B GRPO 能在 TPU 上结合 TorchTitan 引擎和 vLLM rollout 实现端到端运行；#7905 引入了 Raiden checkpoint 引擎以实现高速 TPU 权重同步（目前为单芯片 PoC）。
- **Qwen3.5 配置优化** (#7903)：修改脚本配置以缓解 Qwen3.5 训练时的内存碎片问题。
- **平台钩子** (#7911)：引入了平台级 hooks 机制以扩展定制能力。

### Release 动态
本段时间内无新版本发布。

---

## 🐛 Issues

### #7914 — [[megatron] seq-mean-token-mean guard rejects CP=1 and appears overly restrictive for reconstructed THD outputs](https://github.com/verl-project/verl/issues/7914)
- **作者**: Zhikaiiii  **时间**: 2026-09-18 11:55 CST
- **摘要**: ## System Info  - Observed in a custom training application using verl's v1 trainer / Megatron engine. - Model: Qwen3.5-9B. - Training topology: 16 GPUs, TP=8, PP=1, static CP=2 (DP=1). - Python: 3.12. - `use_remove_padding=True`, `dynamic_context_parallel=False`. - Policy loss: GSPO; aggregation: `…

### #7913 — [[Bug] GMPO geo_mean loss changes when a minibatch is split into more microbatches](https://github.com/verl-project/verl/issues/7913)
- **作者**: gss10282025  **时间**: 2026-09-18 11:35 CST
- **标签**: bug
- **摘要**: ### System Info  ```text verl: 24f25b03aa4b54249a273655ebbcce06f484192b PyTorch: 2.11.0+cu130 Transformers: 5.5.3 Python: 3.12 GPU: NVIDIA GeForce RTX 5090, one visible GPU per process ```  Source: [core_algos.py at 24f25b03](https://github.com/verl-project/verl/blob/24f25b03aa4b54249a273655ebbcce06…

### #7909 — [Multi-LoRA training: resident adapters and mixed-adapter steps on one base model](https://github.com/verl-project/verl/issues/7909)
- **作者**: savaresejeremy  **时间**: 2026-09-18 06:39 CST
- **摘要**: ### Summary  verl trains one LoRA adapter per training engine: `get_peft_model` under the default adapter name, `max_loras: 1` on the vLLM side, the LoRA helpers (`collect_lora_params`, `layered_summon_lora_params`) read the `default` adapter only, and the checkpoint path records one adapter's rank …

## 🔀 Pull Requests

### #7916 — [[BREAKING][algo] fix: require minibatch normalization for geo_mean](https://github.com/verl-project/verl/pull/7916)
- **作者**: gss10282025  **时间**: 2026-09-18 12:23 CST
- **摘要**: ### What does this PR do?  Fixes #7913. Each `geo_mean` microbatch now contributes to the optimizer-minibatch sequence mean through `agg_loss`, including compensation for DP gradient averaging.  #5614 already proposes the same core aggregation fix. This draft overlaps that implementation. Its additi…

### #7915 — [[megatron] fix: allow sequence-mean loss with per-token normalization](https://github.com/verl-project/verl/pull/7915)
- **作者**: Zhikaiiii  **时间**: 2026-09-18 12:01 CST
- **摘要**: ### What does this PR do?  Fixes #7914 by removing the `seq-mean-token-mean` rejection in the Megatron per-token loss callback. The guard rejects CP=1 when `calculate_per_token_loss=True`, and also blocks static-CP THD training even though the forward path reconstructs full-sequence outputs before P…

### #7912 — [[fully_async] fix: reject requests arriving behind the closed gate stead of parking them](https://github.com/verl-project/verl/pull/7912)
- **作者**: wuxibin89  **时间**: 2026-09-18 11:34 CST
- **摘要**: ### What does this PR do?  Fix https://github.com/verl-project/verl/issues/7865, add a additional flag `reject_request=True` to abort_replicas, making replicas reject and return immediately instead of parking them.

### #7911 — [Pr1 platform hooks](https://github.com/verl-project/verl/pull/7911)
- **作者**: askhat-g  **时间**: 2026-09-18 08:35 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #7910 — [[ckpt, ray, trainer] feat: add opt-in sub-second fault tolerance with dynamic NCCL re-rendezvous and in-memory failover](https://github.com/verl-project/verl/pull/7910)
- **作者**: sunnylovestiramisu  **时间**: 2026-09-18 06:40 CST
- **摘要**: ### What does this PR do?  This PR introduces an opt-in fault tolerance architecture for disaggregated RL training (Trainer & Rollout separated across GPU nodes) that prevents pipeline collapse and long restarts when GPU workers or nodes abruptly fail (e.g., hard process crashes, CUDA uncorrectable …

### #7908 — [ Fix grpo memory leak](https://github.com/verl-project/verl/pull/7908)
- **作者**: jialei777  **时间**: 2026-09-18 05:55 CST
- **摘要**: cherry pick commit https://github.com/jialei777/verl-upstream/pull/9/changes/87032204ebdc552ed6df5e99816794ed98307d85

### #7907 — [Lixali/tpu run](https://github.com/verl-project/verl/pull/7907)
- **作者**: lixali  **时间**: 2026-09-18 05:43 CST
- **摘要**: ### What does this PR do?  Minimal fixes needed to run Qwen3-0.6B GRPO end-to-end on TPU with the TorchTitan engine and vLLM rollout.  - `vLLMColocateWorkerExtension` imported `verl.utils.vllm.vllm_fp8_utils` unconditionally at module load. That module pulls in GPU-only FP8 dependencies that are una…

### #7906 — [[trainer, rollout] fix: track response truncation across context limits](https://github.com/verl-project/verl/pull/7906)
- **作者**: yuchenwang3  **时间**: 2026-09-18 01:39 CST
- **摘要**: ### What does this PR do?  Fixes #7823. A response can hit `max_model_len` before it reaches the configured response length. For example, a context limit of 8 and prompt lengths of 2/3/4 produce truncated responses of 6/5/4 tokens; the old width comparison reports a zero clip ratio.  Carry an option…

### #7905 — [feat(tpu): introduce Raiden checkpoint engine for high-speed TPU weight sync](https://github.com/verl-project/verl/pull/7905)
- **作者**: wenjung2007  **时间**: 2026-09-18 01:17 CST
- **摘要**: DO NOT MERGE: this poc is only for single chip use-case.  Integrate Raiden (tpu-sync) weight synchronization backend for TPU-based reinforcement learning (GRPO) training. This implementation leverages direct D2H/H2D DMA transfers and socket-based P2P communication between the trainer and vLLM sample…

### #7903 — [[trainer, cfg] feat: modify script configuration for qwen3_5.](https://github.com/verl-project/verl/pull/7903)
- **作者**: ChibiQuest  **时间**: 2026-09-17 21:28 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  Because memory fragmentation may occasionally cause OOM, modifying this makes it more stable.  ### Checklist Before Starting  - [ ]…
