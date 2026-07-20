# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-20 10:53 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的简洁摘要：

### 🚀 Release (版本发布)
*   **v0.23.0rc1 发布**：发布了 v0.23.0 的首个候选版本（RC1），标志着 v0.23.0 版本线进入测试阶段。（注：同时有 PR 向 `releases/v0.24.0rc` 分支合入修复，说明更高版本的开发也在并行推进）。

### 🛠️ PR (重要变更与合并)
*   **新特性：支持 Mistral Large 3 (675B) NVFP4 量化**
    *   新增了 Ascend 模型适配器、NVFP4 量化框架、回归测试及部署文档 (#12378, #12377)。由于受硬件容量限制无法实测 675B 模型，补充了 Qwen-7B 双卡 TP=2 作为参考验证。
*   **BugFix：修复 TP 精度问题**
    *   Cherry-pick 修复了“因 reorder 时机错误导致 TP 不等进而引发的精度问题”至 `releases/v0.24.0rc` 分支 (#12382)。
*   **文档更新：MemCache 分离部署**
    *   修正了 MemCache 配置说明，并新增了 MemCache 与 vLLM 分离部署模式的独立指南 (#12384)。
*   **主干同步：适配 vLLM 最新代码**
    *   连续三次升级适配 vLLM 主干（commits `54503ece`, `04d553f3`, `9427c453`），主要修改了 Mooncake connector 和 Hunyuan VL processor 的兼容性 (#12372, #12379, #12380)。
*   **CI/基础设施优化**
    *   将 `main2main` 自动化流程从全量同步重构为增量同步 (#12374)；并增加了兜底机制，当目标 commit 版本过旧时自动回退至全新模式 (#12376)。
    *   自动化更新测试估算时间 (#12373)。

### 🐛 Issue (问题与反馈)
*   **严重 Bug：PD 分离部署挂起**
    *   在 vLLM-Ascend 0.22.1.rc1 环境下运行 GLM-5.1 模型时，PD 分离拓扑出现严重挂起问题：ZMQ 通信报错，且 KV Cache 永久占用达 99%，导致新请求完全被阻塞 (#12383)。
*   **CI 异常：main2main 流程中断**
    *   `main2main` 自动化流程两次未完成所有计划步骤即停止，需要人工审核介入 (#12381, #12375)。

---

## 🐛 Issues

### #12383 — [[Bug]: PD Disaggregation Hang: ZMQ Communication Errors + Permanent 99% KV Cache Occupation Block New Requests](https://github.com/vllm-project/vllm-ascend/issues/12383)
- **作者**: Bowen-Leee  **时间**: 2026-07-20 10:41 CST
- **标签**: bug, core-features, pd-disaggregation
- **摘要**: ### Your current environment  Environment Framework: vLLM-Ascend 0.22.1.rc1 Model: GLM-5.1 Hardware: 4 Ascend A3 Nodes Disagg Topology: PD Separation, 1 Prefill Group  + 1 Decode Group  Test Params Concurrency: 32 Request Spec: 100k input tokens, 30k output tokens    ### 🐛 Describe the bug  When run…

### #12381 — [[main2main] main2main manual review required (9427c453)](https://github.com/vllm-project/vllm-ascend/issues/12381)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-20 05:33 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/12380 - Commit range: `04d553f390fd37e09ab111936ef1592881299957`...`9427c453863f3ab9e720748f04b9d6dd404ef602` - Status: `failed`  ## Final Summary  …

### #12375 — [[main2main] main2main manual review required ()](https://github.com/vllm-project/vllm-ascend/issues/12375)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-19 22:17 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR:  - Commit range: ``...`` - Status: `missing`  ## Final Summary  Status: completed Steps: 0/0

## 🔀 Pull Requests

### #12384 — [[v0.23.0][Doc][Misc] Correct MemCache configuration and add a separated deployment guide](https://github.com/vllm-project/vllm-ascend/pull/12384)
- **作者**: Oranbean258  **时间**: 2026-07-20 10:42 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? Corrected the MemCache configuration descriptions. Added a new section describing the separated MemCache–vLLM deployment mode. ### Does this PR introduce _any_ user-facing change? No. ### How was this patch tested? Documentation-only change, no testing require…

### #12382 — [[Cherry-pick][releases/v0.24.0rc][BugFix]Fix precision issues caused by incorrect reordering timing leading to TP inequality (from #12359)](https://github.com/vllm-project/vllm-ascend/pull/12382)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-20 10:30 CST
- **摘要**: Cherry-pick of PR #12359 onto `releases/v0.24.0rc`.  Original PR: #12359 Original author: @wangxiaoteng888  --- ### What this PR does / why we need it? Fix precision issues caused by incorrect reordering timing leading to TP inequality  ### Does this PR introduce _any_ user-facing change? No  ### Ho…

### #12380 — [[Misc]feat: adapt to vLLM main (9427c453)](https://github.com/vllm-project/vllm-ascend/pull/12380)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-20 05:33 CST
- **摘要**: ### What this PR does / why we need it?  Upgrade vLLM commit to `9427c453`  1. Adapt `vllm_ascend/patch/hunyuan_vl_processor_compat.py` due to [00673115](https://github.com/vllm-project/vllm/commit/00673115)    - (1) Upstream removed HunYuanVL processor registry entries, causing early return in comp…

### #12379 — [[Misc]feat: adapt to vLLM main (04d553f3)](https://github.com/vllm-project/vllm-ascend/pull/12379)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-20 02:14 CST
- **摘要**: ### What this PR does / why we need it?  Upgrade vLLM commit to `04d553f3`  1. Adapt `vllm_ascend/distributed/kv_transfer/kv_p2p/mooncake_connector.py`, `vllm_ascend/distributed/kv_transfer/kv_p2p/mooncake_layerwise_connector.py`, `vllm_ascend/patch/platform/patch_speculative_config.py`, `vllm_ascen…

### #12378 — [[Feature] Support Mistral Large 3 NVFP4](https://github.com/vllm-project/vllm-ascend/pull/12378)
- **作者**: chunlei-2026  **时间**: 2026-07-20 01:04 CST
- **标签**: documentation, module:tests, module:core, module:quantization
- **摘要**: Add the Ascend model adapter, NVFP4 quantization scaffold, regression tests, model configurations, deployment documentation, and AI-assisted workflow guidance as one DCO-compliant change. Fixes #7338   ### What this PR does / why we need it? Implement model adapter and NVFP4 quantization support for…

### #12377 — [[Feature] Support mistral large3 675b nvfp4](https://github.com/vllm-project/vllm-ascend/pull/12377)
- **作者**: chunlei-2026  **时间**: 2026-07-19 23:25 CST
- **标签**: documentation, module:tests, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it? 1. 新增Mistral-Large-3-675B-NVFP4的e2e测试yaml配置，放在tests/e2e/models/configs，设计多类测试场景；注明该模型受磁盘、HBM硬件容量限制无法实测，补充Qwen-7B双卡TP=2适配成功作为参考验证案例。 2. 新增对应部署教程文档至docs/source/tutorials/models，覆盖张量并行部署、NPU故障排查、HF授权、上下文适配、资源限制、接口验收等内容。 3. 仓库根目录新增SKILL.md，记录AI辅助开发流程、排障方案、可复用模板、脱敏…

### #12376 — [[CI] Main2main fall back to fresh mode when target commit is behind baseline](https://github.com/vllm-project/vllm-ascend/pull/12376)
- **作者**: wjunLu  **时间**: 2026-07-19 22:53 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? A manual workflow_dispatch specifies a TARGET_COMMIT that is older than the baseline's already-verified VLLM commit, the baseline's adapted code (which targets a newer VLLM) would produce incorrect version guards for the older target.   Detect this with merge-…

### #12374 — [[CI] Refactor main2main from full to incremental synchronization](https://github.com/vllm-project/vllm-ascend/pull/12374)
- **作者**: wjunLu  **时间**: 2026-07-19 22:01 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? Refactor main2main from full to incremental synchronization ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.24.0 - vLLM main: https://github.com/vllm-project/vllm/commit/85c09e9885e346ea1612da30ebff5a75f…

### #12373 — [[CI] Auto-update estimated test times in test_config.yaml](https://github.com/vllm-project/vllm-ascend/pull/12373)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-19 20:34 CST
- **摘要**: ## Summary  This PR was auto-generated by the **Update estimated test times** [workflow](https://github.com/vllm-project/vllm-ascend/actions/runs/29682160424).  It updates the `estimated_times` values in `.github/workflows/scripts/test_config.yaml` based on actual elapsed times collected from CI wor…

### #12372 — [[Misc]feat: adapt to vLLM main (54503ece)](https://github.com/vllm-project/vllm-ascend/pull/12372)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-19 20:20 CST
- **标签**: module:quantization
- **摘要**: ### What this PR does / why we need it?  Upgrade vLLM commit to `54503ece`  1. Adapt `vllm_ascend/distributed/kv_transfer/kv_p2p/mooncake_connector.py`, `vllm_ascend/distributed/kv_transfer/kv_p2p/mooncake_layerwise_connector.py`, `vllm_ascend/patch/platform/patch_speculative_config.py`, `vllm_ascen…

## 🚀 Releases

### [v0.23.0rc1](https://github.com/vllm-project/vllm-ascend/releases/tag/v0.23.0rc1)
- **作者**: yiz-liu  **时间**: 2026-07-19 21:55 CST
- **摘要**: ## v0.23.0rc1 - 2026.07.20  We're excited to announce v0.23.0rc1, the first release candidate for the vLLM Ascend v0.23.0 release line. This release aligns the plugin with upstream vLLM v0.23.0 and expands model, context-parallel, KV-cache offload, and Ascend 950 support. Please follow the [official…
