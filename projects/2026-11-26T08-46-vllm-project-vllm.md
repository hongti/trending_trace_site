# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-26 16:46 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### ⚠️ Issue 与 Release 动态
本次提供的动态列表中暂无 Issue 和 Release 类型的活动，以下仅对 Pull Request 进行归纳。

---

### 🔧 Pull Request 摘要

**1. 安全性增强（#70081, #70082, #70083, #70084）**
本批次包含多项安全加固改进：
- **#70081**：在调度后台 Responses 任务前，强制检查服务器的存储能力和存储策略，防止不安全的后台写入。
- **#70082**：在引擎准入阶段验证传输的 M-RoPE 位置偏移，检查签名暂存边界和加载的旋转表容量，确保位置数据合法。
- **#70083**：在准备每个批次前清理原生 DeepStack 的暂存区，避免上一个批次的残留状态污染当前解码路径。
- **#70084**：在引擎传输期间保持停止 Token（stop IDs）的严格边界，将配置的 EOS 成员与显式停止列表分离，防止越界。

**2. ROCm / AITER 相关优化与修复（#70085, #70086, #70087）**
- **#70087**：为 ROCm 引入 AITER 的原地（in-place）融合加法 RMSNorm 操作，优化 IR 操作性能并填补之前的 TODO。
- **#70086**：优化 ROCm CI，让镜像缓存机器人仅监听特定的两个任务状态，而不是所有构建的最终状态，提升 CI 效率。
- **#70085**：修复当 `fuse_allreduce_rms=True` 且禁用 AITER 时导致的 ROCm 启动崩溃问题，现会自动跳过不可用的 all-reduce 融合。

**3. 其他重要修复（#70080, #70088, #70089）**
- **#70088**：修复在 Intel XPU / WSL2 等环境下 `torch.xpu.mem_get_info()` 持续返回静态空闲内存的问题，现会回退到已分配权重的内存查询，避免内存分配异常。
- **#70080**：修复 `causal_conv1d_fn` 前向 Triton 内核中未正确跳过 `pad_slot_id` 条目的问题，使其与注意力后端的约定保持一致。
- **#70089**：Rust 前端在反序列化请求时，拒绝 `parameters` 非 JSON 对象的函数工具，与 Python 端的校验逻辑对齐。

---

## 🔀 Pull Requests

### #70089 — [[Rust Frontend] Reject non-object function tool parameters](https://github.com/vllm-project/vllm/pull/70089)
- **作者**: BugenZhao  **时间**: 2026-11-26 16:45 CST
- **标签**: rust
- **摘要**: ## Overview  Reject a function tool whose `parameters` is not a JSON object (or `null`/omitted) at request deserialization in the Rust frontend, on chat and Responses and in gRPC Render, as Python and the OpenAI API do. Today any JSON value is accepted and reaches the renderer.  ## Claims  - `/v1/ch…

### #70088 — [fix(memory): fallback to allocated weights memory when mem_get_info returns static free memory](https://github.com/vllm-project/vllm/pull/70088)
- **作者**: hemal9102  **时间**: 2026-11-26 16:42 CST
- **摘要**: ### Problem Fixes #69897  On environments such as Intel XPU with Level Zero drivers under Windows 11 / WSL2 and integrated GPUs, querying raw memory via `torch.xpu.mem_get_info()` continuously returns `free == total` even after several GiBs of model weights have been allocated.  Consequently: ```pyt…

### #70087 — [[ROCm][AITER] Support in-place fused add RMSNorm](https://github.com/vllm-project/vllm/pull/70087)
- **作者**: lunarwhite  **时间**: 2026-11-26 16:39 CST
- **标签**: rocm
- **摘要**: ## Overview  Use AITER's in-place fused add RMSNorm for the mutable IR operation, completing the existing in-place implementation TODO. Keep the functional implementation for functional calls and compilation.  ## Details  The current AITER implementation allocates two output tensors even when the IR…

### #70086 — [[ROCm][CI] Wake the ROCm image cache bot on its two jobs' statuses](https://github.com/vllm-project/vllm/pull/70086)
- **作者**: AndreasKaratzas  **时间**: 2026-11-26 16:29 CST
- **标签**: rocm, ci/build
- **摘要**: - `rocm-image-cache.yml` (and its reusable workflow) listen to the `ci/step: ROCm image cache` and `ci/step: Cache a PR's ROCm images for main` statuses instead of every build's final `buildkite/ci/pr` and `buildkite/ci` status. - `handle_buildkite_status` reads the build number from job links (`...…

### #70085 — [[Bugfix][ROCm] Skip unavailable all-reduce fusion without AITER](https://github.com/vllm-project/vllm/pull/70085)
- **作者**: lunarwhite  **时间**: 2026-11-26 16:25 CST
- **标签**: bug, rocm, torch.compile
- **摘要**: ## Overview  Avoid a ROCm startup crash when `fuse_allreduce_rms=True` is explicitly set with AITER disabled. Keep the CUDA-only FlashInfer import boundary established by #22202.  ## Claims  - Unsupported ROCm/non-AITER fusion settings no longer reference an undefined `AllReduceFusionPass`. - CUDA/F…

### #70084 — [[Security] Preserve stop-token bounds over engine transport](https://github.com/vllm-project/vllm/pull/70084)
- **作者**: jperezdealgaba  **时间**: 2026-11-26 16:22 CST
- **标签**: rust
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Keep explicit stop IDs bounded across engine transport while preserving generation-config EOS behavior.  ## Claims  - Keep configured EOS membership separate from the explicit stop list in the Python and Rust frontends. - Preserve stopping behavior for leg…

### #70083 — [[Security] Clear DeepStack scratch before preparing each batch](https://github.com/vllm-project/vllm/pull/70083)
- **作者**: jperezdealgaba  **时间**: 2026-11-26 16:22 CST
- **标签**: multi-modality, qwen, cohere
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Clear native DeepStack scratch during batch preparation so eager and captured decoder paths consume the current batch's state.  ## Claims  - Reset the prior batch's scratch on the worker stream before preparing the next batch. - Reuse the existing microbat…

### #70082 — [[Security] Validate transferred M-RoPE positions](https://github.com/vllm-project/vllm/pull/70082)
- **作者**: jperezdealgaba  **时间**: 2026-11-26 16:22 CST
- **标签**: documentation, kv-connector
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Validate transferred M-RoPE position offsets at engine admission.  ## Claims  - Check signed staging bounds and the loaded rotary table capacities on decoder ranks. - Apply the same checks to requests from the Python and Rust frontends before dispatch.  ##…

### #70081 — [[Security] Validate storage before scheduling background Responses](https://github.com/vllm-project/vllm/pull/70081)
- **作者**: jperezdealgaba  **时间**: 2026-11-26 16:22 CST
- **标签**: frontend
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Enforce the Python Responses storage policy before scheduling background work.  ## Claims  - Check the normalized background and store flags. - Check the server's storage capability before rendering, engine enqueue, or response retention.  ## Validation  L…

### #70080 — [[Bugfix] causal_conv1d_fn: skip pad_slot_id entries in fwd kernel](https://github.com/vllm-project/vllm/pull/70080)
- **作者**: XFDG  **时间**: 2026-11-26 16:20 CST
- **标签**: bug
- **摘要**: ## Why  `causal_conv1d_fn` documents that a sequence whose `cache_indices` entry equals `pad_slot_id` is **not processed** (the same convention used by the attention backends). The forward Triton kernel `_causal_conv1d_fwd_kernel` does not honor this: it loads each sequence's cache index (`conv_stat…
