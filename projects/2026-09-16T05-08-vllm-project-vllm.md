# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-16 13:08 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 最近动态的中文摘要：

### 📋 Issue 动态
1. **测试忽略 (#57114)**：仅用于测试权限，无实际内容。
2. **[RFC] 混合模型的缓存驱逐与分段重计算机制 (#57111)**：提出了针对混合模型（如交替使用全注意力层与 GDN/Mamba 循环层的 Qwen3.5）的新优化方案。旨在实现“检查点感知”的缓存驱逐和分段重计算，以更高效地管理不同的缓存块和循环状态。

### 🔨 Pull Request 动态
**性能优化与新特性**
* **[Frontend] 暴露本地 DP size (#57116)**：在 gRPC 控制元数据中暴露本地数据并行（DP）大小，为 Dynamo 带来与进程内 vLLM 集成相似的混合负载均衡支持。
* **[Kernel][PCP] PCP KV 缓存直写优化 (#57115)**：将复制的 PCP KV 更新直接发布到 ExtensibleKVCache 自有的存储中，优化内核写入路径。
* **[DSA] 新增 FlashInfer top-k 后端 (#57109)**：引入 FlashInfer 的 `gvr_2` 作为 DSA 稀疏索引解码阶段的 top-k 后端，提升解码性能。
* **[Spec Decode] 减少 Triton 重复编译 (#57107)**：修复了接受度估计器中因 batch size 变化导致 Triton 频繁重新编译的问题，提升推测解码性能。
* **[Qwen3.8-Flash-Next] 修复内存碎片 (#57105)**：为 QSA 索引器预分配最坏情况下的 logits 工作空间，避免运行时的内存碎片化。

**重要 Bug 修复**
* **[Spec Decode] 修复 DSpark 前缀缓存回归 (#57110)**：解决 DSpark 因借用 EAGLE 隐藏状态调度路径而导致的前缀缓存 block drop 回归问题，将两者逻辑解耦。
* **[ModelLoader] 量化层权重追踪修复 (#57108)**：修复 `DefaultModelLoader.track_weights_loading` 在量化模型中未正确检查量化层而导致拒绝加载的问题。
* **[KV Connector] 修复 KV 压力下死锁 (#57104)**：修复了在 KV 缓存压力下，KVConnector 异步加载与 MTP（多 token 预测）结合时引发死锁的严重问题。

**CI 与测试改进**
* **[CI] LM Eval 测试分片加速 (#57113)**：将 `TurboQuant KV Cache` 的 LM 评估测试从 1 个任务拆分为 4 个并行副本（按 `kv-cache-dtype` 区分），大幅缩短 CI 耗时。
* **[ROCm][CI] 修复 TheRock 镜像测试失败 (#57112)**：修复了在 ROCm 的 TheRock 镜像中 MLA RoPE 融合内核测试失败的问题。

### 🚀 Release 动态
*本批次动态中未包含版本发布信息。*

---

## 🐛 Issues

### #57114 — [test issue - please ignore](https://github.com/vllm-project/vllm/issues/57114)
- **作者**: Thangnguyenvn98  **时间**: 2026-09-16 12:41 CST
- **摘要**: testing permissions; will close

### #57111 — [[RFC]: Checkpoint-aware cache eviction and segmented recomputation for hybrid models](https://github.com/vllm-project/vllm/issues/57111)
- **作者**: warriorsniu  **时间**: 2026-09-16 11:45 CST
- **标签**: RFC
- **摘要**: ### Motivation.   #### Background  Hybrid models such as Qwen3.5 interleave Full Attention (FA) layers with recurrent GDN/Mamba layers. vLLM stores FA KV blocks and recurrent states in different cache groups, while a prefix is reusable only when all required groups can resume from compatible token b…

## 🔀 Pull Requests

### #57116 — [[Frontend] Expose local DP size in gRPC Control metadata](https://github.com/vllm-project/vllm/pull/57116)
- **作者**: alec-flowers  **时间**: 2026-09-16 13:00 CST
- **标签**: rust
- **摘要**: ## Purpose  Dynamo's in-process vLLM integration already supports [hybrid load balancing](https://docs.vllm.ai/en/latest/serving/data_parallel_deployment/#hybrid-load-balancing). We are bringing the same multi-node DP topology to Dynamo's native gRPC sidecar integration, with a frontend and sidecar …

### #57115 — [[Kernel][PCP] Publish direct-final KV into ExtensibleKVCache storage](https://github.com/vllm-project/vllm/pull/57115)
- **作者**: foraxe  **时间**: 2026-09-16 12:51 CST
- **标签**: frontend, deepseek, cpu, mrv2, kv-cache-manager
- **摘要**: ## Summary  Publish replicated PCP KV updates directly into ExtensibleKVCache-owned storage. The existing fused Norm/RoPE/cache writer writes each rank's updates into every PCP peer's final cache, then a system-scope publication barrier makes the local replica ready for the unchanged Indexer and spa…

### #57113 — [[CI] Shard LM Eval TurboQuant KV Cache 1->4 by kv-cache-dtype](https://github.com/vllm-project/vllm/pull/57113)
- **作者**: Thangnguyenvn98  **时间**: 2026-09-16 12:40 CST
- **标签**: ci/build, quantization
- **摘要**: ## What  Shard `(H200 MIG 18GB) LM Eval TurboQuant KV Cache` from one job into four parallel replicas, one kv-cache-dtype config per shard, using the same per-shard config-list idiom as the MoE Refactor and Humming jobs in this file. Recent main runs sit at 39–51m of pytest (42–53m wall) against the…

### #57112 — [[ROCm][CI] Fix MLA RoPE fused-kernel tests for TheRock image](https://github.com/vllm-project/vllm/pull/57112)
- **作者**: mawong-amd  **时间**: 2026-09-16 12:27 CST
- **标签**: rocm
- **摘要**: <!-- markdownlint-disable --> The `Core Operations Kernels` test group fails on TheRock image (see [here](https://buildkite.com/vllm/amd-ci/builds/12914/list?sid=01a09eed-c3fc-4155-8ae7-ad55792cdc2c&open=false)). This has been root-caused to the Triton 3.8 upgrade. Specifically, in the computation o…

### #57110 — [[Bugfix][Spec Decode] Separate DSpark from EAGLE prefix-cache block drop](https://github.com/vllm-project/vllm/pull/57110)
- **作者**: ZhangHandi  **时间**: 2026-09-16 11:37 CST
- **标签**: bug, dflash, kv-cache-manager
- **摘要**: ## Purpose  Fix a DSpark prefix-cache regression caused by sharing EAGLE's hidden-state scheduling path.  DSpark intentionally returns true from `use_eagle()` because it consumes target hidden states. Before this change, that also made `use_eagle_block_drop()` return true. The scheduler and KV-cache…

### #57109 — [[Perf][DSA] Add FlashInfer gvr_2 as a sparse-indexer decode top-k backend](https://github.com/vllm-project/vllm/pull/57109)
- **作者**: JaredforReal  **时间**: 2026-09-16 11:35 CST
- **标签**: deepseek, nvidia
- **摘要**: ## Purpose  Add FlashInfer's self-sampling GVR V2 top-k (`top_k_varlen(backend="gvr_2")`, flashinfer-ai/flashinfer#4811 + flashinfer-ai/flashinfer#4986) as a decode top-k backend of the DSA sparse indexer.  - New `kernel_config.sparse_indexer_topk_backend` value `flashinfer_gvr2`. The call is hint-f…

### #57108 — [[Bugfix][ModelLoader] Make enable_weights_track check quantized layers](https://github.com/vllm-project/vllm/pull/57108)
- **作者**: siliangchen-amd  **时间**: 2026-09-16 11:29 CST
- **标签**: bug, quantization
- **摘要**: ## Purpose  `DefaultModelLoader.track_weights_loading` refuses a load that left a parameter uninitialised. It is off by default for a quantized model, and the documented way to turn it on is:  ``` --model-loader-extra-config '{"enable_weights_track": true}' ```  **That opt-in currently checks nothin…

### #57107 — [[Perf][Spec Decode] Avoid triton recompiles in the acceptance estimator](https://github.com/vllm-project/vllm/pull/57107)
- **作者**: TheEpicDolphin  **时间**: 2026-09-16 11:19 CST
- **标签**: speculative-decoding, ready, mrv2
- **摘要**: ## Context Triton keys its compilation cache on whether each int scalar is 1 and whether it is 16-divisible, so the number of tokens/reqs args re-key as the batch size changes at runtime, causing stalls.  ## Summary Fixed by disabling specialization on `num_reqs`/`num_tokens` for the acceptance esti…

### #57105 — [[Qwen3.8-Flash-Next] Avoid memory fragmentation in QSA indexer logits workspace](https://github.com/vllm-project/vllm/pull/57105)
- **作者**: gau-nernst  **时间**: 2026-09-16 11:09 CST
- **标签**: qwen
- **摘要**: ## Purpose  Fixes #56457  Now we always reserve the worst-case logits workspace so that subsequent calls won't hit a new allocation.  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fi…

### #57104 — [[BugFix][KV Connector] Fix Deadlock with KVConnector AsyncLoad + MTP under KV Pressure](https://github.com/vllm-project/vllm/pull/57104)
- **作者**: robertgshaw2-redhat  **时间**: 2026-09-16 11:04 CST
- **标签**: bug, kv-connector, scheduler
- **摘要**: ## Purpose An async KV load is allocated without lookahead slots (see `limit_lookahead_tokens`), and `_request_remaining_blocks()` -- which backs the `_inflight_prefill_reserved_blocks()` admission gate -- does not count them either. Promotion then pads the request to `1 + num_spec_tokens` and asks …
