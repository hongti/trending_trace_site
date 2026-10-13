# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-13 15:02 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🚀 Release
- **v0.30.0rc1** (2026-10-12)
  - **版本亮点**：这是一个以“模型支持”为核心的开发候选版本，基于 v0.29 版本线构建，并与上游 vLLM v0.3 版本保持对齐。

### 🐛 Issue
- **#18533 [性能优化]**：跟踪 DSpark draft KV 长度同步的优化与验证，作为之前相关报告的后续实施方案。
- **#18532 [Bug]**：指出 Padded FIA KV page views 会导致全缓存临时拷贝，并在图预热 阶段引发 OOM（内存溢出）问题。

### 🔧 Pull Request (PR)
**新特性与模型支持**
- **#18534**：为 GLM-Next (GLM-5.3-Flash) 的 KPool 索引器引入 **int8 量化缓存**，修复 `indexer_kv_dtype` 被忽略的问题，并兼容 `dsa_cp`。
- **#18531**：为 gdn 移植并支持可配置的 FLA prefill。
- **#18530**：新增 DeepSeek-V4.1-Flash 的 A2 1P1D 分离式部署指南，以及基于 Mooncake KV Cache Pool 的部署指南。

**Bug 修复**
- **#18536**：修复 mrv1 中 `draft_model` spec decode 的初始化报错 Bug。
- **#18541**：将 MiniMax-M3 MXFP8 A5 nightly 配置更新至 main 分支，并将 sparse attention 保留在 piecewise graphs 之外以避免错误。

**性能优化**
- **#18537**：将 PCP decode O 投影的激活交换机制（AllToAll, ReduceScatter 等）回port到 `v0.30.0` 发布分支，优化 sharded DSV4/V4.1 的解码性能。

**文档与 CI 维护**
- **#18538 / #18535**：自动翻译了 58 个文档文件；并刷新了 Ascend 的支持矩阵、功能指南和模型权重条目（如更新 GLM-5.2 的 max-model-len）。
- **#18539**：优化 CI 机制，要求未打标签的 PR 必须经过 `test_selector.py` 检查，否则将无法通过 ci-gate。

---

## 🐛 Issues

### #18533 — [[Performance]: DSpark draft KV length synchronization — optimization and validation tracking](https://github.com/vllm-project/vllm-ascend/issues/18533)
- **作者**: Liuchenbing-2026  **时间**: 2026-10-13 14:25 CST
- **摘要**: ### Proposal to improve performance  This is a focused implementation/validation follow-up to the original report #16271, linked to PR #16465. It does not replace or contradict that report.  During eligible DSpark draft attention metadata construction, FIA requires host `actual_seq_lengths_kv`. Call…

### #18532 — [[Bug]: Padded FIA KV page views cause full-cache temporary copies and graph-warmup OOM](https://github.com/vllm-project/vllm-ascend/issues/18532)
- **作者**: Liuchenbing-2026  **时间**: 2026-10-13 14:25 CST
- **摘要**: ### Your current environment  Recorded test environment (not a new `collect_env.py` capture):  - ARM64 Ascend 910B4-1; CANN 9.1.0; torch 2.10.0+cpu; torch-npu 2.10.0.post4. - vLLM v0.30.0, `ced6857afa0ea7b2e3f0846a62e1394e90f15607`. - Unmodified Ascend probe base: `c2fa1a926b0aa416d8d088b4a976aedfa7…

## 🔀 Pull Requests

### #18541 — [[BugFix][Model] Add MiniMax-M3 MXFP8 A5 nightly and keep sparse attention outside piecewise graphs](https://github.com/vllm-project/vllm-ascend/pull/18541)
- **作者**: FORFuture37  **时间**: 2026-10-13 15:00 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Replays two local changes onto current main, instead of the diverged `A5_mxfp8_nightly` branch:  - The current MiniMax-M3-MXFP8-A5 nightly config from #18006: TP=4, DP=2, MXFP8 weights, fp8 KV cache, Eagle3, and GPQA at temperature 0.6. TextVQA stays disabled…

### #18540 — [1](https://github.com/vllm-project/vllm-ascend/pull/18540)
- **作者**: qigangc  **时间**: 2026-10-13 14:53 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/8cc9aa5ad350f6f80cf3bebb298d410803ff75a9

### #18539 — [[CI]: require a label for test_selector.py without the full suite](https://github.com/vllm-project/vllm-ascend/pull/18539)
- **作者**: shiqiangA  **时间**: 2026-10-13 14:52 CST
- **标签**: ci/build
- **摘要**: Put test_selector.py on the src path filter so an unlabeled PR fails ci-gate, while ready-precise stays on the precise path. Add a startup probe print so this diff can be checked in CI.  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this p…

### #18538 — [[Doc] Translated Doc files 2026-10-13](https://github.com/vllm-project/vllm-ascend/pull/18538)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-13 14:51 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **58** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/versioning_policy.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Docume…

### #18537 — [[Cherry-pick][releases/v0.30.0][Performance][DSA] Use activation exchange for PCP decode O projection (from #18506)](https://github.com/vllm-project/vllm-ascend/pull/18537)
- **作者**: li1how  **时间**: 2026-10-13 14:50 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Backport #18506 to `releases/v0.30.0`.  Use activation AllToAll, resident PCP O-group projection, and ReduceScatter for sharded DSV4/V4.1 decode.  ### Does this PR introduce _any_ user-facing change?  No API or configuration changes.  ### How was this patch t…

### #18536 — [[Bugfix] fix draft_model spec decode init bug for mrv1](https://github.com/vllm-project/vllm-ascend/pull/18536)
- **作者**: HF-001  **时间**: 2026-10-13 14:39 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? fix draft_model spec decode bug for mrv1 ,error info: ``` File "/vllm-workspace/vllm-ascend/vllm_ascend/worker/model_runner_v1.py", line 4373, in _dummy_run (EngineCore pid=56060)     self.drafter.dummy_run( (EngineCore pid=56060)   File "/usr/local/python3.12…

### #18535 — [[Doc][Misc] Refresh support matrix, feature guides and model weight entries](https://github.com/vllm-project/vllm-ascend/pull/18535)
- **作者**: EliasKaslan  **时间**: 2026-10-13 14:36 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Refresh the Ascend documentation for recently verified capabilities:  - `support_matrix/supported_models.md`: update the `max-model-len` of GLM-5.2 from `200k` to `1M`. - `support_matrix/supported_features.md`: add a `DSpark` row (Experimental, verified on De…

### #18534 — [[Feat][Model] GLM-Next KPool indexer LI C8: int8 quantized pooled cache, dsa_cp compatible](https://github.com/vllm-project/vllm-ascend/pull/18534)
- **作者**: Toneymiller  **时间**: 2026-10-13 14:33 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  GLM-5.3-Flash (glm5_next arch, KDA+SFA hybrid) uses the **KPool** pooled indexer cache. Today `attention_config.indexer_kv_dtype` is silently ignored for this model family: kpool models are not SFA-sparse, so `enable_sparse_li_c8` is never derived — and even …

### #18531 — [feat(gdn): port configurable FLA prefill ](https://github.com/vllm-project/vllm-ascend/pull/18531)
- **作者**: huoyibingli  **时间**: 2026-10-13 14:20 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/8cc9aa5ad350f6f80cf3bebb298d410803ff75a9

### #18530 — [[Doc][Feature] Add A2 PD deployment for DeepSeek-V4.1-Flash](https://github.com/vllm-project/vllm-ascend/pull/18530)
- **作者**: ylyhyqsl  **时间**: 2026-10-13 14:18 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  This PR adds the A2 1P1D disaggregated deployment guide and the A2 1P1D deployment with Mooncake KV Cache Pool guide for the DeepSeek-V4.1-Flash model.  ### Does this PR introduce _any_ user-facing change?  Yes, it updates the documentation to include deploym…

## 🚀 Releases

### [v0.30.0rc1](https://github.com/vllm-project/vllm-ascend/releases/tag/v0.30.0rc1)
- **作者**: ZT-AIA  **时间**: 2026-10-12 21:46 CST
- **摘要**: ## v0.30.0rc1 - 2026.10.12  This is vLLM Ascend v0.30.0rc1, a model-focused release built on the v0.29 release line and aligned with upstream vLLM v0.30.0 via the main2main pipeline. This release focuses on four flagship models: **DeepSeek V4.1**, **GLM-5.3**, **GLM-5.3-Flash**, and **Qwen3.8-27B**,…
