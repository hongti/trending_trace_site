# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-31 09:05 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 📌 Issue 动态
*   **[RFC] MLA 分块上下文逐请求调度** (#50497)：探讨 MLA（Multi-head Latent Attention）在 prefill 阶段的调度机制优化，旨在改进现有上下文收集与分页隐层 KV 的处理方式。
*   **[Bug] NV-Embed-v2 启动失败** (#50495)：`nvidia/NV-Embed-v2` 在 V1 引擎子进程生成时触发 `_LazyConfigMapping` pickle 错误，导致模型无法启动。

---

### 🚀 PR 动态

**🆕 新模型支持**
*   **新增 Apertus 1.5 模型** (#50496)：支持提供 Apertus v1.5 多模态模型（8B 与 70B）的推理服务。

**⚡ 核心架构与特性优化**
*   **NIXL/KVConnector 支持 HMA 布局** (#50494)：在流水线并行（PP）的 push prefill 中支持 attention-HMA（混合 KV 缓存）布局，解除了之前的显式限制。
*   **Kimi-K3 前缀缓存优化** (#50493)：支持 DCP 部分前缀缓存命中，通过将上报单位改为 num_tokens，提升了前缀缓存的复用粒度。
*   **Kimi-K3 推测解码修正** (#50487)：修正了 DFlash drafter 的辅助状态，改用 pre-norm AttnRes 混合流，以匹配其训练时的设定。

**🔧 Bug 修复**
*   **Speculative Decode 批处理修复** (#50488)：修复了 V2 模型运行器中处理推测批时发现的 4 个缺陷。
*   **前端错误类型修复** (#50491)：将 `chat_utils.py` 中 7 个面向用户的 `ValueError` 替换为 `VLLMValidationError`，以符合 vLLM 新的错误层级规范。

**💡 API 与易用性**
*   **Completions API 支持图像** (#50489)：`/completions` 接口新增接受 `image_urls` 的能力，满足多模态强化学习（RL）工作负载需要原始提示与图像结合的场景。
*   **Block Size 静默丢弃警告** (#50486)：当用户指定的 `--block-size` 被后端对齐逻辑静默覆盖时，系统将新增警告提示，避免混淆。

**🛠️ CI 与文档**
*   **ROCm CI 修复** (#50490)：解除因 Kimi K3 上线导致的 ROCm CI 严格限定测试的“可选”门槛，确保此类错误能被正常拦截。
*   **文档更新** (#50492)：新增 MatrixHub 作为模型加载源说明。

---

### 📦 Release 动态
*   近期无新版本发布。

---

## 🐛 Issues

### #50497 — [[RFC]: Per-request scheduling for MLA chunked context (prefill)](https://github.com/vllm-project/vllm/issues/50497)
- **作者**: LucasWilkinson  **时间**: 2026-07-31 08:49 CST
- **标签**: RFC, kimi, k3
- **摘要**: ## Motivation  MLA prefill with existing context gathers paged latent KV into a fixed workspace, up-projects it, attends, and merges the partial into the running output. The schedule is **batch-column shaped** — the workspace is divided evenly across all prefills, and every iteration processes the s…

### #50495 — [[Bug]: nvidia/NV-Embed-v2 fails to start: `_LazyConfigMapping` pickle error during engine-core subprocess spawn](https://github.com/vllm-project/vllm/issues/50495)
- **作者**: wagnerpatriota  **时间**: 2026-07-31 08:15 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (x86_64) GCC version                  : (Ubuntu 11.…

## 🔀 Pull Requests

### #50496 — [[Model] Apertus 1.5](https://github.com/vllm-project/vllm/pull/50496)
- **作者**: Anunay-Yadav  **时间**: 2026-07-31 08:24 CST
- **标签**: documentation, new-model, multi-modality, tool-calling
- **摘要**: ## General Information    This change enables support for serving Apertus v1.5 multimodal (hf tags apertus-ai/Apertus-v1.5-8B,  apertus-ai/Apertus-v1.5-70B)     Special thanks to @blancsw  for creating a workable refactor  from https://github.com/swiss-ai/vllm/tree/apertus_integration that optimizes…

### #50494 — [[KVConnector][NIXL] Support attention-HMA layouts in pipeline-parallel push prefill](https://github.com/vllm-project/vllm/pull/50494)
- **作者**: zixi-qi  **时间**: 2026-07-31 08:13 CST
- **标签**: documentation, kv-connector
- **摘要**: ## Purpose  Follow-up to #45880 (pipeline-parallel prefill in `NixlPushConnector`), which explicitly guarded off hybrid KV-cache (HMA) layouts under PP:      raise NotImplementedError(         "NixlPushConnector does not support pipeline_parallel_size > 1 "         "with hybrid KV cache layouts (HMA…

### #50493 — [[Kimi-K3] support DCP partial prefix cache hit](https://github.com/vllm-project/vllm/pull/50493)
- **作者**: GirasoleY  **时间**: 2026-07-31 07:38 CST
- **标签**: ci/build, nvidia, kimi, k3
- **摘要**: Stacked on #50484, improve prefix cache reuse granularity.  Under DCP block size gets scaled by DCP size (xN) so we need to change the reporting to num_tokens to allow partial hit within the block size. The change has three commits: * refactor * reporting num_tokens caches instead of num_blocks * li…

### #50492 — [[Doc] Add MatrixHub as a model loading source](https://github.com/vllm-project/vllm/pull/50492)
- **作者**: yitingdc  **时间**: 2026-07-31 07:32 CST
- **标签**: documentation
- **摘要**: ## Purpose  Document [MatrixHub](https://github.com/matrixhub-ai/matrixhub) as a model loading source in `docs/models/supported_models.md`, next to the existing Hugging Face Hub and ModelScope sections.  MatrixHub is a self-hosted, Apache 2.0 licensed model registry that caches models from upstream …

### #50491 — [[Bugfix][Frontend] Raise VLLMValidationError for user-facing errors in chat_utils.py](https://github.com/vllm-project/vllm/pull/50491)
- **作者**: latent-9  **时间**: 2026-07-31 06:57 CST
- **标签**: bug, frontend
- **摘要**: ## Purpose  Closes #50253.  After the `VLLMError` hierarchy migration in #49665, seven user-facing errors in `vllm/entrypoints/chat_utils.py` still raised a bare `ValueError`, so they bypassed `VLLMValidationError` and the structured request-error handling (the API `param` field came back `null`).  …

### #50490 — [[ROCm][CI] Un-gate narrowly-scoped tests broken by Kimi K3](https://github.com/vllm-project/vllm/pull/50490)
- **作者**: wjabbour  **时间**: 2026-07-31 06:51 CST
- **标签**: rocm, ci/build, kimi, k3
- **摘要**: ## Purpose  Last week's Kimi K3 onboarding (#50000/#50089) broke ROCm CI twice. Both breaks were caught by tests correctly scoped via `source_file_dependencies` — but marked `optional: true`, so they only ran on the nightly mirror, a day after merge, instead of on the PR:  - #50262: `Kernels KDA Tes…

### #50489 — [[Frontend] accept image_urls in completions](https://github.com/vllm-project/vllm/pull/50489)
- **作者**: walterbm  **时间**: 2026-07-31 06:49 CST
- **标签**: frontend
- **摘要**: ## Purpose  Allow the `/completions` API to accept image_urls. This is especially helpful for multi-modal RL workloads which require raw prompting (only possible through `/completions`) and also need image inputs   All the image processing is following the same pattern as `/chat/completions` while a…

### #50488 — [[Bugfix][Spec Decode][MRV2] Fix speculative-batch handling in the V2 model runner](https://github.com/vllm-project/vllm/pull/50488)
- **作者**: rchalamala  **时间**: 2026-07-31 06:39 CST
- **标签**: bug, speculative-decoding, nvidia, mrv2
- **摘要**: # [Bugfix][Spec Decode][MRV2] Fix speculative-batch handling in the V2 model runner  ## Purpose  Four defects that fire when a speculative batch meets the V2 model runner. They were all found bringing up a single Kimi K3 DFlash lane, but none of them is K3-specific: `RejectionSampler` does not know …

### #50487 — [[Model][Spec Decode] Tap the pre-norm AttnRes mixture as the Kimi K3 DFlash aux state](https://github.com/vllm-project/vllm/pull/50487)
- **作者**: rchalamala  **时间**: 2026-07-31 06:35 CST
- **标签**: kimi, k3
- **摘要**: ## Purpose  The DFlash drafter consumes auxiliary hidden states captured at a fixed set of target layers. K3 captures the post-mixture stream, which is not what the drafter was trained against: the AttnRes residual mixture is applied before the layer norm, and the current capture site reads the valu…

### #50486 — [[Misc] Warn when --block-size is silently discarded by backend alignment](https://github.com/vllm-project/vllm/pull/50486)
- **作者**: rchalamala  **时间**: 2026-07-31 06:18 CST
- **标签**: ci/build, cpu
- **摘要**: ## Purpose  `Platform._align_hybrid_block_size` computes the kernel block alignment as `max(backend_minimum, cache_config.block_size)`, further floored at 128 for MLA. When the backend minimum (or MLA floor) exceeds the requested value, an explicit `--block-size` is silently discarded — and the log …
