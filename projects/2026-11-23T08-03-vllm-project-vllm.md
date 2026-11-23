# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-23 16:03 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issues
1. **性能与前端优化建议** (#69361)：建议在生成 ORCA 负载指标 JSON 响应头时使用 `orjson` 替代标准库 `json.dumps`，并降低日志级别以减少开销。
2. **依赖兼容性 Bug** (#69358)：报告了内置的 `LMCacheConnectorV1` 在最新版本的 `lmcache` (>=0.5.6) 下无法启动的问题，原因是新版移除了 `lmcache.integration.vllm.vllm_v1_adapter` 模块。

### 🔧 Pull Requests
**新特性与功能**
- **推测解码支持语法感知** (#69360)：为 MRv2 推测解码引入 worker 端的语法感知草拟机制，提升 Agent JSON 工具调用等结构化输出的效率。
- **离线 Pooling API 支持多模态** (#69357)：在离线 pooling API 中新增多模态参数，支持直接透传渲染。
- **LoRA 多适配器组合** (#69356)：支持在离线推理中按请求激活多个独立加载的 LoRA 适配器并进行静态缩放组合。
- **LoRA 按需启用** (#69354)：将 `embedding` 和 `lm_head` 的 LoRA 包装改为可选，默认不为其分配 GPU 显存，通过 `--lora-enable-vocab-modules` 显式开启。

**性能优化**
- **ORCA JSON 序列化加速** (#69362)：采用 `orjson` 加速 ORCA 指标头构建，并将相关日志降级至 DEBUG，直接回应了上述 Issue #69361。
- **ROCm 显存与计算优化**：
  - (#69364) 在 gfx12 (RDNA4) 上，对预填充阶段的 W4A16 运算改用“反量化后执行 GEMM”策略，提升大批次计算性能。
  - (#69363) 针对 Qwen4Exp，当 AITER 使用展开权重时跳过折叠 HC 权重，为每张 GPU 节省约 0.63 GiB 显存。
- **KV Cache 卸载优化** (#69359)：调整执行顺序，优先处理当前步骤需要的带围栏 KV 卸载存储，减少后台大块复制造成的 GPU KV 块等待时间。
- **MoE FP8 计算优化** (#69355)：在 SM12x 架构（如 DGX Spark / GB10）的张量并行场景下，优先使用 Triton 而非默认的 DeepGEMM 来处理 Block FP8 MoE 计算，提升效率。

**Bug 修复**
- **SHM 缓存泄漏修复** (#69365)：修复了在客户端提前断开连接时，共享内存（SHM）缓存读取义务未释放的问题，确保 FIFO 内存回收机制能正常推进。

### 🚀 Release
- 本次更新周期内无新的版本发布。

---

## 🐛 Issues

### #69361 — [[Perf][Frontend] Use orjson and reduce logging overhead for ORCA JSON headers](https://github.com/vllm-project/vllm/issues/69361)
- **作者**: amrsorne  **时间**: 2026-11-23 15:56 CST
- **摘要**: ## Problem  When clients request ORCA load reports with `endpoint-load-metrics-format: JSON`, every chat/completion response builds the header with stdlib `json.dumps` and logs at INFO.  ## Proposal  - Use `orjson` for the JSON branch (consistent with other entrypoint serializers). - Log header crea…

### #69358 — [[Bug]: Built-in LMCacheConnectorV1 broken with declared dependency range lmcache>=0.5.6 (latest 0.5.8 removed lmcache.integration.vllm.vllm_v1_adapter)](https://github.com/vllm-project/vllm/issues/69358)
- **作者**: Tao20060602  **时间**: 2026-11-23 15:26 CST
- **摘要**: ### Summary  Serving with the built-in `LMCacheConnectorV1` fails at engine-core startup when lmcache is installed to the latest release:  ``` ModuleNotFoundError: No module named 'lmcache.integration.vllm.vllm_v1_adapter' RuntimeError: Worker failed with error 'No module named 'lmcache.integration.…

## 🔀 Pull Requests

### #69365 — [[Bugfix] Release SHM cache reader obligation on request abort so FIFO reclaim can advance](https://github.com/vllm-project/vllm/pull/69365)
- **作者**: four1er  **时间**: 2026-11-23 16:02 CST
- **摘要**: ## Problem  Fixes #69352.  With `--mm-processor-cache-type shm`, a request whose client disconnects after the SHM write and before any worker read leaves the item's read obligation outstanding. `default_is_free_check` requires `reader_count >= writer_count * n_readers`, and `free_buf` stops at the f…

### #69364 — [[ROCm][Perf] W4A16: dequantize-then-GEMM for prefill-sized batches on gfx12](https://github.com/vllm-project/vllm/pull/69364)
- **作者**: BoyuZhang-amd  **时间**: 2026-11-23 16:00 CST
- **标签**: rocm, quantization
- **摘要**: ## Overview  On gfx12 (RDNA4), `RDNAHybridW4A16LinearKernel` runs prefill-sized batches (M of several hundred to thousands of rows) through the Triton skinny-format W4A16 GEMM. That kernel is well below the dense GEMM rate on gfx1201, so long-prompt TTFT is slower than it needs to be.  **This PR deq…

### #69363 — [[ROCm][Qwen4Exp] Skip the folded HC weight when AITER takes the unfolded one](https://github.com/vllm-project/vllm/pull/69363)
- **作者**: mjkvaak-amd  **时间**: 2026-11-23 16:00 CST
- **标签**: rocm, qwen
- **摘要**: ## Overview  Follow-up to #59442: drops its 0.63 GiB per GPU folded weight copy when AITER exports `SKINNY_DECODE_UNFOLDED` (ROCm/aiter#8087), passing the original weight with `fold_w=False`. Older AITER keeps the folded path, so this takes effect when the AITER pin moves past ROCm/aiter#8087.  ## V…

### #69362 — [[Perf][Frontend] Speed up ORCA JSON headers with orjson](https://github.com/vllm-project/vllm/pull/69362)
- **作者**: amrsorne  **时间**: 2026-11-23 15:56 CST
- **标签**: frontend
- **摘要**: ## What  - Serialize ORCA `endpoint-load-metrics` JSON payloads with `orjson` (same stack as `api_utils.create_json_response`). - Emit the "Created ORCA header" message at **debug** instead of **info** so ORCA-enabled frontends do not log on every response.  ## Why  Clients that set `endpoint-load-m…

### #69360 — [[Feature][Spec Decode] Add worker-side grammar-aware drafting for MRv2 speculators](https://github.com/vllm-project/vllm/pull/69360)
- **作者**: usernamezero  **时间**: 2026-11-23 15:44 CST
- **标签**: structured-output, speculative-decoding, mrv2, scheduler
- **摘要**: ### Purpose  In MRv2 speculative decoding with structured outputs (e.g., Agent JSON tool-call schemas), grammar bitmasks are currently applied only during target verification while the drafter proposes unconstrained tokens. Whenever a schema constrains the next token (structural punctuation, fixed k…

### #69359 — [[Perf][KV Offload] Prioritize fenced stores before background stores](https://github.com/vllm-project/vllm/pull/69359)
- **作者**: Alex-ai-future  **时间**: 2026-11-23 15:42 CST
- **标签**: kv-connector
- **摘要**: <!-- markdownlint-disable --> <!-- Suggested title: [Perf][KV Offload] Prioritize fenced stores before background stores -->  ## Overview  Submit KV offload stores needed by the current step before other pending stores. This reduces the time a step spends waiting to reuse a GPU KV block when a large…

### #69357 — [[Frontend] Add multimodal options to offline pooling APIs.](https://github.com/vllm-project/vllm/pull/69357)
- **作者**: noooop  **时间**: 2026-11-23 15:20 CST
- **标签**: frontend, pooling
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Add the following multimodal parameters to the pooling API and wire these parameters through to render.   Part of https://github.com/vllm-project/vllm/pull/67180  Multimodal & Caching  • mm_processor_kwargs (dict | None): Additional keyword arguments forwa…

### #69356 — [[LoRA] Support per-request multi-adapter composition with static scales](https://github.com/vllm-project/vllm/pull/69356)
- **作者**: arun-elr  **时间**: 2026-11-23 15:18 CST
- **标签**: documentation, cpu, mrv2, scheduler, kv-cache-manager
- **摘要**: ## Overview  Implements Phase 1 of #51940: per-request activation of multiple independently-loaded LoRA adapters with static scales (`y = Wx + sum_i s_i * (alpha_i / r_i) * B_i A_i x`) for the offline API, using a reference multi-slot Punica loop with zero kernel changes.  ## Claims  - Adds `max_lor…

### #69355 — [[Perf][MoE] Prefer Triton for block FP8 MoE on SM12x with TP](https://github.com/vllm-project/vllm/pull/69355)
- **作者**: ZacharyZcR  **时间**: 2026-11-23 15:10 CST
- **摘要**: ## Overview  On SM12x, block FP8 MoE (`kFp8Static128BlockSym` weights, `kFp8Dynamic128Sym` activations) falls through to the default backend order and picks DeepGEMM. On DGX Spark (GB10, SM121) the Triton fused MoE kernel is faster at every decode batch size we measured. This PR moves Triton to the …

### #69354 — [[LoRA] Make embedding and lm_head LoRA opt-in](https://github.com/vllm-project/vllm/pull/69354)
- **作者**: linitra24  **时间**: 2026-11-23 15:10 CST
- **标签**: documentation, needs-rebase, llama, qwen
- **摘要**: ## Overview  Skip input embedding and `lm_head` LoRA wrapping by default to avoid allocating GPU buffers for unused weights. Add `--lora-enable-vocab-modules` to enable either module or both. An explicit `--lora-target-modules` allowlist takes precedence.  Related to #68107 and #51594.  ## Claims  -…
