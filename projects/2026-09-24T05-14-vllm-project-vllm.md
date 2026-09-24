# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-24 13:14 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要：

### ⚠️ Issue 与 Release 动态
本次提供的数据中未包含 Issue 与 Release 相关信息。

### 🚀 Pull Request (PR) 摘要

**一、性能优化与新特性**
1. **ROCm 性能提升**：
   - 针对 gfx950 优化了 DeepSeek V4/V4.1 的 MXFP8 GEMM 性能，支持原生 32x32 块级权重缩放（#58510）。
   - 提升了 ROCm 平台上 HY4 preview FP8 的性能，通过启用 AITER MoE 等方式加速推理（#58509）。
2. **新硬件/后端集成**：
   - 支持 GLM5Next 的 dense 部分在 CPU 上运行，为 kda 层添加了纯 torch 实现路径（#58507）。
   - 集成 Helion KDA 后端，将 SGLang 内核适配至 vLLM 的状态索引与预填充输入中（#58502）。
3. **Kernel 优化**：
   - 避免 SM100 blockwise FP8 GEMM 中冗余的缩放填充操作，提升执行效率（#58506）。

**二、重要 Bug 修复**
1. **模型与调度逻辑修复**：
   - 修复 DeepSeek-V4.1 在 bounded replay 机制下，KV connector 报告非块对齐前缀命中时的边界处理错误（#58504）。
   - 修复 KV 事件中，针对重复前缀缓存块错误发射 `BlockRemoved` 事件的问题，现仅针对最后一个缓存副本触发（#58508）。
2. **前端与解析器修复**：
   - 修复 `/tokenize` 和 pooling chat 请求中，当助手消息的 `content` 为空但包含 `tool_calls` 时导致 HTTP 400 报错的问题（#58503）。
   - 修复推理解析器（Reasoning Parser）与工具解析器（Tool Parser）结合使用时（如 MiniMax M2 的 append-think 模式），错误丢弃结束定界符的问题（#58505, #58501）。

---

## 🔀 Pull Requests

### #58510 — [[ROCm][Perf] MXFP8 GEMM on native 32x32 block scales for gfx950](https://github.com/vllm-project/vllm/pull/58510)
- **作者**: Fangzhou-Ai  **时间**: 2026-09-24 12:53 CST
- **标签**: rocm
- **摘要**: ## Purpose  DeepSeek V4/V4.1 checkpoints store MXFP8 weight scales in 32x32 blocks. The ModelOpt loader expands them to one scale row per weight row, and on gfx950 `RocmDotScaledMxfp8LinearKernel` then runs a single generic `tl.dot_scaled` kernel on those per-row scales. On DeepSeek-V4.1-Flash those…

### #58509 — [[ROCm][Perf] HY4 preview perf boost](https://github.com/vllm-project/vllm/pull/58509)
- **作者**: qli88  **时间**: 2026-09-24 12:03 CST
- **标签**: performance, rocm, ci/build
- **摘要**: ## Purpose Improve the performance of HY 4 preview FP8 on ROCm platform. What this PR contains:  1. Enable AITER MoE for HY4 FP8. Actually AITER FP8 MoE already supports Silu activation, so we just change the vLLM side code to utilize it; 2. Rewrite iHC kernels with Triton to replace the default PyT…

### #58508 — [[Bugfix][KV Events] Emit BlockRemoved only for the last cached copy of a block hash](https://github.com/vllm-project/vllm/pull/58508)
- **作者**: milesial  **时间**: 2026-09-24 11:48 CST
- **标签**: bug, kv-cache-manager
- **摘要**: ## Purpose  vLLM does not merge duplicate prefix-cache blocks. When two requests compute the same prefix, one block hash maps to several physical blocks. Examples are concurrent chunked prefills, identical decode output, and the recomputed last block of a block-aligned prompt.  `BlockPool` emits `Bl…

### #58507 — [run glm5next dense on cpu](https://github.com/vllm-project/vllm/pull/58507)
- **作者**: hamstergamerszhang-ops  **时间**: 2026-09-24 11:37 CST
- **标签**: rocm, cpu, glm
- **摘要**: first half of #57345 - the dense part. the sparse indexer is cuda-only so that stays out of this one.  the kda layers had no cpu path at all, so this adds plain torch versions of the two ops (fused_recurrent_kda for decode, chunk_kda_with_fused_gate for prefill). they follow the triton kernels op fo…

### #58506 — [[Kernel] Avoid redundant SM100 blockwise FP8 scale padding](https://github.com/vllm-project/vllm/pull/58506)
- **作者**: xsank  **时间**: 2026-09-24 11:33 CST
- **标签**: nvidia
- **摘要**: ## Purpose  The SM100 blockwise FP8 GEMM requires the leading dimension of its column-major activation scales to be divisible by four for TMA scale-factor copies.  The current implementation handles a non-aligned `M > 64` by allocating a temporary padded scale tensor and issuing a `cudaMemcpy2DAsync…

### #58505 — [[Bugfix] Keep closing </think> delimiter in MiniMax M2 append-think mode](https://github.com/vllm-project/vllm/pull/58505)
- **作者**: apex-mochen  **时间**: 2026-09-24 11:24 CST
- **标签**: bug, tool-calling, minimax
- **摘要**: ## Summary  The MiniMax M2 tool parser dropped the closing `</think>` delimiter from `content` when it appeared in the CONTENT state — i.e. when combined with the `minimax_m2_append_think` reasoning parser, which intentionally keeps the reasoning delimiters in `content`. The opening `<think>` surviv…

### #58504 — [[Bugfix][DSv4.1] Preserve partial external hits under bounded replay](https://github.com/vllm-project/vllm/pull/58504)
- **作者**: wangyicong52  **时间**: 2026-09-24 11:07 CST
- **标签**: bug, scheduler, kv-cache-manager, DSv4.1
- **摘要**: ## Purpose  When a KV connector reports a non-block-aligned prefix hit for DeepSeek-V4.1 bounded replay, the scheduler currently rounds the hit down to a whole block before rewinding the replay window. With a 64-token block, a 128-token replay window, and an 8,439-token remote hit, this changes the …

### #58503 — [[Bugfix][Frontend] Materialize tool_calls in /tokenize and pooling chat requests](https://github.com/vllm-project/vllm/pull/58503)
- **作者**: sohom-cs  **时间**: 2026-09-24 11:01 CST
- **标签**: bug, frontend, tool-calling
- **摘要**: ## Purpose  Fixes #57730.  `POST /tokenize` with a chat body whose assistant message has `"content": null` and a `tool_calls` list returns HTTP 400:  ``` cannot pickle 'pydantic_core._pydantic_core.ValidatorIterator' object ```  Root cause: `ChatCompletionAssistantMessageParam.tool_calls` is typed `…

### #58502 — [[helion] Helion KDA backend integration](https://github.com/vllm-project/vllm/pull/58502)
- **作者**: ethche  **时间**: 2026-09-24 10:49 CST
- **标签**: performance, kimi, k3
- **摘要**: ## Purpose  Changes: - fixes integration, adapts the ported SGLang kernels to vLLM's state indexing and prefill inputs (`ops/kda/kda_prefill.py`, `kda_decode.py`, `kda_replayssm.py`), hardens backend selection, and adds coverage for padded state pages and dynamic shapes. - performance: three changes…

### #58501 — [[Bugfix][Reasoning] Keep the closing delimiter when append-think meets the tool parser](https://github.com/vllm-project/vllm/pull/58501)
- **作者**: he-yufeng  **时间**: 2026-09-24 10:49 CST
- **标签**: bug, tool-calling, minimax
- **摘要**: ## Purpose  Fixes #58486.  With `--reasoning-parser minimax_m2_append_think --tool-call-parser minimax_m2`, a tool call dropped the closing `</think>` from `content`. The append parser keeps reasoning markup inside content by design, but the composed tool engine still treated `</think>` as a reasoni…
