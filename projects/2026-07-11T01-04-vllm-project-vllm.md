# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-11 09:04 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库最近的动态摘要：

### .issue 动态 (Issues)
近期 Issue 主要集中在 **DeepseekV3 相关后端**的配置与兼容性问题：
*   **配置缺失问题**：`DeepseekV32IndexerBackend` 运行强制要求 `--block-size 64`，但目前既未在文档中说明，也未实现自动检测，导致用户使用受阻 (#48286)。
*   **架构兼容性崩溃**：使用 `TRITON_MLA_SPARSE` 后端在 SM80 架构上进行 CUDA graph 捕获时，其 fallback 机制会发生崩溃 (#48285)。

---

### 🔀 PR 动态 (Pull Requests)
近期 PR 涵盖了新模型支持、核心架构升级、关键 Bug 修复及 CI 优化等重要变更：

**1. 新模型与文档支持**
*   **新增 Cosmos3 Edge Reasoner 模型**：引入了全新的混合 Transformer 架构模型，包含 Reasoner 和 Generator 双塔结构 (#48291)。
*   **完善模型文档**：将 `DeepseekV32ForCausalLM` 添加至 `supported_models.md` 文档中 (#48293)。

**2. 核心架构与特性升级**
*   **MRV2 扩展适用范围**：Model Runner v2 (MRV2) 默认启用范围进一步扩大，现已覆盖所有 pooling 模型 (#48290)。
*   **Offloading 机制增强**：为 KV Offload（Mooncake 等）引入了内部 session id 传递机制 (#48288)；同时为 FS/OBJ KV 事件添加了 P2P 可达性元数据，优化卸载路由 (#48281)。
*   **MoE 并行优化**：新增 pad-aware swiglu limit kernel，改善了专家并行下 MoE 输入在 token 维度的 padding 处理效率 (#48287)。
*   **可观测性提升**：在 Core 迭代细节日志中新增 `result wait time` 指标，用于测量 `future.result()` 调用的实际耗时 (#48292)。

**3. 关键 Bug 修复**
*   **修复权重重载破坏模型问题**：运行时重载权重会破坏混合 Mamba 模型（如 NemotronH），此 PR 通过让非量化模型跳过逐层重载来修复该问题 (#48284)。
*   **修复流式输出丢事件 Bug**：修复了 SSE 流式输出中，零增量项的生命周期事件（added/done）被静默丢弃的严重问题 (#48282)。

**4. CI 与构建优化**
*   **macOS 构建改进**：在新的 macmini Buildkite 队列上原生构建 macOS Apple Silicon (arm64) CPU wheel，并自动化发布至官方源 (#48289)。

---

### 🚀 Release 动态
*   本期监测范围内**无新版本发布**。

---

## 🐛 Issues

### #48286 — [DeepseekV32IndexerBackend requires --block-size 64 — not documented or auto-detected](https://github.com/vllm-project/vllm/issues/48286)
- **作者**: biondogs  **时间**: 2026-07-11 06:40 CST
- **摘要**: **Title:** `DeepseekV32IndexerBackend` requires `--block-size 64` — not documented or auto-detected  **vLLM version:** v0.24.0  **Backend:** `TRITON_MLA_SPARSE` / `DeepseekV32IndexerBackend` (PR [#38476](https://github.com/vllm-project/vllm/pull/38476))  ---  `DeepseekV32IndexerBackend` (used by GLM…

### #48285 — [TRITON_MLA_SPARSE fallback crashes on SM80 during CUDA graph capture](https://github.com/vllm-project/vllm/issues/48285)
- **作者**: biondogs  **时间**: 2026-07-11 06:39 CST
- **摘要**: **Title:** TRITON_MLA_SPARSE fallback crashes on SM80 during CUDA graph capture  **vLLM version:** v0.24.0  **Backend:** `TRITON_MLA_SPARSE` (PR [#38476](https://github.com/vllm-project/vllm/pull/38476), head `3740c02`)  **Hardware:** NVIDIA A100 SM80 (12 GPUs, TP=4, PP=3)  **Model:** FenomAI/GLM-5.…

## 🔀 Pull Requests

### #48293 — [[Doc] Add DeepseekV32ForCausalLM to supported_models.md](https://github.com/vllm-project/vllm/pull/48293)
- **作者**: Gavin-Morris-04  **时间**: 2026-07-11 07:47 CST
- **标签**: documentation, deepseek
- **摘要**: ## Purpose  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such as providing test command. - [ ] The …

### #48292 — [[Core] Log result wait time in iteration details](https://github.com/vllm-project/vllm/pull/48292)
- **作者**: SillageNULL  **时间**: 2026-07-11 07:46 CST
- **标签**: v1
- **摘要**: ## Purpose  This PR adds `result wait time` to the existing iteration-detail logs.  The metric measures the wall-clock interval from immediately before calling `future.result()` until the call returns. It is recorded in both `EngineCore.step()` and `EngineCore.step_with_batch_queue()`.  The metric i…

### #48291 — [Add Cosmos3 Edge Reasoner model](https://github.com/vllm-project/vllm/pull/48291)
- **作者**: adsridhar  **时间**: 2026-07-11 07:31 CST
- **标签**: documentation, new-model, needs-rebase, multi-modality
- **摘要**: ## Purpose This PR brings in a new Cosmos3 Edge Reasoner model (not yet released) to vllm. Cosmos3 is a mixture of transformers model consisting of a Reasoner and a Generator tower. This PR introduces custom model layers, processors, and performs checkpoint mapping. ## Test Plan Checkpoint loading a…

### #48290 — [[Model Runner v2] Enable MRV2 for all pooling models by default](https://github.com/vllm-project/vllm/pull/48290)
- **作者**: taneem-ibrahim  **时间**: 2026-07-11 07:23 CST
- **摘要**: ##Purpose Follow up on #46646, which enables ModelRunner v2 by default for non-pooling models. This PR extends the default MRV2 eligibility check to pooling models while keeping other runner types, such as draft runners, excluded.  ## Test plan  `python -m pytest tests/test_config.py::test_is_defaul…

### #48289 — [[CI] Build macOS arm64 CPU wheel natively on the macmini queue](https://github.com/vllm-project/vllm/pull/48289)
- **作者**: mgoin  **时间**: 2026-07-11 07:03 CST
- **标签**: ready, ci/build
- **摘要**: Builds the macOS Apple Silicon (arm64) CPU wheel natively on the new `macmini` Buildkite queue, then publishes it to `wheels.vllm.ai` via the normal release upload path.  Supersedes #46660: now that a macOS Buildkite agent exists, we build directly on it instead of dispatching a GitHub-hosted runner…

### #48288 — [feat: Internal session id for offloading ](https://github.com/vllm-project/vllm/pull/48288)
- **作者**: karen-sy  **时间**: 2026-07-11 06:58 CST
- **标签**: frontend, v1, kv-connector, rust
- **摘要**: ## Purpose  Lightweight propagation of session id to vLLM offloading (Mooncake, OffloadingConnector) constructs. This PR is stacked on #48048 and depends on its typed request-level `session_id` plumbing.  Context: - DEP/RFC: #48049 - Base session ID plumbing PR: #48048 - Related motivation: sglang#2…

### #48287 — [add pad-aware swiglu limit kernel](https://github.com/vllm-project/vllm/pull/48287)
- **作者**: gnovack  **时间**: 2026-07-11 06:45 CST
- **摘要**: ## Purpose When serving with expert-parallelism enabled, the MoE inputs and intermediate states are padded along the token dimension to handle the worst case (i.e. all tokens routed to a single rank; see https://github.com/vllm-project/vllm/blob/main/vllm/model_executor/layers/fused_moe/prepare_fina…

### #48284 — [Skip layerwise reload for non-quantized models](https://github.com/vllm-project/vllm/pull/48284)
- **作者**: yuchenwang3  **时间**: 2026-07-11 06:20 CST
- **标签**: v1
- **摘要**: Reloading weights at runtime (`reload_weights(is_checkpoint_format=True)`) corrupts hybrid Mamba models such as NemotronH. Reloading the model's own original weights already changes greedy decoding (e.g. `" 360"` becomes `" 150 150 ..."`), which breaks RLHF/RLVR weight sync for these models.  `is_ch…

### #48282 — [[Bugfix] Emit added/done lifecycle events for zero-delta streaming items](https://github.com/vllm-project/vllm/pull/48282)
- **作者**: mimran-khan  **时间**: 2026-07-11 05:45 CST
- **标签**: bug, frontend
- **摘要**: Fixes #48274  ## What's going on  There's a known bug in `emit_previous_item_done_events` where items that complete with zero deltas get silently dropped from the SSE stream. The code already acknowledges it:  ```python if not state.sent_output_item_added and not state.is_first_function_call_delta: …

### #48281 — [[KV Offload] Add P2P reachability metadata to FS/OBJ KV events](https://github.com/vllm-project/vllm/pull/48281)
- **作者**: Change72  **时间**: 2026-07-11 05:42 CST
- **标签**: documentation, v1, kv-connector
- **摘要**: ## Purpose  Follow up on [the #47923 review discussion](https://github.com/vllm-project/vllm/pull/47923#discussion_r3541681707) by adding an explicit, optional reachability signal for the built-in FS and OBJ secondary tiers.  The same FS tier can represent node-local or shared storage, so `medium` a…
