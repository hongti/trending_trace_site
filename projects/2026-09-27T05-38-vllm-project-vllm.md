# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-27 13:38 CST

## AI 总结

# vLLM 仓库近期动态摘要

## 📌 Issue（1 条）

- **#58870** — `MRotaryEmbedding` + YaRN 的 Bug：`max_position_embeddings` 被放大 4 倍（为 Qwen2.5-VL 视频位置预留 cos/sin 缓存），但该放大后的值被传递给基类，导致 **YaRN 修正范围和缓存大小不正确**，影响长上下文推理精度。

---

## 🔧 Pull Request（10 条）

### Bug 修复
- **#58879** — 修复上述 #58870，确保 mRoPE YaRN 修正使用**原始上下文长度**而非放大后的值。

### 性能优化
- **#58876** — 在 **Host 端构建 EP Expert Map**，避免在 XLA/TPU 后端初始化时触发设备同步和图重编译，显著加速模型初始化。
- **#58872** — 针对 **DeepSeek-V4.1**，在无序列并行时按 token 跨 TP rank 拆分 Attention 输入投影，减少冗余计算。
- **#58869**（WIP）— 将 ROCm **a4w4 MoE** 默认配置从 DeepSeek V4.1 扩展到 V4（架构共享，尚待测试）。

### 内核 / 硬件支持
- **#58877** — 在 **SM12x**（Blackwell 架构）上启用 OAI Triton MXFP4 MoE 内核。
- **#58878** — 将 CUTLASS W4A8 内核测试**限定为 SM90**，修正架构门控不匹配问题。

### 基础设施 / 构建
- **#58871** — 预编译安装时自动查找兼容的已发布 wheel，解决 main 分支提交后 wheel 尚未发布的空窗期问题。
- **#58873** — 清理 `hadacore_transform_kernel` 中的 TODO，优化存储布局以降低延迟。

### 可观测性 / 连接器
- **#58874** — 为 P/D 异步 KV 加载添加 **KV-fetch 阶段 Gauge 指标**，提升等待请求的可观测性。
- **#58875** — NIXL KVConnector 对未识别请求（租约过期 / ID 不匹配）的完成通知进行计数，避免无效 KV 块日志噪声。

---

## 🚀 Release

本期间**无新版本发布**。

---

### 总结

近期活动以 **DeepSeek 系列模型优化**和 **硬件覆盖扩展**为主线：性能上针对 DeepSeek-V4.1/V4 的 MoE 与 Attention 投影做了 TP 优化和 ROCm 量化支持；硬件上将 MXFP4 MoE 扩展至 Blackwell SM12x。同时修复了影响 Qwen2.5-VL 长上下文的 mRoPE/YaRN 关键 Bug，并持续改善 TPU 后端初始化、预编译构建流程和 P/D 架构可观测性。

---

## 🐛 Issues

### #58870 — [[Bug]: MRotaryEmbedding + YaRN uses 4x original_max_position_embeddings for the YaRN correction range and cache size](https://github.com/vllm-project/vllm/issues/58870)
- **作者**: PenguinPowerUp  **时间**: 2026-09-27 09:54 CST
- **摘要**: ### Your current environment  vLLM v0.30.0 (`vllm/vllm-openai:v0.30.0`), transformers 5.17.0. The same code is on `main` at eb0f2ca. The repro below runs on CPU.  ### 🐛 Describe the bug  When `rope_parameters` contain `mrope_section` and `rope_type: "yarn"` (e.g. Qwen3.5 / Qwen3-VL with YaRN enabled…

## 🔀 Pull Requests

### #58879 — [[Bugfix] Use original context length for mRoPE YaRN correction range](https://github.com/vllm-project/vllm/pull/58879)
- **作者**: adimalkar  **时间**: 2026-09-27 13:18 CST
- **标签**: bug
- **摘要**: ## Purpose  Fixes #58870.  `MRotaryEmbedding` enlarges `max_position_embeddings` 4x as cos/sin cache headroom (for Qwen2.5-VL video positions) and passes the enlarged value to the base class. For YaRN it borrows `YaRNScalingRotaryEmbedding._compute_inv_freq`, which reads `self.max_position_embedding…

### #58878 — [[Test][Quantization] Gate CUTLASS W4A8 kernel tests to SM90](https://github.com/vllm-project/vllm/pull/58878)
- **作者**: Kevinli24  **时间**: 2026-09-27 13:00 CST
- **标签**: nvidia, quantization
- **摘要**: ## Purpose `test_cutlass_w4a8.py` and `test_cutlass_w4a8_moe.py` gate on compute capability >= 90, but the CUTLASS W4A8 kernels are built for 9.0a only (`W4A8_ARCHS` in CMakeLists.txt) and `CutlassW4A8LinearKernel.can_implement` already requires exactly SM90. On any Blackwell GPU the tests run and f…

### #58877 — [[Kernel] Enable OAI Triton MXFP4 MoE on SM12x](https://github.com/vllm-project/vllm/pull/58877)
- **作者**: wtdcode  **时间**: 2026-09-27 12:54 CST
- **标签**: gpt-oss, quantization
- **摘要**: ## Purpose  Disclaimer: The code was largely assisted by Claude Code but the PR itself is handwritten and well tested.  In general, this PR is a follow-up to #41028.  #41028 tries to allow sm120 to use Triton kernels for GPT-OSS models instead of Marlin kernels. However, current implementation of tr…

### #58876 — [[Perf] Build the EP expert map on the host](https://github.com/vllm-project/vllm/pull/58876)
- **作者**: caojx-google  **时间**: 2026-09-27 11:48 CST
- **摘要**: ## Purpose  On lazy/XLA device backends (e.g. TPU via vllm-torchtpu), `ExpertMapManager` triggered a device sync and graph compile per MoE layer during model init:  1. `get_compressed_map_string()` called `.item()` on every element of the device-resident expert map. Its result is an `info_once()` ar…

### #58875 — [[KVConnector][NIXL] Count completion notifications for unrecognized requests](https://github.com/vllm-project/vllm/pull/58875)
- **作者**: liuzijing2014  **时间**: 2026-09-27 11:35 CST
- **标签**: documentation, kv-connector
- **摘要**: ## Purpose  When a decoder finishes reading KV for a request the prefiller no longer tracks (its lease expired, or the request ids don't match), the pull worker logs `Potentially invalid KV blocks for unrecognized request` and moves on. The blocks may already have been reused by then, so this is wor…

### #58874 — [[Metrics][P/D] Add KV-fetch stage gauges for async KV loads](https://github.com/vllm-project/vllm/pull/58874)
- **作者**: liuzijing2014  **时间**: 2026-09-27 11:35 CST
- **标签**: kv-connector, scheduler
- **摘要**: ## Purpose  Requests waiting on an async KV load (a P/D decode request pulling KV from the prefiller, or a `MultiConnector` child loading from a KV store) only show up as `num_requests_waiting_by_reason{reason="deferred"}`, together with LoRA, grammar and streaming waits. When decode TTFT goes up, t…

### #58873 — [Updated TODO Doc](https://github.com/vllm-project/vllm/pull/58873)
- **作者**: prakharPant  **时间**: 2026-09-27 11:26 CST
- **标签**: nvidia
- **摘要**: ## Purpose Resolves an open `TODO` in `hadacore_transform_kernel` regarding the removal of `matrix_transpose_m8_n8_b16_inplace` to decrease latency.   The TODO suggested finding an alternative storage method to avoid the transpose overhead. However, empirical testing on an RTX 3070 Ti (sm_86) confir…

### #58872 — [[Perf][DSv4.1] Split attention input projections by token across TP r…](https://github.com/vllm-project/vllm/pull/58872)
- **作者**: ShuoleiWang  **时间**: 2026-09-27 11:22 CST
- **标签**: deepseek, DSv4.1
- **摘要**: ## Purpose  Without sequence parallelism, every TP rank holds all tokens and runs DeepSeek-V4.1's attention input projections on all of them: - `fused_wqa_wkv` (5120 → 1280 + 512) in every layer; - the compressor's `fused_wkv_wgate` and the indexer's `weights_proj` in the KV and index source layers.…

### #58871 — [[Build] Find compatible published wheels for precompiled installs](https://github.com/vllm-project/vllm/pull/58871)
- **作者**: Gregory-Pereira  **时间**: 2026-09-27 10:39 CST
- **标签**: documentation, ci/build, nvidia
- **摘要**: ## Purpose  Addresses the gap where recent commits on vLLM main don't exist in the wheels index in the 1-2 hours it takes to populate them. When VLLM_USE_PRECOMPILED is enabled and the VLLM_PRECOMPILED_WHEEL_LOCATION is not a path, this logic will walk backwards through 50 commits in git history to …

### #58869 — [[ROCm][Perf][WIP] Extend a4w4 MoE default to DeepSeek V4 (untested)](https://github.com/vllm-project/vllm/pull/58869)
- **作者**: Fangzhou-Ai  **时间**: 2026-09-27 09:31 CST
- **标签**: rocm, needs-rebase, deepseek, quantization, DSv4
- **摘要**: ## Purpose  Follow-up to #58819 (DeepSeek V4.1 a4w4 default). DeepSeek V4 shares the same MoE shape and AITER `GateMode.SEPARATED` dispatch path as V4.1, so this extends `_use_mxfp4_w4a4_moe_activation`'s auto-detection to `deepseek_v4`/`deepseek_v4_text` as well.  **Status: DeepSeek V4 is untested …
