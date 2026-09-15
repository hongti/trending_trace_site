# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-15 13:12 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文简洁摘要：

### 🐛 Issue 动态
*   **#16567 [Bug]**: 在 `vllm-ascend` main 分支（v0.26.0rc）中，当 worker 崩溃后，每个请求都会抛出 `EngineDeadError`。由于复用同一个异常实例，导致异常堆栈帧不断累积并刷屏日志。

### 🚀 PR 动态
**新特性**
*   **#16568 [Feature]**: 在 Ascend 950 设备上为 veRL 支持 **W4A8 MXFP rollout**（FP4 E2M1 权重 + FP8 激活），引入了 `restore_weights_for_rl_loading` 以兼容 veRL，并增加了幂等性和形状检查。

**Bug 修复**
*   **#16570 [BugFix]**: 针对 Issue #16567，在 GLM 用法封装中重新抛出全新的 `EngineDeadError` 实例，避免多请求共享同一异常实例导致的栈帧累积和日志洪水。
*   **#16565 [BugFix]**: 修复 W8A8 场景下 `dispatch_ffn_combine` 的 workspace 和行步长计算错误，改为按 `max(N,K)` 而非 `K` 来分配空间。
*   **#16564 [BugFix]**: 修复 KV Transfer 中混合 KV cache 描述符解析到同一底层分配时，引发的 Mooncake 注册区域重叠问题。
*   **#16574 [BugFix]**: 修复了三个 MRV2 CPU 单元测试用例失效的问题。
*   **#16572 [BugFix]**: 修复 doctest 变更检测逻辑（原两点 Diff 会误纳入 base 分支的其他 PR 变更），避免触发无关测试。

**重构与清理**
*   **#16571 [Refactor]**: 统一 GDN attention 路径，复用上游 output projection 逻辑进行增量重构。
*   **#16569 [Refactor]**: 移除已过时的 `MiniMax-M2` 配置补丁文件。

**CI 与文档**
*   **#16573 [CI]**: 恢复 MRV2 下 DSV4 DSpark 的 CI 验收，将判定标准从“验收率”改为“验收长度”。
*   **#16566 [Doc]**: 为 MiniMax-M3 模型添加 KV cache pool PD 指南，并补充 EAGLE3 草稿模型权重链接。

### 📦 Release 动态
*   本次动态中未包含版本发布信息。

---

## 🐛 Issues

### #16567 — [[Bug]:[main] Per-request EngineDeadError tracebacks accumulate stack frames and flood logs after a worker crash](https://github.com/vllm-project/vllm-ascend/issues/16567)
- **作者**: linnea-lin-00638949  **时间**: 2026-09-15 11:17 CST
- **摘要**: ### Your current environment  | Item | Value | | :--- | :--- | | vllm-ascend | main / v0.26.0rc (incident log from an internal build; relevant frame line numbers match main) | | vllm | v1 engine, Python 3.11 | | Model | any model — the mechanism is model-agnostic (incident observed on a GLM-family M…

## 🔀 Pull Requests

### #16574 — [[BugFix][Test] Fix MRV2 CPU UT regression](https://github.com/vllm-project/vllm-ascend/pull/16574)
- **作者**: MrZ20  **时间**: 2026-09-15 12:47 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  This PR fixes three broken MRV2 CPU unit tests:  - `test_initialize_kv_cache_installs_aclgraph_factory_and_pcp` - `test_prepare_inputs_common_path` - `test_prepare_inputs_covers_draft_full_dcp_pp_and_rswa`  The failure was observed in https://github.com/vllm-…

### #16573 — [[CI][MRV2] Recover the accptance of DSV4 DSpark](https://github.com/vllm-project/vllm-ascend/pull/16573)
- **作者**: wxsIcey  **时间**: 2026-09-15 12:36 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? [CI][MRV2] Recover the accptance of DSV4 DSpark，changing the acceptance rate determination for each location to an acceptance length determination.  ### Does this PR introduce _any_ user-facing change? N/A  ### How was this patch tested? CI passed with new add…

### #16572 — [[BugFix][CI] Fix doctest change detection](https://github.com/vllm-project/vllm-ascend/pull/16572)
- **作者**: MrZ20  **时间**: 2026-09-15 11:56 CST
- **标签**: ci/build, module:tests
- **摘要**: ### What this PR does / why we need it?  The doctest planner used a two-dot diff between the PR base and head. This could include changes merged into the base branch by other PRs and cause unrelated doctests to run.  This PR uses a three-dot diff and the corresponding merge base, so doctests are sel…

### #16571 — [[Refactor][Model] Unify GDN attention paths: reuse upstream output projection](https://github.com/vllm-project/vllm-ascend/pull/16571)
- **作者**: kk-ss1999  **时间**: 2026-09-15 11:45 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Draft: incremental GDN attention refactoring on top of #14639.  **Dependency:** #14639 is still open. This branch includes its metadata-builder changes and merges Ascend main `a643db116`. The main-based diff therefore includes that prerequisite. The attention…

### #16570 — [[BugFix][Platform] Re-raise fresh EngineDeadError in GLM usage wrappers](https://github.com/vllm-project/vllm-ascend/pull/16570)
- **作者**: linnea-lin-00638949  **时间**: 2026-09-15 11:36 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Fixes #16567  When the engine dies, `OutputProcessor.propagate_error()` hands **one shared `EngineDeadError` instance** to every in-flight request. Each request that propagates that same instance up its generator chain prepends its own frames onto the shared …

### #16569 — [[Refactor][Patch] Remove obsolete MiniMax-M2 config patches](https://github.com/vllm-project/vllm-ascend/pull/16569)
- **作者**: AceCoder0  **时间**: 2026-09-15 11:27 CST
- **标签**: ready-precise
- **摘要**: ## Description  Remove `vllm_ascend/patch/platform/patch_minimax_m2_config.py` entirely. Both hooks it installed are obsolete on current main:  ### 1. `_verify_quantization` (force `quantization=None` for MiniMax-M2 fp8 on NPU)  The original premise no longer holds:  - fp8 is now in the Ascend platf…

### #16568 — [[Feature] Support W4A8 MXFP rollout for veRL on Ascend 950](https://github.com/vllm-project/vllm-ascend/pull/16568)
- **作者**: zaney9880  **时间**: 2026-09-15 11:18 CST
- **标签**: module:quantization
- **摘要**: Enables W4A8 MXFP (FP4 E2M1 weights + FP8 activations) rollout in veRL when using Ascend 950 devices. Introduces restore_weights_for_rl_loading for veRL compatibility and adds idempotency and shape-recording checks to protect the weight, w13 and w2 attributes when quantization is enabled.  ### What …

### #16566 — [[doc]: add KV cache pool PD guide for MiniMax-M3](https://github.com/vllm-project/vllm-ascend/pull/16566)
- **作者**: FORFuture37  **时间**: 2026-09-15 11:13 CST
- **标签**: documentation
- **摘要**: Changes in docs/source/tutorials/models/MiniMax-M3.md:  Add the MiniMax-M3-EAGLE3 draft model weight link (ModelScope) in Section 3.1, used by the EAGLE3 speculative decoding examples. Reword the Section 5.1 single-node guidance: BF16 on Atlas 800 A3 is not recommended for single-node deployment; du…

### #16565 — [[BugFix] dispatch_ffn_combine W8A8: size GMM1 C workspace and row strides by max(N,K) instead of K](https://github.com/vllm-project/vllm-ascend/pull/16565)
- **作者**: Mr-qiji  **时间**: 2026-09-15 11:12 CST
- **摘要**: ### What this PR does / why we need it? Fixes #16561  In the W8A8 variant of `dispatch_ffn_combine` (`csrc/mc2/dispatch_ffn_combine/`), the int16 workspace holding the GMM1 output (M×N, N = 2×moe_intermediate, the SwiGLU input) is **allocated with row width K** (host tiling: `maxOut × n2 × sizeof(in…

### #16564 — [[BugFix][KV Transfer] Merge overlapping Mooncake registration regions](https://github.com/vllm-project/vllm-ascend/pull/16564)
- **作者**: iKeybot-code  **时间**: 2026-09-15 10:42 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Hybrid KV cache descriptors can refer to overlapping ranges when shared descriptor buffers and private layer storages resolve into the same backing allocation. The Mooncake transfer engine rejects registering those ranges independently with `Transfer Engine d…
