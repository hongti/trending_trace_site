# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-31 14:10 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
1. **双向 NIXL 请求挂起 Bug** (#64859)
   - 在启用 `bidirectional_kv_xfer` 时，Prefill 节点的请求在首次 D->P 拉取后会无限挂起在 `WAITING_FOR_REMOTE_KVS` 状态。
2. **稀疏注意力索引器输出异常** (#64855)
   - 在 AMD MI355X 环境下，使用 native padded decode (`requires_padding`) 在 ragged 投机解码步骤中会产生错误的 logits / top-k 结果。
3. **新增特性请求：无侵入式清理前缀缓存** (#64853)
   - 希望支持 `reset_prefix_cache` 在不清除运行中请求的情况下，安全地清除已缓存的前缀。

### 🔧 Pull Request 动态
**重要修复**
- **修复双向 NIXL 挂起问题** (#64860)：解决 Prefill 节点首次 D->P 拉取后无限等待的问题，释放被卡住的 KV blocks。
- **修复 Mistral Tokenizer 角色映射** (#64861)：将 `developer` 角色自动映射为 `system`，修复了使用 `--tokenizer-mode mistral` 时相关 API 拒绝该角色的问题。
- **修复 MiniMax-M3.1 内存显存溢出** (#64856)：在实现初始化时预留 Q8KV4 prefill 分割片段，修复了因未计入内存分析导致超出 `--gpu-memory-utilization` 限制的问题。

**新特性与前端支持**
- **Rust 前端支持 Beam Search** (#64862)：在 `/v1/completions` 和 `/v1/chat/completions` 端点实现了与 Python 端一致的在线 Beam Search 功能。
- **支持 Anthropic 客户端工具** (#64858)：`/v1/messages` 端点现支持通过内置 schemas 接收 Anthropic 定义的客户端工具（如 `bash_*`, `text_editor_*`, `memory_*`）。
- **Responses API 支持音频输入** (#64851)：修复并支持了在 `/v1/responses` 用户消息中传入 `input_audio` 内容。

**性能与算子优化**
- **ROCm (gfx942) 稀疏 MLA 解码优化** (#64857)：针对 AMD MI300X，每 64 列仅加载一次 e8m0 缩放因子并将 `BLOCK_K` 设为 16，显著提升性能。
- **MiniMax-M3 算子融合与写入优化** (#64852)：在 NVIDIA TP1 模式下，使用 Triton kernel 将 input Gemma RMSNorm 与 qkv_proj 的 MXFP8 输入写入合并。
- **融合 tanh-GELU 激活与静态 FP8 量化** (#64854)：新增融合算子以优化静态 FP8 量化路径。

**测试与代码健壮性**
- 扩充了 `collection_utils` 辅助函数的测试覆盖范围 (#64850)。

### 🚀 Release 动态
- 近期暂无新版 Release 信息。

---

## 🐛 Issues

### #64859 — [[Bug]: Bidirectional NIXL: prefill-side request hangs in WAITING_FOR_REMOTE_KVS after its first D->P pull](https://github.com/vllm-project/vllm/issues/64859)
- **作者**: sohom-cs  **时间**: 2026-10-31 13:24 CST
- **标签**: kv-connector
- **摘要**: ### Your current environment  <details> <summary>Environment</summary>  CPU-only unit reproduction; no GPU run. The bug is in the NIXL connector's scheduler-side logic, so the scheduler unit tests below reproduce it without NIXL hardware.  - vLLM main at 2b009fc6d1 - Python 3.12.1, torch 2.13.0, mac…

### #64855 — [[Bug] Sparse-attention indexer: native padded decode (`requires_padding`) gives wrong logits/top-k on ragged spec-decode steps](https://github.com/vllm-project/vllm/issues/64855)
- **作者**: jbelofamd  **时间**: 2026-10-31 12:05 CST
- **标签**: rocm
- **摘要**: ### Environment  - GPU: AMD Instinct MI355X (gfx950) - ROCm (the bug is in host-side metadata and input slicing, independent of the ROCm version) - AITER: amd-aiter 0.1.26 - vLLM: a 0.32.1 dev build; code references below are to main @ 602811546b035a5ff35256a0a008b61071b13904  ### Summary  When a de…

### #64853 — [[Feature]: Allow `reset_prefix_cache` to clear cached prefixes without preempting running requests](https://github.com/vllm-project/vllm/issues/64853)
- **作者**: chalshik  **时间**: 2026-10-31 11:49 CST
- **标签**: feature request
- **摘要**: ### 🚀 The feature, motivation and pitch  ### What I'm doing  I'm studying how a vLLM server recovers after losing its prefix cache. I run a single replica under steady open-loop load, wipe the cache with `POST /reset_prefix_cache`, and measure whether latency returns to normal. The wipe should chang…

## 🔀 Pull Requests

### #64862 — [[Rust Frontend] Support beam search (use_beam_search, length_penalty)](https://github.com/vllm-project/vllm/pull/64862)
- **作者**: aaroncheng00  **时间**: 2026-10-31 14:08 CST
- **标签**: ci/build, rust
- **摘要**: ## Overview  Implements beam search in the Rust frontend for `/v1/completions` and `/v1/chat/completions` (`use_beam_search`, beam width `n`, `length_penalty`), mirroring Python's online beam search. Part of the Rust frontend parity roadmap #44280 ("Beam search via the existing `use_beam_search` req…

### #64861 — [[Bugfix] Map `developer` role to `system` for Mistral tokenizers](https://github.com/vllm-project/vllm/pull/64861)
- **作者**: Shawnsuun  **时间**: 2026-10-31 13:40 CST
- **标签**: bug, mistral
- **摘要**: ## Overview  Fix #64793: with `--tokenizer-mode mistral`, `/v1/responses` and `/v1/chat/completions` reject any input message with `"role": "developer"`. The Mistral renderer now maps `developer` → `system`, reusing the conversion the HF renderer already uses.  ## Claims  - `developer` messages rend…

### #64860 — [[Bugfix][NIXL] Fix bidirectional P-side hang after the first D->P pull](https://github.com/vllm-project/vllm/pull/64860)
- **作者**: sohom-cs  **时间**: 2026-10-31 13:25 CST
- **标签**: bug, kv-connector, nixl
- **摘要**: ## Overview  With `bidirectional_kv_xfer`, a turn-2 request on the prefill node that goes back through the connector lookup after its first D->P pull waits in `WAITING_FOR_REMOTE_KVS` forever and keeps its blocks. This makes that second lookup prefill locally.  Fixes #64859  ## Claims  - After its f…

### #64858 — [[Frontend] support anthropic-defined client tools on /v1/messages](https://github.com/vllm-project/vllm/pull/64858)
- **作者**: dundysm  **时间**: 2026-10-31 12:56 CST
- **标签**: documentation, frontend
- **摘要**: ## Overview  `/v1/messages` now accepts anthropic-defined client tools (`bash_*`, `text_editor_*`, `memory_*`) by supplying their built-in schemas to the chat template; second work item of #64810. follows #64575, which rejected these with a named 400.  ## Claims  - requests that declare `{"type": "b…

### #64857 — [[ROCm][Perf] gfx942 sparse MLA decode: load e8m0 scales once per 64 columns, BLOCK_K 16](https://github.com/vllm-project/vllm/pull/64857)
- **作者**: ishara0925  **时间**: 2026-10-31 12:36 CST
- **标签**: rocm
- **摘要**: Contributed by **Krai Ltd.** (hasith@krai.ai) as part of its MLPerf Endpoints v1.0 DeepSeek-V4.1-Flash submission on 8x AMD Instinct MI300X; this commit is one of the patches disclosed in that submission.  ## Purpose  On gfx942 (MI300X/MI325X) the DeepSeek-V4/V4.1 sparse MLA decode runs `_sparse_att…

### #64856 — [[Bugfix][MiniMax-M3.1] Reserve the Q8KV4 prefill split partials at impl init](https://github.com/vllm-project/vllm/pull/64856)
- **作者**: gau-nernst  **时间**: 2026-10-31 12:16 CST
- **标签**: bug, nvidia, minimax
- **摘要**: ## Overview  Fixes item 7 of #60791: the MiniMax-M3 MSA sparse prefill (NVFP4 KV cache) split partials are allocated per call but never counted by memory profiling, so the engine exceeds `--gpu-memory-utilization` and OOMs — at startup in the FlashInfer-autotune dummy run (the issue's repro), or on …

### #64854 — [[Kernel][Fusion] Fuse tanh-GELU gated activation with static FP8 quant](https://github.com/vllm-project/vllm/pull/64854)
- **作者**: t239573445  **时间**: 2026-10-31 11:56 CST
- **标签**: documentation, performance, quantization
- **摘要**: ## Relation to #64212  #64212 adds `gelu_tanh_and_mul_per_token_quant` for the **dynamic per-token** FP8 path (`kFp8DynamicTokenSym`), which is what compressed-tensors `*-FP8-dynamic` Gemma checkpoints use. This PR covers the **static per-tensor** path (`kFp8StaticTensorSym`), used by modelopt-style…

### #64852 — [[Perf][MiniMax-M3] (NVIDIA-only) TP1 input norm writes qkv_proj's MXFP8 input](https://github.com/vllm-project/vllm/pull/64852)
- **作者**: gau-nernst  **时间**: 2026-10-31 11:40 CST
- **标签**: nvidia, minimax
- **摘要**: <!-- markdownlint-disable --> ## Overview On NVIDIA TP1, MiniMax-M3's input Gemma RMSNorm always uses one Triton kernel for supported BF16 inputs, including layer 0's plain norm. Per call, it writes qkv_proj's MXFP8 input when the consumer takes MXFP8, otherwise the plain BF16 norm; TP>1 keeps fused…

### #64851 — [[Frontend][Responses] Accept input_audio content parts in user messages (#47659)](https://github.com/vllm-project/vllm/pull/64851)
- **作者**: shadowmodder  **时间**: 2026-10-31 11:20 CST
- **标签**: frontend
- **摘要**: Supersedes #47662, which was closed when its branch was recreated during a rebase onto current `main`. Same fix, rebased and re-verified.  ## Summary  Audio inputs that work on `/v1/chat/completions` fail on `/v1/responses` with an opaque 422 `ValidationError` (#47659), even though the payload is id…

### #64850 — [[Test] Add tests for collection_utils helpers](https://github.com/vllm-project/vllm/pull/64850)
- **作者**: samaleka  **时间**: 2026-10-31 11:16 CST
- **摘要**: ## Overview  Extend `tests/utils_/test_collection_utils.py` (which previously covered only `is_list_of_numbers`, `common_prefix`, and `swap_dict_values`) with tests for `as_list`, `chunk_list`, `full_groupby`, and `LazyDict`, which had no tests anywhere in `tests/`.  ## Claims  - 12 new cases, inclu…
