# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-22 13:15 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的简洁摘要：

### 🐛 Issue 动态
- **KV Offload 元数据丢失 Bug (#63037)**
  报告了一个关于 KV 缓存的 Bug：复用 `OffloadKey` 时，可能会因延迟的 `BlockStored` 事件而丢失元数据。该问题已在最新的 `main` 分支上复现并进行了回归测试。

### 🚀 Pull Request 动态
本次 PR 主要围绕 **ROCm/AMD 显卡适配优化**、**核心调度与性能提升** 以及 **多模态修复** 展开：

**1. ROCm / AMD 显卡相关优化（大量更新）**
- **MiMo 模型支持**：为 MiMo-V2.6-Pro 在 gfx950 上添加了 TP8 MegaMoE 后端 (#63039)；并为 MiMo-V2 密集 MLP 添加了实验性 FP8 blockscale 融合算子 (#63034)。
- **Qwen3.8 与 Kimi-K3**：在 MI355X 上为 Qwen3.8-Flash-Next 引入了专用的 `--monokernel` 解码优化 (#63032)；为 Kimi-K3 启用了支持 MoRIIO 的专用解码，并扩展至分离式服务 (#63031)。
- **DeepSeek-V4.1**：支持在 A4W4 MoE 中通过 MoRI 调度预量化的 MXFP4 数据 (#63029)。

**2. 核心调度与性能优化**
- **调度器上下文限制 (#63035)**：新增 `--max-num-batched-context-tokens` 参数，限制单步预填充所处理的上下文 Token 数量，以防内存溢出或卡顿。
- **DSA 性能优化 (#63038)**：根据压缩后的 logits 宽度对稀疏索引器的预填充块进行预算控制，提升 DeepGEMM 性能。
- **推测解码优化 (#63030, WIP)**：在 CUDA Graph 的 WHILE 节点中实现草稿模型提前退出，无需主机同步即可停止生成，保留了异步调度能力。

**3. 多模态与文档修复**
- **PaliGemma 注意力修复 (#63033)**：修复了 PaliGemma 模型的双向注意力范围，使其在处理图像、BOS、前缀文本等整个提示词时保持一致。
- **NIXL 文档修正 (#63036)**：更新了 NIXL 兼容性矩阵，明确其支持混合 SSM/Mamba 模型（含 GDN）的异构 TP，而非仅限于同构 TP。

### 📦 Release 动态
近期无新的 Release 版本发布。

---

## 🐛 Issues

### #63037 — [[Bug][KV Offload] Reused OffloadKey can lose metadata from delayed BlockStored events](https://github.com/vllm-project/vllm/issues/63037)
- **作者**: Alex-ai-future  **时间**: 2026-10-22 13:05 CST
- **摘要**: ## Your current environment  The issue was reproduced and regression-tested against the current `main` branch.  Relevant validation from [#62414](https://github.com/vllm-project/vllm/pull/62414):  - 440 offloading connector unit tests passed - 52 CPU offload manager tests passed - Pre-commit checks …

## 🔀 Pull Requests

### #63039 — [[ROCm][MiMo] Add TP8 MegaMoE backend](https://github.com/vllm-project/vllm/pull/63039)
- **作者**: LiuYinfeng01  **时间**: 2026-10-22 13:09 CST
- **标签**: documentation, rocm, ci/build, deepseek, DSv4
- **摘要**: ## Scope  Draft implementation of `aiter_mimo_mega_moe_tp` for MiMo-V2.6-Pro routed experts on gfx950 TP8/DP1. Separate from MiMo mono MLP and DeepSeek-V4 TP MegaMoE.  Geometry: E384, top-k8, hidden6144, TP-local intermediate2048, sigmoid routing with correction bias.  ## Evidence  - focused ROCm co…

### #63038 — [[Perf][DSA] Budget sparse-indexer prefill chunks by the compressed logits width](https://github.com/vllm-project/vllm/pull/63038)
- **作者**: nbommi19  **时间**: 2026-10-22 13:07 CST
- **标签**: rocm, deepseek, cpu, DSv4
- **摘要**: ## Overview  Since #61149, DeepGEMM returns the CUDA sparse-indexer prefill logits as `[rows, max_seqlen_k]`, where `max_seqlen_k` is the longest key range in the chunk. The chunk planner (`_split_indexer_prefill_chunks`) still budgets each chunk as `rows x total keys of all requests`. This PR budge…

### #63036 — [[docs] correct NIXL hybrid SSM heterogeneous TP support](https://github.com/vllm-project/vllm/pull/63036)
- **作者**: hlin99  **时间**: 2026-10-22 13:04 CST
- **标签**: documentation, kv-connector
- **摘要**: Correct the NIXL compatibility matrix to reflect existing heterogeneous TP support for hybrid SSM/Mamba models, including GDN.                                                                                                • Replace the outdated same-TP-only restriction with support for both TP direc…

### #63035 — [[Core][Scheduler] Bound the context a prefill step attends over](https://github.com/vllm-project/vllm/pull/63035)
- **作者**: gcanlin  **时间**: 2026-10-22 12:51 CST
- **标签**: scheduler
- **摘要**: ## Overview  Adds `--max-num-batched-context-tokens`, an opt-in bound on the context that one step's prefill chunks attend over. Each chunk counts its request's already-computed tokens plus the tokens it prefills, summed over the step. `max_num_batched_tokens` bounds only the new tokens, so it does …

### #63034 — [[ROCm][MiMo] Add experimental FP8 blockscale mono MLP](https://github.com/vllm-project/vllm/pull/63034)
- **作者**: LiuYinfeng01  **时间**: 2026-10-22 12:47 CST
- **标签**: rocm
- **摘要**: ## Scope  Adds an opt-in experimental gfx950 eager-decode kernel for MiMo-V2 dense MLPs. It fuses SwiGLU, per-128 FP8 activation quantization, and the FP8 blockscale down projection without materializing the activated intermediate tensor.  Enabled only with `VLLM_ROCM_MIMO_MONO_MLP=1`; requires gfx9…

### #63033 — [[Bugfix][Multimodal] Make PaliGemma's whole prompt one bidirectional block](https://github.com/vllm-project/vllm/pull/63033)
- **作者**: ejrtks1020  **时间**: 2026-10-22 12:37 CST
- **标签**: bug, documentation, mrv2, scheduler
- **摘要**: ## Overview  Fixes #62968. PaliGemma attends bidirectionally over its whole prompt (image, BOS, prefix text, trailing newline) in Transformers, but vLLM only made the image span bidirectional. This PR builds the PrefixLM range over the whole prompt for PaliGemma, and keeps each prompt in a single pr…

### #63032 — [[ROCm][Perf] Qwen3.8-Flash-Next specialized decode on MI355X (--monokernel)](https://github.com/vllm-project/vllm/pull/63032)
- **作者**: mjkvaak-amd  **时间**: 2026-10-22 12:19 CST
- **标签**: rocm, ci/build, qwen
- **摘要**: ## Overview  Opt-in `--monokernel` decode for Qwen3.8-Flash-Next (`amd/Qwen3.8-Flash-Next-Quark-MXFP4`) at TP2 on MI355X. Laid out per the specialized-decode RFC #60904 under `vllm/models/qwen4_exp/amd/specialized_decode/`. A decode step of 1–16 rows runs four persistent FlyDSL launches a layer; eve…

### #63031 — [[ROCm][Kimi-K3] Support MoRIIO consumers with specialized decode](https://github.com/vllm-project/vllm/pull/63031)
- **作者**: YukioZzz  **时间**: 2026-10-22 12:19 CST
- **标签**: documentation, rocm, kimi, k3
- **摘要**: ## Overview  Enable Kimi-K3 specialized decode on built-in MoRIIO consumers, extending #60683 to disaggregated serving. Reuse the READ completion barrier from #60808 and the existing WRITE admission barrier.  ## Claims  - Accept MoRIIO READ and WRITE consumers without restricting their transport at …

### #63030 — [[WIP][Spec Decode] Drafter early exit in a CUDA graph WHILE node](https://github.com/vllm-project/vllm/pull/63030)
- **作者**: qiching  **时间**: 2026-10-22 12:14 CST
- **标签**: speculative-decoding, nvidia, mrv2
- **摘要**: Autoregressive drafters (MTP, EAGLE) can stop drafting on the GPU without a host sync, so async scheduling is preserved. With VLLM_DYN_DRAFT set, the captured FULL decode graph runs draft steps 1..k-1 inside one CUDA graph WHILE node; a Triton kernel decides after each step whether the batch's expec…

### #63029 — [[ROCm][DSv4.1] Dispatch pre-quantized MXFP4 through MoRI for the A4W4 MoE](https://github.com/vllm-project/vllm/pull/63029)
- **作者**: maslan-coop26  **时间**: 2026-10-22 12:09 CST
- **标签**: rocm, quantization, DSv4.1
- **摘要**: ## Motivation  With `VLLM_ROCM_USE_AITER_MOE_A4W4_DSV4=1`, DeepSeek-V4.1's routed experts run AITER's A4W4 MoE: the activations are quantized to MXFP4 (group-32, E8M0 scales) inside `fused_moe`. Under MoRI expert parallelism they are therefore dispatched as BF16, 2 bytes per element, and every rank …
