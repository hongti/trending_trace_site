# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-10 10:43 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 🐛 Issue 动态
- **LoRA prompt_logprobs 计算错误** (#51594)：当 LoRA 适配器作用于 `lm_head` 时，其 delta 增量在 `prompt_logprobs` 中被静默丢弃，导致 teacher-forced 阶段的 prompt logprobs 返回错误结果（生成阶段的 logprobs 正常）。
- **DeepSeek-V4-Flash MTP 推测解码挂起** (#51593)：在 SM120 架构上使用 MTP 推测解码时，高负载运行 12-14 分钟后，当 batch 缩减至 3 个请求时，引擎会因 `shm_broadcast` 超时发生确定性挂起。

### 🛠️ PR 动态
**1. 重要 Bug 修复**
- **修复 LoRA lm_head delta 丢失** (#51597)：直接修复 #51594，确保 `lm_head` 的 delta 增量被正确应用于 `prompt_logprobs`。
- **修复异步 Mamba 对齐问题** (#51599)：解耦异步 Mamba align D2H 计数与 InputBatch 行偏移，修复了在 `use_async_scheduling=True` 下使用推测解码/MTP 时的状态更新错误。

**2. 性能与内存优化**
- **CUDA Graph 内存预算修正** (#51590)：修复了 `profile_cudagraph_memory()` 中内存计算偏少的问题，改为测量完整的 CUDA graph 捕获占用空间，使 KV cache 内存预算更准确。
- **DeepEP v2 Prefill 调度优化** (#51589)：新增自适应“无同步” prefill 调度机制，通过环境变量 `VLLM_DEEPEP_V2_MAX_SYNCLESS_TOKENS` 控制阈值，提升性能。

**3. 硬件支持与适配**
- **Intel XPU** (#51600)：移除 `VLLM_XPU_ENABLE_XPU_GRAPH` 标志，因为 PyTorch XPU 2.14 已将 XPU Graph 从 SYCL Graph 切换至 Level Zero Graph，解决了先前的限制。
- **AMD ROCm** (#51598)：为 gfx1100 (RDNA3) 启用作用域严格的 AITER W8A8 支持，此路径采用失败封闭设计，不会放宽对现有 CDNA 或 RDNA4 的限制。

**4. 新特性与功能增强**
- **RL 自定义权重传输** (#51596)：允许通过 `register_engine` API 注册并使用自定义的 `WeightTransferEngine` 和 `WeightTransferTrainer` 后端。
- **KVConnector 异步加载计数** (#51595)：在 MultiConnector 中增加了按请求统计异步加载（async loads）的功能，与原有的异步保存逻辑对齐。
- **Rust Benchmark 参数对齐** (#51592)：对齐 Rust 与 Python 的 speed-bench CLI 参数，并添加参数一致性测试，修复了标志位漂移问题。

**5. 文档更新**
- **Metal 快速入门** (#51591)：为 Apple Silicon 用户添加提示，推荐使用 MLX 优化的 `mlx-community` 模型，替代原先的 `facebook/opt-125m`。

### 🚀 Release 动态
- 本期暂无新版发布。

---

## 🐛 Issues

### #51594 — [[Bug]: LoRA lm_head delta is silently dropped from prompt_logprobs](https://github.com/vllm-project/vllm/issues/51594)
- **作者**: XiaohanZhangCMU  **时间**: 2026-08-10 09:33 CST
- **摘要**: ### Summary  When a LoRA adapter targets `lm_head`, its delta is **silently dropped from `prompt_logprobs`**. Generation logprobs are correct; teacher-forced prompt logprobs for the same tokens are not. Nothing warns, and the returned numbers look plausible.  ### Evidence  Measured on 2x B300, TP=2,…

### #51593 — [[Bug]: DeepSeek-V4-Flash MTP hangs after the batch drains to 3 requests (engine core shm_broadcast timeout) on SM120](https://github.com/vllm-project/vllm/issues/51593)
- **作者**: lucifer1004  **时间**: 2026-08-10 08:57 CST
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information... ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (x86_64) GCC …

## 🔀 Pull Requests

### #51600 — [[XPU] remove VLLM_XPU_ENABLE_XPU_GRAPH](https://github.com/vllm-project/vllm/pull/51600)
- **作者**: zhenwei-intel  **时间**: 2026-08-10 10:41 CST
- **标签**: intel-gpu, ci/build
- **摘要**: ## Purpose XPU Graph delivers significant performance gains for small models and MoE models under low concurrency.  PyTorch XPU 2.14 switches XPU Graph from SYCL Graph to Level Zero Graph, addressing previous multi-GPU limitations and high memory usage.  This PR:  - Removes the `VLLM_XPU_ENABLE_XPU_…

### #51599 — [fix(v1): decouple async Mamba align D2H counts from InputBatch row shifts (#51571)](https://github.com/vllm-project/vllm/pull/51599)
- **作者**: bandham-manikanta  **时间**: 2026-08-10 10:31 CST
- **摘要**: ### Target Issue Closes #51571  ### Description When running speculative decoding / MTP in `align` mode with `use_async_scheduling=True`, `GPUModelRunner._update_states_after_model_execute()` passes `input_batch.num_accepted_tokens_cpu_tensor` as the D2H target for `postprocess_mamba_align_gpu()`.  …

### #51598 — [[ROCm][RFC] Enable scoped AITER W8A8 support on gfx1100](https://github.com/vllm-project/vllm/pull/51598)
- **作者**: 01xjw  **时间**: 2026-08-10 10:24 CST
- **标签**: rocm
- **摘要**: ## Purpose  Provide a narrowly scoped, fail-closed AITER W8A8 path for gfx1100 (RDNA3) without widening vLLM's existing CDNA or RDNA4 gates to unsupported CK/ASM operations.  Addresses vllm-project/vllm#51136. This Draft depends on ROCm/aiter#4512 and ROCm/aiter#4648.  The linked issue already recor…

### #51597 — [[LoRA] Fix lm_head delta silently dropped from prompt_logprobs (#51594)](https://github.com/vllm-project/vllm/pull/51597)
- **作者**: jialoop-git  **时间**: 2026-08-10 10:16 CST
- **摘要**: ## Summary  Fixes #51594.  When a LoRA adapter targets `lm_head`, `prompt_logprobs` silently returns wrong values — the lm_head delta is not applied. Generation logprobs are correct; only the prompt-logprobs path is broken.  **Root cause**: `add_lora_logits` in `punica_gpu.py` uses `prompt_mapping_m…

### #51596 — [[RL] Allow custom weight transfer backend](https://github.com/vllm-project/vllm/pull/51596)
- **作者**: wangxiyuan  **时间**: 2026-08-10 09:58 CST
- **摘要**: ## Purpose vLLM support register custom  `WeightTransferEngine` and `WeightTransferTrainer` via `register_engine` api. While once the new one is registered, there is no way to use it with vLLM. It because that the input pydantic check fail.  This PR allow pass the custom backend to WeightTransferCon…

### #51595 — [[KVConnector] Count async loads per request in MultiConnector](https://github.com/vllm-project/vllm/pull/51595)
- **作者**: rebel-jinhwan  **时间**: 2026-08-10 09:35 CST
- **标签**: kv-connector
- **摘要**: ## Purpose  `MultiConnector` already tracks how many sub-connectors are still async-**saving** a request (`_extra_async_saves`) and only forwards `finished_sending` once the last one has reported, so the scheduler frees the blocks exactly once.  `finished_recving` had no equivalent accounting — ever…

### #51592 — [[Rust][Benchmark] Align speed-bench CLI flags with Python and add flag parity test](https://github.com/vllm-project/vllm/pull/51592)
- **作者**: esmeetu  **时间**: 2026-08-10 08:55 CST
- **标签**: performance, rust
- **摘要**: ## Purpose  `VLLM_USE_RUST_BENCH=1` delegation (#50081) execs `vllm-bench` with the raw argv, so Python-documented flags must parse in the Rust CLI. The speed-bench flags had drifted:  - `--speed-bench-output-len` did not exist in Rust — it failed with `unexpected argument`, and without it the outpu…

### #51591 — [docs: clarify Metal quickstart model choice](https://github.com/vllm-project/vllm/pull/51591)
- **作者**: Gloria72  **时间**: 2026-08-10 07:57 CST
- **标签**: documentation
- **摘要**: ## Summary - Add an Apple Silicon note near the offline inference quickstart model example. - Point vLLM-Metal users to an MLX-optimized `mlx-community` model instead of `facebook/opt-125m`.  Fixes #50165  ## Testing - `git diff --check`  Docs build not run locally because `mkdocs` is not installed …

### #51590 — [[Memory] Measure complete CUDA graph capture footprint for KV budgeting](https://github.com/vllm-project/vllm/pull/51590)
- **作者**: xiaohuguo2023  **时间**: 2026-08-10 07:55 CST
- **标签**: nvidia
- **摘要**: ## Purpose  Fix CUDA graph memory undercounting in `profile_cudagraph_memory()`.  The old path profiled only two descriptors per graph mode and extrapolated the rest. It also summed per-mode deltas even though FULL and PIECEWISE share one runtime pool, and it did not budget memory allocated during p…

### #51589 — [[Perf] Add adaptive sync-less DeepEP v2 prefill dispatch](https://github.com/vllm-project/vllm/pull/51589)
- **作者**: LucasWilkinson  **时间**: 2026-08-10 06:59 CST
- **摘要**: ## Summary  - Add an opt-in `VLLM_DEEPEP_V2_MAX_SYNCLESS_TOKENS` threshold, disabled by default. - Select sync-less eager prefill dispatch from the maximum post-sequence-parallel token count across DP ranks. - Reconstruct aligned expert counts and expert IDs from DeepEP GPU prefix sums, avoiding the…
