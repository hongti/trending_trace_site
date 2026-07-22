# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-22 09:03 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

###  ISSUE 动态
共 2 个突出问题，均与底层机制或模型初始化异常相关：
*   **推理解析器初始化失败 (#49379)**：服务 `poolside/Laguna-S-2.1` 模型时，其专属的 `poolside_v1` 推理/工具解析器无法正确自动初始化，异常回退到默认的 `IdentityReasoningParser`。
*   **KV Cache 错误命中风险 (#49377)**：当请求的 Token 被原地截断时，若 `Request.block_hashes` 未同步更新，残留的旧哈希值会导致后续指纹比对失误，引发错误的 KV Cache 命中。

---

### 🐍 PR 动态
共 10 个 PR，涵盖新内核引入、架构重构、多类 Bug 修复、性能优化及 CI/文档改进：

**🚀 新特性与架构重构**
*   **新增 FlashInfer FP4 量化内核 (#49382)**：引入 `FlashInferW4A16NvFp4LinearKernel` (mm_bf16_fp4)，支持在内核内部对 FP4 权重进行反量化，同时保持激活值为 bf16，充分利用 FlashInfer 0.6.14 新特性。
*   **ModelOpt 量化方法重构 (#49381)**：针对原本 6 个近乎重复的 `LinearMethod` 类（对应 FP8/NVFP4 等不同格式），重新设计为基于通用 `QuantKey` 驱动的方法，大幅减少代码冗余。

**🔧 核心 Bug 修复**
*   **Aria 模型 TP>1 修复 (#49378)**：修复 Aria 模型在张量并行(TP>1)时，`shared_experts` 因未设置 `reduce_results=False` 导致输出被双重 All-Reduce 的错误。
*   **ColQwen3.5 Attention 不匹配修复 (#49372)**：修正之前代码让 ColQwen3.5 池化模型无条件使用双向 Attention 的错误，使其符合该检索模型官方 Checkpoint 的 Attention 机制契约。
*   **ROCm AITER KV Cache 融合修复 (#49373)**：修复 ROCm 环境下 AITER 注意力融合在处理 packed KV-cache 布局 `[BLOCKS, HEADS, BLOCK_SIZE, 2*HEAD_DIM]` 时，因旧版 `unbind` 逻辑导致的维度拆分错误。
*   **ROCm FP8 KV Cache 测试修复 (#49380)**：修复 ROCm (MI300/gfx94x) 环境下 FP8 KV Cache dtype 在 Attention backend 测试中产生非有限值的问题。

**⚡ 性能优化**
*   **Mamba2 Prefill 性能提升 (#49371)**：将 Mamba2 在 prefill 阶段保存 SSM 状态的操作，从 Python 循环逐次拷贝优化为单次批量索引拷贝，大幅减少 GPU->CPU 同步开销。

**🧪 CI 与文档改进**
*   **扩充 ROCm AITER 测试覆盖 (#49375)**：为 ROCm 专属的 AITER 量化（FP8/MXFP4/FP4）及融合 MoE 路径增加针对性单元测试。
*   **延长 H200 CI 超时时间 (#49374)**：因队列迁移后执行时间变长（已耗时超40分钟），将 H200 35GB 队列的 Model Executor CI 超时阈值从 45 分钟上调至 60 分钟。
*   **补充 NVFP4 内核文档 (#49376)**：在 ModelOpt 指南中新增说明，阐述 NVFP4 GEMM 内核的自动选择逻辑及 Marlin weight-only 回退机制，帮助用户排查性能不及预期的原因。

---

### 📦 Release 动态
*   近期无新版本发布信息。

---

## 🐛 Issues

### #49379 — [[Bug]: Poolside Laguna-S-2.1 poolside_v1 reasoning parser falls back to IdentityReasoningParser and fails auto initialization](https://github.com/vllm-project/vllm/issues/49379)
- **作者**: hic-messaoudi  **时间**: 2026-07-22 07:45 CST
- **标签**: bug
- **摘要**: ### Your current environment   vLLM: 0.25.1  ### 🐛 Describe the bug  I'm trying to serve poolside/Laguna-S-2.1 with vLLM using the provided Poolside reasoning/tool parsers.  **Command:**  vllm serve \     --model poolside/Laguna-S-2.1 \     --tensor-parallel-size 8 \     --reasoning-parser poolside_…

### #49377 — [[Bug]: Token truncation leaves stale Request.block_hashes and can cause incorrect KV cache hits](https://github.com/vllm-project/vllm/issues/49377)
- **作者**: Change72  **时间**: 2026-07-22 07:18 CST
- **摘要**: ### Your current environment  <details> <summary>Environment</summary>  Not environment-dependent: this is a code-level invariant break in the scheduler / prefix-hashing path, reproduced against `main` @ `f25953cc59f9b4ba9b04b16228d2b86dcfbcbdb1` with the pure-Python script below (no GPU required). …

## 🔀 Pull Requests

### #49382 — [[Kernel] Add FlashInferW4A16NvFp4LinearKernel (FlashInfer mm_bf16_fp4)](https://github.com/vllm-project/vllm/pull/49382)
- **作者**: yichengj0  **时间**: 2026-07-22 08:01 CST
- **标签**: nvidia
- **摘要**: ## Purpose  FlashInfer 0.6.14, the version vLLM already pins, added `mm_bf16_fp4` (flashinfer-ai/flashinfer#3597): a GEMM that keeps activations in bf16 and dequantizes the FP4 weight inside the kernel, tuned for DGX Spark (SM121). This PR wires it into vLLM's NVFP4 weight-only (W4A16) linear path, …

### #49381 — [[ModelOpt] Redesign the LinearMethod classes using the generic QuantKey-driven method](https://github.com/vllm-project/vllm/pull/49381)
- **作者**: juhi10071998  **时间**: 2026-07-22 07:53 CST
- **摘要**: ## TL;DR  ModelOpt linear quantization is implemented today as **six near-duplicate `LinearMethod` classes**, one per format (FP8 per-tensor, FP8 per-channel/per-token, FP8 block-weight-only, NVFP4 W4A4, NVFP4 W4A16, MXFP8). This PR replaces all six with **one generic `ModelOptLinearMethod`**, compo…

### #49380 — [[CI][Bugfix] Fix ROCm FP8 KV cache dtype in attention backend test](https://github.com/vllm-project/vllm/pull/49380)
- **作者**: peizhang56  **时间**: 2026-07-22 07:51 CST
- **标签**: bug, rocm, ready, v1
- **摘要**: ## Purpose  `tests/v1/attention/test_attention_backends.py::test_causal_backend_correctness[fp8*]` fails on ROCm (gfx94x / MI300) with `[AttentionBackendEnum.TRITON_ATTN] produced non-finite values`.  Root cause is in the test, not the backend. The test stores the FP8 KV cache using a hardcoded `tor…

### #49378 — [[Bugfix] Aria: set reduce_results=False on shared_experts to fix TP>1](https://github.com/vllm-project/vllm/pull/49378)
- **作者**: mansourthr  **时间**: 2026-07-22 07:43 CST
- **标签**: bug
- **摘要**: Aria passes shared_experts to FusedMoE but does not set reduce_results=False. So the shared expert all-reduces its output inside LlamaMLP, then FusedMoE all-reduces the total again. The shared part is counted twice. At tp=1 this does not matter. At tp=2 or higher, output is wrong.  Fix: add reduce_r…

### #49376 — [[Docs] Document NVFP4 GEMM kernel selection and Marlin weight-only fallback](https://github.com/vllm-project/vllm/pull/49376)
- **作者**: harjothkhara  **时间**: 2026-07-22 07:15 CST
- **标签**: documentation
- **摘要**: ## Purpose  Users hitting slow NVFP4 performance have no docs explaining which GEMM kernel vLLM picked or why. This adds a short note to the ModelOpt guide covering:  - Kernel selection happens automatically at load time, based on what the GPU supports. - GPUs without a native FP4 GEMM kernel fall b…

### #49375 — [[ROCm][CI] Add More AITER quantization/MoE kernel tests](https://github.com/vllm-project/vllm/pull/49375)
- **作者**: micah-wil  **时间**: 2026-07-22 06:13 CST
- **标签**: rocm, ci/build
- **摘要**: This PR expands ROCm kernel test coverage for AITER quantization and fused-MoE paths. Previously these ROCm-specific code paths (FP8, MXFP4/FP4, and AITER fused MoE) had little to no targeted kernel-level testing on AMD hardware.

### #49374 — [[CI] Increase Model Executor timeout on h200_35gb](https://github.com/vllm-project/vllm/pull/49374)
- **作者**: khluu  **时间**: 2026-07-22 05:57 CST
- **标签**: ci/build
- **摘要**: ## Summary  Increase the `Model Executor` Buildkite timeout from 45 to 60 minutes on the `h200_35gb` queue.  ## Why  After the queue migration in #43024, the job took 40:01 in nightly build 78813 and 41:43 in nightly build 79066. The latest run had only 3:17 of headroom under the existing 45-minute …

### #49373 — [[Bugfix][ROCm] Fix ROCM_AITER_FA & ROCM_AITER_UNIFIED_ATTN QK-Norm+RoPE+KVCache fusion for the packed KV-cache [BLOCKS, HEADS, BLOCK_SIZE, 2*HEAD_DIM] layout](https://github.com/vllm-project/vllm/pull/49373)
- **作者**: jhu960213  **时间**: 2026-07-22 05:29 CST
- **标签**: bug, rocm, v1
- **摘要**: ## Purpose - FA fused do_qk_norm_rope_kvcache_update used stale unbind(1) (broke on the (nb, nkvh, bs, 2*hs) packed layout, num_kv_heads != 2); fixed via a shared _split_kv_cache (transpose(1,2).split) matching unified. - FA fused_qk_norm_rope_kvcache_supported is now gated and not is_shuffle_kv_cac…

### #49372 — [[Bugfix] Respect declared attention contract for ColQwen3.5 retrievers](https://github.com/vllm-project/vllm/pull/49372)
- **作者**: athrael-soju  **时间**: 2026-07-22 05:15 CST
- **标签**: bug, multi-modality, qwen
- **摘要**: ## Purpose  Fix the attention mismatch between vLLM and released ColQwen3.5 retrieval checkpoints.  Assisted-by: OpenAI Codex  #46108 made ColQwen3.5 pooling models unconditionally bidirectional. The VultronRetriever checkpoints were trained and evaluated with causal attention, while the older `athr…

### #49371 — [[Perf] Batch Mamba2 prefill SSM state saves into one indexed copy](https://github.com/vllm-project/vllm/pull/49371)
- **作者**: samuelkim7  **时间**: 2026-07-22 05:02 CST
- **摘要**: ## Purpose  In the `mamba_cache_mode="all"` prefill path, `MambaMixer2` saves the block-aligned intermediate SSM states with a Python loop over the prefill sequences. Each iteration costs a GPU->CPU sync and a separate copy kernel.  This PR batches the save into one indexed copy for the whole batch.…
