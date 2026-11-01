# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-01 14:40 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
*   **配置验证与运行时崩溃问题**：多位开发者报告了配置通过验证但运行时崩溃的 Bug。例如 `--attention-backend TORCH_SDPA`（一个仅限 ViT 的标签）和 `seed=-1` 在 `SamplingParams` 中合法，但分别在引擎初始化或 `np.random.seed` 中引发未捕获的错误。
*   **量化维度不匹配**：使用 `kv_cache_dtype="turboquant_3bit_nc"` 在 Phi-2（head_size=80）上运行时崩溃。原因是 Hadamard 旋转矩阵强制构建为 2 的幂次（128×128），导致与 80 维的 Key 执行 matmul 时报错。
*   **新特性请求**：请求为 Qwen3.8-Flash-Next（`Qwen4Exp` 架构）添加端到端的流水线并行 支持。

### 🔧 Pull Request 动态
*   **新模型支持**：新增对 LiquidAI LFM3-1.7B 模型（`Lfm3ForCausalLM`）的原生支持。
*   **性能与编译优化**：
    *   引入 MoE FlashInfer 融合编译 pass，将 MoE finalize、残差加、all-reduce 与 RMSNorm 融合（pattern code 7）。
    *   优化 MoE 路由性能，跳过 Muse Spark 预备路由路径中的 CUDA-graph padding，并将分组路由的 padding masks 融合到 top-k 内核中。
*   **投机解码 增强**：
    *   引入独立的初步草稿图上限（`draft_prefill_cudagraph_max_tokens`），允许保留大目标 prefill 覆盖范围。
    *   避免 EAGLE3 路径中冗余的辅助状态拼接操作。
    *   修复缓存提升后窗口化草稿 AOT 保护丢失、自适应验证成本分析漏算 MoE 路由开销等 Bug。
*   **底层与硬件支持修复**：
    *   [ROCm] 为 Kimi-K3 支持带有专用解码的 Mooncake 消费者，避免回退到存储路径。
    *   [Kernel] 修复 Marlin MoE 内核中由于共享缓冲区重用导致的内存竞争问题。
*   **基准测试工具**：修复 Rust 版 `vllm-bench` 中 `--output-len` 参数在 `sharegpt` 和 `hf` 数据集下被静默忽略的问题。

### 🚀 Release 动态
*   本次数据中未包含近期的 Release 信息。

---

## 🐛 Issues

### #64956 — [[Bug]: kv_cache_dtype="turboquant_3bit_nc" crashes on Phi-2 (head_size=80) — supports_head_size() claims "any positive value", but the Hadamard rotation is built at the next power of two (128×128) and matmul'd with an 80-dim key](https://github.com/vllm-project/vllm/issues/64956)
- **作者**: jyfct356  **时间**: 2026-11-01 12:41 CST
- **标签**: bug
- **摘要**: ### Your current environment  ``` ==============================         System Info ============================== OS                           : Ubuntu 22.04.4 LTS (x86_64) GCC version                  : (Ubuntu 12.3.0-1ubuntu1~22.04.3) 12.3.0 Clang version                : 17.0.6 (https://github.…

### #64954 — [[Bug]: `--attention-backend TORCH_SDPA` passes config validation, then dies with "Backend TORCH_SDPA must be registered before use" — a ViT-only tag offered as a valid option, whose `ValueError` bypasses the "Selected backend ... is not valid" path](https://github.com/vllm-project/vllm/issues/64954)
- **作者**: jyfct356  **时间**: 2026-11-01 12:33 CST
- **标签**: bug
- **摘要**: ### Your current environment  ``` ==============================         System Info ============================== OS                           : Ubuntu 22.04.4 LTS (x86_64) GCC version                  : (Ubuntu 12.3.0-1ubuntu1~22.04.3) 12.3.0 Clang version                : 17.0.6 (https://github.…

### #64953 — [[Bug]:  `seed=-1` passes config validation and kills engine init inside `np.random.seed` ("Seed must be between 0 and 2**32 - 1") — while `SamplingParams` already treats `-1` as "no seed"](https://github.com/vllm-project/vllm/issues/64953)
- **作者**: jyfct356  **时间**: 2026-11-01 12:28 CST
- **标签**: bug
- **摘要**: ### Your current environment  ``` ==============================         System Info ============================== OS                           : Ubuntu 22.04.4 LTS (x86_64) GCC version                  : (Ubuntu 12.3.0-1ubuntu1~22.04.3) 12.3.0 Clang version                : 17.0.6 (https://github.…

### #64952 — [[Feature]: Add pipeline parallelism support for Qwen3.8-Flash-Next (Qwen4Exp)](https://github.com/vllm-project/vllm/issues/64952)
- **作者**: Lethobenthos20  **时间**: 2026-11-01 12:26 CST
- **标签**: feature request
- **摘要**: ### 🚀 The feature, motivation and pitch  ## Motivation  Qwen3.8-Flash-Next (the `Qwen4Exp` architecture in vLLM) is not currently supported end to end with pipeline parallelism (PP). This blocks downstream integrations from enabling PP on their main branches, even when backend-specific pieces are av…

## 🔀 Pull Requests

### #64959 — [[Model] Add LiquidAI LFM3 (Lfm3ForCausalLM)](https://github.com/vllm-project/vllm/pull/64959)
- **作者**: atmikshetty  **时间**: 2026-11-01 13:58 CST
- **标签**: documentation, new-model
- **摘要**: ## Overview  Add native support for [LiquidAI/LFM3-1.7B](https://huggingface.co/LiquidAI/LFM3-1.7B) (`Lfm3ForCausalLM`), step 2 of #64645.  Depends on #64647 by @abhijithneilabraham (width-7 `causal_conv1d`). That commit is included here unchanged, with its original authorship, so this branch runs a…

### #64958 — [[ROCm][Kimi-K3] Support Mooncake consumers with specialized decode](https://github.com/vllm-project/vllm/pull/64958)
- **作者**: whx-sjtu  **时间**: 2026-11-01 13:42 CST
- **标签**: rocm, kimi, k3, mooncake
- **摘要**: Draft for early review. No blocking issues.  ## Overview  The Kimi-K3 specialized decode on MI355X accepts only a MoRIIO consumer as its KV connector (#63031), so a Mooncake decode falls back to the stock path. A built-in Mooncake consumer meets the same condition: its loads finish before the reques…

### #64957 — [feat(compilation): add MoE FlashInfer fusion pass for kMoEFinalizeARResidualRMSNorm (#40544)](https://github.com/vllm-project/vllm/pull/64957)
- **作者**: YuvrajSinghBhadoria2  **时间**: 2026-11-01 12:59 CST
- **标签**: torch.compile, nvidia
- **摘要**: ## Summary  Implements FlashInfer fused MoE finalize + residual add + all-reduce + RMSNorm (pattern code 7) as a compiler pass.  ## Changes  1. **vllm/compilation/passes/fusion/moe_flashinfer_fusion.py** (new)    - MoEFlashInferFusionPass — pattern matcher for UnfinalizedMoEOutput → finalize → add →…

### #64955 — [[Perf][MoE] Fuse padding skipping in grouped and Muse prepared routing](https://github.com/vllm-project/vllm/pull/64955)
- **作者**: WoosukKwon  **时间**: 2026-11-01 12:35 CST
- **标签**: documentation, ready, ci/build
- **摘要**: ## Overview  Skip CUDA-graph padding in Muse Spark's prepared NVFP4 routing path, and fuse existing grouped-routing padding masks into the CUDA top-k kernels. Padding produces `-1` expert IDs and zero weights without separate CUDA masking launches.  ## Claims  - Muse's prepared TRT-LLM path now supp…

### #64951 — [[Bugfix][Spec Decode] Preserve windowed draft AOT guard after cache promotion](https://github.com/vllm-project/vllm/pull/64951)
- **作者**: mesaleh  **时间**: 2026-11-01 12:18 CST
- **标签**: bug, speculative-decoding, ci/build, mrv2, dflash
- **摘要**: ## Overview  Keep the existing DFlash windowed-draft FA3 AOT safeguard after full cache allocation. Use the draft implementations' computation windows when the allocation-spec helper no longer reports one.  ## Claims  - Preserve the disabled AOT schedule for windowed DFlash/DFlash2/DSpark computatio…

### #64950 — [[Spec Decode] Add an independent first-draft graph ceiling](https://github.com/vllm-project/vllm/pull/64950)
- **作者**: mesaleh  **时间**: 2026-11-01 12:18 CST
- **标签**: documentation, speculative-decoding, ci/build, nvidia, mrv2
- **摘要**: ## Overview  Add `draft_prefill_cudagraph_max_tokens` for target-dependent V2 speculators. Users can retain large target prefill coverage while limiting first-draft PIECEWISE captures and preserving FULL decode graphs.  ## Claims  - Optional startup control for EAGLE/EAGLE3, single-module MTP and Ge…

### #64949 — [[Bugfix][Kernel] Synchronize Marlin MoE shared-buffer reuse](https://github.com/vllm-project/vllm/pull/64949)
- **作者**: mesaleh  **时间**: 2026-11-01 12:18 CST
- **标签**: bug
- **摘要**: ## Overview  Synchronize Marlin MoE's final shared weight reads before reduction reuses the same storage.  ## Claims  - Removes the observed discarded-prefetch shared-memory race. - Preserves exact output and workspace bytes in the captured GPTQ and MXFP8 native calls.  ## Validation  H100, CUDA 13.…

### #64948 — [[Spec Decode] Avoid redundant EAGLE3 auxiliary concatenation with fc_norm](https://github.com/vllm-project/vllm/pull/64948)
- **作者**: mesaleh  **时间**: 2026-11-01 12:17 CST
- **标签**: speculative-decoding, llama, mrv2
- **摘要**: ## Overview  Avoid the raw auxiliary-state concatenation before per-chunk `fc_norm` in the V2 Llama-family EAGLE3 path. Current code concatenates, splits for normalization, then concatenates again.  ## Claims  - Removes one raw auxiliary allocation while preserving the incumbent norm and FC operatio…

### #64947 — [[Bugfix] Include MoE routes in adaptive verification cost profiling](https://github.com/vllm-project/vllm/pull/64947)
- **作者**: mesaleh  **时间**: 2026-11-01 12:17 CST
- **标签**: bug, speculative-decoding, mrv2
- **摘要**: ## Overview  Include routed-expert work when calibrating adaptive verification costs, while preserving FULL CUDA graph dispatch.  ## Claims  - Cost-profile dummy tokens route to MoE experts with `VLLM_MOE_SKIP_PADDING=1`. - Ordinary dummy tokens remain padding; cost profiling does not force eager ex…

### #64946 — [[Rust][Benchmark] Map --output-len to the sharegpt and hf datasets](https://github.com/vllm-project/vllm/pull/64946)
- **作者**: ZhenchengLin  **时间**: 2026-11-01 12:14 CST
- **标签**: rust
- **摘要**: ## Purpose  Fixes #64939.  `--output-len` is silently ignored for `--dataset-name sharegpt` and `hf` in the Rust `vllm-bench`: each request's output length comes from the dataset's reference answer instead. Python `vllm bench serve` maps the flag onto both datasets, and Rust already does so for rand…
