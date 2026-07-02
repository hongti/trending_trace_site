# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-02 11:29 UTC

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期的动态摘要：

### 📌 Issue 动态
1. **[Feature] 适配 Unlimited-OCR** (#11357)：用户提出适配 Unlimited-OCR 模型的需求，认为其精度和效率优于 Deepseek OCR。
2. **[RFC] PD 分离架构下 Mamba 状态独立通道路由** (#11356)：针对 DeepSeek V4 等带 Mamba 层的混合模型，指出当前 PD 分离会导致前缀缓存命中率降至 0。提议增加 `mamba_state_payload` 等机制实现独立通道路由，解决全局 LCM 对齐下 Mamba 状态无法适配的结构性冲突。

---

### 🚀 PR 动态
**1. 核心性能优化：AscendStore 与 KV Cache**
* 对 **AscendStore** 进行了系列深度性能重构：引入协调器抽象与延迟分组哈希生成（#11348），优化 lookup/put 热路径并增加提前终止机制（#11349），优化键构造以避免热路径创建 `PoolKey` 对象并缓存稳定前缀（#11351）。
* 为 **KV Cache 调度热路径**（HBM 前缀查找等）增加 KVTRACE 性能追踪日志，便于诊断生产环境前缀缓存性能回退（#11350）。
* 补齐测试：为 Ascend KV 传输和 Store 连接器组件增加基于 Mock 的全面单元测试（#11354）。

**2. 新特性与调度策略**
* **短请求优先调度** (#11352)：新增默认关闭的 `ShortRequestFirst` 策略，旨在减少长 prefill 请求造成的队头阻塞，改善混合 prompt 长度负载下的延迟。
* **MoE 多流重叠扩展** (#11355)：为 W4A4_MXFP4 量化 MoE 模型的共享专家启用 4 阶段多流重叠，使其性能与现有的 W8A8 路径对齐。

**3. Bug 修复**
* **修复 310P 设备 Mamba 挂起** (#11353)：修复 310P 设备上 Mamba align 后处理在 MTP/spec decode 场景下挂起的问题，增加了 310P 专属回退机制，绕开 Triton 融合路径，改用 CPU 元数据更新与状态拷贝镜像。

**4. CI 与基础设施**
* 修复 main2main workflow 以启用 vllm-ascend 缓存（#11359）。
* 修复 nightly CI 命令（#11358）。

---

### 📦 Release 动态
* 近期无新版本发布信息。

---

## 🐛 Issues

### #11357 — [[Feature]: 能适配下Unlimited-OCR吗？](https://github.com/vllm-project/vllm-ascend/issues/11357)
- **作者**: liushubao0109  **时间**: 2026-07-02 10:47 UTC
- **标签**: feature request
- **摘要**: ### 🚀 The feature, motivation and pitch  Unlimited-OCR的精度和效率都比Deepseek OCR的更好，能适配一下Unlimited-OCR  ### Alternatives  _No response_  ### Additional context  _No response_

### #11356 — [[RFC] PD-separated Mamba state independent channel routing](https://github.com/vllm-project/vllm-ascend/issues/11356)
- **作者**: Xuan-yi-yan  **时间**: 2026-07-02 10:35 UTC
- **标签**: RFC
- **摘要**: ### Motivation.  PD separation currently drops prefix cache hit rate to zero for hybrid models (DeepSeek V4 with Mamba layers). Global LCM alignment forces Mamba states through block-based alignment where they structurally cannot fit.  ### Proposed Change.  1. Add mamba_state_payload field to ReqMet…

## 🔀 Pull Requests

### #11359 — [[CI] Fix main2main workflow to enable vllm-ascend cache](https://github.com/vllm-project/vllm-ascend/pull/11359)
- **作者**: wjunLu  **时间**: 2026-07-02 11:27 UTC
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/a30addc7548a9a8b9b3323a7bc3eb7d7c4895d1c

### #11358 — [[CI] Fix nighly command](https://github.com/vllm-project/vllm-ascend/pull/11358)
- **作者**: zhangxinyuehfad  **时间**: 2026-07-02 10:58 UTC
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/a30addc7548a9a8b9b3323a7bc3eb7d7c4895d1c

### #11355 — [[Feature] Enable 4-stage multistream overlap for W4A4_MXFP4 shared experts](https://github.com/vllm-project/vllm-ascend/pull/11355)
- **作者**: zhenwenqi2024  **时间**: 2026-07-02 10:24 UTC
- **标签**: module:tests, module:ops, module:quantization
- **摘要**: ### What this PR does / why we need it? This PR enables the 4-stage multistream overlap of shared experts for W4A4_MXFP4 quantized MoE models, bringing it to parity with the existing W8A8 path.  Why we need it: When multistream_overlap_shared_expert is enabled on W4A4_MXFP4, profiling shows the shar…

### #11354 — [[Test][Feature] Add comprehensive unit tests for Ascend KV transfer and store connector components](https://github.com/vllm-project/vllm-ascend/pull/11354)
- **作者**: Mango03111  **时间**: 2026-07-02 09:52 UTC
- **标签**: module:tests
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
- **标签**: module:tests
- **摘要**: ## Summary  - Add direct key-string generation for AscendStore token/chunk iteration to avoid creating `PoolKey` objects on hot lookup, load, and store paths. - Cache stable key prefixes per KV cache group/cache role/cache family. - Make grouped block hashes lazy so callers only rehash chunks they a…

### #11350 — [feat: add performance tracing for KV cache scheduling](https://github.com/vllm-project/vllm-ascend/pull/11350)
- **作者**: DreamerLeader  **时间**: 2026-07-02 09:22 UTC
- **摘要**: ## Motivation  KV cache scheduling hot paths (HBM prefix lookup and `find_longest_cache_hit`) lacked timing instrumentation, making it difficult to diagnose prefix caching performance regressions in production.  ## Changes  - **core/recompute_scheduler.py**: Add KVTRACE logging around `get_computed_…

### #11349 — [perf: optimize AscendStore lookup and put paths](https://github.com/vllm-project/vllm-ascend/pull/11349)
- **作者**: DreamerLeader  **时间**: 2026-07-02 09:20 UTC
- **摘要**: ## Motivation  AscendStore's lookup and put paths had several performance bottlenecks: lookup iterated groups sequentially without early termination, put created intermediate `PoolKey` objects for every chunk, and `wait_for_save()` blocked on `queue.join()` even when only specific requests needed to…

### #11348 — [perf: add AscendStore coordinator and lazy grouped hash generation](https://github.com/vllm-project/vllm-ascend/pull/11348)
- **作者**: DreamerLeader  **时间**: 2026-07-02 09:19 UTC
- **摘要**: ## Motivation  AscendStore's hash generation was eager — all grouped block hashes were materialized upfront even when only a subset was needed for lookup or put. Additionally, there was no coordinator abstraction to manage store/lookup masks across KV cache groups, and block hash conversion for exte…
