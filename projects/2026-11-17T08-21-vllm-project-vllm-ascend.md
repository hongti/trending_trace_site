# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-17 16:21 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期的动态摘要：

### 🚀 Pull Requests (PR)
本期 PR 活动主要围绕性能优化、新特性支持、Bug 修复以及 CI 流程改进展开：

**1. 新特性与功能增强**
*   **KV 传输与共享**：#22157 为 `MooncakeConnectorV2` 引入了 KV 共享层支持（允许某层读取其他层的 KV cache）；#22158 改进了 AscendStore KV 事件，使 Mooncake Conductor 等外部消费者能正确解析混合注意力缓存。
*   **模型支持**：#22153 为 Qwen GDN RecoverSSM 添加了实验性的 PD（Prefill/Decode）分离支持。
*   **可观测性**：#22155 增加了分层 KV pool 加载耗时的上报功能。

**2. 性能优化与重构**
*   **缓存优化**：#22161 优化了 A2/A3 SparseFlashMla CSA 的地址缓存路径，并添加了可选的诊断功能；#22160 在 MRV2 中为 Qwen3.5 混合缓存复用了上游 vLLM v0.33.0 的 KV cache 分配器。

**3. Bug 修复**
*   **Attention 修复**：#22159 修复了 GLM DFlash2 FA3 HND 缓存及目标图预热问题；#22156 修复了在 FULL-graph 投机解码中，DSpark 和 DFlash 批量填充导致的 attention 元数据不匹配问题。

**4. CI 与测试**
*   **流程优化**：#22154 优化了 CI 流程，CPU UT 仅在选择用例包含时才运行；#22152 添加了 P1 pool A/B 测试用例，用于复现 CI 运行器上的 FabricMem 32 TiB 预留失败问题。

### ⚠️ Issues
*   本期暂无公开的 Issue 动态。

### 🏷️ Releases
*   本期暂无新的 Release 发布。

---

## 🔀 Pull Requests

### #22161 — [perf: optimize A2/A3 SparseFlashMla CSA address cache and add diagnostics](https://github.com/vllm-project/vllm-ascend/pull/22161)
- **作者**: lixiang518  **时间**: 2026-11-17 16:15 CST
- **摘要**: ## What this PR does / why we need it  Optimize the A2/A3 SparseFlashMla CSA address-cache path while retaining guarded scalar fallback. Add opt-in host/device diagnostics to distinguish host-side eligibility from actual device cache copies, with documentation and regression coverage.  - Update Spar…

### #22160 — [[Refactor][MRV2] Reuse upstream KV allocation for Qwen3.5 hybrid cache](https://github.com/vllm-project/vllm-ascend/pull/22160)
- **作者**: lilillian  **时间**: 2026-11-17 16:08 CST
- **摘要**: ### What this PR does / why we need it?  Reuse the vLLM v0.33.0 KV-cache allocator for Qwen3.5-style FullAttention/GDN hybrid models in MRV2. The upstream allocator places complete manager pages in the shared backing allocation. Ascend then splits those pages into K/V kernel-block views and exposes …

### #22159 — [[BugFix][Attention] Fix GLM DFlash2 FA3 HND caches and target graph warmup](https://github.com/vllm-project/vllm-ascend/pull/22159)
- **作者**: aisong1988  **时间**: 2026-11-17 16:00 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Standalone follow-up to #21850, rebased onto main at `8cd3c3762`. Prerequisite #20167 is now merged; this PR contains only the six-file follow-up increment, not its prerequisite commits.  With a GLM MLA/SFA target and an eager DFlash2 draft, the draft needs a…

### #22158 — [[Feature] Mooncake support publish hybrid KV metadata](https://github.com/vllm-project/vllm-ascend/pull/22158)
- **作者**: Misak2333  **时间**: 2026-11-17 15:55 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Improve AscendStore KV events so external consumers such as Mooncake Conductor can correctly interpret hybrid attention caches.  Individual group events do not describe the complete deployment layout. Without knowing all required cache groups and Store object …

### #22157 — [[Feature][KV Transfer] Support KV-sharing layers in MooncakeConnectorV2](https://github.com/vllm-project/vllm-ascend/pull/22157)
- **作者**: dwoai22  **时间**: 2026-11-17 15:50 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  A model can let a layer read another layer's KV cache instead of keeping its own, by giving it `kv_sharing_target_layer_name` (upstream examples: Gemma 3n and Gemma 4). Such a *borrower* gets no KV cache spec and no `kv_cache_tensors` entry, so it owns no blo…

### #22156 — [[Bugfix][SpecDecode] Align padded draft attention metadata for DSpark and DFlash](https://github.com/vllm-project/vllm-ascend/pull/22156)
- **作者**: shenyunyang  **时间**: 2026-11-17 15:44 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Fixes attention request-metadata mismatches during FULL-graph speculative decoding with padded batches. The service can start successfully, then fail during inference when the active request count no longer fills the selected draft graph.  The attention build…

### #22155 — [[Feature][Observability] Report layerwise KV pool load get duration](https://github.com/vllm-project/vllm-ascend/pull/22155)
- **作者**: ywwuyi-rice  **时间**: 2026-11-17 15:42 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  The AscendStore connector reports load duration only for the non-layerwise paths. Layerwise loads issue one copy call per layer and bypass that channel, so the per-layer load time stays invisible in `/metrics` whenever layerwise transfer is enabled.  This PR …

### #22154 — [[CI]chore: run CPU UT only when selected cases include it](https://github.com/vllm-project/vllm-ascend/pull/22154)
- **作者**: shiqiangA  **时间**: 2026-11-17 15:36 CST
- **标签**: ci/build
- **摘要**: UT no longer starts on every PR pipeline trigger. After test selection, CPU UT runs only when the selected cases include UT, and E2E starts after that UT succeeds. A selection with no UT skips CPU UT.  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ###…

### #22153 — [[Feature][Model] Add experimental Qwen GDN RecoverSSM PD support](https://github.com/vllm-project/vllm-ascend/pull/22153)
- **作者**: Nothing-98  **时间**: 2026-11-17 15:31 CST
- **标签**: module:tests, module:ops, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  Qwen4Exp/Flash Next currently has standalone GDN RecoverSSM kernels but lacks a model-runner integration that preserves its transferable convolution/SSM cache layout. This draft adds an explicit, disabled-by-default integration for speculative verification fo…

### #22152 — [test: add P1 pool A/B case on base 3f1de8b04](https://github.com/vllm-project/vllm-ascend/pull/22152)
- **作者**: lizy124  **时间**: 2026-11-17 15:16 CST
- **标签**: module:tests
- **摘要**: Minimal P1-on-base branch for reproducing the FabricMem 32 TiB reservation failure on the CI runner.  - Base: 3f1de8b04 (upstream main) - P1 test and kv_pool_watch.py from 296741b31 - conftest.py: only the VllmRunner.get_metrics() / DPVllmRunner.get_metrics() additions - kv_pool_runtime.py and kv_po…
