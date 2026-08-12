# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-12 11:07 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 🚀 版本发布
* **v0.27.1**：这是基于 v0.27.0 的补丁版本。**版本亮点**：新增对量化 DSpark Markov heads 的支持。

---

### 🐛 Issue 动态
* **Bug 反馈**：在 v0.27.1 + DSpark 环境下，DeepSeek-V4-Flash-0731 模型间歇性输出格式错误的 DSML 工具调用起始包装符（#51914）。
* **功能需求**：请求允许在日志中仅记录模型输出文本，而不必捆绑输出 `output_token_ids`，以提升日志可读性（#51912）。

---

### 🔀 PR 动态

**1. 重要修复与回滚**
* **回滚默认参数**：因 CI 夜间构建失败，回滚了将 `_max_num_batched_tokens` 默认值从 8192 提升至 16384 的变更（#51908）。
* **推测解码修复**：修复了在开启 Full CUDA Graphs 并使用动态推测解码（DSD）与自回归 MTP/EAGLE draft 模型时的崩溃问题（#51907）。
* **量化推理修复**：修复了 MiniMax-M3 使用 Compressed Tensors (CT) 格式的 MXFP4 推理时，因视觉塔 `KeyError` 导致加载失败的问题（#51910）。
* **缓存块逻辑优化**：优化缓存块的淘汰逻辑，区分命中与未命中的 block_hash，防止未命中块优先被错误淘汰（#51909）。

**2. 模型与硬件支持**
* **[ROCm]** 在 `deepseek_v32` 路径上启用 GLM-5.2-MXFP4 模型支持，并修复了稀疏注意力的正确性问题（#51915）。

**3. 新特性与性能优化**
* **[Rust 前端]** 在 token generate 路由（`/inference/v1/generate`）中新增对停止字符串的支持（#51904）。
* **[API 前端]** 新增 `routed_experts_prompt_start` 参数，允许客户端在返回结果中省略已知的 prompt 前缀（#51906）。
* **[Attention 优化]** 将 `context_lens_tensor` 的计算移入 GDN prefill 分支内，避免在非 prefill 阶段进行多余计算（#51913）。

**4. CI 与基础设施**
* **x86 CPU CI**：为镜像构建添加 Docker Registry 层缓存，避免每次从头编译 csrc/rust，大幅加速构建（#51911）。
* **Intel GPU CI**：全局统一使用 `VLLM_DISABLE_COMPILE_CACHE=1` 环境变量（#51905）。

---

## 🐛 Issues

### #51914 — [[Bug] DeepSeek-V4-Flash-0731 intermittently emits malformed DSML tool-call start wrapper on v0.27.1 + DSpark](https://github.com/vllm-project/vllm/issues/51914)
- **作者**: jinbagi  **时间**: 2026-08-12 10:54 CST
- **摘要**: ## Summary  With **DeepSeek-V4-Flash-0731** on **vLLM v0.27.1** with **DSpark enabled**, we have intermittently observed malformed DSML tool-call output where the opening wrapper is corrupted from:  ```text <｜DSML｜tool_calls> ```  to something like:  ```text <｜DSML｜toolcalls> ```  while the rest of …

### #51912 — [[Feature]: Allow logging model output text without output token IDs](https://github.com/vllm-project/vllm/issues/51912)
- **作者**: ruanwenjun  **时间**: 2026-08-12 10:28 CST
- **摘要**: ### 🚀 The feature, motivation and pitch  In vLLM v0.27.1, `--enable-log-outputs` logs the generated text, `output_token_ids`, and finish reason together in the same INFO record:  ```text Generated response <request_id>: output: '...', output_token_ids: [...], finish_reason: stop ```  For production …

## 🔀 Pull Requests

### #51915 — [[ROCm][Model][Bugfix] Enable GLM-5.2-MXFP4 on the deepseek_v32 path and fix sparse attention correctness](https://github.com/vllm-project/vllm/pull/51915)
- **作者**: jhu960213  **时间**: 2026-08-12 10:59 CST
- **标签**: bug, rocm, deepseek
- **摘要**: ## Purpose  Enables GLM-5.2 (`GlmMoeDsaForCausalLM`) end-to-end on `vllm/models/deepseek_v32/amd/` for gfx942/gfx950. Routing is opt-in via `--model-class-overrides`; the registry entry is unchanged, so the default path for GLM-5.2 and DeepSeek-V3.2 is untouched.   The following issues were also fix…

### #51913 — [[Attention] Move context_lens_tensor compute into GDN prefill path](https://github.com/vllm-project/vllm/pull/51913)
- **作者**: xyang16  **时间**: 2026-08-12 10:47 CST
- **摘要**: ## Purpose  Move `context_lens_tensor = m.compute_num_computed_tokens()` in GDNAttentionMetadataBuilder.build() from the top of the method into the `if num_prefills > 0` branch because it's only used in prefill. Since `context_lens_tensor` is not used in decode path, this change avoids the tensor co…

### #51911 — [[CI] Add registry layer cache to x86 CPU image build](https://github.com/vllm-project/vllm/pull/51911)
- **作者**: bigPYJ1151  **时间**: 2026-08-12 10:27 CST
- **标签**: ci/build
- **摘要**: ## Summary  The x86 CPU CI image build (`.buildkite/image_build/image_build_cpu.sh`) used classic `docker build` with no layer cache, so every build recompiled csrc/rust from scratch. This mirrors the registry-based BuildKit layer cache already used for the CUDA CI image build (`image_build.sh`).  #…

### #51910 — [[Quantization][CT] fix the mxfp4 inference for MiniMax-M3 with CT format.](https://github.com/vllm-project/vllm/pull/51910)
- **作者**: lkk12014402  **时间**: 2026-08-12 10:20 CST
- **标签**: quantization
- **摘要**: ## Description  Running **MiniMax-M3** MXFP4 checkpoints quantized with the **compressed-tensors (CT)** `mixed-precision` format currently fails at model load with a `KeyError` on the vision tower. In these checkpoints the **vision module is not quantized** (it is listed in the CT `ignore` list and …

### #51909 — [Cached blocks never hit eviction first](https://github.com/vllm-project/vllm/pull/51909)
- **作者**: shanrow-amd  **时间**: 2026-08-12 10:15 CST
- **摘要**: ## Purpose Most cached blocks with block_hash that never hit. Now they are appended to the free list mixed with fewer hit blocks. We should differentiate the both cases because of the non-hit blocks amount that is huge and far greater than hit blocks. So we should append the non-hit blocks first and…

### #51908 — [Revert "[Config] Update default `_max_num_batched_tokens` from 8192 to 16384" (#51726)](https://github.com/vllm-project/vllm/pull/51908)
- **作者**: vllm-agent  **时间**: 2026-08-12 10:14 CST
- **摘要**: Auto-generated by CI failure analyzer from nightly build [#83443](https://buildkite.com/vllm/ci/builds/83443) (commit `3e372c5`).  Reverts #51726 — "[Config] Update default `_max_num_batched_tokens` from 8192 to 16384".  ## Why  This default flip is linked to **2 new nightly failures** on B200 hardw…

### #51907 — [[BugFix][Spec Decode] Fix dynamic-SD draft decode capture](https://github.com/vllm-project/vllm/pull/51907)
- **作者**: Suppressor72  **时间**: 2026-08-12 10:05 CST
- **标签**: bug, speculative-decoding, nvidia, mrv2
- **摘要**: ## Purpose  Fixes crash when FULL CUDA graphs are enabled under **dynamic speculative decoding (DSD)** with autoregressive MTP/EAGLE draft models.  Related: #48494, #49652 (same fix, rebased), #48329. Prerequisite for #50885 (FlashInfer FULL under spec-decode).  ### Root cause  DSD expands CUDA-grap…

### #51906 — [[Frontend] Add routed-experts prompt offset](https://github.com/vllm-project/vllm/pull/51906)
- **作者**: aoshen02  **时间**: 2026-08-12 10:00 CST
- **标签**: frontend, ready
- **摘要**: ## Summary  - Add `routed_experts_prompt_start` to OpenAI chat/completion requests and `SamplingParams`, allowing clients to omit an already-known prompt prefix from returned R3. - Centralize NumPy-to-base64 serialization used by existing R3 responses and document the `int32` expert-ID representatio…

### #51905 — [[XPU][CI]Change to use global VLLM_DISABLE_COMPILE_CACHE=1 in Intel GPU CI](https://github.com/vllm-project/vllm/pull/51905)
- **作者**: zxd1997066  **时间**: 2026-08-12 09:36 CST
- **标签**: intel-gpu, ci/build
- **摘要**: ## Purpose Change to use global VLLM_DISABLE_COMPILE_CACHE=1 in Intel GPU CI  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [x] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)…

### #51904 — [[Rust Frontend] Support stop strings in the token generate route](https://github.com/vllm-project/vllm/pull/51904)
- **作者**: ricky-chaoju  **时间**: 2026-08-12 08:48 CST
- **标签**: ci/build, rust
- **摘要**: This PR adds stop-string support to `/inference/v1/generate` in the Rust frontend. Python accepts `stop` there (its `sampling_params` is the full `SamplingParams`, and it builds a detokenizer unless `--tokens-only` is set, which the Rust frontend lists as unsupported), and the Rust gRPC token route …

## 🚀 Releases

### [v0.27.1](https://github.com/vllm-project/vllm/releases/tag/v0.27.1)
- **作者**: khluu  **时间**: 2026-08-11 18:47 CST
- **摘要**: This is a patch release on top of v0.27.0.  - Support quantized DSpark Markov heads (#50424)
