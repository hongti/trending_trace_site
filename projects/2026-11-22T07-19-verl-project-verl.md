# verl-project/verl — 动态追踪

> 生成时间: 2026-11-22 15:19 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📋 Issue
1. **配置与调试体验**：
   - 静态批处理校验报错信息不清晰，仅输出最终大小，未展示各参数（如 `train_batch_size`、`rollout.n`、FSDP 等）是如何推导出该结果的（#8793）。
   - V1 trainer 的 NPU profiler 配置存在静默失效问题，必须同时配置全局和 Actor 级别的 profiler 参数才能生效（#8789）。
2. **性能瓶颈**：
   - V1 FSDP 路径下的 `update_weights` 每步都全量广播模型参数（如 Qwen2.5-32B 每步耗时约 9 秒），缺乏增量同步或分桶同步机制，严重拖慢训练速度（#8790）。

### 🔧 Pull Request
**新特性与性能优化**
- **内存优化**：为 FSDP 引入融合分块 logprobs 和 entropy 操作，消除了词表维度的内存峰值（#8786）。
- **检查点加速**：合并 Megatron 异步检查点的预加载操作，并对卸载到 CPU 的张量进行快照，减少每步多次独立写入的开销（#8784）。
- **实验门槛降低**：新增快速失败配置校验机制，并为单卡（1xH20）环境下的 Qwen2.5-0.5B GRPO 提供调优配置模板（#8782）。

**核心 Bug 修复**
- **检查点恢复**：修复了开启 CPU offload 时，Megatron 检查点恢复预分配优化器状态会导致模型权重被静默修改的严重问题（#8787）。
- **并行计算**：修复了静态上下文并行（CP > 1）下计算 per-token loss 时 token-count 余数丢失的问题（#8785）。
- **错误提示**：优化静态批大小校验报错，现在会明确指出导致不可整除的具体参数来源（#8792）。

**CI 与硬件适配（Ascend / vLLM）**
- 为失败的 Ascend 端到端测试自动收集并上传 HCCL 诊断日志（#8791）。
- 修复了 Ascend pytest 任务中 `torch_npu 2.10` 的日志销毁竞争条件（#8788）。
- 在稳定的 vLLM Docker 镜像中补全了 Megatron 运行时依赖（如 PuLP、SciPy、FLA 等）（#8783）。

### 🚀 Release
- **本期暂无版本发布信息。**

---

## 🐛 Issues

### #8793 — [[cfg] Batch divisibility errors do not explain the effective batch sizes](https://github.com/verl-project/verl/issues/8793)
- **作者**: archted  **时间**: 2026-11-22 15:10 CST
- **摘要**: ## Problem  Static batch validation reports only the effective batch size and minimum batch size, without showing how they were derived.  For example, `train_batch_size=4`, `rollout.n=2`, and FSDP with 2 nodes × 3 devices produce:  ```text AssertionError: real_train_batch_size (8) must be divisible …

### #8790 — [[V1 FSDP] update_weights broadcasts full params every step (~9s/step on 32B); no delta or bucketed sync mode](https://github.com/verl-project/verl/issues/8790)
- **作者**: dahai80  **时间**: 2026-11-22 11:22 CST
- **摘要**: ## Summary  On the V1 FSDP path, `update_weights` (actor → rollout engine weight sync) broadcasts the **full parameter set every training step**. Measured cost on Qwen2.5-32B TP8: ~9 s/step, every step, regardless of how few parameters changed. Over hundreds of GRPO steps this is a fixed tax proport…

### #8789 — [[V1 trainer] NPU profiler config requires BOTH global_profiler.steps and actor.profiler; missing either side silently no-ops](https://github.com/verl-project/verl/issues/8789)
- **作者**: dahai80  **时间**: 2026-11-22 11:22 CST
- **摘要**: ## Summary  On verl V1 (`trainer.use_v1=True`), enabling the NPU profiler only works when **both** sides are configured:  - `global_profiler.steps=[1]` — the trigger, read in `ray_trainer.py` - `actor_rollout_ref.actor.profiler.{tool,save_path,tool_config}` — worker instance config, read by the acto…

## 🔀 Pull Requests

### #8792 — [[cfg] fix: explain static batch divisibility failures](https://github.com/verl-project/verl/pull/8792)
- **作者**: archted  **时间**: 2026-11-22 13:30 CST
- **摘要**: Static batch validation currently reports only the product and minimum batch size. On a multi-node run, the error does not show which training, rollout, or device settings produced those values.  This reports train_batch_size, rollout.n, devices per node, node count, and the actual backend-specific …

### #8791 — [[ci, hardware] feat: collect HCCL diagnostics for failed Ascend e2e runs](https://github.com/verl-project/verl/pull/8791)
- **作者**: lxb007981  **时间**: 2026-11-22 11:24 CST
- **摘要**: ### What does this PR do?  Capture console output and native Ascend logs for each training run. Upload them when a failed run contains an HCCL error signature, and discard them otherwise. It aims to diagnose intermittent CI failures like https://github.com/verl-project/verl/actions/runs/44711712430/…

### #8788 — [[ci, hardware] fix: avoid the torch_npu 2.10 logger teardown race in Ascend pytest jobs](https://github.com/verl-project/verl/pull/8788)
- **作者**: lxb007981  **时间**: 2026-11-22 10:43 CST
- **摘要**: ### What does this PR do?  Run Ascend model, NPU unit, and vLLM tests through a pytest wrapper. Let Ray and torch_npu run their Python cleanup hooks, then exit the driver with pytest's status before native teardown can free a logger that an ACL worker thread may still be using (UAF).  ### Checklist …

### #8787 — [[megatron, ckpt] fix: keep resumed model weights intact when pre-allocating optimizer state](https://github.com/verl-project/verl/pull/8787)
- **作者**: taruki11  **时间**: 2026-11-21 22:44 CST
- **摘要**: ### What does this PR do?  Fixes #8765. Resuming a Megatron checkpoint with optimizer state enabled (`optimizer_cpu_offload=True`, i.e. `HybridDeviceOptimizer`) could silently change the model weights during `load_checkpoint()`: the v2 split loader loads the model weights first and then builds the o…

### #8786 — [[perf, fsdp, worker, cfg] feat: add fused chunked logprobs and entropy op to eliminate vocab memory peak](https://github.com/verl-project/verl/pull/8786)
- **作者**: ApacheWang  **时间**: 2026-11-21 20:47 CST
- **摘要**: ### What does this PR do?  This PR introduces an opt-in `use_fused_chunked_loss` path (`verl.utils.torch_functional.fused_chunked_logprobs_and_entropy`) for the FSDP actor forward/backward pipeline to eliminate large-vocabulary peak HBM spikes when computing per-token `log_probs` and policy `entropy…

### #8785 — [[megatron] fix: preserve token-count remainder under static context parallelism](https://github.com/verl-project/verl/pull/8785)
- **作者**: donghyq  **时间**: 2026-11-21 20:43 CST
- **摘要**: ### What does this PR do?  Under Megatron `calculate_per_token_loss=True` with static context parallelism (`context_parallel_size > 1`, `dynamic_context_parallel=False`), `MegatronEngineWithLMHead.postprocess_micro_batch_func` currently reports `local_num_tokens = self._routed_num_tokens(data) // cp…

### #8784 — [[megatron, ckpt, cfg] perf: coalesce per-step async checkpoint preloads and snapshot offloaded CPU tensors](https://github.com/verl-project/verl/pull/8784)
- **作者**: donghyq  **时间**: 2026-11-21 20:42 CST
- **摘要**: ### What does this PR do?  When `checkpoint.async_save=True` is enabled on Megatron, `MegatronCheckpointManager.save_checkpoint` schedules up to three separate `dist_checkpointing` writes per step (`model/dist_ckpt`, `optimizer/dist_ckpt`, `extra/dist_ckpt`). Because Megatron-Core's persistent async…

### #8783 — [[megatron, vllm] fix: install Megatron runtime dependencies in the stable vLLM image](https://github.com/verl-project/verl/pull/8783)
- **作者**: amenomori  **时间**: 2026-11-21 18:01 CST
- **摘要**: ### What does this PR do?  Addresses #8770 in the stable vLLM Dockerfile. Megatron-Bridge imports ModelOpt at startup, and ModelOpt in turn imports PuLP and SciPy. Megatron Core also needs FLA for Qwen3.5 GatedDeltaNet. AI assistance was used to prepare the change and run the tests.  ### Checklist B…

### #8782 — [[cfg, perf, doc] feat: add fail-fast config validation and 1xH20 tuning tiers for Qwen2.5-0.5B GRPO](https://github.com/verl-project/verl/pull/8782)
- **作者**: StephenAvA  **时间**: 2026-11-21 15:21 CST
- **摘要**: ## What does this PR do?  Two related changes that lower the entry cost of small-scale RL experiments on a single GPU:  1. **Fail-fast config preflight** (`verl/utils/config.py`): a new `_validate_parallel_and_memory_config()` called at the top of `validate_config()` catches 6 classes of common misc…
