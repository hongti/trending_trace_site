# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-23 10:11 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
近期问题主要集中在**性能回退**与**核心算子/后端的严重 Bug**：
*   **性能回退**：自动态 K=0 修复合入后，DSD（动态推测解码）在 K=0 工作负载下，draft-state 同步前向传播仍带来 4-11% 的性能开销，且对 MTP 与 EAGLE3 两种 drafter 架构影响差异显著 (#53416)。
*   **显存溢出 (OOM)**：GLM-5.2 FP8 模型在 8×H200 上运行 FlashMLA sparse decode 混合批次路径时，因 workspace 随 `--max-num-batched-tokens` 线性增长且无上限，导致 CUDA OOM (#53413)。
*   **输出异常**：DeepSeek v4 Pro Base 使用默认 MoE 后端时输出退化且 logprobs 异常偏高，对比 `deep_gemm`，当前自动选择的 FlashInfer TRTLLM 后端存在正确性问题 (#53411)。

---

### 🔧 Pull Request 动态
PR 活动涵盖了前端重构、重要错误修复及推测解码性能优化：

**🚀 架构与前端重构**
*   **Rust Frontend Derender 三部曲推进**：Phase 2 (#53418) 实现了推理和工具调用解析；Phase 3 (#53419) 实现了流式 derender 及双进程端到端测试。

**🐛 关键 Bug 修复**
*   **DeepSeek V4 流式工具调用泄漏**：修复了使用 `deepseek_v4` 解析器时，原始 DSML 标记泄漏到工具调用参数中的问题 (#53417)。
*   **算子溢出修复**：修复了 fused SiLU block quant 中 `token_idx` 使用 32 位整型导致大 batch 下偏移量溢出的问题 (#53409)。
*   **流水线并行修复**：为 `DeepSeekV4MTP` 实现 `SupportsPP` 接口，修复了其在开启 Pipeline Parallelism 时崩溃的问题 (#53408)。
*   **版本兼容性修复**：修复了 `_prev_minor_version_was` 函数因硬编码主版本号 `0` 导致未来 `1.x`+ 版本直接 Assert 崩溃的问题 (#53415)。
*   **ROCm 推测解码回退修复**：修复了 AMD ROCm 环境下，推测解码有一半的 decode batch size 错误回退到 eager 模式而非 CUDA Graph 的问题 (#53407)。

**⚡ 性能优化**
*   **TurboQuant CG 恢复**：在修复正确性后，重新让 spec-decode verify batches 在 FULL cudagraphs 下作为 decodes 运行，恢复性能 (#53410)。
*   **H2D 拷贝优化**：将 `xdrope_positions` 的 Host-to-Device 拷贝拆分为逐行传输，解决了 pinned buffer 步长视图静默回退到 pageable 内存导致的 Host 跑前性能杀手问题 (#53412)。

**🧩 模型与量化支持**
*   **MTP 量化旁路扩展**：扩展了 fc 量化旁路逻辑，支持 Qwen3.5 和 Qwen3-next MTP 模型在 compressed-tensors WNA16 格式下的 BF16 权重 (#53414)。

---

### 🚀 Release 动态
*   **近期暂无新版本发布**。

---

## 🐛 Issues

### #53416 — [[Performance]: DSD K=0 draft-state sync forward costs 4-11% in a K≡0 workload; resume impact differs sharply between the two tested drafter architectures (MTP vs EAGLE3)](https://github.com/vllm-project/vllm/issues/53416)
- **作者**: Suppressor72  **时间**: 2026-08-23 09:50 CST
- **摘要**: ### Misc discussion on performance / Report of performance regression  ## Summary  Since the dynamic-K=0 fix (#51510 / #51575 lineage), a step that resolves K=0 skips the draft *decode* steps but still runs one draft "prefill" forward per step, deliberately kept as per-step draft KV/state sync (`v1/…

### #53413 — [[Bug]: GLM-5.2 FP8 on 8×H200 dies with runtime CUDA OOM in sparse_decode_fwd](https://github.com/vllm-project/vllm/issues/53413)
- **作者**: yuzisun  **时间**: 2026-08-23 07:37 CST
- **标签**: bug, glm
- **摘要**: ### 🐛 Describe the bug     ## Summary     On the FlashMLA sparse **fp8 mixed-batch** path, the sparse decode kernel allocates a workspace that    scales linearly with `--max-num-batched-tokens` and is **not bounded by any workspace limit and not    accounted for during startup memory profiling**. Th…

### #53411 — [[Bug]: DeepSeek v4 Pro Base - The default MoE backend selection produces incorrect output and inflated logprobs (FlashInfer TRTLLM vs deep_gemm)](https://github.com/vllm-project/vllm/issues/53411)
- **作者**: sebastiandero  **时间**: 2026-08-23 07:35 CST
- **标签**: bug, deepseek, kimi
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information... ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (aarch64) GCC…

## 🔀 Pull Requests

### #53419 — [[Rust Frontend] /derender: streaming derender + two-process e2e test (phase 3/3)](https://github.com/vllm-project/vllm/pull/53419)
- **作者**: ezhoureal  **时间**: 2026-08-23 10:06 CST
- **标签**: rust
- **摘要**: ## Purpose  **Stacked on #53418 (phase 2/3)** — review only the top commit(s). Split per @sagearc's review on #53223, mirroring the Python derender phasing in #42729 (#43606 detok → #45919 parsing → #48617 streaming).  Phase 3: **streaming derender** for both `/v1/chat/completions/derender` and `/v1…

### #53418 — [[Rust Frontend] /derender: reasoning and tool-call parsing (phase 2/3)](https://github.com/vllm-project/vllm/pull/53418)
- **作者**: ezhoureal  **时间**: 2026-08-23 10:05 CST
- **标签**: rust
- **摘要**: ## Purpose  **Stacked on #53223 (phase 1/3)** — review only the top commit; the rest is the phase-1 PR. Split per @sagearc's review, mirroring the Python derender phasing in #42729 (#43606 detok → #45919 parsing → #48617 streaming).  Phase 2: non-streaming **reasoning/tool-call parsing** in `/v1/cha…

### #53417 — [fix(parser): prevent DSML markup leak in DeepSeek V4 streaming tool calls](https://github.com/vllm-project/vllm/pull/53417)
- **作者**: SparshM8  **时间**: 2026-08-23 10:04 CST
- **标签**: tool-calling, deepseek, DSv4
- **摘要**: ### Description Fixes #53227.  Streaming tool calls on DeepSeek-V4-Pro leak raw DSML markup into tool call arguments when using the `deepseek_v4` tool call parser. This happens because `_PARTIAL_PARAM_RE` uses a greedy `(.*)$` match that captures trailing DSML tags (like `</｜DSML｜parameter`) when th…

### #53415 — [[Bugfix] Fix _prev_minor_version_was crashing on major version >= 1](https://github.com/vllm-project/vllm/pull/53415)
- **作者**: promptsmith1990  **时间**: 2026-08-23 08:47 CST
- **标签**: bug
- **摘要**: ## Summary  `_prev_minor_version_was()` in `vllm/version.py` has a hardcoded `assert __version_tuple__[0] == 0`, so the function will raise `AssertionError` for any future `1.x`+ release instead of doing the minor-version comparison it's meant for. It's called from `VllmConfig` observability handlin…

### #53414 — [fix(quant): bypass fc quantization for compressed-tensors MTP checkpo…](https://github.com/vllm-project/vllm/pull/53414)
- **作者**: williamclymire-tamu  **时间**: 2026-08-23 07:42 CST
- **标签**: qwen, quantization
- **摘要**: …ints  Qwen3.5 and Qwen3-next MTP models store mtp.fc as BF16 in compressed-tensors WNA16 checkpoints, but the existing fc_quant bypass only handled modelopt_fp4. Extend it to also cover compressed-tensors, preventing shape mismatches during weight loading.  Fixes #53387     ## Purpose Fix Qwen3.5 a…

### #53412 — [[Perf] Split xdrope_positions H2D copy into per-row transfers](https://github.com/vllm-project/vllm/pull/53412)
- **作者**: JasonKeyiL  **时间**: 2026-08-23 07:37 CST
- **标签**: mrv1-only
- **摘要**: ## Summary  PR #51841 identified a subtle host-run-ahead killer: `copy_(..., non_blocking=True)` on a **strided view of a pinned buffer** silently falls back to a pageable intermediate, and pageable H2D synchronizes the stream regardless of `non_blocking`. That PR fixed `mrope_positions`. Eight line…

### #53410 — [[Perf] TurboQuant: run spec-decode verify batches as decodes with FULL cudagraphs](https://github.com/vllm-project/vllm/pull/53410)
- **作者**: giannisanni  **时间**: 2026-08-23 07:13 CST
- **标签**: nvidia, quantization
- **摘要**: ## Purpose  Performance follow-up to #53406 (the correctness fix for #52475). That fix stopped FULL cudagraph capture of spec-decode verify batches because the TurboQuant path serving them (the per-request Python prefill loop) is not graph-capturable. Correct, but expensive: with MTP enabled, every …

### #53409 — [[Bugfix] Fix int32 token offset overflow in fused SiLU block quant](https://github.com/vllm-project/vllm/pull/53409)
- **作者**: canlahlah  **时间**: 2026-08-23 07:12 CST
- **标签**: bug, quantization
- **摘要**: ## Purpose  Fixes #53390.  `silu_and_mul_per_block_quant` used a 32-bit `token_idx` when computing global input and output row offsets. For sufficiently large token batches, `token_idx * (2 * hidden_size)` overflowed `INT32_MAX`, wrapped the input pointer, and caused an illegal CUDA memory access.  …

### #53408 — [[Bugfix] DeepSeekV4MTP: implement SupportsPP so the draft can start under PP](https://github.com/vllm-project/vllm/pull/53408)
- **作者**: JasonKeyiL  **时间**: 2026-08-23 07:11 CST
- **标签**: bug, deepseek, DSv4
- **摘要**: ## Summary  `DeepSeekV4MTP` (all three of `nvidia/amd/xpu`) inherits from `nn.Module` only. The moment `--pipeline-parallel-size > 1` is combined with an MTP/DSpark speculative config, `SpeculativeConfig._verify_args` runs `draft_model_config.verify_with_parallel_config(draft_parallel_config)` — `dr…

### #53407 — [[Bugfix][MRV2][ROCm] Dispatch uniform decode to a padded FULL cudagraph](https://github.com/vllm-project/vllm/pull/53407)
- **作者**: xiaohuguo2023  **时间**: 2026-08-23 06:41 CST
- **标签**: bug, rocm, nvidia, mrv2, verified
- **摘要**: ## Purpose  Under speculative decoding, **half of all decode batch sizes** silently fall back to running attention eagerly every decode step instead of replaying a captured CUDA graph. This makes them dispatch to a captured graph instead.  **Current state.** With a separate decode routine (`FULL_AND…
