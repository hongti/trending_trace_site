# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-30 13:52 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文简洁摘要：

### 🐛 Issue 动态
近期主要报告了两个影响运行的 Bug：
*   **Qwen3.5 模型推理异常**：在 H20 GPU 上使用 v0.19.1 版本运行 BF16 格式的 Qwen3.5-35B-A3B 融合 SFT 模型时，间歇性出现 NaN logits 并输出仅含感叹号的乱码 (#59381)。
*   **引擎启动时信号处理失效**：API 服务器在启动阶段（模型加载和预热时）收到的 SIGTERM 信号会被 zmq_socket_ctx 吞掉，导致服务器挂起，直到 600 秒超时后才退出 (#59375)。

### 🔀 PR 动态
近期 PR 主要围绕新特性支持、Bug 修复、代码重构与测试优化：

**新特性与支持**
*   **量化支持**：在 Compressed-Tensors 中新增对 `MXFP4xFP8_DYNAMIC` 量化格式的支持 (#59376)。
*   **LoRA 内核优化**：引入可选的 `VLLM_LORA_DETERMINISTIC_SPLIT_K=8`，在需要批处理不变性时提供确定性的 LoRA shrink 内核 (#59377)。
*   **KV Connector 优化**：优化 LMCache 代理，使其向 prefiller 发送有效的单 token 请求，以更高效地填充 KV 缓存 (#59370)。
*   **强化学习(RL)前端**：在 `finish_weight_update` 中增加针对基线的权重验证功能，完善训练器驱动的权重更新流程 (#59371)。

**Bug 修复**
*   **多模态修复**：修复 DeepSeek-V4 VL 模型未按最坏情况（即消耗 token 最多）选取 dummy 图像尺寸的问题，避免越界 (#59373)。
*   **ROCm 构建修复**：修复了由于依赖路径变更导致的 TheRock/MoRI Dockerfile 构建失败问题（涉及 #59372 与 #59378）。

**重构与测试**
*   **接口重构**：重构 KV offload 复制后端接口，引入共享的 `CopyRunDescriptor` 协议，使逻辑更清晰 (#59374)；重构 `BlockHashToBlockMap.get_one_block` 方法使用提前返回模式 (#59380)。
*   **测试修复**：在非 CUDA 平台（如 Intel XPU CI）上跳过依赖 `.cuda()` 的 IPC 权重转移测试 (#59379)。

### 🚀 Release 动态
*   本次提供的数据集中未包含新的 Release 版本发布信息。

---

## 🐛 Issues

### #59381 — [[Bug]: Qwen3.5-35B-A3B merged SFT BF16 intermittently produces NaN logits and exclamation-only output on H20 (v0.19.1; HF control finite)](https://github.com/vllm-project/vllm/issues/59381)
- **作者**: ChenYahui210  **时间**: 2026-09-30 13:21 CST
- **标签**: bug, quantization
- **摘要**: ### Your current environment  Privacy-reviewed, targeted environment collection (not a full collect_env.py dump):  ```text GPU used by the reproduction: 1 x NVIDIA H20-3e GPU memory reported by nvidia-smi: 143771 MiB Compute capability: 9.0 NVIDIA driver: 580.159.03 OS: Linux x86_64, kernel 5.4.119-…

### #59375 — [[Bug]: SIGTERM during engine startup is swallowed by zmq_socket_ctx; API server hangs until VLLM_ENGINE_READY_TIMEOUT_S](https://github.com/vllm-project/vllm/issues/59375)
- **作者**: TSUMUGI-XE  **时间**: 2026-09-30 12:36 CST
- **标签**: intel-gpu
- **摘要**: SIGTERM sent to the API server during engine startup (spawn → READY, which includes model load and warmup) is ignored. The server waits until `VLLM_ENGINE_READY_TIMEOUT_S` (600 s) and exits with `TimeoutError`. Nothing is logged at the default level.  **Cause** (main @ 678baf537): `_interrupt_init` …

## 🔀 Pull Requests

### #59380 — [[Refactor] Use early-return pattern in BlockHashToBlockMap.get_one_block](https://github.com/vllm-project/vllm/pull/59380)
- **作者**: zvier  **时间**: 2026-09-30 13:14 CST
- **标签**: kv-cache-manager
- **摘要**: Align with the style used by contain, pop, and insert methods.  ## Purpose Refactor BlockHashToBlockMap.get_one_block to use an early-return pattern, replacing the nested if blocks is not None: guard. This aligns the method's structure with contain, pop, and insert in the same class,   all of which …

### #59379 — [[Test] Skip IPC weight-transfer test on XPU platforms](https://github.com/vllm-project/vllm/pull/59379)
- **作者**: chaojun-zhang  **时间**: 2026-09-30 13:14 CST
- **标签**: intel-gpu, nvidia
- **摘要**: ## Why  test_ipc_weight_transfer_restores_reset_weights fails on non-CUDA platforms (e.g., Intel XPU CI) with AssertionError: Torch not compiled with CUDA enabled. It relies on .cuda() and the CUDA-only IPC backend (ipc_engine.py), so making .cuda() device-agnostic won't help. No related open PRs/is…

### #59378 — [[ROCm][Build] Restore TheRock dependency discovery for MoRI](https://github.com/vllm-project/vllm/pull/59378)
- **作者**: AndreasKaratzas  **时间**: 2026-09-30 13:05 CST
- **标签**: rocm, ci/build
- **摘要**: Fix the ROCk base-image failure introduced by `467d81d9` by restoring TheRock's CMake dependency search path for MoRI. Scope `CMAKE_PREFIX_PATH` to the wheel build so `hsakmt` finds NUMA, preserving existing prefixes and keeping MoRI `v1.2.3.post1`.  Prepared with OpenAI Codex assistance.

### #59377 — [[Kernel][LoRA] Add deterministic split-K=8 LoRA shrink for batch invariance](https://github.com/vllm-project/vllm/pull/59377)
- **作者**: ShengleiFu  **时间**: 2026-09-30 12:54 CST
- **标签**: documentation
- **摘要**: ## Purpose  With `VLLM_BATCH_INVARIANT=1` the LoRA shrink kernel runs with `SPLIT_K=1`, because split-K normally reduces with float atomics. This adds an opt-in `VLLM_LORA_DETERMINISTIC_SPLIT_K=8` (default `0`) that keeps K parallelism without atomics: the existing shrink kernel writes unscaled FP32…

### #59376 — [[Quantization] Add support for MXFP4xFP8_DYNAMIC in Compressed-Tensors](https://github.com/vllm-project/vllm/pull/59376)
- **作者**: kylesayrs  **时间**: 2026-09-30 12:47 CST
- **标签**: quantization
- **摘要**: ## Purpose ## Add ct support for MXFP4xFP8_DYNAMIC for future quantizations and to verify the validity of https://github.com/vllm-project/compressed-tensors/pull/902 when used on DSV4  ## Changes ## * Add routing for `deepseek_v4_mxfp4_moe` backend to `CompressedTensorsW4A4Mxfp4MoEMethod`  ## Testin…

### #59374 — [[Refactor] KV offload copy backend interface](https://github.com/vllm-project/vllm/pull/59374)
- **作者**: Alex-ai-future  **时间**: 2026-09-30 12:32 CST
- **摘要**: # Refactor KV offload copy backend interface  ## Summary  This PR refactors the CPU KV offload transfer path around a shared logical `CopyRunDescriptor` protocol. The worker builds structured run information, while the selected copy backend expands it for the existing copy API and owns the descripto…

### #59373 — [[BugFix][Multimodal] Pick worst-case DeepSeek-V4 VL dummy image size (#59271)](https://github.com/vllm-project/vllm/pull/59373)
- **作者**: YidaWeng  **时间**: 2026-09-30 12:31 CST
- **标签**: bug, multi-modality, deepseek, DSv4
- **摘要**: ## Purpose  Fixes #59271.  `DeepseekV4VLProcessingInfo.get_image_size_with_most_features` assumed a square maximizes ViT patches within the token budget. Because each aligner row costs an extra `IMAGE_NEW_LINE`, a wide grid (`1932x336`) produces more placeholder tokens and patches than the previous …

### #59372 — [[ROCm][BugFix][The Rock] Fix mori build for the rock](https://github.com/vllm-project/vllm/pull/59372)
- **作者**: rasmith  **时间**: 2026-09-30 12:26 CST
- **标签**: bug, rocm, ci/build
- **摘要**: ## Purpose This https://github.com/vllm-project/vllm/pull/56073 broke the Dockerfile.rock_base build.  This PR fixes the build by setting the `CMAKE_PREFIX_PATH` and using the appropriate version of `setuptools_scm`.  There will be a PR into MORI so that these work-arounds aren't needed and then a f…

### #59371 — [[Frontend][RL] Verify weights against a baseline in finish_weight_update](https://github.com/vllm-project/vllm/pull/59371)
- **作者**: aoshen02  **时间**: 2026-09-30 12:25 CST
- **标签**: documentation, frontend, rust
- **摘要**: ## Purpose  Follow-up to #51350 (RFC #52689). The weight checker verifies a transfer as checksum → reset → transfer → compare, but the compare was a separate call, so a trainer that drives the update through `send_weights()` could not check it in the same round trip. This PR lets `finish_weight_upda…

### #59370 — [[KV Connector] Send a valid single-token request to the prefiller in the LMCache proxy](https://github.com/vllm-project/vllm/pull/59370)
- **作者**: konfan12345678  **时间**: 2026-09-30 12:18 CST
- **标签**: documentation, kv-connector
- **摘要**: ## Purpose  `examples/disaggregated/lmcache/disagg_prefill_lmcache_v1/disagg_proxy_server.py` forwards every client request to P first and throws P's response away - P only exists to populate the KV cache that D will read. Its prefill-leg rewrite overrides `max_tokens` and nothing else:  ```python r…
