# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-15 09:59 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue (问题反馈)
1. **NVFP4 模型崩溃 (#52407)**：开启 `VLLM_BATCH_INVARIANT=1` 后，加载 NVFP4 (`modelopt_fp4`) 模型会在初始化时崩溃，原因是批次不变性禁用了硬件 NVFP4，导致查表(CPU)与索引(GPU)设备不匹配。
2. **DeepSeek V4 高并发输出乱码 (#52404)**：在 4 张 B200 GPU 上使用 DP4+EP 或 TP4+EP 部署 DeepSeek V4 Flash 0731 时，处理数百个文本提取任务会出现输出乱码现象。
3. **多模态 CI 失败 (#52403)**：Gemma4 模型的 ViT CUDA Graph 测试（图像/视频）失败，报错引擎核心初始化失败。
4. **生成接口丢失选项 (#52398)**：调用非流式 `/inference/v1/generate` 接口且设置 `n > 1` 时，非确定性返回少于 `n` 个的 choices，且无报错提示。

---

### 🔀 Pull Request (代码合并)

**🚀 新特性与支持**
* **Kimi-K3 FP8 支持 (#52406)**：为 Kimi-K3 的 MLA 和 KDA 注意力机制支持 ModelOpt `FP8_PB_WO` 权重，并处理了 per-block FP8 所需的 128 行粒度对齐填充。

**🛠️ 核心错误修复**
* **修正 NVFP4 缩放计算 (#52405)**：集中化了 TRT-LLM NVFP4 MoE 的 `g1_scale_c` 计算，避免在内核已通过 `output1_scale_gate_scalar` 接收 `g1_alphas` 时将其折叠进 SiTU 输出缩放。
* **DeepSeek V4 Cudagraph 修复 (#52401)**：按 model runner 选取 DeepSeek V4 eager cudagraph 区域，修复了此前因缩小 cudagraph 区域导致 MRV1 输出损坏的问题，同时避免了强制默认 MRV2 给 ROCm 带来的性能损耗。
* **生成接口修复 (#52399)**：修复了 `/inference/v1/generate` 在 `n > 1` 时非流式请求丢失 choices 的问题（关闭 #52398）。
* **KV Offload 断言修复 (#52397)**：修复当 `max_offload_tokens` 低于 partial-tail 边界时，原生 CPU/分层 KV 卸载路径引发的断言错误。
* **推测解码修复 (#52396)**：修复了 DSpark 推测解码使用未量化 draft 模型时，因 `hf_overrides` 非字典类型导致的引擎初始化崩溃。
* **结构化输出校验修复 (#52394)**：结构化输出验证器改用抛出 `VLLMValidationError`，防止不良的 `response_format` 被包装成难以处理的 `EngineGenerateError`。

**🟩 ROCm 及 CI 优化**
* **DSv4 稀疏注意力优化 (#52402)**：针对 gfx942 架构优化 DSv4 sparse-attn indexer，引入原生 fp8 MFMA 并修正 LDS 占用率门控。
* **Docker 依赖修复 (#52400)**：从 `Dockerfile.rocm` 中移除 pybind11 安装，防止其与 `rocm_base` 中构建 AITER 内核的 pybind11 版本冲突。
* **多模态前缀支持修复 (#52395)**：因未实现 Prefix-LM，令 ROCm 和统一注意力后端的 `supports_mm_prefix` 返回 False，避免潜在错误。

---

### 🚀 Release (版本发布)
* 近期暂无新的版本发布记录。

---

## 🐛 Issues

### #52407 — [[Bug]: VLLM_BATCH_INVARIANT=1 crashes NVFP4 models: emulation break_fp4_bytes device mismatch (lookup table on CPU, indices on GPU)](https://github.com/vllm-project/vllm/issues/52407)
- **作者**: royalbert  **时间**: 2026-08-15 09:01 CST
- **标签**: bug, quantization
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information... ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (aarch64) GCC…

### #52404 — [[Bug]: 4 B200 GPU ，DP 4 + EP（or TP 4 + EP），Deepseek V4 Flash 0731 , simultaneously processing hundreds of text extraction tasks, outputting garbled characters.](https://github.com/vllm-project/vllm/issues/52404)
- **作者**: majestichou  **时间**: 2026-08-15 08:01 CST
- **标签**: bug
- **摘要**: ### Your current environment  Ubuntu 22.04, vllm docker v0.27.1 image, B200 server  ### 🐛 Describe the bug  I used four B200 GPUs, with Ubuntu 22.04 as the operating system, and the latest Docker image for VLLM, vllm/vllm-openai:v0.27.1. I started the DeepSeek V4 Flash 0731 model (https://huggingfac…

### #52403 — [[CI Failure]: multi-modal-models-standard-4-other-plus-whisper - models/multimodal/generation/test_vit_cudagraph.py::test_vit_cudagraph_(image|video)[gemma4]](https://github.com/vllm-project/vllm/issues/52403)
- **作者**: jyan-R  **时间**: 2026-08-15 07:43 CST
- **摘要**: ### Which test(s) are failing?  ```text FAILED models/multimodal/generation/test_vit_cudagraph.py::test_vit_cudagraph_image[gemma4] - RuntimeError: Engine core initialization failed. FAILED models/multimodal/generation/test_vit_cudagraph.py::test_vit_cudagraph_video[gemma4] - RuntimeError: Engine co…

### #52398 — [[Bug]: `/inference/v1/generate` silently drops choices when `n > 1`](https://github.com/vllm-project/vllm/issues/52398)
- **作者**: qgallouedec  **时间**: 2026-08-15 06:56 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (x86_64) GCC version                  : (Ubuntu 11.…

## 🔀 Pull Requests

### #52406 — [[Model] Support ModelOpt FP8_PB_WO in Kimi-K3 attention](https://github.com/vllm-project/vllm/pull/52406)
- **作者**: Edwardf0t1  **时间**: 2026-08-15 08:53 CST
- **标签**: quantization, kimi, k3
- **摘要**: ## Summary  - support ModelOpt `FP8_PB_WO` weights in Kimi-K3 MLA and KDA attention - pad fused projection outputs to the 128-row granularity required by per-block FP8, then remove the zero padding before attention consumes them - directly dequantize standard 2D ModelOpt FP8 block-scale layouts duri…

### #52405 — [[Bugfix] Correct TRTLLM NVFP4 SiTU output scale](https://github.com/vllm-project/vllm/pull/52405)
- **作者**: Edwardf0t1  **时间**: 2026-08-15 08:53 CST
- **标签**: bug, nvidia
- **摘要**: ## Summary  - centralize the TRT-LLM NVFP4 MoE `g1_scale_c` calculation - avoid folding `g1_alphas` into the SiTU output scale when the kernel already receives it through `output1_scale_gate_scalar` - use the same calculation during initialization and post-load processing - add focused coverage for …

### #52402 — [[ROCm][gfx942] DSv4 sparse-attn indexer: native fp8 MFMA + corrected LDS occupancy gate](https://github.com/vllm-project/vllm/pull/52402)
- **作者**: MohitAMD  **时间**: 2026-08-15 07:34 CST
- **标签**: rocm
- **摘要**: > **Draft** — sharing early for review with the data we have so far; additional E2E A/B data points (1P1D / 1P2D disaggregated) will be appended as benchmark jobs complete.  ## Summary Optimizes the **gfx942 (MI300X/MI325X)** path of `fp8_mqa_logits` — the DeepSeek-V4 sparse-attention *indexer* in `…

### #52401 — [[Bugfix] Pick the DeepSeek V4 eager cudagraph region per model runner](https://github.com/vllm-project/vllm/pull/52401)
- **作者**: njhill  **时间**: 2026-08-15 07:10 CST
- **标签**: bug, ready, deepseek, nvidia
- **摘要**: #51430 narrowed the DeepSeek V4 eager cudagraph region, which corrupts MRV1 output, and #51768 responded by defaulting the model to MRV2 and rejecting MRV1 + PIECEWISE. That default costs ROCm, where MRV1 is still the faster runner for this model.  Choose the region from the runner instead: MRV1 wra…

### #52400 — [[ROCm]: Drop pybind11 from Dockerfile.rocm to prevent version mismatch](https://github.com/vllm-project/vllm/pull/52400)
- **作者**: Rohan138  **时间**: 2026-08-15 07:10 CST
- **标签**: rocm, ready, ci/build
- **摘要**: ## Purpose Currently, the AITER kernels in `Dockerfile.rocm_base` were built with pybind11 3.0.4, whereas the pybind11 install in `Dockerfile.rocm` installs pybind 3.1.0 (released 08/08/2026). The API incompatibility breaks Kimi K2.6/K3 among other models: https://github.com/ROCm/aiter/issues/4770  …

### #52399 — [[Bugfix][Frontend] Return all choices from /inference/v1/generate when n > 1](https://github.com/vllm-project/vllm/pull/52399)
- **作者**: qgallouedec  **时间**: 2026-08-15 07:06 CST
- **标签**: bug, frontend
- **摘要**: ## Purpose  closes #52398  `serve_tokens` sets `sampling_params.output_kind` only for streaming requests, so non-streaming ones keep the `SamplingParams` default, `CUMULATIVE`. With `n > 1`, `serve_tokens_full_generator` keeps only the last streamed `RequestOutput` and reads `final_res.outputs`, whi…

### #52397 — [Fix `max_offload_tokens` assertion error.](https://github.com/vllm-project/vllm/pull/52397)
- **作者**: yizhuoliang  **时间**: 2026-08-15 06:48 CST
- **标签**: kv-connector
- **摘要**: # [Bugfix][KV Offload] Assertion failure when `max_offload_tokens` is below the partial-tail boundary  ## Purpose  The issue is in `OffloadingConnector` — the native CPU/tiered KV offload path (not P/D disaggregation, not the Mooncake store connector) — on a hybrid Mamba attention model, like the Qw…

### #52396 — [[Bugfix] Don't raise on non-dict hf_overrides when resolving draft quant config](https://github.com/vllm-project/vllm/pull/52396)
- **作者**: stefanskiasan  **时间**: 2026-08-15 06:47 CST
- **标签**: bug, quantization
- **摘要**: ## Purpose  Enabling DSpark speculative decoding with an **unquantized draft** crashes at engine init:  ``` File "vllm/v1/worker/gpu/spec_decode/dspark/utils.py", load_dspark_model   draft_vllm_config.quant_config = get_draft_quant_config(vllm_config) File "vllm/model_executor/models/utils.py", get_…

### #52395 — [[CI/Build][BugFix][The Rock] Make supports_mm_prefix  return False for ROCm attn and unified attn since Prefix-LM not implemented](https://github.com/vllm-project/vllm/pull/52395)
- **作者**: rasmith  **时间**: 2026-08-15 06:46 CST
- **标签**: bug, rocm
- **摘要**: ## Purpose This PR changes `supports_mm_prefix` in for ROCm attention and unified attention to return false since  `RocmAttentionMetadata`   and `RocmAttentionImpl.forward` do not have `mm_prefix` (or similar) fields / parameters like the `TritonAttentionMetadata` and `TritonAttentionImpl` countepar…

### #52394 — [[Bugfix] Raise `VLLMValidationError` from structured output validators](https://github.com/vllm-project/vllm/pull/52394)
- **作者**: jeffreywang88  **时间**: 2026-08-15 06:27 CST
- **标签**: bug, structured-output
- **摘要**: ## Purpose  Structured output validators raise raw `ValueError`. `AsyncLLM.generate` re-raises only `VLLMClientError` untouched and wraps the rest in `EngineGenerateError`, so a bad `response_format` schema returns 500 instead of 400.  This issue is surfaced from ray serve LLM because vLLM 0.27 wrap…
