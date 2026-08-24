# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-24 10:09 CST

## AI 总结

## vllm-project/vllm 近期动态摘要

---

### 🐛 Issue

1. **KV 缓存分区问题**（#53495）：`ExampleConnector` 基于 raw prompt tokens 进行外部 KV 缓存键值匹配，导致 `cache_salt`、LoRA 和 `prompt_embeds` 无法正确分区，影响缓存隔离。
2. **MTP 推测解码导致 prompt_logprobs 损坏**（#53488）：启用 MTP（multi-token prediction）推测解码 + chunked prefill 时，部分请求的 `prompt_logprobs` 被静默损坏，涉及 Qwen3.5 系列模型。
3. **RFC: 二级存储层使用指标**（#53486）：提议为磁盘/SSD 等 KV offload 层增加使用量监控指标，当前已有 `FileS` manager 支持磁盘层级，但缺少对应的 metrics 暴露。

---

### 🔀 Pull Request

**新特性 / 性能优化：**
- **[ROCm] 启用 Aiter 稀疏 MLA Gluon 内核**（#53492）：为 MI355 上 GLM 5.2 模型加速，默认关闭，通过 flag 开启。
- **[Kimi-K3] 拆分 KDA mixer 的 CUDA Graph**（#53487）：将 KDA mixer（causal-conv + recurrent KDA + gated RMSNorm）从分段 CUDA graph 中分离，减少 eager trace 开销。
- **[Elastic EP] 无需 warmup 即初始化通信组**（#53483）：新组初始化不再强制运行 warmup collective，避免与推理前向通信重叠导致的问题。
- **[XPU] 更新支持模型列表**（#53494）：新增 DeepSeek-V4-Flash、Phi-3.5-mini、Qwen3-30B-A3B-FP8、Qwen3-Next-80B-A3B、InternVL3.5-30B-A3B 等。

**Bug 修复：**
- **[LoRA] Gemma 4 MoE 模型 LoRA 服务崩溃**（#53482）：缺少 `get_expert_mapping` 实现，导致任何 LoRA adapter 加载均报 `AttributeError`。
- **[V1] Mamba 混合模型前缀缓存失效**（#53479）：`_mamba_block_aligned_split` 存在两个耦合缺陷——状态稀疏且对齐模式下推测回退逻辑错误，导致前缀缓存在非精确重复流量下基本无效。
- **[推测解码] 无需 logprobs 时仍克隆完整 target logits**（#53489）：修复 rejection sampler 在 `logprobs` 未请求时不必要的张量克隆，降低显存和计算开销。
- **[Benchmark] MoE 调优器 OOM**（#53490）：Triton 3.7.1 中 `triton.runtime.cache.clear()` 不可调用导致缓存清理失败，长时搜索仍会 OOM。
- **[Core] CPU↔GPU 同步检查遗漏分页异步拷贝**（#53491）：扩展 `torch.cuda.set_sync_debug_mode` 同步检查范围，覆盖 paged async copy 场景。

**文档：**
- **新增 GPU 架构兼容性矩阵 + 扩展 KServe 集成指南**（#53493）：填补生产部署中发现的文档缺口。

---

### 📦 Release

本期无新版本发布。

---

## 🐛 Issues

### #53495 — [[Bug]: ExampleConnector keys external KV on raw prompt tokens, so cache_salt, LoRA and prompt_embeds do not partition it](https://github.com/vllm-project/vllm/issues/53495)
- **作者**: oneKn8  **时间**: 2026-08-24 10:05 CST
- **摘要**: ### Your current environment  main at 185cada3 (2026-08-23), `VLLM_TARGET_DEVICE=cpu`, no model loaded. The reproduction is a pure scheduler-side unit test, so no GPU is involved.  ### Describe the bug  `ExampleConnector` derives its external storage key from the prompt token bytes only (`_generate_…

### #53488 — [[Bug]: `prompt_logprobs` silently corrupted for some requests when MTP speculative decoding is enabled (Qwen3.5-family, chunked prefill; two builds, two checkpoints)](https://github.com/vllm-project/vllm/issues/53488)
- **作者**: nyhhome  **时间**: 2026-08-24 07:55 CST
- **标签**: speculative-decoding, quantization
- **摘要**: ### Your current environment  - **Hardware**: NVIDIA DGX Spark (GB10, compute capability 12.1, aarch64, 121 GiB unified memory), single GPU - **Build A**: `vllm/vllm-openai:nightly-aarch64` — vLLM `0.26.1rc1.dev1102+ge9d1398d9`, flashinfer 0.6.17, torch 2.13.0+cu130 - **Build B** (independent source…

### #53486 — [[RFC]: Secondary Tier Usage Metrics](https://github.com/vllm-project/vllm/issues/53486)
- **作者**: Srinivasoo7  **时间**: 2026-08-24 06:32 CST
- **标签**: RFC
- **摘要**: ### Motivation.  #43763 asks whether `SimpleCPUOffloadConnector` will support disk/SSD tiers. The answer is: a different, newer connector already does `vllm/v1/kv_offload/tiering/fs/manager.py` (`FileSystemTierManager`) is a working disk-backed secondary tier, and `obj/manager.py` covers object stor…

## 🔀 Pull Requests

### #53494 — [[XPU] update key supported models](https://github.com/vllm-project/vllm/pull/53494)
- **作者**: yma11  **时间**: 2026-08-24 09:58 CST
- **标签**: documentation, intel-gpu
- **摘要**: ## Purpose  ## Test Plan Update:  Text-only：`DeepSeek-V4-Flash`、`Phi-3.5-mini-instruct`、`Qwen3-30B-A3B-FP8`、`Qwen3-Next-80B-A3B-Instruct`、`Qwen3-Next-80B-A3B-Thinking` Multimodal：`InternVL3_5-30B-A3B`、`Muse-Glimmer-30B`、`Qwen3-VL-32B-Instruct`、`Qwen3.5-35B-A3B`、g`emma-3-27b-it`、`gemma-4-31B-it`、`gem…

### #53493 — [docs: add GPU architecture compatibility matrix and expand KServe integration guide](https://github.com/vllm-project/vllm/pull/53493)
- **作者**: mgonzalezo  **时间**: 2026-08-24 09:37 CST
- **标签**: documentation
- **摘要**: … guide      ## Purpose  To fix documentation gaps discovered during production deployments of vLLM on KServe:  **1. GPU architecture compatibility matrix**  The official vLLM Docker images and pip wheels are compiled for specific CUDA compute capabilities (sm_70, sm_75, sm_80, etc.). When a GPU's a…

### #53492 — [[ROCm][MLA] Enable sparse MLA Gluon kernel from Aiter](https://github.com/vllm-project/vllm/pull/53492)
- **作者**: cagrikymk  **时间**: 2026-08-24 09:22 CST
- **标签**: rocm
- **摘要**: `ROCMAiterMLASparseBackend` extended to enable Aiter's gluon backend, hidden behind a flag, off by default.  ## Purpose The main goal is to improve performance for GLM 5.2 on MI355 by accelerating the sparse MLA attention using the new gluon backend, both for decode and prefill.  Model used for test…

### #53491 — [[Core] Enhance cpu<->gpu sync checking to include paged async copies](https://github.com/vllm-project/vllm/pull/53491)
- **作者**: njhill  **时间**: 2026-08-24 09:19 CST
- **摘要**: The CPU<->GPU sync checking added in https://github.com/vllm-project/vllm/pull/44800 / https://github.com/vllm-project/vllm/pull/43107 makes use of pytorch's `torch.cuda.set_sync_debug_mode` function.  However this does not catch stalls due to "non-blocking" tensor copies to or from non-pinned host …

### #53490 — [[Benchmark] Fix MoE tuner cache cleanup and OOM recovery](https://github.com/vllm-project/vllm/pull/53490)
- **作者**: gaoxiaomo  **时间**: 2026-08-24 08:18 CST
- **标签**: performance
- **摘要**: ## Purpose  Long MoE tuning searches can still OOM because `clear_triton_cache()` probes `triton.runtime.cache.clear()`, which is not callable in Triton 3.7.1. Compiled kernels remain referenced by `JITFunction.device_caches`, so the worker accumulates memory across configurations.  This change:  - …

### #53489 — [[Bugfix] Avoid cloning target logits when logprobs are disabled](https://github.com/vllm-project/vllm/pull/53489)
- **作者**: KilJaeeun  **时间**: 2026-08-24 08:06 CST
- **标签**: bug
- **摘要**: ## Purpose  Avoid cloning the full target-logits tensor during speculative decoding when logprobs are not requested.  In raw-logprobs mode, the rejection sampler currently clones `raw_target_logits` before applying in-place logits processors. The raw tensor is only consumed by `_get_logprobs_tensors…

### #53487 — [[Kimi-K3] Split the KDA mixer out of piecewise CUDA graphs](https://github.com/vllm-project/vllm/pull/53487)
- **作者**: xiaohuguo2023  **时间**: 2026-08-24 06:49 CST
- **标签**: torch.compile, nvidia, kimi, k3
- **摘要**: ## Purpose  Kimi-K3 piecewise CUDA graphs already split at MLA (`vllm::unified_mla_attention_with_output`), but the KDA mixer (causal-conv + recurrent KDA + gated RMSNorm) was still traced as many eager Triton launches. `@eager_break_during_capture` on `_forward` is a no-op unless `VLLM_USE_BREAKABL…

### #53483 — [[Elastic EP] Initialize new groups without warmup collectives](https://github.com/vllm-project/vllm/pull/53483)
- **作者**: itayalroy  **时间**: 2026-08-24 06:18 CST
- **标签**: rocm, nvidia
- **摘要**: Elastic EP previously force-initialized communication groups for the target configuration by running a warmup collective on each new group. These collectives overlap with forward-pass collectives from the main serving thread, which seems to be risky in some cases (ROCm ran into issues with this over…

### #53482 — [[Bugfix][LoRA] Implement get_expert_mapping for Gemma 4 MoE models](https://github.com/vllm-project/vllm/pull/53482)
- **作者**: RichWoollcott  **时间**: 2026-08-24 06:05 CST
- **标签**: bug
- **摘要**: ## Purpose  Serving **any** LoRA adapter against a **MoE Gemma 4** model currently fails at engine start:  ``` INFO  [utils.py:101] MoE model detected. Using fused MoE LoRA implementation. AttributeError: To support LoRA for MoE model, 'get_expert_mapping' must be implemented ```  `LoRAModelManager`…

### #53479 — [[Bugfix][V1] Mamba align: materialize a state at every boundary and drop the speculative one-block back-off](https://github.com/vllm-project/vllm/pull/53479)
- **作者**: kamb-code  **时间**: 2026-08-24 04:27 CST
- **标签**: bug, kv-connector, scheduler, kv-cache-manager
- **摘要**: ## Purpose  Two coupled defects in `_mamba_block_aligned_split` (#52897) leave hybrid-model prefix caching mostly inert outside of exact-repeat traffic:  1. **States are sparse.** Align-mode states materialize only at chunk ends, and a quiet prefill produces one deep mid-prompt chunk end — so a sibl…
