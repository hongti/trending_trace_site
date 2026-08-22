# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-22 10:01 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的简洁摘要：

### 🚨 Issue
* **CI 自动化流程中断需人工审查**：#14751 指出 `main2main` 自动化流程在完成所有计划步骤前停止，需人工介入审查（关联 Draft PR #14750）。

### 🔧 Pull Request
**1. 新特性与模型优化**
* **Qwen3.5 GDN decode 流水线融合**：#14749 在 Ascend 平台上融合了 Qwen3.5-9B GDN 的 decode 流水线（整合因果卷积、Q/K 归一化、门控等），减少中间张量物化和独立算子启动，提升性能。
* **dspark Markov head 复制**：#14748 在 `finegrained_tp_config` 中新增 `markov_tensor_parallel_size` 参数，实现 Markov head 的复制路由（0=禁用，1=复制）。
* **适配 dcp 和 dflash**：#14744 提出了针对 dcp 和 dflash 算子的适配方案。

**2. Bug 修复**
* **修复 MegaMoe 挂起问题**：#14747 修复了当主模型与 Draft 模型的 MoE 量化模式不一致（如主模型量化、Draft 模型浮点）时导致的挂起问题。
* **降低 Kimi-K3 显存峰值**：#14745 优化了 Kimi-K3 的 attention residual 计算方式，有效降低了显存峰值占用。

**3. 上游同步**
* **适配 vLLM Main 分支**：#14750 和 #14746 持续跟进上游 vLLM main 分支更新，当前已同步适配至 8 月 12 日及 8 月 21 日的提交（对应 vLLM v0.27.1）。

**4. CI 与测试**
* **优化 CI Rerun 策略**：#14752 将 CI 重跑策略从 per-job 级别简化为 run-level API，解决了可复用工作流矩阵任务重跑时返回 HTTP 403 的问题。
* **新增 Qwen3.8 测试用例**：#14743 为 Qwen3.8-27B-w8a8 模型添加了每周常规的性能与精度测试用例。

**5. 文档**
* **文档中文化**：#14753 自动翻译了 34 个文档文件至中文（zh_CN 目录）。

### 🚀 Release
* 近期无新版本发布动态。

---

## 🐛 Issues

### #14751 — [[main2main] main2main manual review required (7f7a32cf)](https://github.com/vllm-project/vllm-ascend/issues/14751)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-22 03:45 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/14750 - Commit range: `58d3918e3ea0a544ffedadad2ba84559e9c51d8f`...`7f7a32cfec0f1bc5b73c37200b86631523a1ea8f` - Status: `failed`  ## Final Summary  …

## 🔀 Pull Requests

### #14753 — [[Doc] Translated Doc files 2026-08-22](https://github.com/vllm-project/vllm-ascend/pull/14753)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-22 09:55 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **34** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/contributors.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/versioning_policy.po</co…

### #14752 — [[CI] simplify rerun strategy to use run-level API instead of per-job rerun](https://github.com/vllm-project/vllm-ascend/pull/14752)
- **作者**: zhangxinyuehfad  **时间**: 2026-08-22 09:33 CST
- **摘要**: ### What this PR does / why we need it?  Per-job rerun API (POST /jobs/{job_id}/rerun) returns HTTP 403 for reusable workflow caller jobs (matrix jobs using uses:), causing failed/cancelled test jobs to be silently skipped. Only inline jobs like ci-gate were successfully re-run, leading to inconsist…

### #14750 — [[Misc]feat: adapt to vLLM main (7f7a32cf)](https://github.com/vllm-project/vllm-ascend/pull/14750)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-22 03:44 CST
- **标签**: module:ops, ready
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to August 12.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | — | [751f2ccd](https://github.com/vllm-project/vllm/commit/751f2ccdd3b7e…

### #14749 — [[Feat][Model] Fuse the Qwen3.5 GDN decode pipeline on Ascend](https://github.com/vllm-project/vllm-ascend/pull/14749)
- **作者**: nodeeeeee  **时间**: 2026-08-22 00:52 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Qwen3.5-9B GDN decode currently materializes full intermediate tensors and launches separate operators for causal convolution, Q/K normalization, gating, the recurrent update, and gated RMSNorm. For single-token decode, these operators repeatedly read and wri…

### #14748 — [[Feat] dspark: replicate markov head via finegrained_tp_config](https://github.com/vllm-project/vllm-ascend/pull/14748)
- **作者**: Levi-JQ  **时间**: 2026-08-21 22:25 CST
- **标签**: module:ops, module:core
- **摘要**: Add markov_tensor_parallel_size to finegrained_tp_config (0=disabled, 1=replicated). When enabled, markov_w1/markov_w2 are routed to a world_size=1 comm group via prefix match, so each rank holds the full V x r / r x V table. This removes the per-step TP all-reduce (w1) and all-gather (w2) from the …

### #14747 — [[BugFix][v0.26.0rc] Fix MegaMoe hang when main and draft models have inconsistent MoE quantization](https://github.com/vllm-project/vllm-ascend/pull/14747)
- **作者**: kunpengW-code  **时间**: 2026-08-21 22:16 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it? If the quantization modes of the main model and draft model are inconsistent, for example, the main model is a quantized model and the draft model is a floating-point model, the MEGAMOE operator reports an error, indicating that this format is not supported. T…

### #14746 — [[Ci] main2main vllm 0821](https://github.com/vllm-project/vllm-ascend/pull/14746)
- **作者**: zhangxinyuehfad  **时间**: 2026-08-21 20:45 CST
- **标签**: ci/build, module:tests, module:ops, module:core, ready-all
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14745 — [[BugFix][Kimi-K3] Reduce attention residual peak memory](https://github.com/vllm-project/vllm-ascend/pull/14745)
- **作者**: qijiajin  **时间**: 2026-08-21 19:51 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  This is a stacked follow-up on #14454. The only additional commit is `6dd7f644c`, which changes two files.  Kimi K3 currently computes attention-residual scores with:  ```python (normalized_without_gamma * score_weight).sum(-1) ```  This materializes a broadc…

### #14744 — [[Feature] proposed adaptation of dcp and dflash](https://github.com/vllm-project/vllm-ascend/pull/14744)
- **作者**: Laasap  **时间**: 2026-08-21 19:27 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/d02df748bf9efd99022f1a062597dc3cb3808485

### #14743 — [[CI] Add weely performance and accuracy test case for Qwen3.8-27B-w8a8- #14741](https://github.com/vllm-project/vllm-ascend/pull/14743)
- **作者**: weixinAc  **时间**: 2026-08-21 19:07 CST
- **标签**: module:tests, model-dataset-download
- **摘要**: ### What this PR does / why we need it?  [CI] Add weely performance and accuracy test case for Qwen3.8-27B-w8a8- #14741  ### Does this PR introduce _any_ user-facing change? NO ### How was this patch tested? by the running the test  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-projec…
