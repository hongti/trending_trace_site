# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-02 14:39 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文简洁摘要：

### 🐛 Issue 动态
* **CI 失败报告 (#65081)**：`main` 分支上的 `pre-commit mypy` 检查失败。原因是 `vllm/models/muse_spark/amd/model.py` 文件中 `process_weights_after_loading` 函数被重复定义。

### 🔧 PR 动态
近期的 PR 主要集中在 Bug 修复、性能优化（特别是针对 ROCm/AMD 硬件）以及基准测试工具的改进：

* **重要 Bug 修复**：
  * **Ling3 模型精度修复 (#65088)**：修正了 Ling RL 模型中 SwiGLU 门限截断顺序错误的问题（原先在 SiLU 之前应用了门限限制）。
  * **Muse Spark CI 修复 (#65080)**：合并了重复定义的 `process_weights_after_loading` 钩子，直接修复了上述 #65081 Issue。
  * **Encoder Cache 冗余传输修复 (#65079)**：修复了 ECCPU 单独 CPU 层被驱逐时，触发冗余远程传输的问题，现在会优先复用消费者 runner 缓存中的命中数据。
  * **基准测试修复 (#65085, #65083)**：修复了 `vllm bench forward` 默认 `max_num_batched_tokens` 过大影响内存分析的问题（改为根据用例推导）；修复了 `CustomDataset` 将纯数字字符串 Prompt（如 "00123"）错误推断为整数类型的问题。

* **核心性能优化**：
  * **前缀缓存查找优化 (#65086)**：通过“跨步跳跃”优化了滑动窗口前缀缓存的查找逻辑，解决了代码中遗留的 TODO。
  * **KV Offload 向量化 (#65082)**：向量化了 GPU 与 CPU 之间 KV Cache 传输的描述符填充过程，提升跨层传输效率。

* **ROCm (AMD) 性能优化**：
  * **MiniMax-M3.1 解码加速 (#65087)**：在 gfx950 上为 NVFP4 indexer 引入四波解码 top-k 策略并精确合并结果，提升解码性能。
  * **Qwen3.8 单次解码支持 (#65078)**：在 128-CU gfx950 (MI350P) 上为 Qwen3.8-27B 实现了密集单次解码。

* **多模态重构 (#65084)**：重构了 encoder CUDA graph，以支持共享图缓冲区的重用。

### 🚀 Release 动态
* 本次提供的动态列表中**无新版本发布**信息。

---

## 🐛 Issues

### #65081 — [[CI Failure]: pre-commit mypy on main - process_weights_after_loading redefined in vllm/models/muse_spark/amd/model.py](https://github.com/vllm-project/vllm/issues/65081)
- **作者**: Alex-Wengg  **时间**: 2026-11-02 13:48 CST
- **标签**: rocm, ci-failure
- **摘要**: ### Name of failing test  `pre-commit` workflow, hook `mypy` ("Run mypy for the oldest supported Python version"), on `main`.  ### Basic information  - [ ] Flaky test - [x] Can reproduce locally - [ ] Caused by external libraries (e.g. bug in `transformers`)  ### 🧪 Describe the failing test  Every p…

## 🔀 Pull Requests

### #65088 — [[Bugfix][Model] Fix Ling3 SwiGLU clamping](https://github.com/vllm-project/vllm/pull/65088)
- **作者**: zexplorerhj  **时间**: 2026-11-02 14:38 CST
- **标签**: bug
- **摘要**: ## Overview  While investigating an accuracy issue during training–inference alignment for Ling RL, we found that routed experts applied the SwiGLU gate limit before SiLU, whereas the training/reference implementation applies it after SiLU. This mismatch can affect RL accuracy through inconsistent t…

### #65087 — [[ROCm][Perf][MiniMax-M3.1] Four-wave decode top-k for the NVFP4 indexer](https://github.com/vllm-project/vllm/pull/65087)
- **作者**: chao-xu-spec  **时间**: 2026-11-02 14:23 CST
- **标签**: rocm, minimax
- **摘要**: ## Overview  On gfx950, the MiniMax-M3.1 NVFP4 indexer picks the 16 best blocks of every decode row with one wave per row. This PR gives each decode row four waves and merges their top-16 lists exactly, so the ordered block ids are unchanged. No other kernel, flag or call site changes.  ## Claims  -…

### #65086 — [[Perf][Core] Optimize sliding-window prefix cache lookup with stride jumping](https://github.com/vllm-project/vllm/pull/65086)
- **作者**: frieddeli  **时间**: 2026-11-02 14:15 CST
- **标签**: kv-cache-manager
- **摘要**: # [Perf][Core] Optimize sliding-window prefix cache lookup with verified stride jumping  ## Overview  Resolves the open `# TODO` at line 1002 of `vllm/v1/core/single_type_kv_cache_manager.py` by replacing the $O(N)$ backwards linear block scan with a stride jump search ($O(N/W + W)$). The algorithm …

### #65085 — [[Bugfix][Benchmark] Derive forward benchmark batch budget from cases](https://github.com/vllm-project/vllm/pull/65085)
- **作者**: jianzs  **时间**: 2026-11-02 14:15 CST
- **标签**: bug, documentation, performance
- **摘要**: ## Overview  `vllm bench forward` defaulted `max_num_batched_tokens` to at least 8192 even when no case used a prefix-cache hit. The engine uses that value as the memory-profiling batch, so small decode/verify-only case sets profiled a much larger batch than they run, which can make KV-cache plannin…

### #65084 — [[Multimodal] Refactor encoder cuda graph to support share graph buffer reuse](https://github.com/vllm-project/vllm/pull/65084)
- **作者**: Isotr0py  **时间**: 2026-11-02 14:02 CST
- **标签**: documentation, nvidia
- **摘要**: ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this PR.  - [ ] **Design Fit:** Minimizes…

### #65083 — [[Bugfix][Benchmark] Keep numeric-string prompts as strings in CustomDataset](https://github.com/vllm-project/vllm/pull/65083)
- **作者**: Vegetog  **时间**: 2026-11-02 13:55 CST
- **标签**: bug, performance
- **摘要**: ## Overview  Fixes #65071. `CustomDataset.load_data()` lets `pd.read_json` infer the `prompt` column's dtype, so a JSONL file whose prompts are all numeric strings (`"00123"`, `"00456"`) is loaded as integers. Leading zeros are lost and `sample()` then fails in the tokenizer.  ## Claims  - JSON stri…

### #65082 — [[Perf][KV Offload] Vectorize direct-layout descriptor fill across layers](https://github.com/vllm-project/vllm/pull/65082)
- **作者**: AdvayMonga  **时间**: 2026-11-02 13:50 CST
- **标签**: nixl
- **摘要**: ## Overview  `_fill_direct_ops` builds the (src, dst, size) copy descriptors for every GPU<->CPU KV transfer in the `OffloadingConnector`. It loops over the layer refs and calls `compute_sub_block_ptrs` twice per ref, so one transfer costs 32 calls on an 8B model and 80 on 70B TP8, even for a single…

### #65080 — [[Bugfix][ROCm] Muse Spark: merge the two process_weights_after_loading hooks](https://github.com/vllm-project/vllm/pull/65080)
- **作者**: Alex-Wengg  **时间**: 2026-11-02 13:47 CST
- **标签**: bug, rocm, ready, verified
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Fixes #65081, the pre-commit failure on `main` since #63440 merged:  ``` vllm/models/muse_spark/amd/model.py:1214: error: Name "process_weights_after_loading" already defined on line 1207  [no-redef] ```  #64357 and #63440 each added `process_weights_after…

### #65079 — [[Bugfix][EC] Reuse runner cache hits before remote admission](https://github.com/vllm-project/vllm/pull/65079)
- **作者**: rebel-eunji  **时间**: 2026-11-02 13:43 CST
- **标签**: bug, documentation, cpu, scheduler, mooncake, nixl
- **摘要**: ## Overview  Avoid fetching encoder outputs that are already in the consumer's runner cache. Previously, eviction from ECCPU's separate CPU tier could trigger a redundant transfer and fail a metadata-only request if the producer was unavailable.  Related: #55639. No duplicate open PR was found; this…

### #65078 — [[ROCm][Perf][Qwen3.8] Dense mono decode on 128-CU gfx950 (MI350P)](https://github.com/vllm-project/vllm/pull/65078)
- **作者**: jhartika-amd  **时间**: 2026-11-02 13:39 CST
- **标签**: new-model, rocm, needs-rebase, ci/build, qwen
- **摘要**: ## Overview  Dense mono decode for Qwen3.8-27B on 128-CU gfx950 (MI350P). **Stacked on #61573**, which adds the dense mono decode for MI355X; the first two commits are #61573's and the rest are this PR's. Review the later commits. Draft until #61573 lands.  ## Claims  - The K1 (GDN mixer) and K2 (MX…
