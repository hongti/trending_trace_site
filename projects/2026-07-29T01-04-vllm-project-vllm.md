# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-29 09:04 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 🐛 Issue
- **MoE 后端并发崩溃 (#50189)**：在 Blackwell 架构（SM120, RTX PRO 6000）下，使用 `flashinfer_b12x` MoE 后端对 Qwen3.5-122B-A10B-NVFP4 模型进行并发 chunked prefill 时，触发 Xid 31 MMU fault（非法写入）及 `cudaErrorIllegalAddress` 导致引擎崩溃。
- **前缀缓存与投机解码冲突 (#50188)**：开启显式前缀缓存（`--enable-prefix-caching`）并配合 MTP 投机解码与工具调用时，若重复相同提示词，会导致工具调用输出损坏。

### 🔧 Pull Request
**重要修复：**
- **CPU FP8 Attention 修复 (#50194)**：修复了 CPU attention 调度器中 FP8 KV cache 的 tile 大小计算错误，解决了 BF16 查询搭配 FP8 KV 时分配空间过大的问题。
- **投机解码修复 (#50187, #50183)**：
  - 修复了 EAGLE3 辅助隐藏层方法名的拼写错误，该错误影响了 DeepSeek-V2、Qwen3.5 等模型的正确调用。
  - 修复了拒绝采样器 `tl.argmax` 在处理全 NaN 块时产生越界索引导致异常的问题。
- **通信器逻辑修复 (#50182)**：修复了 `SymmMemCommunicator` 的初始化逻辑，现在仅在选择 `multimem` 算法时才强制要求 multicast 指针，避免了不必要的通信器禁用。
- **V1 初始化稳定 (#50192)**：增强了 V1 GPU 模型运行器的初始化过程，修复了 AMD CI 中的间歇性失败。

**性能与功能改进：**
- **Attention 内核延迟优化 (#50185)**：通过使用向量化加载、将 `B` 和 `N` 作为编译时常量以及设置 `NC=3` 等手段，显著降低了 `attn_res` 内核的延迟。
- **Helion 改进 (#50193)**：改进了 fused QK 的配置选择逻辑。

**工程与 CI 优化：**
- **前端规范化 (#50191)**：在 OpenAI 兼容前端的批量请求 URL 校验中，统一使用类型化的 `VLLMValidationError` 替代原生 `ValueError`。
- **ROCm CI 稳定 (#50190)**：稳定了 ROCm 环境下 ngram 和 suffix 的正确性测试。
- **依赖升级 (#50186)**：通过 Dependabot 批量升级了 173 个依赖包（小版本更新）。

### 🚀 Release
- 本期暂无新版本发布。

---

## 🐛 Issues

### #50189 — [[Bug]: Xid 31 MMU fault (illegal write) with flashinfer_b12x MoE backend under concurrent chunked prefill (SM120, Qwen3.5-122B-A10B-NVFP4)](https://github.com/vllm-project/vllm/issues/50189)
- **作者**: jhsmith409  **时间**: 2026-07-29 07:17 CST
- **摘要**: ### Summary  Serving `nvidia/Qwen3.5-122B-A10B-NVFP4` with the forced `flashinfer_b12x` MoE backend on SM120 (RTX PRO 6000 Blackwell Max-Q, TP=1), the engine crashes with `cudaErrorIllegalAddress` + a driver-level **Xid 31 MMU fault (`FAULT_PDE ACCESS_TYPE_VIRT_WRITE`)** about 30 s into a load test …

### #50188 — [[Bug]: Explicit --enable-prefix-caching with MTP spec decode + tool calling corrupts tool-call emission on repeated identical prompts](https://github.com/vllm-project/vllm/issues/50188)
- **作者**: gr4ig  **时间**: 2026-07-29 07:16 CST
- **标签**: quantization
- **摘要**: ### Your current environment  `collect_env` was not available on this host — abridged environment:  - vLLM: **0.23.0** (venv install) - GPU: **NVIDIA GeForce RTX 5090** (32,607 MiB), driver **595.71.05**, CUDA **13.0** - `FLASHINFER_CUDA_ARCH_LIST=12.0f`, `PYTORCH_CUDA_ALLOC_CONF=expandable_segments…

## 🔀 Pull Requests

### #50194 — [[CPU] Fix FP8 attention scratchpad sizing](https://github.com/vllm-project/vllm/pull/50194)
- **作者**: tianmu-li  **时间**: 2026-07-29 08:30 CST
- **标签**: cpu
- **摘要**: ## Purpose  The CPU attention scheduler computed FP8 KV tile geometry using `sizeof(kv_cache_t)`. With BF16 queries and one-byte FP8 KV cache entries, this selected larger tiles than the BF16-backed scratchpad can hold during large AMX prefills. The resulting out-of-bounds writes cause corrupted out…

### #50193 — [[Helion] Improve fused QK config selection](https://github.com/vllm-project/vllm/pull/50193)
- **作者**: yushangdi  **时间**: 2026-07-29 07:57 CST
- **摘要**: ## Purpose  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such as providing test command. - [ ] The …

### #50192 — [[Bugfix] Stabilize V1 GPU model runner initialization](https://github.com/vllm-project/vllm/pull/50192)
- **作者**: AndreasKaratzas  **时间**: 2026-07-29 07:33 CST
- **标签**: bug, v1
- **摘要**: Hardens V1 GPU model-runner initialization after the intermittent AMD CI failure in [Buildkite build 11367](https://buildkite.com/vllm/amd-ci/builds/11367/list?jid=019fa7f3-f1e7-4758-a230-01d7b66fed3e&tab=output). The profiling warmup covered only mixed sampling metadata, while suffix can invoke the…

### #50191 — [[Frontend] Use VLLMValidationError for batch request URL validation](https://github.com/vllm-project/vllm/pull/50191)
- **作者**: tanchao  **时间**: 2026-07-29 07:26 CST
- **标签**: frontend
- **摘要**: ## Purpose  Small follow-up to the ongoing effort to adopt the typed `VLLMValidationError` in the OpenAI-compatible frontend (merged #49214, #49217). This converts the remaining raw `ValueError` raises in `download_bytes_from_url` (`vllm/entrypoints/openai/run_batch.py`) to `VLLMValidationError`. Th…

### #50190 — [[ROCm][CI] Stabilize ngram and suffix correctness test](https://github.com/vllm-project/vllm/pull/50190)
- **作者**: AndreasKaratzas  **时间**: 2026-07-29 07:24 CST
- **标签**: rocm, ready, v1
- **摘要**: Stabilizes the ROCm `test_ngram_and_suffix_correctness` group after the intermittent AMD CI failure in [Buildkite build 11367](https://buildkite.com/vllm/amd-ci/builds/11367/list?jid=019fa7f3-f1e7-4758-a230-01d7b66fed3e&tab=output). The failing run completed the ngram case, force-killed its engine a…

### #50187 — [[Bugfix][Spec Decode] Fix misspelled EAGLE3 aux-hidden-state layer method overrides](https://github.com/vllm-project/vllm/pull/50187)
- **作者**: Henry-Cheng  **时间**: 2026-07-29 06:45 CST
- **标签**: bug, qwen, deepseek, mistral, kimi
- **摘要**: `SupportsEagle3` declares `get_eagle3_default_aux_hidden_state_layers`, and the runtime (gpu_model_runner, eagle3 utils) only ever calls that name. However deepseek_v2, qwen3_5, pixtral, and kimi_k25 overrode a misspelled `get_eagle3_aux_hidden_state_layers` (missing `default`), so those overrides w…

### #50186 — [Bump the minor-update group across 1 directory with 173 updates](https://github.com/vllm-project/vllm/pull/50186)
- **作者**: dependabot[bot]  **时间**: 2026-07-29 06:20 CST
- **标签**: rocm, ci/build, cpu, nvidia, dependencies
- **摘要**: Bumps the minor-update group with 173 updates in the / directory:  | Package | From | To | | --- | --- | --- | | [tblib](https://github.com/ionelmc/python-tblib) | `3.1.0` | `3.2.2` | | [regex](https://github.com/mrabarnett/mrab-regex) | `2026.2.28` | `2026.7.19` | | [requests](https://github.com/ps…

### #50185 — [attn_res kernel latency improvements](https://github.com/vllm-project/vllm/pull/50185)
- **作者**: gnovack  **时间**: 2026-07-29 05:53 CST
- **标签**: kimi, k3
- **摘要**: ## Purpose A few small improvements to reduce the latency of the attention residual kernel: - Use vectorized loads when populating `q_cache` - Treat `B` and `N` as compile-time constants - Set `NC = 3` for low batch size path - Unroll the main loop over `num_chunks`  ## Test Plan  ``` pytest -v test…

### #50183 — [[Bugfix][Spec Decode] Fix NaN handling in rejection sampler tl.argmax](https://github.com/vllm-project/vllm/pull/50183)
- **作者**: gabriel-peracio  **时间**: 2026-07-29 05:06 CST
- **标签**: bug, v1, v2
- **摘要**: ## Summary  `tl.argmax` on an all-NaN block returns an out-of-range block index (pointing into the padded region beyond `num_blocks - 1`). That index is used to load from the local-argmax tensor, producing an out-of-bounds read and an illegal memory access downstream in the rejection sampler.  Two k…

### #50182 — [[Bugfix] Only require symm-mem multicast when multimem is the selected algorithm](https://github.com/vllm-project/vllm/pull/50182)
- **作者**: iemAnshuman  **时间**: 2026-07-29 05:02 CST
- **标签**: bug
- **摘要**: ## Purpose  Fixes #50179.  `SymmMemCommunicator.__init__` disables the communicator whenever the rendezvous handle has no multicast pointer:  ```python if handle.multicast_ptr == 0:     logger.warning("SymmMemCommunicator: symmetric memory "                    "multicast operations are not supported…
