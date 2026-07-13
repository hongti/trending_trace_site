# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-12 09:06 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库最近动态的中文简洁摘要：

### 📌 Issue 概要
* **[RFC] 为 Ampere 架构引入 fp8 KV cache** (#48374)：提议在 `TRITON_MLA_SPARSE` 路径中通过软件反量化支持 fp8 KV cache。目前该路径仅支持 bf16，导致 24GB 显存成为瓶颈，作者已实现 sm_80/sm_86 的工作路径。

### 🛠 PR 概要
**新特性与前端**
* **新增离线前缀缓存分析器** (#48369)：引入 `vllm analyze-prefix-cache` CLI 工具，无需 GPU 即可预估请求数据集对前缀缓存的友好度，优化部署决策。

**核心 Bug 修复**
* **修复混合模型 Mamba 状态损坏** (#48375)：修复了混合模型（如 Qwen3.6）在同时启用 MTP/EAGLE 投机解码与前缀缓存时，MambaManager 循环状态被破坏的问题。
* **修复 MoE 专家并行越界读取** (#48371)：修复了在专家并行（EP）下，`topk_ids` 包含 `-1` 哨兵值时导致 `expert_map` 越界读取的 OOB 错误。
* **修复 KV Connector 吞吐量计算** (#48368)：修正了 Nixl/Mooncake KV Connector 在并发传输场景下，吞吐量指标（MB/s）计算逻辑错误（将各传输时间总和误作总耗时）。
* **修复 DeepSeek V3 工具调用解析** (#48367)：处理了模型输出包含外部工具调用标记但缺少完整匹配时的畸形解析问题。
* **防止 XPU MLA 内核 NaN 污染** (#48366)：修复了 XPU sparse-MLA 内核在处理全掩码索引块时输出 NaN 的问题。

**硬件与性能调优 (ROCm)**
* **调优 AMD MI355X Mamba 内核** (#48372, #48373)：将 MI355X 上 `selective_state_update` 解码内核的 float16/float32 启动配置，重新调优至与 MI300X 统一的 `effective_batch` 网格上。

**文档与 CI**
* **修正文档默认值** (#48376)：将后缀解码（suffix decoding）的最大投机 token 数文档默认值从 32 更正为 24。
* **恢复模型 CI 测试** (#48370)：重新启用了 3 个现已公开的模型（Granite4Vision、GlmOcrMTP、Qwen3DSpark）的注册表测试。

### 🚀 Release 概要：v0.25.0
* **版本亮点**：**Model Runner V2 现已成为默认运行器**，带来底层执行架构的重大升级。
* **规模统计**：包含 558 个提交，来自 232 位贡献者（其中 64 位为新贡献者）。

---

## 🐛 Issues

### #48374 — [[RFC]: fp8 KV cache for the Ampere sparse-MLA path (TRITON_MLA_SPARSE) via software dequant](https://github.com/vllm-project/vllm/issues/48374)
- **作者**: nickus  **时间**: 2026-07-12 07:43 CST
- **摘要**: ### 🚀 The feature, motivation and pitch  **`TRITON_MLA_SPARSE` (#47629) is bf16-KV-only, and on 24 GB cards that is the binding constraint.** I have a working sm_80/sm_86 fp8 KV path and would like to know whether the project wants it before I polish it into a mergeable PR.  #### The gap, in the PR'…

## 🔀 Pull Requests

### #48376 — [[Doc] Correct suffix decoding default](https://github.com/vllm-project/vllm/pull/48376)
- **作者**: abatilo  **时间**: 2026-07-12 08:40 CST
- **标签**: documentation
- **摘要**: ## Summary  Correct the documented default maximum speculative token count for suffix decoding from 32 to 24. This matches `suffix_decoding_max_tree_depth` and the fallback assignment in `SpeculativeConfig._validate_suffix_decoding`.  ## Duplicate work  I searched open pull requests for suffix decod…

### #48375 — [[BugFix] Honor drop_eagle_block in MambaManager](https://github.com/vllm-project/vllm/pull/48375)
- **作者**: potto007  **时间**: 2026-07-12 08:29 CST
- **标签**: bug, v1
- **摘要**: ## Purpose  On a hybrid model (full-attention + Mamba/GDN layers, e.g. Qwen3.6-35B-A3B), enabling MTP/EAGLE speculative decoding together with prefix caching corrupts the recurrent state, and the server serves wrong answers. #43559 reports tool-eval accuracy dropping from ~90% to ~50%. Nothing raise…

### #48373 — [[ROCm] Retune MI355 selective_state_update float32 config on the unified effective_batch grid](https://github.com/vllm-project/vllm/pull/48373)
- **作者**: vanshbhatia-amd  **时间**: 2026-07-12 05:35 CST
- **标签**: rocm
- **摘要**: ## Summary  Re-tunes the **AMD Instinct MI355X** `selective_state_update` (Mamba SSU decode kernel) `cache_dtype=float32` launch config onto the **same `effective_batch` grid** already used by the MI300X (#47947) and MI350X (#48159) configs, so all AMD Instinct SSU configs are sampled on a uniform s…

### #48372 — [[ROCm] Retune MI355 selective_state_update float16 config on the unified effective_batch grid](https://github.com/vllm-project/vllm/pull/48372)
- **作者**: vanshbhatia-amd  **时间**: 2026-07-12 05:35 CST
- **标签**: rocm
- **摘要**: ## Summary  Re-tunes the **AMD Instinct MI355X** `selective_state_update` (Mamba SSU decode kernel) `cache_dtype=float16` launch config onto the **same `effective_batch` grid** already used by the MI300X (#47945) and MI350X (#48159) configs, so all AMD Instinct SSU configs are sampled on a uniform s…

### #48371 — [[Bugfix] Fix OOB expert_map read in moe_fused_mul_sum with invalid topk_ids](https://github.com/vllm-project/vllm/pull/48371)
- **作者**: jahnavi-yelamanchi  **时间**: 2026-07-12 04:57 CST
- **标签**: bug
- **摘要**: ## Purpose  Fixes #47281. Under expert parallelism, `topk_ids` can contain `-1` sentinels for token/expert pairs that got routed to a different EP rank (documented in `fused_moe/utils.py`). The `moe_fused_mul_sum` Triton kernel indexed `expert_map` with these ids without guarding against `-1`, so it…

### #48370 — [[CI/Build] Re-enable registry tests for 3 now-public model checkpoints](https://github.com/vllm-project/vllm/pull/48370)
- **作者**: guoriyue  **时间**: 2026-07-12 04:34 CST
- **摘要**: ## Purpose  `Granite4VisionForConditionalGeneration`, `GlmOcrMTPModel` and `Qwen3DSparkModel` carry `is_available_online=False` in `tests/models/registry.py`, set while their checkpoints were unpublished. All three repos have since gone public, so the flags now only disable their CI coverage (initia…

### #48369 — [[Frontend] Add offline prefix-cache workload analyzer CLI](https://github.com/vllm-project/vllm/pull/48369)
- **作者**: harsh543  **时间**: 2026-07-12 04:21 CST
- **标签**: frontend
- **摘要**: ## Purpose  Adds `vllm analyze-prefix-cache`: an offline, no-GPU CLI that estimates how prefix-cache-friendly a request dataset is, before spending GPU time on a serving run.  Fixes #47993.  **Motivation:** vLLM's automatic prefix caching reuses KV-cache blocks only when the full-block hash chain ma…

### #48368 — [[Bug] Fix KV connector throughput metric under concurrent transfers](https://github.com/vllm-project/vllm/pull/48368)
- **作者**: harsh543  **时间**: 2026-07-12 03:48 CST
- **标签**: bug, v1, kv-connector
- **摘要**: ## What's wrong  `NixlKVConnectorStats.reduce()` and `MooncakeKVConnectorStats.reduce()` both report:  ```text Throughput (MB/s) = total_mb / sum(per_transfer_durations) ```  Because transfers are posted without waiting for completion, independent per-handle stopwatch spans overlap in steady state. …

### #48367 — [[Bugfix] Handle malformed DeepSeek V3 tool calls](https://github.com/vllm-project/vllm/pull/48367)
- **作者**: Abhayrkhot  **时间**: 2026-07-12 03:38 CST
- **标签**: bug, tool-calling, deepseek
- **摘要**: ## Summary  Fix the DeepSeek V3 non-streaming tool parser fallback when model output contains the outer tool-call marker but no complete tool-call match.  Previously, `tool_call_regex.findall()` could return no matches while the parser still returned `tools_called=True` with an empty `tool_calls` li…

### #48366 — [[Bugfix] Prevent NaN poisoning in xpu_mla_sparse for fully-masked index chunks](https://github.com/vllm-project/vllm/pull/48366)
- **作者**: nickus  **时间**: 2026-07-12 02:41 CST
- **标签**: bug, intel-gpu, v1
- **摘要**: FIX #48364  ## Purpose  `_bf16_mla_sparse_kernel` (the XPU sparse-MLA kernel behind `XPU_MLA_SPARSE`, DeepSeek-V4 XPU prefill, and the fp8 decode wrapper) NaN-poisons its output whenever the first `BLOCK_N` (=16) topk index entries of a row are all masked, even though valid keys follow later:  - the…

## 🚀 Releases

### [v0.25.0](https://github.com/vllm-project/vllm/releases/tag/v0.25.0)
- **作者**: khluu  **时间**: 2026-07-12 04:06 CST
- **摘要**: # vLLM v0.25.0 Release Notes  ## Highlights  This release features 558 commits from 232 contributors (64 new)!  * **Model Runner V2 is now the default for all dense models** (#44443). Building on quantized-model support from the previous release, MRv2 is now the standard execution path, with new sup…
