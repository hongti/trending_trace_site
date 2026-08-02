# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-02 09:06 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
近期 Issue 主要集中在**新硬件兼容性**、**推测解码**及**混合模型/KV缓存**的崩溃或输出异常：

*   **新硬件与底层兼容性**：
    *   **SM120 (Blackwell) 启动失败**：在 RTX PRO 6000 (sm_120) 上，若本地 CUDA Toolkit 低于 12.9，FlashInfer JIT 编译失败会导致引擎初始化直接崩溃，且未降级回退（#50705）。
    *   **SM121 (GB10/DGX Spark) 崩溃**：DFlash 推测解码在 SM121 上自动选择注意力机制时，为非因果 draft 错误选择了 FLASH_ATTN，导致 CUDA 设备端断言失败（#50707）。
*   **推测解码体验差**：使用 DFlash 并设置较大的 `num_speculative_tokens`（如16）时，因默认 batching 限制导致配置构建失败，但报错信息仅给出负数的 `max_num_scheduled_tokens`，未提示修复标志（#50708）。
*   **混合模型与量化 Bug**：
    *   **TurboQuant 混合模型崩溃**：在 v0.25.0+ 版本中，因无法识别 `auto` 缓存数据类型，在 `determine_available_memory` 阶段崩溃（#50709）。
    *   **KV 缓存与 Prefix Caching 冲突**：使用 int8_per_token_head KV 且 KV 池 100% 占用时，Prefix Caching 会导致输出损坏（#50702）。
*   **多模态模型初始化异常**：Mistral Small 3.2 24B (HF格式) 在纯文本 LLM 初始化时，因强制进行多模态性能分析（处理 `[IMG]`）而失败（#50706）。

---

### 🛠️ Pull Request 动态
PR 活动主要围绕**核心 Bug 修复**、**性能优化**及**架构改进**：

*   **重要 Bug 修复**：
    *   **Qwen3.5-MoE 兼容性**：修复了在 transformers 5.x 环境下无法加载 Qwen3.5-MoE 纯文本模型的问题（#50704）。
    *   **DeepSeek-V4 混合 KV 加载**：修复了调度器在处理 DeepSeek-V4-Flash 等具有多个物理 KV 缓存的模型时，获取 block ID 失败的崩溃问题（#50700）。
    *   **KV Offload 数据丢失**：修复了包含 Mamba 层的模型在 CPU->GPU KV 卸载加载时，新分配的 KV 块被静默清零的严重问题（#50696）。
    *   **推测解码修复**：修复了 DFlash/DSpark 在 draft 与 target 模型 GQA 比率不同时，FA3 AOT 调度器头数不匹配的 RuntimeError（#50694）；修复了 DSpark 在无 sparse index buffer 时的 warmup 崩溃（#50693）。
*   **性能与架构优化**：
    *   **Inkling 性能提升**：将 shared-expert 的部分加法操作融合到 Lamport 集合通信中，减少了 element-wise add 内核的启动开销（#50697）。
    *   **Rust Frontend 扩展**：为 Elastic EP（弹性专家并行）扩容添加了 supervisor 端控制通道，以协调 Rust 前端与 Python 进程（#50698）。
*   **文档与工程**：
    *   修复了 `FusedMoE` 重命名为 `FusedMoEFactory` 后文档中的过时引用（#50701）。
    *   修复了 benchmarking 文档中的失效链接（#50695），并添加了 first-interaction GitHub Action（#50703）。

---

### 🚀 Release 动态
*   近期**无新版本发布**。（注：Issue 中提及的 v0.25.0 / v0.26.0 为用户反馈环境版本，非本期新增 Release）。

---

## 🐛 Issues

### #50709 — [[Bug]: TurboQuant hybrid model crashes at determine_available_memory with 'Unknown cache dtype: auto' on v0.25.0+](https://github.com/vllm-project/vllm/issues/50709)
- **作者**: UzkiS  **时间**: 2026-08-02 08:36 CST
- **标签**: bug
- **摘要**: ### Your current environment  - vLLM: v0.25.0 / v0.26.0 - GPU: NVIDIA GeForce RTX 3080 (SM86) × 2, 20GB - OS: Linux (Ubuntu) - CUDA: 13.0 - Python: 3.12 - Model: Qwen3.5-35B-A3B / Qwen3.6-35B-A3B (AWQ INT4, compressed-tensors) - KV cache dtype: turboquant_4bit_nc / turboquant_k8v4 - Launch: tensor-p…

### #50708 — [[Bug] Speculative decoding with a large num_speculative_tokens fails with a bare negative max_num_scheduled_tokens instead of naming the flags that fix it](https://github.com/vllm-project/vllm/issues/50708)
- **作者**: scottleimroth  **时间**: 2026-08-02 07:48 CST
- **摘要**: ﻿ ### Summary  Starting a server with DFlash speculative decoding at `num_speculative_tokens = 16` and default batching fails at config build, before any weights load, with a pydantic validation error reporting a negative token budget. The number is correct; the message gives no indication of which …

### #50707 — [[Bug] DFlash on SM121 (GB10 / DGX Spark): attention autoselect picks FLASH_ATTN for non-causal draft attention and device-asserts in _vllm_fa2_C.varlen_fwd](https://github.com/vllm-project/vllm/issues/50707)
- **作者**: scottleimroth  **时间**: 2026-08-02 07:47 CST
- **摘要**: ### Summary  On GB10 (DGX Spark, `sm_121a`, compute capability 12.1), DFlash speculative decoding crashes at engine startup with a CUDA device-side assert inside the FlashAttention2 kernel. The Python-level backend capability check passes, so FA2 is selected and dispatched, and the failure only appe…

### #50706 — [[Bug]: Mistral3 (HF format): default text-only LLM() init fails in multimodal profiling — "Failed to apply PixtralProcessor on data={'text': '[IMG]'}"](https://github.com/vllm-project/vllm/issues/50706)
- **作者**: manunicholasjacob  **时间**: 2026-08-02 07:27 CST
- **摘要**: ### Your current environment  <details><summary>collect_env output (same runtime family as the reproduction)</summary>  ```text fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc rep_good nopl…

### #50705 — [[Bug]: sm_120 + local CUDA toolkit < 12.9: FlashInfer JIT failures kill engine init in three default paths (sampler, fused-MoE, FP8 KV) instead of falling back](https://github.com/vllm-project/vllm/issues/50705)
- **作者**: manunicholasjacob  **时间**: 2026-08-02 07:26 CST
- **摘要**: ### Your current environment  <details><summary>collect_env output</summary>  ```text fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc rep_good nopl nonstop_tsc cpuid extd_apicid aperfmperf …

### #50702 — [[Bug]: int8_per_token_head KV + prefix caching corrupts output when the KV pool is pinned at 100% (Gemma-4 hybrid, Triton)](https://github.com/vllm-project/vllm/issues/50702)
- **作者**: MDEVOSTATOR  **时间**: 2026-08-02 05:26 CST
- **摘要**: ## Environment - vLLM v0.26.0, V1 engine, Model Runner V1 - 2× RTX 3090 (Ampere/sm86), TP=2, no NVLink (`--disable-custom-all-reduce`) - Model: `cyankiwi/gemma-4-31B-it-AWQ-4bit` (Gemma-4 31B, hybrid attention: 50 sliding layers head_dim 256 + 10 global head_dim 512) - `--attention-backend=TRITON_AT…

## 🔀 Pull Requests

### #50704 — [[Bugfix][Models] Accept Qwen3_5MoeTextConfig in Qwen3_5MoeProcessingInfo for transformers 5.x compatibility](https://github.com/vllm-project/vllm/pull/50704)
- **作者**: loulanyue  **时间**: 2026-08-02 07:09 CST
- **标签**: bug, qwen
- **摘要**: ## Purpose  Fix #50428: vLLM fails to load Qwen3.5-MoE text-only models (e.g. `Qwen3.6-35B-A3B`) when transformers 5.x is installed.  In transformers 5.x, the top-level config class for Qwen3.5-MoE text-only models was renamed from `Qwen3_5MoeConfig` (the multimodal VL wrapper) to `Qwen3_5MoeTextCon…

### #50703 — [Update workflow comments and add first-interaction action.](https://github.com/vllm-project/vllm/pull/50703)
- **作者**: fremontfirstnotlast-netizen  **时间**: 2026-08-02 06:43 CST
- **标签**: ci/build
- **摘要**: ## Purpose  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such as providing test command. - [ ] The …

### #50701 — [[Bugfix][Doc] Fix references to FusedMoE in doc](https://github.com/vllm-project/vllm/pull/50701)
- **作者**: bnellnm  **时间**: 2026-08-02 05:24 CST
- **标签**: bug, documentation, ready
- **摘要**: ## Purpose  The rename of `FusedMoE` -> `FusedMoEFactory` did not update related uses in the docs. This PR replaces most instances of `FusedMoE` with "fused MoE" since it is more of a generic description of the layer. One specific link to the code is replaced with `MoERunner` where the naive all2all…

### #50700 — [[BugFix][KV Connector] Handle DeepSeek-V4 hybrid KV load failures](https://github.com/vllm-project/vllm/pull/50700)
- **作者**: xijiaat  **时间**: 2026-08-02 04:53 CST
- **标签**: bug, deepseek, kv-connector
- **摘要**: ## Purpose  Fixes #50687.  `Scheduler._update_requests_with_invalid_blocks()` currently assumes that `get_block_ids()` returns exactly one block-id list. DeepSeek-V4-Flash-0731 uses five physical KV cache groups with different block grids, so the first connector load failure raises:  ```text ValueEr…

### #50698 — [Rust Frontend: Add supervisor-side control channel for Elastic EP scaling](https://github.com/vllm-project/vllm/pull/50698)
- **作者**: coder3101  **时间**: 2026-08-02 03:52 CST
- **标签**: frontend, rust
- **摘要**: ## Purpose  Phase 3 of https://github.com/vllm-project/vllm/issues/45154   Because the elastic EP orchestration does not live in the frontend process, Rust frontend should needs to co-ordinate with Python supervisor for it. This PR introduces the communication channel for it. More details about the …

### #50697 — [[Perf][Inkling] Fuse shared-expert partial addition into the Lamport collective](https://github.com/vllm-project/vllm/pull/50697)
- **作者**: gcanlin  **时间**: 2026-08-02 02:11 CST
- **摘要**: ## Purpose  Reduce element-wise add kernel launch time.  Before:  ```text   Routed MoE ──> routed_partial ─┐                                  ├─> torch.add_ ─> Lamport RS + SConv + AG + Norm   Shared Sink ─> shared_partial ─┘ ```   After:  ```   Routed MoE ──> routed_partial ─┐                      …

### #50696 — [[KV offload] Order CPU->GPU loads against the compute stream](https://github.com/vllm-project/vllm/pull/50696)
- **作者**: Etelis  **时间**: 2026-08-02 01:45 CST
- **摘要**: On models that zero freshly allocated KV blocks (any model with mamba layers, i.e. `needs_kv_cache_zeroing`), a CPU->GPU load in the offloading connector can be silently wiped.  `SingleDirectionOffloadingHandler.transfer_async` waited on the compute stream only for GPU->CPU. Loads ran on their own s…

### #50695 — [docs: fix broken link in docs/benchmarking/cli.md](https://github.com/vllm-project/vllm/pull/50695)
- **作者**: AgenticSpark  **时间**: 2026-08-02 00:59 CST
- **标签**: documentation
- **摘要**: Replaced a broken relative link to ../cli/bench/mm_processor.md with the canonical online reference (https://docs.vllm.ai/en/latest/cli/bench/mm_processor/). Ran scan_docs.py to verify the original broken-link finding is gone. No other files changed.

### #50694 — [[Spec Decode] Fix FA3 AOT scheduler head count mismatch for DFlash/DSpark](https://github.com/vllm-project/vllm/pull/50694)
- **作者**: elvircrn  **时间**: 2026-08-02 00:08 CST
- **标签**: mrv2
- **摘要**: ## Summary  - Fix `RuntimeError: scheduler_metadata must have shape (metadata_size)` when using DFlash/DSpark speculative decoding with FA3 on models where the draft and target have different GQA ratios (e.g. GLM-5.2-FP8 + DSpark speculator). - Fix `VllmConfig` pydantic validation error when using `…

### #50693 — [Fix DSpark warmup without sparse index buffer](https://github.com/vllm-project/vllm/pull/50693)
- **作者**: xijiaat  **时间**: 2026-08-02 00:04 CST
- **摘要**: ## Purpose  Fixes #50615.  During DSpark startup, `profile_run` calls `forward_mqa` without attention metadata to size the warmup workspace. For the SWA-only draft layer (`compress_ratio <= 1`), `topk_indices_buffer` is intentionally not allocated, but the warmup path asserted that the buffer existe…
