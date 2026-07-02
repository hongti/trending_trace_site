# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-02 12:56 UTC

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📌 Issue 动态
- **CI 稳定性优化 (#11360)**：提议暂时移除基于 mooncake layerwise connector 的 Qwen3-235B-A22B PD 分离不稳定测试用例，并计划重新开发该测试。
- **新特性需求 (#11357)**：用户请求适配 Unlimited-OCR，认为其在精度和效率上优于现有的 Deepseek OCR。
- **架构设计 RFC (#11356)**：提出 PD 分离场景下 Mamba 状态的独立通道路由方案。旨在解决 DeepSeek V4 等混合模型在 PD 分离时，因全局 LCM 对齐导致 Mamba 状态结构不兼容、前缀缓存命中率降为零的问题。

---

### 📌 PR 动态
**🛠️ Bugfix（缺陷修复）**
- **修复 GLM-5.1 IndexCache 权重加载 (#11363)**：纠正了现有补丁的逻辑误判，之前将 `skip_topk=True` 视为 checkpoint 缺少 Indexer 的标志（仅适用 GLM-5.2），导致 GLM-5.1 启用 IndexCache 时权重丢失。
- **避免 310P 芯片 Mamba align 后处理挂起 (#11353)**：针对 310P 芯片在 MTP/推测解码场景下的挂起问题，引入了专属回退机制，绕过 Triton 融合路径，改用 CPU 元数据更新及 CP 行为镜像。

**✨ Feature & Optimization（新特性与优化）**
- **W4A4_MXFP4 共享专家 4 阶段多流重叠 (#11355)**：为 W4A4_MXFP4 量化的 MoE 模型启用 4-stage multistream overlap，使其与 W8A8 路径性能对齐，减少共享专家延迟。
- **新增 ShortRequestFirst 调度策略 (#11352)**：引入默认关闭的“短请求优先”调度策略，以缓解混合长度负载下，长 prefill 请求造成的队头阻塞（HOL blocking）问题。
- **优化 AscendStore 键构造 (#11351)**：避免在热查找/加载/存储路径上创建 `PoolKey` 对象，缓存稳定前缀，并实现分组块哈希的惰性计算，提升性能。

**🧪 CI & Test（测试与流水线）**
- **完善 AscendStore 单元测试 (#11354)**：为 AscendStore 相关组件（config、KV transfer、scheduler、worker 等）增加了基于 mock 的全面单元测试覆盖。
- **修复 EPLB loader 测试配置 (#11361)**：修复了 RFork 原生 EPLB loader 测试中因缺少 `parallel_config` 导致 `AttributeError` 的设置错误。
- **多项 CI 流水线修复 (#11359, #11362, #11358)**：包括移除 main2main workflow 的 `restore-keys`；修复 csrc 缓存路径与 checkout 目录布局（改为同级目录结构）以确保缓存生效；以及修复 nightly command 问题。

---

### 📌 Release 动态
**本期无新版本发布。**

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

### #11363 — [[Bugfix] Fix GLM-5.1 IndexCache weight loading](https://github.com/vllm-project/vllm-ascend/pull/11363)
- **作者**: ZYang6263  **时间**: 2026-07-02 12:33 UTC
- **标签**: module:tests
- **摘要**: ## What this PR does / why we need it?  The existing patch treated `skip_topk=True` as proof that the checkpoint omitted the layer's `Indexer`. This is valid for GLM-5.2 shared-indexer layers, but not for GLM-5.1 when IndexCache is enabled through runtime HF overrides. GLM-5.1 checkpoints still cont…

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
