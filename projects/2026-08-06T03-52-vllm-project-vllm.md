# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-06 11:52 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 📢 Release（版本发布）
*   本期暂无新的 Release 版本发布。但从近期动态来看，社区正为 **DeepSeek V4** 的高性能推理及 **Intel XPU** 的底层支持做密集的功能铺垫。

---

### 💡 Issue（功能请求与 RFC）
*   **DeepSeek V4 标记保留** (#51223)：请求在 Chat Completion 内容中增加保留 `<think>` 标记的选项，以适配 DeepSeek-V4-Flash 的推理/工具调用模式。
*   **Rust 前端 RL 控制面对齐** (#51220)：RFC 提议完善 Rust 前端开发模式路由器中的 RL（强化学习）控制面端点，使其与 Python 端对齐（如 `/pause`, `/resume`, `/abort_requests` 等）。
*   **Intel XPU 推理上游路线图** (#51214)：提出 ARK H2 计划，旨在将 Intel AutoRound 量化工具包及 INC Path XPU 推理能力上游至 vLLM。

---

### 🔧 PR（代码合并与优化）

**🚀 新特性与模型支持**
*   **DeepSeek-V4 IndexCache** (#51209)：为 DeepSeek-V4 引入 DSA IndexCache 支持，结合 DSpark 加速，大幅提升 V4-Flash 的服务性能。
*   **Cosmos3-Edge 视频修剪** (#51221)：为 `nvidia/Cosmos3-Edge` 模型新增 EVS 视频修剪支持（复用 Qwen3-VL 的 prune + mRoPE 路径）。

**🐛 Bug 修复**
*   **EPD 编码器缓存崩溃** (#51222)：修复了纯编码器实例在处理多模态项时触发 `RuntimeError: Encoder cache miss` 的问题。
*   **遥测 HTTP 会话泄漏** (#51219)：修复了使用遥测时 HTTP 会话未正确关闭导致的资源泄漏问题。
*   **KV Cache 规格类型误报** (#51218)：修复了 `UniformTypeKVCacheSpecs` 在多内部类型时错误返回 `UNKNOWN` 而非 `FULL_ATTENTION` 的问题。

**⚡ 性能与核心优化**
*   **MoE 激活泛化** (#51217)：集中处理了 padded expert-major 张量的 count-aware masked MoE 激活逻辑，统一了 Humming、Triton 和 Marlin 等多后端的调用路径。
*   **ModelRunner V2 索引优化** (#51210)：通过改进取值方式（`__getitem__`）及使用更匹配的数据类型（`np.intp`, `torch.int64`），索引速度提升约 6%。

**🖥️ 硬件生态适配**
*   **ROCm/AMD 稀疏索引增强** (#51216)：允许 ROCm 稀疏 MLA 索引后端使用 16-token 的 KV 缓存块大小，从而激活 AITER 的预洗牌 FP8 paged-attention 能力。
*   **Intel XPU Attention 文档** (#51215)：补充了 Intel XPU 平台支持的 Attention 后端列表（`FLASH_ATTN`, `TRITON_ATTN`）。

**🧪 测试改进**
*   **MultiConnector 测试解耦** (#51213)：移除了硬编码的 KV cache 块大小和外部匹配 token 数，使多连接器一致性测试不再依赖特定的 block-size。

---

## 🐛 Issues

### #51223 — [[Feature]: DeepSeek V4: option to preserve <think>/</think> markers in chat completion content](https://github.com/vllm-project/vllm/issues/51223)
- **作者**: Bowen-Leee  **时间**: 2026-08-06 11:51 CST
- **标签**: feature request
- **摘要**: ### 🚀 The feature, motivation and pitch  ### Environment - vLLM v0.25.1 - DeepSeek-V4-Flash - Startup: --tokenizer-mode deepseek_v4 --tool-call-parser deepseek_v4 --enable-auto-tool-choice   (--reasoning-parser is NOT configured)  ### Repro POST /v1/chat/completions with: {   "chat_template_kwargs":…

### #51220 — [[RFC][Rust Frontend] Complete RL control-plane endpoint parity](https://github.com/vllm-project/vllm/issues/51220)
- **作者**: aoshen02  **时间**: 2026-08-06 10:48 CST
- **摘要**: ## Motivation  The Rust frontend's development-mode router already implements the basic RL control endpoints:  - `POST /pause` - `POST /resume` - `GET /is_paused` - `POST /abort_requests` - `GET /get_world_size`  However, it does not implement the weight-transfer / online weight-update lifecycle exp…

### #51214 — [[RFC]: ARK H2 Upstream Roadmap for vLLM INC Path XPU Inference](https://github.com/vllm-project/vllm/issues/51214)
- **作者**: Zhenzhong1  **时间**: 2026-08-06 09:28 CST
- **标签**: RFC, quantization
- **摘要**: # ARK H2 Upstream Roadmap for vLLM INC Path XPU Inference  ## 1. Objective  Intel AutoRound is an advanced toolkit designed to support the quantization of LLMs on Intel platforms.  The H2 goal is to make AutoRound Kernel (ARK) a production-ready acceleration toolkit for AutoRound-quantized models on…

## 🔀 Pull Requests

### #51222 — [[Bugfix][EPD][Model Runner V2] Skip gather mm embeddings for encoder only instance](https://github.com/vllm-project/vllm/pull/51222)
- **作者**: gty111  **时间**: 2026-08-06 11:25 CST
- **标签**: bug, mrv2
- **摘要**: ## Purpose  An EPD encoder instance dies with `RuntimeError: Encoder cache miss` as soon as it is handed a multi-modal item the EC connector already holds.  ```   gpu/model_runner.py:1392  → model_state.get_mm_embeddings(...)   gpu/mm/encoder_runner.py:180 → raise RuntimeError(f"Encoder cache miss f…

### #51221 — [[Model] Add EVS video pruning support for Cosmos3-Edge](https://github.com/vllm-project/vllm/pull/51221)
- **作者**: JoeScharpf  **时间**: 2026-08-06 11:16 CST
- **标签**: performance, multi-modality
- **摘要**: ## Summary - Enable `--video-pruning-rate` on `nvidia/Cosmos3-Edge` by implementing `SupportsMultiModalPruning` (same Qwen3-VL EVS prune + mRoPE recompute path; no deepstack). - Forward `timestamps` through video input parsing and prune/rewrite video embeddings after the vision tower. - Add processo…

### #51219 — [[Bugfix] Close usage telemetry HTTP sessions](https://github.com/vllm-project/vllm/pull/51219)
- **作者**: matteso1  **时间**: 2026-08-06 10:37 CST
- **标签**: bug
- **摘要**: ## The bug  #6600 intentionally moved vLLM's HTTP callers onto a shared reusable session. That remains useful for repeated asset and media requests. Usage reporting sends during startup and then once every ten minutes. The shared client leaves an external HTTPS socket in the frontend between those r…

### #51218 — [[Bugfix] Report FULL_ATTENTION for uniform-base UniformTypeKVCacheSpecs groups instead of UNKNOWN](https://github.com/vllm-project/vllm/pull/51218)
- **作者**: yifjiang  **时间**: 2026-08-06 10:30 CST
- **标签**: bug
- **摘要**: ## Problem  `get_kv_cache_spec_kind()` returns `KVCacheSpecKind.UNKNOWN` for a `UniformTypeKVCacheSpecs` whose members have more than one *inner* kind:  ```python if isinstance(kv_cache_spec, UniformTypeKVCacheSpecs):     inner_kinds = {get_kv_cache_spec_kind(spec) for spec in kv_cache_spec.kv_cache…

### #51217 — [[MoE] Generalize masked activation for batched experts](https://github.com/vllm-project/vllm/pull/51217)
- **作者**: mgoin  **时间**: 2026-08-06 10:28 CST
- **摘要**: ## Summary  - centralize count-aware masked MoE activation for padded expert-major tensors - route Humming, batched Triton, Marlin, and Triton MoE through `ApplyMoEActivationConfig` - make Humming choose its GEMM layout automatically from the activation format and parallelism, while retaining valida…

### #51216 — [[ROCm][AMD] Enable preshuffled sparse indexing for 16-token blocks](https://github.com/vllm-project/vllm/pull/51216)
- **作者**: jamesETsmith  **时间**: 2026-08-06 10:03 CST
- **标签**: rocm
- **摘要**: ## Purpose  Allow ROCm sparse MLA indexer backends to use KV-cache block sizes aligned to 16 tokens. Previously, `--block-size 16` selected kernel block size 1, disabling AITER’s preshuffled FP8 paged-MQA path. The preshuffled AITER kernel was more performant than the non-shuffled variant so we were…

### #51215 — [[Docs] List Intel XPU attention backends](https://github.com/vllm-project/vllm/pull/51215)
- **作者**: baodii  **时间**: 2026-08-06 09:51 CST
- **标签**: documentation, intel-gpu, ready, verified, build-docs
- **摘要**: ## Motivation  Document the attention backends available on Intel XPU platforms in the quickstart guide.  ## Modifications  - Added the supported Intel XPU attention backends:   `FLASH_ATTN`, `TRITON_ATTN`, `TRITON_MLA`, `XPU_MLA_SPARSE`,   `TORCH_SDPA`, and `TURBOQUANT`.  ## Testing  Not run; docum…

### #51213 — [[Test] Make MultiConnector consistency test block-size agnostic](https://github.com/vllm-project/vllm/pull/51213)
- **作者**: zhenwei-intel  **时间**: 2026-08-06 09:22 CST
- **标签**: kv-connector
- **摘要**: ## Purpose `test_multi_example_connector_consistency` hard-codes the number of KV cache blocks (`num_blocks=[7]`) and external matched tokens (`96`) in its `update_state_after_alloc` assertions. These values depend on the KV cache block size (16 by default), so the test only passes on backends using…

### #51210 — [[ModelRunner V2] Minor indexing optimizations](https://github.com/vllm-project/vllm/pull/51210)
- **作者**: njhill  **时间**: 2026-08-06 07:51 CST
- **标签**: ready, mrv2
- **摘要**: - Use `__getitem__` rather than `get` with `np.fromiter(map(...))` (~6% faster) - Use `np.intp` for indexing types (avoids copy/type-conversion every time it's used) - Use `torch.int64` for gpu tensor index mapping (same reason)  Idea came from this PR: https://github.com/vllm-project/vllm/pull/5053…

### #51209 — [[Feature] IndexCache for DeepSeek-V4 (validated on V4-Flash-0731, including DSpark on to accelerate on top of ](https://github.com/vllm-project/vllm/pull/51209)
- **作者**: DiegoCao  **时间**: 2026-08-06 07:47 CST
- **标签**: documentation, deepseek
- **摘要**: ## Summary:  This work is built to support Dspark + IndexerCache to achieve best deepseek v4 flash serving performance.  ## Purpose  Adds DSA IndexCache support to DeepSeek-V4, so C4A layers marked shared reuse the top-k the previous C4A layer left in the shared `topk_indices_buffer` instead of runn…
