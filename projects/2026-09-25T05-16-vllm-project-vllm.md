# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-25 13:16 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 📌 Issue 动态
* **用例提案：支付网关控制的私有化 vLLM 端点** (#58657)
  作者提出了一个新场景：团队在私有化部署 vLLM 的 OpenAI 兼容服务器时，希望通过 HTTP 402 状态码结合 Nano 加密货币，为 AI Agents 提供按量计费/支付网关控制的访问机制。这展示了社区在 vLLM 商业化与微支付集成方面的探索。

### 🛠️ PR 动态

**1. Bug 修复**
* **启动阶段快速失败机制** (#58666)：在 vLLM 启动阶段引入了快速失败（fail-fast）阻塞逻辑，提升系统初始化的稳定性。
* **FlashInfer 注意力机制修复** (#58663)：修复了 FlashInfer SM90 sparse MLA 中 `index_kpool` 的处理错误，该错误曾错误复用了主 MLA 缓存的布局参数。

**2. 性能优化**
* **SM100 架构加速** (#58662)：在小批量 BF16 输出投影中自动选用 FlashInfer TGV，显著加速 Qwen3.8-27B 等模型的注意力计算。
* **ROCm 平台 MLA 优化** (#58661)：通过 Grid-stride 页索引扩展优化 aiter MLA 内核，每次调用可在一个工作组织中至少节省 62.8 微秒。
* **前端多模态响应优化** (#58660)：针对 `/render` 接口返回超大 JSON（如四图请求达 57.7MB）的问题，改为直接序列化多模态渲染响应，大幅降低开销。

**3. 算子融合与内核**
* **Silu 融合算子** (#58664)：实现了融合版的 `silu_and_mul_per_token_quant` 操作并完成独立测试，为后续融合操作打下基础。
* **模型融合扩展** (#58665)：将手动激活与量化融合（manual activation+quant fusion）应用于 Granite、Seed-OSS 和 EXAONE 4.0 模型。
* **ROCm GEMM 归属权** (#58659)：规范了 ROCm block32 GEMM 打包内核及 split-K 归约逻辑的来源致谢。

**4. 新特性与功能完善**
* **视觉/音频 LoRA 支持** (#58658)：为 Nemotron Nano/Omni VL 模型的视觉塔、音频塔及其连接器添加了 LoRA 微调支持。
* **投机解码指标补全** (#58656)：修复了 `/v1/responses` 接口缺失 `metrics.speculative_decoding` 指标的问题，统一了前端 API 的度量返回标准。

### 🚀 Release 动态
* 本期提供的动态数据中 **未包含版本发布** 信息。

**总结**：近期 vLLM 的更新集中在**性能压榨**（特别是针对 SM100 架构和 ROCm 平台）、**多模态前端处理优化**，以及**对新模型架构的算子融合与 LoRA 适配**。同时，社区也开始探讨 vLLM 在去中心化支付场景下的应用潜力。

---

## 🐛 Issues

### #58657 — [Use-case proposal: payment-gated self-hosted vLLM for agents (HTTP 402 + Nano)](https://github.com/vllm-project/vllm/issues/58657)
- **作者**: dhyabi2  **时间**: 2026-09-25 10:56 CST
- **摘要**: # Use-case / pattern proposal: payment-gated, self-hosted vLLM endpoints for AI agents (HTTP 402 + Nano)  **The use case:** teams self-host vLLM behind its OpenAI-compatible server and want to let *autonomous agents* pay for inference **per token** — without creating accounts, issuing API keys to ev…

## 🔀 Pull Requests

### #58666 — [[Bugfix] Implement fail-fast blocking during the startup phase](https://github.com/vllm-project/vllm/pull/58666)
- **作者**: willweimike  **时间**: 2026-09-25 13:14 CST
- **标签**: bug, kv-connector
- **摘要**: ## Purpose  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such as providing test command. - [ ] The …

### #58665 — [[Fusion] Apply manual activation+quant fusion to Granite, Seed-OSS and EXAONE 4.0](https://github.com/vllm-project/vllm/pull/58665)
- **作者**: rishabhsinha17  **时间**: 2026-09-25 12:48 CST
- **标签**: quantization
- **摘要**: ## Purpose  Second model batch for #43501, following the batching plan from https://github.com/vllm-project/vllm/issues/43501#issuecomment-5716717828. Applies the manual fusion pattern that #51415 established for `LlamaMLP` to three more SiluAndMul families. Each MLP now routes through `maybe_fused_…

### #58664 — [[Kernels][Fusion] Fused silu mul per token quant](https://github.com/vllm-project/vllm/pull/58664)
- **作者**: ElizaWszola  **时间**: 2026-09-25 12:46 CST
- **标签**: documentation, ci/build, quantization
- **摘要**: Implements fused `silu_and_mul_per_token_quant`. This PR only implements the operation and tests it in isolation without performing any fusion, but it will be required for some fusions to take effect in https://github.com/vllm-project/vllm/pull/57514 (Mistral)  #### Testing ``` pytest tests/kernels/…

### #58663 — [[Bugfix][Attention] Fix index_kpool handling in FlashInfer SM90 sparse MLA](https://github.com/vllm-project/vllm/pull/58663)
- **作者**: andakai  **时间**: 2026-09-25 12:37 CST
- **标签**: bug, nvidia
- **摘要**: ## 1. Root cause  The FlashInfer SM90 sparse MLA builder incorrectly used the main MLA cache's `tokens_per_state` as the sparse indexer's `index_kpool`. These values describe different layouts: GLM-5.3-Flash stores one main MLA cache state per token (`tokens_per_state=1`), while its sparse indexer p…

### #58662 — [[Perf][SM100] Use TGV for small-batch BF16 output projections](https://github.com/vllm-project/vllm/pull/58662)
- **作者**: xutingl  **时间**: 2026-09-25 12:33 CST
- **标签**: nvidia
- **摘要**: ## Purpose  Automatically select FlashInfer TGV for biasless BF16 projections with `(N, K) = (5120, 6144)` and flattened batch sizes 1, 2, 4 or 8 on SM100. This accelerates Qwen3.8-27B's attention and GDN output projections at TP1.  Reuse the existing BF16 custom op and TGV kernel. Other shapes, lar…

### #58661 — [[Perf][ROCm][MLA] Grid-stride page-index expansion, saving at least 62.8us/call in one workgroup](https://github.com/vllm-project/vllm/pull/58661)
- **作者**: fululi12  **时间**: 2026-09-25 12:27 CST
- **标签**: rocm
- **摘要**: ## Purpose  `_expand_page_indices_kernel` turns block-table entries into the per-token flat page indices the aiter MLA kernel consumes (that kernel always runs `page_size=1` internally, with `kv_buffer` flattened via `.view(-1, 1, 1, H)`).  It launched with grid `(num_reqs,)` and looped:  ```python …

### #58660 — [[Perf][Frontend] Serialize multimodal render responses directly](https://github.com/vllm-project/vllm/pull/58660)
- **作者**: Levius-Fubuki  **时间**: 2026-09-25 11:44 CST
- **标签**: frontend
- **摘要**: ## Purpose  `/render` returns preprocessed multimodal features as large JSON strings. For a four-image Qwen3-VL request, the response is 57.7 MB. The current `JSONResponse(content=result.model_dump())` materializes a Python dictionary and then encodes it using the standard JSON encoder; 75.1% of inc…

### #58659 — [[ROCm] Credit ROCm/aiter for the block32 GEMM's packed kernel and in-launch split-K](https://github.com/vllm-project/vllm/pull/58659)
- **作者**: valarLip  **时间**: 2026-09-25 11:40 CST
- **标签**: rocm, ready
- **摘要**: ## Purpose  The packed small-M kernel and the in-launch split-K reduction on one XCD in `rocm_block32_gemm.py` (#58510) are adapted from the group32 GEMM in ROCm/aiter#5750:  - the K panels packed into MFMA rows, keeping only the block diagonal (aiter 4ffc0a7098); - `_split_tile`, `_sum_splits`, `_s…

### #58658 — [[Model][LoRA] Add tower/connector LoRA support for Nemotron Nano/Omni VL (vision + audio)](https://github.com/vllm-project/vllm/pull/58658)
- **作者**: Nan2018  **时间**: 2026-09-25 11:08 CST
- **标签**: multi-modality
- **摘要**: ## Purpose  Adds LoRA support for the vision tower (RADIO ViT), audio tower (Parakeet conformer encoder), and their respective connectors on `NemotronH_Nano_VL_V2` / `NemotronH_Nano_Omni_Reasoning_V3`, using the existing `enable_tower_connector_lora` mechanism. Language-model-only LoRA for this mode…

### #58656 — [[Frontend][Spec Decode] Populate metrics.speculative_decoding on /v1/responses](https://github.com/vllm-project/vllm/pull/58656)
- **作者**: ra-srivid  **时间**: 2026-09-25 10:37 CST
- **标签**: documentation, frontend
- **摘要**: ## Purpose  Per-request spec-decode acceptance stats (added in #48915) are attached on **chat** and **completions** responses via `response.metrics.speculative_decoding`, but were never populated on the **`/v1/responses`** endpoint — even though `ResponsesResponse` already declares the `metrics: Per…
