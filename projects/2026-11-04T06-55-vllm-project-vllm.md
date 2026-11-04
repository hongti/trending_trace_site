# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-04 14:55 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要。本次更新内容仅包含 Pull Request (PR) 活动记录，未涉及 Issue 和 Release。

### ⚙️ Pull Request (PR) 动态归纳

近期合并的 PR 主要围绕**性能优化、硬件适配（ROCm/XPU/Blackwell）、Bug 修复以及新功能扩展**展开：

#### 1. 性能优化
*   **QSA 预填充分片 (#65606)**：针对 Qwen4Exp 模型，在 SM90 架构（TP4）上将大型 QSA 预填充查询行进行分片处理，通过复用现有的 scorer 和 Top-K 提升效率。
*   **调度器成本限制 (#65604)**：在缓存压力导致大量相同前缀被驱逐时，限制了调度器的 CPU 工作量，有效保留了缓存代表和块分配，提升了重复前缀的处理性能。
*   **ROCm FP8 稀疏注意力优化 (#65599)**：在 gfx942 (MI300X/MI325X) 上，当 KV 缓存为 FP8 时，使用单 warp 启动 Step-5 CSA 预填充稀疏注意力内核。

#### 2. Bug 修复
*   **MiniMax-M3 预热死锁 (#65602)**：修复了在流水线并行 (PP>1) 场景下，MiniMax-M3 混合模型架构在注意力预热时发生的死锁问题。
*   **FlashInfer 内存访问非法 (#65598)**：修复了在 Blackwell (SM100) 架构上使用 CUDA graph 重放独立草稿模型预填充时，引发的非法内存访问/段错误问题，通过将 TRTLLM 预填充查询偏移量保留在持久缓冲区中解决。

#### 3. 新功能与特性
*   **多模态目标边界框生成 (#65605)**：在 `/v1/systemone` 前端为多模态模型新增了生成式边界框回答功能，支持在图像中定位文本描述的目标。
*   **Logprobs 重放支持 (#65603)**：将生成 token 的 logprobs 支持移植到重新设计的 AuxOutput 连接器上，纯 logprobs 请求现在使用可变大小的工件存储。
*   **KV 事件块大小暴露 (#65601)**：在 `BlockStored` 事件中添加了可选的 token 跨度 `chunk_size`，帮助使用者区分可独立检索的 GPU 块和包含多个已发布块头部的卸载块。
*   **ROCm UMBP 等待请求预取 (#65597)**：为 `UMBPStoreConnector` 引入了可选的等待请求预取功能，利用 MORI 的 `batch_prefetch` API 填充本地 DRAM 层。

#### 4. 硬件/测试兼容性
*   **XPU 流捕获检查修复 (#65600)**：修补了在纯 CPU 构建的 PyTorch 上运行 SM90 sparse MLA 计划测试时，因调用 `torch.cuda.is_current_stream_capturing()` 而引发的 `RuntimeError`。

---
*注：近期暂无 Issue 讨论与版本发布的记录。*

---

## 🔀 Pull Requests

### #65606 — [[Perf][Model] Shard large QSA prefill query rows on SM90 TP4](https://github.com/vllm-project/vllm/pull/65606)
- **作者**: positive666  **时间**: 2026-11-04 14:54 CST
- **标签**: documentation, performance, ci/build, qwen
- **摘要**: ## Overview  Qwen4Exp QSA repeats prefill scoring and Top-K on every TP rank. Assign complete prefill query rows to ranks and gather the final `int32` indices, reusing the existing scorer and Top-K with replicated K/cache.  This applies the row-ownership principle from #54394 to QSA's separate model…

### #65605 — [[Frontend] Add generative box answers for multimodal models on /v1/systemone](https://github.com/vllm-project/vllm/pull/65605)
- **作者**: Mlight7  **时间**: 2026-11-04 14:53 CST
- **标签**: documentation, frontend
- **摘要**: ## Overview  Add generated bounding-box answers to `/v1/systemone` for text-described targets in images. The implementation reuses the served multimodal model, its chat renderer, and the existing generation engine to return normalized box coordinates.  Related: [#62247](https://github.com/vllm-proje…

### #65604 — [[Perf][Core] Bound FIFO eviction cost for duplicate prefixes](https://github.com/vllm-project/vllm/pull/65604)
- **作者**: pratikkorat26  **时间**: 2026-11-04 14:49 CST
- **标签**: documentation, scheduler, kv-cache-manager
- **摘要**: ## Overview  Bound scheduler CPU work when cache pressure evicts many physical copies of the same prefix. Preserve cache representatives, block assignments, and generated outputs while replacing repeated scans of deleted dictionary slots in large duplicate sets.  ## Claims  - Real `KVCacheManager` i…

### #65603 — [[Feature][AuxOutput] Support logprobs replay on the redesigned connector](https://github.com/vllm-project/vllm/pull/65603)
- **作者**: floatlibai  **时间**: 2026-11-04 14:44 CST
- **标签**: documentation, mrv2, scheduler, kv-cache-manager
- **摘要**: ## Overview  Port generated-token logprobs support to the redesigned AuxOutput connector on `rl/aux-output`. Logprobs-only requests use a variable-size artifact store while routed-expert and indexer outputs retain their existing data paths.  ## Claims  - Retains generated-token logprobs by KV block …

### #65602 — [[Bugfix] Fix MiniMax-M3 attention warmup deadlock under PP>1 (#60791)](https://github.com/vllm-project/vllm/pull/65602)
- **作者**: bandham-manikanta  **时间**: 2026-11-04 14:13 CST
- **标签**: bug, mrv2, minimax
- **摘要**: Follow-up to #64338 and resolves Item 8 in #60791.  ### Root Cause & Context MiniMax-M3 is a hybrid model architecture: early layers (layers 0–2) use dense attention + standard MLPs, while subsequent layers (layers 3+) use MiniMax-M3 Sparse Attention (MSA) + MoE.  When partitioned across pipeline st…

### #65601 — [[KV Events] Expose chunk size in BlockStored events](https://github.com/vllm-project/vllm/pull/65601)
- **作者**: Change72  **时间**: 2026-11-04 13:47 CST
- **标签**: documentation, kv-connector, kv-cache-manager, nixl
- **摘要**: ## Overview  Add an optional token-span `chunk_size` to `BlockStored` so consumers can distinguish an independently retrievable GPU block from one offloaded chunk containing multiple advertised block hashes. This is a small, independently reviewable step toward #52103.  ## Claims  - Native GPU event…

### #65600 — [[XPU][UT] Patch stream-capture check in SM90 sparse MLA plan test](https://github.com/vllm-project/vllm/pull/65600)
- **作者**: ZhouXiaoya25  **时间**: 2026-11-04 13:43 CST
- **标签**: intel-gpu, nvidia
- **摘要**: ## Overview  `test_plan_uses_state_params` calls `_SM90State.plan()`, which calls `torch.cuda.is_current_stream_capturing()`. On CPU-only torch builds this raises `RuntimeError: Tried to instantiate dummy base class _cuda_isCurrentStreamCapturing`. Patch it to return False in the test, as other test…

### #65599 — [[ROCm][Step5] Use one warp for FP8 CSA prefill sparse attention on gfx942](https://github.com/vllm-project/vllm/pull/65599)
- **作者**: akii96  **时间**: 2026-11-04 13:35 CST
- **标签**: rocm
- **摘要**: ## Overview  Launch the Step-5 CSA prefill sparse attention kernel (`sparse_attention_paged`) with one warp when the KV cache is FP8 on gfx942 (MI300X/MI325X). Follow-up to #61837.  ## Claims  - FP8-KV prefill sparse attention is 1.6x faster on MI325X, with bit-identical output. - Step-5-Preview ser…

### #65598 — [[Bugfix][Attention] Keep FlashInfer TRTLLM prefill query offsets in a persistent buffer](https://github.com/vllm-project/vllm/pull/65598)
- **作者**: Pb314314  **时间**: 2026-11-04 13:23 CST
- **标签**: bug, nvidia
- **摘要**: ## Purpose  Fix an illegal memory access / segfault during CUDA graph replay of the standalone draft-model prefill on Blackwell (SM100), reported in https://github.com/vllm-project/vllm/pull/60989#issuecomment-6620438731.  Since #60989, the standalone draft model (`method: "draft_model"`) captures F…

### #65597 — [[ROCm][UMBP] Add waiting-request prefetch to UMBPStoreConnector](https://github.com/vllm-project/vllm/pull/65597)
- **作者**: BadrBasowid  **时间**: 2026-11-04 13:19 CST
- **标签**: documentation, rocm, kv-connector
- **摘要**: ## Overview  Add opt-in UMBP prefetch for waiting requests through `prefetch_waiting_requests`. Uses MORI's [`batch_prefetch`](https://github.com/ROCm/mori/pull/850) API to populate the local DRAM tier before normal restoration.  ## Claims  - Supports embedded and standalone UMBP modes. - Prefetch r…
