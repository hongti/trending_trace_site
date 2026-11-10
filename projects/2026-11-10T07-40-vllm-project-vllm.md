# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-10 15:40 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 最近动态的中文摘要：

### 📌 Issue 动态
1. **RFC: 投机解码的 Draft Window KV 优化 (#66751)**
   - **提案亮点**：提出在投机解码中，由于 Draft 模型仅需读取最近的 `W` 个 token，建议仅保留并复用实际被读取的滑动窗口 KV，而不是在共享池中保留完整的历史 Main KV，以优化内存和计算效率。
2. **CI 失败: 调度器测试报错 (#66750)**
   - **问题定位**：`tests/v1/core/test_scheduler.py` 测试中出现未定义的 `_start_decoding` 调用，导致 Ruff 和 mypy 检查失败，目前已在本地复现。

---

### 🔨 Pull Request 动态

**新特性与后端支持**
- **[Kernel/MoE] 引入 FlashInfer CuTe-DSL MoE 后端 (#66749)**：为 Hopper (SM90) 架构新增基于 FlashInfer CuTe-DSL 的非量化 MoE 专家后端，提升 MoE 计算效率。
- **[ROCm/DSA] DCP global-key indexer 预填充 (#66748)**：为 ROCm DSA 拆分并引入了 DCP global-key indexer 的预填充支持。
- **[XPU/LoRA] 单适配器预填充使用 Dense GEMM (#66752)**：在 XPU LoRA 线性层中，当预填充批次仅使用单个适配器时，用 Dense GEMM 替代 per-token BGMV，提升性能。

**性能优化**
- **[Attention] 缓存 Triton K/V cache views (#66745/66746/66747)**：针对 `TritonAttentionImpl` 优化，避免在每次注意力计算调用时重复推导 K/V cache views（如 transpose、split 等），减少开销。
- **[ROCm] FP8 注意力与投机解码验证优化 (#66741/66742/66743)**：一系列针对 ROCm 平台的性能堆栈优化，包括在 AITER FA 中启用 paged MTP 验证、增加带高效 Q 量化的 paged FP8 extend attention 路径，以及支持 split-major 调度的 FP8 宽批次验证。

**Bug 修复**
- **[Tests] 修复调度器测试设置 (#66744)**：修复了上述 Issue #66750 中提到的 `_start_decoding` 未定义错误，恢复了测试的正常运行。

---

### 🚀 Release 动态
- 本次提供的动态中**无新版本发布**信息。

---

## 🐛 Issues

### #66751 — [[RFC]: Draft Window KV: Retaining Only the Window Actually Read During Speculative Decoding](https://github.com/vllm-project/vllm/issues/66751)
- **作者**: SSSunzt  **时间**: 2026-11-10 15:36 CST
- **标签**: RFC, speculative-decoding
- **摘要**: ### Motivation.  Main attention requires the full reusable history, while a sliding-window Draft model reads only the most recent `W` tokens. In the original shared-pool design, Main KV and Draft KV share the same physical block pool, cache-hit boundary, and lifetime. This forces Draft to retain and…

### #66750 — [[CI Failure]: pre-commit undefined _start_decoding in tests/v1/core/test_scheduler.py](https://github.com/vllm-project/vllm/issues/66750)
- **作者**: wjinxu  **时间**: 2026-11-10 15:34 CST
- **标签**: ci-failure
- **摘要**: ### Name of failing test  `tests/v1/core/test_scheduler.py::test_iteration_details_count_recomputation_as_prefill`  ### Basic information  - [ ] Flaky test - [x] Can reproduce locally - [ ] Caused by external libraries  ### Describe the failing test  The test calls `_start_decoding`, which is no lon…

## 🔀 Pull Requests

### #66752 — [[XPU][LoRA] Use dense GEMM for single-adapter prefill batches](https://github.com/vllm-project/vllm/pull/66752)
- **作者**: gc-fu  **时间**: 2026-11-10 15:37 CST
- **标签**: intel-gpu
- **摘要**: ## Overview  Use dense GEMMs instead of per-token BGMV for XPU LoRA linear layers when a prefill-sized batch uses a single adapter. Works under the default compile + graph mode.  ## Claims  - Single-adapter prefill with LoRA is **~40% faster** on Qwen3.6-27B (TP2), making it as fast as the same prom…

### #66749 — [[Kernel][MoE] Add FlashInfer CuTe-DSL unquantized MoE backend for Hopper (SM90)](https://github.com/vllm-project/vllm/pull/66749)
- **作者**: Aneureka  **时间**: 2026-11-10 15:32 CST
- **标签**: documentation, ci/build, nvidia, quantization
- **摘要**: ## Overview  Adds `FlashInferCuteDSLSm90Experts`, an opt-in unquantized (bf16/fp16) MoE experts backend for Hopper built on FlashInfer's SM90 CuTe-DSL fused MoE (`flashinfer.fused_moe.cute_dsl_fused_moe_bf16`). Select it with `--moe-backend flashinfer_cutedsl`; Triton remains the Hopper default. Rel…

### #66748 — [[ROCm][DSA] DCP global-key indexer prefill (stacked on #65685)](https://github.com/vllm-project/vllm/pull/66748)
- **作者**: ZJY0516  **时间**: 2026-11-10 15:28 CST
- **标签**: documentation, rocm, needs-rebase, ci/build, kv-connector, nvidia, mooncake, nixl
- **摘要**: ## Overview  **ROCm DCP global-key indexer prefill for DSA**, split out of #65685 so that PR stays reviewable. Stacked on #65685: until it merges, the diff against `main` includes it. **Review only the last commit, 78319b321a** (+423 −1 in 4 files).  Not a duplicate: the code comes from #65685, whic…

### #66747 — [[Perf][Attention] Cache Triton K/V cache views per tensor identity](https://github.com/vllm-project/vllm/pull/66747)
- **作者**: huangsiyuan0422  **时间**: 2026-11-10 15:21 CST
- **摘要**: ## Summary  Fixes #64572.  `TritonAttentionImpl` re-derives K/V cache views (`transpose(1, 2).split` + optional FP8 `view`) on every attention call across `forward`, `do_kv_cache_update`, and `do_rope_and_kv_cache_update`, even though these views depend only on the `kv_cache` tensor's storage/shape/…

### #66746 — [[Perf][Attention] Cache Triton K/V cache views per tensor identity](https://github.com/vllm-project/vllm/pull/66746)
- **作者**: huangsiyuan0422  **时间**: 2026-11-10 15:21 CST
- **摘要**: ## Summary  Fixes #64572.  `TritonAttentionImpl` re-derives K/V cache views (`transpose(1, 2).split` + optional FP8 `view`) on every attention call across `forward`, `do_kv_cache_update`, and `do_rope_and_kv_cache_update`, even though these views depend only on the `kv_cache` tensor's storage/shape/…

### #66745 — [[Perf][Attention] Cache Triton K/V cache views per tensor identity](https://github.com/vllm-project/vllm/pull/66745)
- **作者**: huangsiyuan0422  **时间**: 2026-11-10 15:19 CST
- **摘要**: ## Summary  Fixes #64572.  `TritonAttentionImpl` re-derives K/V cache views (`transpose(1, 2).split` + optional FP8 `view`) on every attention call across `forward`, `do_kv_cache_update`, and `do_rope_and_kv_cache_update`, even though these views depend only on the `kv_cache` tensor's storage/shape/…

### #66744 — [[Bugfix][Tests] Fix iteration-details recomputation test setup](https://github.com/vllm-project/vllm/pull/66744)
- **作者**: wjinxu  **时间**: 2026-11-10 15:18 CST
- **标签**: bug, scheduler
- **摘要**: ## Overview  Fix the undefined `_start_decoding` call in `test_iteration_details_count_recomputation_as_prefill`, which currently fails both Ruff and mypy on main.  ## Claims  - Set up the existing regression through a real scheduler prefill and sampled output, preserving its recomputation and itera…

### #66743 — [[ROCm][Perf] FP8 wide-batch verification with split-major scheduling](https://github.com/vllm-project/vllm/pull/66743)
- **作者**: sawsa307  **时间**: 2026-11-10 15:18 CST
- **标签**: documentation, rocm
- **摘要**: ## Overview  Stacked on #66742 (which is stacked on #66741). The PR targets main; review the additional decode-width specialization and split-major grid commits after the prerequisite stack.  Extend the paged FP8 attention implementation with a specialized full-attention verify mode for wider unifor…

### #66742 — [[ROCm][Perf] Paged FP8 extend attention with efficient Q quantization](https://github.com/vllm-project/vllm/pull/66742)
- **作者**: sawsa307  **时间**: 2026-11-10 15:17 CST
- **标签**: documentation, rocm, quantization
- **摘要**: ## Overview  Stacked on #66741. The PR targets main, so its diff includes that prerequisite; review the additional FP8 extend/quantization commits.  Add a paged FP8 full-attention extend path to ROCM_AITER_FA and avoid excessive atomic contention in its per-tensor Q quantization. New queries attend …

### #66741 — [[ROCm][Perf] Opt-in paged MTP verification in AITER FA](https://github.com/vllm-project/vllm/pull/66741)
- **作者**: sawsa307  **时间**: 2026-11-10 15:17 CST
- **标签**: documentation, rocm
- **摘要**: ## Overview  Allow eligible uniform causal speculative-verification batches in ROCM_AITER_FA to use AITER paged_attention_v1's MTP support instead of the unified-attention fallback. This changes the target-model verify implementation, not the draft attention backend.  ## Claims  - Build uniform mult…
