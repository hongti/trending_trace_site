# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-26 13:20 CST

## AI 总结

# vLLM 仓库近期动态摘要

## 📋 PR 活动（共 10 项）

### 🐛 Bug 修复（5 项）

- **#58822** — 修复 DeepSeek V4.1 Flash 在启用流式工具调用时无法流式返回客户端的问题
- **#58821** — 修复投机解码中 CUDA 图的动态查询长度问题，避免 draft decode 图形构建异常
- **#58816** — 修复 Anthropic SDK 1.x 兼容性问题（移除了 `temperature`/`top_p`/`top_k` 关键字参数）
- **#58814** — 修复 Kimi-K3 DSpark 中 KV 缓存重绑定后指针未刷新导致的 GPU 内存故障和静默内存损坏
- **#58813** — 修复 Qwen3.5 MTP 头在密集检查点（bf16 分片）下加载时崩溃的问题，改为以非量化方式构建

### ⚡ 性能优化（3 项）

- **#58820** — 新增 Fast MoE 三模态加载引擎，解决 PCIe Gen5/Gen6 系统上 MoE 模型检查点加载瓶颈，支持冷启动 NVMe 流式加载与 3D 批量 DMA
- **#58819** — ROCm 平台上将 DeepSeek V4/V4.1 MoE 默认从 a8w4 改为 a4w4（FP4 激活），提升 gfx950 性能
- **#58818** — 跨 KV 缓存组共享 FlashMLA 解码计划，避免重复 tile-scheduler 初始化开销

### 🔧 其他（2 项）

- **#58815** — 优化 Qwen3.8-Flash-Next 的 47.7 GiB PLE n-gram 表内存占用，改为从加载器映射的检查点分片按需提供
- **#58817** — XPU CI 测试尝试 b50 版本

## 📰 Release / Issue

本次数据中未包含 Release 或 Issue 活动。

## 📌 总结

近期 vLLM 重点围绕 **DeepSeek V4/V4.1** 生态进行多项性能与稳定性优化，同时覆盖 **Qwen3.5/3.8、Kimi-K3** 等模型的 bug 修复与加载性能改进。硬件适配方面涵盖 **ROCm（gfx950）和 XPU** 平台。

---

## 🔀 Pull Requests

### #58822 — [[Bugfix] DeepSeek V4.1 Flash failed to stream back to the client](https://github.com/vllm-project/vllm/pull/58822)
- **作者**: willweimike  **时间**: 2026-09-26 13:02 CST
- **标签**: bug, tool-calling, deepseek, DSv4.1
- **摘要**: … tool-calling with streaming enabled     ## Purpose Fixed #58640   ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] T…

### #58821 — [[Bugfix][Spec Decode] Avoid dynamic query lengths for draft decode graphs](https://github.com/vllm-project/vllm/pull/58821)
- **作者**: CleverPhysician  **时间**: 2026-09-26 12:43 CST
- **标签**: bug, nvidia, mrv2
- **摘要**: ## Motivation  When dynamic speculative decoding is enabled, `CudaGraphManager` derives target-model query lengths from the configured maximum speculative token count. The autoregressive draft decode manager is constructed with `decode_query_len=1`, so applying that calculation produces non-positive…

### #58820 — [[Perf][Model Loader] Fast MoE Tri-Modal Ingestion Engine: Cold NVMe Streaming and Layer-Bounded 3D Bulk DMA](https://github.com/vllm-project/vllm/pull/58820)
- **作者**: eppaneamd  **时间**: 2026-09-26 12:43 CST
- **摘要**: ## Purpose  Addresses checkpoint loading bottlenecks on Mixture-of-Experts (MoE) models under tensor parallelism on high-bandwidth PCIe Gen5/Gen6 systems:  1. **Cold start disk queue serialization**:    - *Problem*: On cold starts, 4–8 TP workers concurrently read shards from disk, causing parallel …

### #58819 — [[ROCm][Perf] Default DeepSeek V4/V4.1 MoE to a4w4 (FP4 activations) on AITER](https://github.com/vllm-project/vllm/pull/58819)
- **作者**: Fangzhou-Ai  **时间**: 2026-09-26 12:30 CST
- **标签**: rocm, deepseek, DSv4
- **摘要**: ## Purpose  DeepSeek V4/V4.1's MXFP4 MoE currently runs AITER's a8w4 (FP8-activation) kernel on gfx950. AITER's own `q_dtype_a` activation-dtype heuristic in `fused_moe()` would pick a4w4 (FP4 activations) for this model's Silu+`GateMode` combination, but vLLM's MXFP4 weight shuffle uses `GateMode.I…

### #58818 — [[Perf][DSv4.1] Share the FlashMLA decode plan across KV cache groups](https://github.com/vllm-project/vllm/pull/58818)
- **作者**: ShuoleiWang  **时间**: 2026-09-26 12:24 CST
- **标签**: mrv2, DSv4.1
- **摘要**: ## Purpose  `DeepseekSparseSWAMetadataBuilder.build()` creates one empty `FlashMLASchedMeta` per DeepSeek-V4 layer type. The first `flash_mla_with_kvcache` call of each type runs FlashMLA's tile-scheduler planner, and the later layers of that type reuse the plan (the comment calls it "shared across …

### #58817 — [[XPU][CI] try b50](https://github.com/vllm-project/vllm/pull/58817)
- **作者**: zxd1997066  **时间**: 2026-09-26 12:17 CST
- **标签**: intel-gpu, ci/build
- **摘要**: ## Purpose  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such as providing test command. - [ ] The …

### #58816 — [[Bug][Entrypoint] Fix Anthropic messages test for Anthropic SDK 1.x](https://github.com/vllm-project/vllm/pull/58816)
- **作者**: liwenjie200543  **时间**: 2026-09-26 11:50 CST
- **标签**: bug
- **摘要**: ## What `test_anthropic_streaming_cache_usage` passed `temperature=0.0` as a keyword to `AsyncMessages.create()`. Anthropic SDK 1.0 removed `temperature` / `top_p` / `top_k` from `messages.create()`; the documented replacement is `extra_body`. This moves the value accordingly.  ## Why vLLM pins `ant…

### #58815 — [[Qwen4Exp] Serve PLE rows from the loader's mapped checkpoint shards](https://github.com/vllm-project/vllm/pull/58815)
- **作者**: Trosfy  **时间**: 2026-09-26 10:33 CST
- **标签**: documentation, qwen
- **摘要**: ## Purpose  Qwen3.8-Flash-Next carries a 47.7 GiB FP8 PLE n-gram table. The storage backends from #54371 keep the whole table in GPU memory or copy it into pinned host memory, so about 48 GiB stays allocated for the life of the server, even though a generation reads only a small part of it. When wei…

### #58814 — [[Bugfix][Kimi-K3] Refresh DSpark context KV cache pointers after the KV cache is re-bound](https://github.com/vllm-project/vllm/pull/58814)
- **作者**: okorzh-amd  **时间**: 2026-09-26 10:12 CST
- **标签**: bug, ready, dflash, kimi, k3
- **摘要**: ##Purpose  Fixes a GPU memory fault and silent memory corruption in the Kimi-K3 DSpark draft, introduced by #57632.  ``` 21:05:36  JIT kernel warmup finished            (all 8 ranks)           Warning: Queue error - HSA_STATUS_ERROR_MEMORY_FAULT           Memory access fault by GPU node-14 ... on ad…

### #58813 — [[Bugfix][Model] Build the Qwen3.5 MTP head unquantized when the checkpoint ships it dense](https://github.com/vllm-project/vllm/pull/58813)
- **作者**: he-yufeng  **时间**: 2026-09-26 09:37 CST
- **标签**: bug, qwen, quantization
- **摘要**: ## Purpose  Fixes #58807.  A compressed-tensors pack-quantized checkpoint that ships its MTP head dense (in a separate bf16 shard, plain `mtp.*.weight` names) crashes at weight load: `Qwen3_5MultiTokenPredictor` builds its Linears from the target's quant config, so the loader expects packed names (`…
