# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-02 12:25 UTC

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📋 Issue 动态
- **架构探讨 (RFC)**：[#11356] 提出 **PD分离的 Mamba 状态独立通道路由**。旨在解决当前 PD 分离导致混合模型（如 DeepSeek V4 的 Mamba 层）前缀缓存命中率降至零的问题，避免全局 LCM 对齐造成的结构冲突。
- **特性请求**：[#11357] 希望适配 **Unlimited-OCR**，作者指出其精度和效率优于 Deepseek OCR，请求提供支持。
- **CI 问题**：[#11360] 计划暂时移除 **Qwen3-235B-A22B PD分离搭配 mooncake 分层 connector** 的不稳定测试用例，待后续重新开发。

### 🔧 PR 动态
**✨ 新特性与性能优化**
- **多流重叠扩展**：[#11355] 为 W4A4_MXFP4 量化 MoE 模型的共享专家启用 **4阶段多流重叠**，性能对齐现有的 W8A8 路径，解决此前开启该功能时的性能瓶颈。
- **调度策略增强**：[#11352] 引入默认关闭的 **ShortRequestFirst（短请求优先）** 调度策略，以减少混合长短提示词工作负载下的 Prefill 队头阻塞问题。
- **KV Cache 追踪**：[#11350] 为 KV cache 调度热路径（如 HBM 前缀查找）添加 **性能追踪（KVTRACE）**，便于诊断生产环境中的前缀缓存性能退化。
- **存储键构造优化**：[#11351] 优化 **AscendStore 键构造**，避免在热路径上创建 `PoolKey` 对象，缓存稳定前缀并实现分组块哈希延迟计算，提升效率。

**🐛 Bug 修复**
- **310P 挂死修复**：[#11353] 修复了 **310P 设备上 MTP/推测解码时 Mamba align 后处理挂起**的问题，为 310P 添加了特定的回退机制（避免启动 Triton 融合路径，改用 CPU 元数据更新与状态拷贝）。

**🧪 测试与 CI 修复**
- **单元测试补全**：[#11354] 为 AscendStore 及 KV transfer 相关组件（配置、调度、Worker、Connector等）添加了基于 mock 的全面单元测试。
- **EPLB 测试修复**：[#11361] 修复了 RFork 原生 EPLB loader 单元测试中的 `AttributeError`（构造的轻量级 vllm_config 缺少 `parallel_config`）。
- **CI 缓存与流水线修复**：[#11359] 修复 main2main workflow 缓存路径问题，对齐 checkout 布局；[#11362] 移除该 workflow 缓存中的 `restore-keys`；[#11358] 修复 nightly CI 命令。

### 🚀 Release 动态
- **近期无新版本发布**。

---

## 🐛 Issues

### #11360 — [[CI]:Temporarily remove the unstable test case for Qwen3-235B-A22B PD separation with mooncake layerwise connector; redevelop the test case.](https://github.com/vllm-project/vllm-ascend/issues/11360)
- **作者**: U1stRsouland  **时间**: 2026-07-02 11:28 UTC
- **摘要**: ### Anything you want to discuss about vllm on ascend.  Temporarily remove the unstable test case for Qwen3-235B-A22B PD separation with mooncake layerwise connector; redevelop the test case.

### #11357 — [[Feature]: 能适配下Unlimited-OCR吗？](https://github.com/vllm-project/vllm-ascend/issues/11357)
- **作者**: liushubao0109  **时间**: 2026-07-02 10:47 UTC
- **标签**: feature request
- **摘要**: ### 🚀 The feature, motivation and pitch  Unlimited-OCR的精度和效率都比Deepseek OCR的更好，能适配一下Unlimited-OCR  ### Alternatives  _No response_  ### Additional context  _No response_

### #11356 — [[RFC] PD-separated Mamba state independent channel routing](https://github.com/vllm-project/vllm-ascend/issues/11356)
- **作者**: Xuan-yi-yan  **时间**: 2026-07-02 10:35 UTC
- **标签**: RFC
- **摘要**: ### Motivation.  PD separation currently drops prefix cache hit rate to zero for hybrid models (DeepSeek V4 with Mamba layers). Global LCM alignment forces Mamba states through block-based alignment where they structurally cannot fit.  ### Proposed Change.  1. Add mamba_state_payload field to ReqMet…

## 🔀 Pull Requests

### #11362 — [[CI] Remove restore-keys from main2mian workflow cache](https://github.com/vllm-project/vllm-ascend/pull/11362)
- **作者**: wjunLu  **时间**: 2026-07-02 12:16 UTC
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? Remove restore-keys from main2mian workflow cache ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/a30addc7548a9a8b9b3323a7bc3eb7d7c4895d1c

### #11361 — [[Test][RFork] Fix native EPLB loader config test](https://github.com/vllm-project/vllm-ascend/pull/11361)
- **作者**: yangsonglin13  **时间**: 2026-07-02 11:35 UTC
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it?  This PR fixes the RFork native EPLB loader unit test setup.  The test constructed a lightweight `vllm_config` without `parallel_config`, then set `vllm_config.parallel_config.enable_eplb = True`, which failed during test setup with `AttributeError` before exe…

### #11359 — [[CI] Fix main2main workflow to enable vllm-ascend cache](https://github.com/vllm-project/vllm-ascend/pull/11359)
- **作者**: wjunLu  **时间**: 2026-07-02 11:27 UTC
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it?  Fix csrc cache path and align checkout layout  - vllm and vllm-ascend as sibling dirs under workspace (drop exclude hack) - csrc cache `path:` relative not absolute — tar keeps dir structure so   restore lands .so at the right level (was extracting to workspa…

### #11358 — [[CI] Fix nighly command](https://github.com/vllm-project/vllm-ascend/pull/11358)
- **作者**: zhangxinyuehfad  **时间**: 2026-07-02 10:58 UTC
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/a30addc7548a9a8b9b3323a7bc3eb7d7c4895d1c

### #11355 — [[Feature] Enable 4-stage multistream overlap for W4A4_MXFP4 shared experts](https://github.com/vllm-project/vllm-ascend/pull/11355)
- **作者**: zhenwenqi2024  **时间**: 2026-07-02 10:24 UTC
- **标签**: module:tests, module:ops, module:quantization
- **摘要**: ### What this PR does / why we need it? This PR enables the 4-stage multistream overlap of shared experts for W4A4_MXFP4 quantized MoE models, bringing it to parity with the existing W8A8 path.  Why we need it: When multistream_overlap_shared_expert is enabled on W4A4_MXFP4, profiling shows the shar…

### #11354 — [[Test][Feature] Add comprehensive unit tests for Ascend KV transfer and store connector components](https://github.com/vllm-project/vllm-ascend/pull/11354)
- **作者**: Mango03111  **时间**: 2026-07-02 09:52 UTC
- **标签**: module:tests, merge-conflicts
- **摘要**: Add mock-based unit coverage for AscendStore config, KV transfer, scheduler, worker, connector, and backend paths.    ## What this PR does / why we need it?    This PR adds mock-based unit test coverage for AscendStore-related components, including config data handling, KV transfer receive paths, po…

### #11353 — [[BugFix] Avoid 310P Mamba align postprocess hang for MTP](https://github.com/vllm-project/vllm-ascend/pull/11353)
- **作者**: Alex-stack-hub  **时间**: 2026-07-02 09:49 UTC
- **标签**: module:tests
- **摘要**: ## What  This PR adds a 310P-specific fallback for the Mamba align-mode postprocess path used by MTP/spec decode.  On 310P, the fallback avoids launching the Triton fused `postprocess_mamba_align_gpu` path. It updates accepted-token CPU metadata and mirrors the Mamba state-copy behavior through a CP…

### #11352 — [[Feature][Scheduler] Add ShortRequestFirst scheduling](https://github.com/vllm-project/vllm-ascend/pull/11352)
- **作者**: immengzi  **时间**: 2026-07-02 09:27 UTC
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  This PR adds a default-off `ShortRequestFirst` scheduling policy to reduce prefill head-of-line blocking under mixed prompt-length workloads.  Under the default FCFS-style waiting behavior, a long prefill request at the front of the queue can delay shorter pr…

### #11351 — [Optimize AscendStore key construction](https://github.com/vllm-project/vllm-ascend/pull/11351)
- **作者**: DreamerLeader  **时间**: 2026-07-02 09:25 UTC
- **标签**: module:tests, merge-conflicts
- **摘要**: ## Summary  - Add direct key-string generation for AscendStore token/chunk iteration to avoid creating `PoolKey` objects on hot lookup, load, and store paths. - Cache stable key prefixes per KV cache group/cache role/cache family. - Make grouped block hashes lazy so callers only rehash chunks they a…

### #11350 — [feat: add performance tracing for KV cache scheduling](https://github.com/vllm-project/vllm-ascend/pull/11350)
- **作者**: DreamerLeader  **时间**: 2026-07-02 09:22 UTC
- **摘要**: ## Motivation  KV cache scheduling hot paths (HBM prefix lookup and `find_longest_cache_hit`) lacked timing instrumentation, making it difficult to diagnose prefix caching performance regressions in production.  ## Changes  - **core/recompute_scheduler.py**: Add KVTRACE logging around `get_computed_…
