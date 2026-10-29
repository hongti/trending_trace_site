# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-29 14:18 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文简洁摘要：

### 📋 Issue 动态
*   **性能优化提案 (#20198)**：提议将上游 vLLM 的“最后 KV-source 层裁剪”优化适配到 vllm-ascend 中的 DeepSeek V4.1 模型，旨在降低 Decoder Replay 阶段的计算开销。
*   **性能 Bug 报告 (#20197)**：在 v0.30 版本中，Kimi K3 采用 PD 分离架构（Decode 节点关闭 DCP，256k 上下文配置）时，并发从 16 增至 32 会导致 TTFT（首字延迟）劣化十几倍。相比之下，1M 配置开启 DCP 时表现正常。

### 🔧 PR 动态
**新特性与模型支持**
*   **Draft 模型权重更新支持 (#20205)**：共享的 Ascend worker 现可通过上游 API 支持 Draft 模型（不仅是 Target 模型）基于 HCCL 的权重更新。
*   **Qwen3.8-Flash-Next 注意力回退机制 (#20201)**：为该模型增加了可切换的 QSA 全注意力 FIA 回退路径，优化短序列场景的处理。

**性能优化**
*   **A3 V2 元数据重叠机制 (#20206)**：在 A3 架构上，为 DeepSeek V4.1 Model Runner V2 新增了 opt-in 异步元数据准备重叠功能，提升执行效率。
*   **Cache Slot Mapping 内核加速 (#20194)**：优化了 `build_packed_cache_slot_mapping` 的 Triton 内核，解决其在 A2/A3 上的性能瓶颈，替代了原先低效的多次小算子启动。

**Bug 修复**
*   **修复 KV Cache 内存计算问题 (#20203)**：修复了由上游变更引起的 Ascend 多分配布局下 KV Cache 内存计算错误，该错误此前会导致 DeepSeek V4.1 初始化失败。

**CI 与测试维护**
*   **环境与依赖升级**：将 CI 的 CANN 镜像从华为云 SWR 迁移至公共 `quay.io/ascend`，并全面升级至 CANN 9.2.1 (#20204, #20202)；更新 triton-ascend 依赖至 3.6.0.post2 (#20200)。
*   **测试用例同步与完善**：新增 v0.29.0 GPU 测试用例并修复 npu_fla 错误 (#20199)；将 Kimi K3 MRV2 测试覆盖和 DeepSeek 每周定时调度任务同步至 main 分支 (#20195)。

### 🚀 Release 动态
*   本周期内无新版发布。

---

## 🐛 Issues

### #20198 — [[RFC][Performance] DeepSeek V4.1: Include the Last KV-source Layer into Decoder Replay](https://github.com/vllm-project/vllm-ascend/issues/20198)
- **作者**: lio1226  **时间**: 2026-10-29 11:28 CST
- **标签**: RFC
- **摘要**: ### Motivation.  We propose adapting the last KV-source layer trimming optimization from upstream [vLLM PR #59894](https://github.com/vllm-project/vllm/pull/59894) to vllm-ascend, reducing computation during long prefills.  Currently, decoder bounded replay in vllm-ascend starts at cut + 1, where cu…

### #20197 — [[Bug]: 0.30 kimi-k3 pd分离 decode节点关闭dcp 上下文256k配置，测评128k/1k 并发增大后，TTFT会劣化十几倍。](https://github.com/vllm-project/vllm-ascend/issues/20197)
- **作者**: Sfeching  **时间**: 2026-10-29 11:26 CST
- **标签**: bug
- **摘要**: ### Your current environment  A3 配置参考0.30 branch 最新服务配置  PD分离  ### 🐛 Describe the bug  32并发的TTFT 相比于16并发  劣化十几倍。而1M配置decode 开dcp 测评 却很正常

## 🔀 Pull Requests

### #20206 — [[Performance][v0.30.0] Add opt-in A3 V2 target metadata overlap](https://github.com/vllm-project/vllm-ascend/pull/20206)
- **作者**: Abstrey  **时间**: 2026-10-29 13:50 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  On A3, the DeepSeek V4.1 Model Runner V2 target still prepares SAS/QLI device metadata before model execution. The existing V1 asynchronous executor and V2 A5 packed-cache path do not enable graph-external producers for this target. This adds an **opt-in, dis…

### #20205 — [[Feature] Support draft model weight updates over HCCL](https://github.com/vllm-project/vllm-ascend/pull/20205)
- **作者**: wang2001-hust  **时间**: 2026-10-29 12:47 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  The shared Ascend worker currently starts weight-update transactions against the target model only. Calling the upstream `start_draft_weight_update()` API therefore cannot select the speculative draft as the HCCL receiver.  This change adds draft-session sele…

### #20204 — [[releases/v0.30.0][CI]: migrate CANN images to quay.io and bump to 9.2.1](https://github.com/vllm-project/vllm-ascend/pull/20204)
- **作者**: zhangxinyuehfad  **时间**: 2026-10-29 12:42 CST
- **标签**: ci/build, ready-all
- **摘要**: ### What this PR does / why we need it?  Migrates the CI container images from the HuaweiCloud SWR mirrors to the public `quay.io/ascend` registry, and bumps the CANN image tag from `9.2.0_20261025172600` to `9.2.1`.  Changes:  - `.github/workflows/_selected_tests.yaml`: run test jobs on   `quay.io/…

### #20203 — [[Bugfix] Fix KV cache memory accounting for Ascend multi-allocation layouts](https://github.com/vllm-project/vllm-ascend/pull/20203)
- **作者**: wxsIcey  **时间**: 2026-10-29 12:41 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it? This PR fixes a KV cache initialization failure when serving DeepSeek V4.1 on Ascend, caused by https://github.com/vllm-project/vllm/pull/53558. When https://github.com/vllm-project/vllm-ascend/pull/18359 merged, this patch can be removed.  The newer builder-b…

### #20202 — [[CI] update runner_label.json cann=9.2.1](https://github.com/vllm-project/vllm-ascend/pull/20202)
- **作者**: xqchen7  **时间**: 2026-10-29 12:29 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? update runner_label.json cann=9.2.1  ### Does this PR introduce _any_ user-facing change? NA  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditio…

### #20201 — [[Feature][Model] Add QSA full-attention FIA fallback for Qwen3.8-Flash-Next (issue #20117)](https://github.com/vllm-project/vllm-ascend/pull/20201)
- **作者**: wxhhuawei  **时间**: 2026-10-29 12:28 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization, merge-conflicts
- **摘要**: ## What does this PR do?  Adds a switchable **full-attention fallback path** for the Qwen3.8-Flash-Next QSA attention, ported from the vllm-ascend 0.26 patch attached to issue #20117 onto the short-sequence FIA branch (`qsa-shortseq-fia`, PR #19444).  With `VLLM_ASCEND_QSA_FULL_ATTENTION` (default o…

### #20200 — [[CI] update triton-ascend requirement to 3.6.0.post2](https://github.com/vllm-project/vllm-ascend/pull/20200)
- **作者**: jiaqi-lee  **时间**: 2026-10-29 11:52 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it?  update triton-ascend requirement to 3.6.0.post2  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundar…

### #20199 — [[Test] [Upstream] Add v0.29.0 test case](https://github.com/vllm-project/vllm-ascend/pull/20199)
- **作者**: meiling6226  **时间**: 2026-10-29 11:39 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? This PR adds GPU test case migration for v0.29.0 and fixes the npu_fla error.  ### Does this PR introduce _any_ user-facing change? NO  ### How was this patch tested? How was this patch tested? By running the test  vllm version:v0.30.0  https://github.com/vllm…

### #20195 — [[Test] Port Kimi K3 MRV2 coverage and DeepSeek weekly scheduling to main](https://github.com/vllm-project/vllm-ascend/pull/20195)
- **作者**: MaybeChz  **时间**: 2026-10-29 11:15 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Port #19774 from `releases/v0.30.0` to `main` by cherry-picking merged commit `c18dc97856708359e3e6546a22a319c5e03c540d` with `-x -s`. The cherry-pick applies without conflicts and preserves source attribution.  - Add Kimi K3 MRV2 four-card E2E coverage using…

### #20194 — [[Ops][Perf] Speed up the packed-cache slot mapping kernel and enable it on A2/A3](https://github.com/vllm-project/vllm-ascend/pull/20194)
- **作者**: Wfd567  **时间**: 2026-10-29 11:14 CST
- **标签**: module:ops
- **摘要**: ### What this PR does / why we need it?  `build_packed_cache_slot_mapping` (#17927) replaced the eager slot-mapping chain (~20 small aclnn launches) with a single Triton launch, but on A2/A3 it has two problems:  1. **It is slower than the chain it replaces.** At decode batch sizes (`num_tokens` in …
