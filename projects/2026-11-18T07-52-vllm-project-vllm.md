# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-18 15:52 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期的动态摘要：

### 🛠️ Pull Requests (PR)
本期共有 10 个 PR，主要聚焦于 **ROCm 性能优化、前端 API 扩展、快速启动及底层架构改进**：

1. **ROCm 性能与硬件支持**：
   - **#68334**：优化 ViT fp8 attention 路径，避免实例化不必要的填充 bf16 张量，提升性能。
   - **#68332**：修复 MXFP4 ASM GEMM 被无关 Triton 表误禁用的问题。
   - **#68325**：为 AMD Radeon AI PRO R9700 (gfx1201) 添加了 W8A8 block-FP8 Triton GEMM 的调优配置。
2. **前端与 API 扩展**：
   - **#68333**：`/v1/completions` 和 `/v1/chat/completions` 端点开始支持接收 `kv_hints` 字段并传递给引擎。
3. **架构与新特性**：
   - **#68331**：实现了 RFC #51639，为 KVConnector 增加了通用的控制 RPC 通道，允许外部调用者发送不透明字节负载。
   - **#68326**：针对 Qwen4Exp，在 NVLink 连接的 DP ranks 之间按行拆分 PLE GPU 表，优化显存与性能。
4. **加载与启动优化**：
   - **#68328**：在 Marlin 后端（FP16/BF16 激活）下，支持加载预先处理的 compressed-tensors WNA16 权重，加速 Fast Start。
   - **#68329**：移除了 `torchao` 的 reload decorator（`support_quantized_model_reload_from_hp_weights`），简化了第三阶段的 reload 生命周期。
5. **基准测试与构建**：
   - **#68327**：在多轮基准测试中，新增了服务端侧的前缀缓存命中率统计。
   - **#68330**：将 CUDA 构建的 `tilelang` 依赖版本升级至 0.1.14。

### ⚠️ Issue 动态
* 本次提供的动态数据中未包含 Issue 相关内容。

### 🚀 Release 动态
* 本次提供的动态数据中未包含 Release 相关内容。

---

## 🔀 Pull Requests

### #68334 — [[ROCm][Perf] Avoid materializing padded bf16 in the ViT fp8 attention path](https://github.com/vllm-project/vllm/pull/68334)
- **作者**: sahirema  **时间**: 2026-11-18 15:47 CST
- **标签**: rocm
- **摘要**: ## Purpose  `_vit_fp8_asm_segment` quantizes q/k/v for the padded-head fp8 assembly FMHA. For each of the three tensors it currently:  1. materializes a padded BF16 tensor (head_dim → 128), 2. takes `abs().amax()` over that **padded** tensor, 3. builds an FP32 copy of the padded tensor, converts to …

### #68333 — [[Frontend] Accept kv_hints on /v1/completions and /v1/chat/completions](https://github.com/vllm-project/vllm/pull/68333)
- **作者**: liranschour  **时间**: 2026-11-18 15:47 CST
- **标签**: frontend
- **摘要**: ## Overview  Implements the OpenAI-entrypoint part of RFC #53421: `/v1/completions` and `/v1/chat/completions` accept a `kv_hints` field and pass the `KvHintsEnvelope` to `engine_client.generate(kv_hints=...)`. #53423 added the envelope and carried it from `AsyncLLM` to the scheduler, but the Python…

### #68332 — [[ROCm][Perf] Stop disabling the MXFP4 ASM GEMM on an unrelated Triton table](https://github.com/vllm-project/vllm/pull/68332)
- **作者**: sahirema  **时间**: 2026-11-18 15:45 CST
- **标签**: rocm
- **摘要**: ## Purpose  `MxFp4LinearLayer.process_weights_after_loading` enables the aiter assembly a4w4 GEMM only when the shape is tuned for **both** the ASM kernel and the Triton preshuffle kernel:  ```python self.use_asm_gemm = rocm_aiter_ops.is_asm_fp4_gemm_tuned(n, k_bytes) \     and rocm_aiter_ops.is_tri…

### #68331 — [[KVConnector] Add generic control RPC channel (RFC #51639)](https://github.com/vllm-project/vllm/pull/68331)
- **作者**: liranschour  **时间**: 2026-11-18 15:38 CST
- **标签**: documentation, frontend, kv-connector, scheduler
- **摘要**: ## Overview  Implements [RFC #51639](https://github.com/vllm-project/vllm/issues/51639): a single, generic, connector-defined control channel. An external caller can now send an opaque byte payload to the scheduler-side KV connector and get a byte response back, without adding a bespoke endpoint, en…

### #68330 — [[Build] Bump tilelang to 0.1.14 for CUDA](https://github.com/vllm-project/vllm/pull/68330)
- **作者**: Mramaran  **时间**: 2026-11-18 15:35 CST
- **标签**: ci/build, nvidia
- **摘要**: ## Overview  Bumps the CUDA tilelang pin from 0.1.12 to 0.1.14. Part of #65014.  ## Claims  - `requirements/cuda.txt` now pins `tilelang==0.1.14`. Nothing else changes: `apache-tvm-ffi==0.1.11` already fits 0.1.14's `>=0.1.11,<0.1.13`, and no other package moves when the full CUDA requirements are i…

### #68329 — [[Model Loader] Remove the torchao reload decorator](https://github.com/vllm-project/vllm/pull/68329)
- **作者**: aoshen02  **时间**: 2026-11-18 15:29 CST
- **摘要**: ## Overview  Implements the removal proposed in #65913: the torchao reload decorator (`support_quantized_model_reload_from_hp_weights`, added in #26327) is removed. It was a third reload lifecycle that ran only when user code called `model.load_weights` directly on an already loaded torchao model.  …

### #68328 — [[Fast Start] Support compressed-tensors WNA16 pre-processed weights](https://github.com/vllm-project/vllm/pull/68328)
- **作者**: UNIDY2002  **时间**: 2026-11-18 15:28 CST
- **标签**: documentation, quantization
- **摘要**: ## Overview  Add Fast Start pre-processed weight support for compressed-tensors WNA16 dense linear layers when the selected backend is Marlin with FP16/BF16 activations.  ## Claims  - Reuse Marlin-packed WNA16 weights and scales across CUDA IPC engine restarts. - Register Marlin's empty symmetric ze…

### #68327 — [[Bench] Add server-side cache hit rate to multi-turn benchmark](https://github.com/vllm-project/vllm/pull/68327)
- **作者**: idiot723  **时间**: 2026-11-18 15:26 CST
- **标签**: performance
- **摘要**: ## Overview  Adds a server-reported prefix cache hit rate to the multi-turn serving benchmark. Streaming requests now set `stream_options.include_usage=True` and parse the final usage chunk for `prompt_tokens` and `prompt_tokens_details.cached_tokens`, exposing `cache_hit_rate` per request and in th…

### #68326 — [[Perf][Qwen4Exp] Split PLE GPU tables over NVLink-connected DP ranks](https://github.com/vllm-project/vllm/pull/68326)
- **作者**: ZJY0516  **时间**: 2026-11-18 15:21 CST
- **标签**: documentation, qwen, deepseek
- **摘要**: ## Overview  With `cpu_offload: false`, TP=1 and NVLink between an engine's DP ranks, Qwen4Exp PLE tables are now split by rows over those GPUs, as DeepSeek V4.1 Engram tables already are. Each rank reads its tokens' rows from the owning GPU's HBM, with no DP collective. The peer-row addressing, gro…

### #68325 — [[ROCm][Perf] Add tuned W8A8 block-FP8 Triton GEMM configs for AMD Radeon AI PRO R9700 (gfx1201)](https://github.com/vllm-project/vllm/pull/68325)
- **作者**: BoyuZhang-amd  **时间**: 2026-11-18 15:21 CST
- **标签**: rocm, quantization
- **摘要**: ## Overview  On gfx1201 (AMD Radeon AI PRO R9700) vLLM has no tuned config for the Triton block-FP8 GEMM (`_w8a8_triton_block_scaled_mm`), so every block-FP8 linear layer falls back to the hard-coded default (`BLOCK_SIZE_M=64, BLOCK_SIZE_N=128, BLOCK_SIZE_K=128, num_warps=4, num_stages=2`) and logs …
