# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-02 14:09 UTC

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📌 Issue 动态
1. **架构讨论 (RFC)**：提出 **PD分离下 Mamba 状态独立通道路由**方案 (#11356)。旨在解决当前 PD 分离导致混合模型（如 DeepSeek V4+Mamba层）前缀缓存命中率降为 0 的问题，避免全局 LCM 对齐导致 Mamba 状态结构不匹配。
2. **新特性请求**：社区请求适配 **Unlimited-OCR** (#11357)，理由是其精度和效率优于 Deepseek OCR。
3. **CI 稳定性**：暂时移除 **Qwen3-235B-A22B PD分离+Mooncake逐层连接器**的不稳定测试用例，计划重新开发该测试 (#11360)。

---

### 🔧 PR 动态 (重要变更与修复)

**🚀 新特性与性能优化**
- **调度策略优化**：新增默认关闭的 **ShortRequestFirst 调度策略** (#11352)，旨在减少长 Prefill 请求造成的队头阻塞，提升短请求的响应速度。
- **MoE 计算加速**：为 W4A4_MXFP4 量化 MoE 模型的共享专家启用 **4阶段多流重叠** (#11355)，性能表现与现有的 W8A8 路径对齐。
- **Embedding 模型支持**：增加 `e5-mistral-7b-instruct` Embedding 模型的端到端测试及使用教程，覆盖在线 `/v1/embeddings` 服务与离线 `LLM.embed()` 接口 (#11364)。

**🐛 Bug 修复**
- **GLM-5.1 权重加载修复**：修复 IndexCache 权重加载逻辑 (#11363)。原代码错误地将 GLM-5.2 共享索引层的逻辑（`skip_topk=True`）应用于启用了 IndexCache 的 GLM-5.1 层，导致权重漏加载。
- **310P 设备 Mamba 挂起修复**：修复 310P 设备在 MTP/推测解码场景下，Mamba align 后处理路径导致的挂起问题 (#11353)，为 310P 引入了特定的 CPU 回退机制。

**🧪 CI 与测试改进**
- **KV传输组件测试**：为 AscendStore 相关组件（配置、KV transfer、调度器等）新增基于 Mock 的全面单元测试 (#11354)。
- **EPLB 测试修复**：修复 RFork 原生 EPLB loader 单元测试中因缺少 `parallel_config` 导致的 `AttributeError` (#11361)。
- **Workflow 与缓存修复**：修复 main2main 工作流的 csrc 缓存路径与目录布局 (#11359)；移除 workflow cache 的 `restore-keys` (#11362)；修复 nightly 测试命令 (#11358)。

---

### 📦 Release 动态
- 本次追踪周期内**无新版本发布**。

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

### #11364 — [[Feature] Add e5-mistral embedding model test](https://github.com/vllm-project/vllm-ascend/pull/11364)
- **作者**: EheinWang  **时间**: 2026-07-02 13:08 UTC
- **标签**: documentation, module:tests
- **摘要**: ## Summary  - Add config-driven e2e coverage for `intfloat/e5-mistral-7b-instruct` embedding inference on vLLM Ascend. - Add a model tutorial covering online `/v1/embeddings` serving and offline `LLM.embed()` usage. - Add the issue skill note for AI-assisted model adaptation experience.  ## Validati…

### #11363 — [[Bugfix] Fix GLM-5.1 IndexCache weight loading](https://github.com/vllm-project/vllm-ascend/pull/11363)
- **作者**: ZYang6263  **时间**: 2026-07-02 12:33 UTC
- **标签**: module:tests, ready
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
