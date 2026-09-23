# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-23 13:03 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issues
* **性能优化提议 (#58266)**：指出视觉语言模型（VLM）的多模态预处理受限于 Python GIL，单线程执行且扩大线程池无效，建议引入进程池来优化 `_mm_executor`。
* **新特性请求 (#58263)**：提议将 `VLLM_BATCH_INVARIANT=1` 批次无关模式绑定到 KV-offload 命名空间及 KV-connector 兼容性检查中，以防出现不兼容情况。

### 🔧 Pull Requests
* **硬件与内核优化**：
  * **ROCm/RDNA 优化**：为 RDNA3 (gfx1100) 调优线性注意力配置，性能提升约 15% (#58265)；将 RDNA3/RDNA4 的 `prefix_prefill` 内核 `num_warps` 改为 8 以提升性能 (#58258)。
  * **ROCm MoE 支持**：支持在 gfx942 上使用 AITER 的 Triton MXFP4 MoE 内核运行 MiMo-V2.6 模型 (#58262)。
  * **RISC-V 优化**：针对窄 M 形状的注意力 GEMM 使用 16 列 RVV tile，提升解码和预填充性能 (#58257)。
* **重要 Bug 修复**：
  * **MLA 崩溃修复 (#58260)**：修复 SM90 sparse MLA 后端在单步调度超过 16384 个查询行时引发的 CUDA 非法内存访问崩溃（通过引入分块执行修复）。
  * **投机解码修复 (#58256)**：修复 ROCm 平台上 GLM-5.x MTP 投机解码在首个草稿步骤崩溃的问题。
  * **调度竞态修复 (#58259)**：修复可恢复请求与异步调度交接时的竞态条件问题。
  * **日志修复 (#58261)**：修复因格式字符串与参数数量不匹配导致运行时被吞掉的日志报错。
* **测试与代码质量**：
  * 将容错端到端测试的故障检测超时时间从 45 秒放宽至 75 秒以减少误报 (#58264)。
  * 修复 Zamba2 模型的 mypy 类型检查问题 (#58255)。

### 🚀 Release
* **v0.30.0 版本亮点**：
  * 本版本包含 **762 次提交**，共有 **315 位贡献者**参与（其中 104 位为新加入的贡献者）。
  * 带来了全新模型支持：重点集成了 **DeepSeek-V4.1-Flash**。

---

## 🐛 Issues

### #58266 — [[Performance]: multimodal preprocessing is single-threaded per API server; a wider thread pool does not help (GIL) — process pool for _mm_executor?](https://github.com/vllm-project/vllm/issues/58266)
- **作者**: tboser  **时间**: 2026-09-23 12:51 CST
- **标签**: multi-modality
- **摘要**: ### Proposal to improve performance  **Summary.** For a VLM serving one image per request, the renderer's single-worker multimodal executor (`vllm/renderers/base.py`, `_mm_executor = ThreadPoolExecutor(max_workers=1)`, "must stay single-worker per #38418") is a hard wall in front of the engine, and …

### #58263 — [[Feature]: Bind batch-invariant mode into KV-offload namespaces and KV-connector compatibility checks](https://github.com/vllm-project/vllm/issues/58263)
- **作者**: SeverinVisionary  **时间**: 2026-09-23 12:29 CST
- **摘要**: ### 🚀 The feature, motivation and pitch  **Summary.** `VLLM_BATCH_INVARIANT=1` is not part of any KV-offload namespace or KV-connector compatibility check. An instance in batch-invariant mode can therefore load KV blocks that were produced under a different numerical configuration (non-invariant ker…

## 🔀 Pull Requests

### #58265 — [[Kernel][ROCm]Tuned config for linear attention on RDNA3](https://github.com/vllm-project/vllm/pull/58265)
- **作者**: jundali77  **时间**: 2026-09-23 12:43 CST
- **标签**: rocm
- **摘要**: ## Purpose Add a tuned config to boost the linear attention performance on RDNA3(gfx1100). on kernel side we can see about 15% uplift.  ## Test Plan Test the kernel with tuned config, measure the performance and numerical correctness.   ## Test Result Kernel benchmark results: AMD Radeon PRO W7900 —…

### #58264 — [[CI] Raise fault-detection deadline 45s -> 75s in FT E2E test](https://github.com/vllm-project/vllm/pull/58264)
- **作者**: vllm-agent  **时间**: 2026-09-23 12:32 CST
- **摘要**: ## Problem  `test_injected_fault_retry_recovers_all_ranks` has failed twice this week (dashboard #2028, #2037): the fault was injected and detected, but detection landed just past the 45s deadline on loaded CI hosts.  ## Fix  Raise the deadline from 45s to 75s (slowest fallback is 30s; the margin wa…

### #58262 — [[ROCm][MoE] Support MiMo-V2.6 MXFP4 on gfx942](https://github.com/vllm-project/vllm/pull/58262)
- **作者**: vllmellm  **时间**: 2026-09-23 12:14 CST
- **标签**: rocm
- **摘要**: ## Purpose  MiMo-V2.6 ships MXFP4-quantized MoE weights, and on gfx942 we should be able to use AITER's Triton MXFP4 MoE kernel (`--moe-backend aiter_triton_mxfp4_bf16`). But now using it will failed to load weight:   ``` ValueError: Mxfp4 MoE backend 'AITER_TRITON_MXFP4_BF16' does not support the d…

### #58261 — [[Bugfix][Logging] Fix three logging calls whose message is swallowed at runtime](https://github.com/vllm-project/vllm/pull/58261)
- **作者**: haosenwang1018  **时间**: 2026-09-23 12:14 CST
- **标签**: bug, kv-connector
- **摘要**: ## Purpose  `logging` interpolates with `msg % args` inside `LogRecord.getMessage()`. When the format string and the argument count disagree it raises, the handler prints `--- Logging error ---` plus a traceback to stderr, and **the intended message never appears**. Three call sites in the tree do t…

### #58260 — [[Bugfix][MLA] Fix SM90 sparse MLA IMA on >16k-row steps by chunking plan/run](https://github.com/vllm-project/vllm/pull/58260)
- **作者**: positive666  **时间**: 2026-09-23 12:13 CST
- **标签**: bug, nvidia
- **摘要**: ## Purpose  Fixes a CUDA illegal-memory-access crash whenever a single scheduled step packs more than 16384 query rows on the SM90 sparse MLA backend (the GLM-5.3-Flash default), e.g. `--max-num-batched-tokens 32768` with a 28k-token prefill chunk, or 32k tokens scheduled across concurrent requests.…

### #58259 — [[Bugfix] Fix resumable request + async scheduling handoff race](https://github.com/vllm-project/vllm/pull/58259)
- **作者**: yzong-rh  **时间**: 2026-09-23 12:02 CST
- **标签**: bug, scheduler
- **摘要**: ## Purpose  Fixes a resumable-request handoff race when async scheduling is used. When a streaming turn stops, `_update_request_as_session()` uses `request.num_computed_tokens` as the prefix boundary for the next turn. That value is an optimistic frontier: it can already include input positions from…

### #58258 — [[Kernel][ROCm] change prefix_prefill kenrel num_warps=8 for RDNA GPUs ](https://github.com/vllm-project/vllm/pull/58258)
- **作者**: jundali77  **时间**: 2026-09-23 11:37 CST
- **标签**: rocm
- **摘要**: ## Purpose the default num_warps=4 performs poorly on RDNA3/RDNA4 GPUs, change it back to num_warps=8 on the gfx1100 and gfx1201 to get better performance.  ## Test Plan Test the performance and numerical correctness of different kernel config.   ## Test Result Kernel bench results: AMD Radeon PRO W…

### #58257 — [[Performance][Hardware][RISC-V] Use a 16-column RVV tile for narrow-M attention GEMMs](https://github.com/vllm-project/vllm/pull/58257)
- **作者**: 6eanut  **时间**: 2026-09-23 11:36 CST
- **标签**: cpu
- **摘要**: ## Purpose  The RISC-V RVV CPU attention kernel currently drives every output tile with an 8-column (`LMUL_256`) B-row load and one accumulator per row. For the narrow-M shapes that dominate decode and GQA (`m_size <= 4`), that shape has too little independent work to keep the core busy and re-issue…

### #58256 — [[ROCm][Bugfix][Spec Decode] Add compact_topk_indices to the ROCm DSA MTP drafter](https://github.com/vllm-project/vllm/pull/58256)
- **作者**: ZhengGong-amd  **时间**: 2026-09-23 11:26 CST
- **标签**: bug, rocm, speculative-decoding, deepseek
- **摘要**: ## Purpose  On ROCm, MTP speculative decoding for GLM-5.x crashes on the first draft step on current `main`, with the reduction flag off and nothing unusual in the config:  ``` File "vllm/v1/spec_decode/llm_base_proposer.py", line 613, in propose     self.model.model.compact_topk_indices(token_indic…

### #58255 — [[Mypy] Fix mypy typing for Zamba2 models](https://github.com/vllm-project/vllm/pull/58255)
- **作者**: ashraf-bhuiyan  **时间**: 2026-09-23 11:16 CST
- **标签**: speculative-decoding, multi-modality, qwen, mistral, dflash
- **摘要**: ## Purpose  **Depends on W-model PR [#58254](https://github.com/vllm-project/vllm/pull/58254), which builds on V-model PR [#58251](https://github.com/vllm-project/vllm/pull/58251). Merge order: Q → U → V → W → Z.** This branch starts at W's `cb71508394`. Its diff against upstream `main` includes the…

## 🚀 Releases

### [v0.30.0](https://github.com/vllm-project/vllm/releases/tag/v0.30.0)
- **作者**: khluu  **时间**: 2026-09-22 13:20 CST
- **摘要**: # v0.30.0  ## Highlights  This release features 762 commits from 315 contributors (104 new)!  * **New models**: DeepSeek-V4.1-Flash (#56214, #56228, #56208) with the whole KV stored in MXFP8 through the FlashMLA V4.1 record on SM100 (#56893), DeepGEMM Mega-mHC (#56962), and async Engram prefetch wit…
