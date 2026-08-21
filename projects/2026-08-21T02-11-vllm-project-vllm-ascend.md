# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-21 10:11 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期的动态摘要：

### 🐛 Issue 动态
*   **CI 同步异常**：#14692 指出 `main2main` 自动同步流程在完成所有计划步骤前中断，需人工介入审核（关联 Draft PR #14691）。
*   **GLM-5.2 模型 Bug**：#14686 报告了 GLM-5.2-W4A8C8 在**双节点共置**场景下，随机出现异步 MTP（多 Token 预测）进入“解析器先行的周期性原始 Token 循环”问题，该问题独立于 P/D 分离架构。

### 🔀 PR 动态
**1. 新特性与模型支持**
*   **MiniMax-M3 适配**：#14685 新增支持 MiniMax-M3 在 Ascend A3 上的 KV-gather-Q 稀疏 prefill 路径，扩展了原有的 A5 支持。

**2. 重要修复与版本准备（重点）**
*   **v0.26.0rc 核心修复合入**：#14687 作为 v0.26.0 的版本阻塞级 PR，回溯合入了 **19 项**经过审计的高严重度正确性与稳定性修复。
*   **Spec Decode 回退**：#14690 回退了与 speculative decode (#13470) 相关的提交。

**3. 测试与 CI 优化**
*   **Kimi K3 诊断回溯**：#14693 向 v0.26.0 分支回溯了针对 Kimi K3 混合 MLA/Mamba 架构的 KV Cache 别名冲突检测工具。
*   **GLM-5.2 夜间守卫**：#14683 使用 GLM-5.2 W4A8C8 配置（TP16/DCP16）替换了过时的 DeepSeek-V3.2 SFA+DCP 夜间测试。
*   **跳过不稳定测试**：#14688 临时将不稳定的 4 卡图模式精度 e2e 测试从 CI 中移除，避免阻塞 PR 合入。
*   **Mooncake 测试**：#14694 新增了 Main mooncake 相关测试。

**4. 上游同步与文档**
*   **适配 vLLM 主干**：#14691 将 vllm-ascend 适配至 8 月 12 日的 vLLM main 分支最新提交。
*   **文档翻译**：#14684 自动翻译并更新了 v0.23.0 版本的 19 个中文文档文件。

### 🚀 Release 动态
*   近期无正式版本发布，但 **v0.26.0rc** 正在紧锣密鼓筹备中（见 PR #14687 与 #14693），主要聚焦于高优先级的正确性与稳定性修复合入。

---

## 🐛 Issues

### #14692 — [[main2main] main2main manual review required (7f7a32cf)](https://github.com/vllm-project/vllm-ascend/issues/14692)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-21 05:05 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/14691 - Commit range: `58d3918e3ea0a544ffedadad2ba84559e9c51d8f`...`7f7a32cfec0f1bc5b73c37200b86631523a1ea8f` - Status: `failed`  ## Final Summary  …

### #14686 — [[Bug][GLM-5.2-W4A8C8][co-located] Stochastic async MTP enters a parser-before periodic raw-token loop](https://github.com/vllm-project/vllm-ascend/issues/14686)
- **作者**: ZhengDeL  **时间**: 2026-08-20 21:31 CST
- **摘要**: ## Why this is a separate report  This is an independent **two-node co-located** reproduction, not P/D disaggregation. It is related to #14463 and #14656, but adds two isolation results that those issue bodies do not contain:  1. the failure reproduces through the vLLM OpenAI-compatible endpoint dir…

## 🔀 Pull Requests

### #14694 — [[Test] Main mooncake](https://github.com/vllm-project/vllm-ascend/pull/14694)
- **作者**: chen-commits  **时间**: 2026-08-21 09:56 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14693 — [[Test][KV Cache] Detect Kimi K3 hybrid MLA/Mamba alias conflicts](https://github.com/vllm-project/vllm-ascend/pull/14693)
- **作者**: Yaphets24  **时间**: 2026-08-21 09:33 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Backports the validation-only hybrid KV-cache alias diagnostic from https://github.com/Yaphets24/vllm-ascend/pull/1 to the v0.26.0 release branch. It detects unsafe physical aliasing between MLA latent-K and Mamba/KDA state blocks without changing cache alloc…

### #14691 — [[Misc]feat: adapt to vLLM main (7f7a32cf)](https://github.com/vllm-project/vllm-ascend/pull/14691)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-21 05:04 CST
- **标签**: module:ops, ready
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to August 12.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | `vllm_ascend/_310p/model_runner_310p.py`<br>`vllm_ascend/worker/model_ru…

### #14690 — [main from main_pr_m2.5](https://github.com/vllm-project/vllm-ascend/pull/14690)
- **作者**: czydyy  **时间**: 2026-08-21 01:01 CST
- **标签**: module:tests
- **摘要**: …pec decode (#13470)"  This reverts commit 27a94764b5ead50ed3e42ab52a257c2173032750.  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d391…

### #14689 — [main from main_pr_14298](https://github.com/vllm-project/vllm-ascend/pull/14689)
- **作者**: czydyy  **时间**: 2026-08-21 00:14 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14688 — [[CI] temporarily skip graph mode precision e2e test](https://github.com/vllm-project/vllm-ascend/pull/14688)
- **作者**: keyi-zz  **时间**: 2026-08-20 22:48 CST
- **摘要**: ### Why  The four-card e2e test 	ests/e2e/pull_request/four_card/test_graph_mode.py has an unstable precision check that is currently blocking PR CI runs.  ### What  Temporarily exclude the test file from the compilation_aclgraph and worker_v1 test selections via the native skip_tests mechanism in .…

### #14687 — [[v0.26.0rc][BugFix] Backport critical correctness and stability fixes](https://github.com/vllm-project/vllm-ascend/pull/14687)
- **作者**: jiaqi-lee  **时间**: 2026-08-20 22:22 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization, ready
- **摘要**: ### What this PR does / why we need it?  This is the v0.26.0 release-blocker backport rollup. It contains only the 19 audited high-severity correctness and stability fixes that were still missing from releases/v0.26.0rc at 1f95052cb.  The branch has 23 physical commits because #14619 is preserved as…

### #14685 — [[Feature][Model] Support MiniMax-M3 A3 prefill KV gather Q](https://github.com/vllm-project/vllm-ascend/pull/14685)
- **作者**: yingyingzizi  **时间**: 2026-08-20 21:19 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This PR enables the KV-gather-Q sparse prefill path for MiniMax-M3 on Ascend A3.  It builds on the MiniMax-M3/A5 support from #14412 while keeping the existing A5 path and ACLNN interface unchanged.  The A3 legacy sparse-attention path gathers KV blocks indep…

### #14684 — [[v0.23.0][Doc] Translated Doc files 2026-08-20](https://github.com/vllm-project/vllm-ascend/pull/14684)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-20 20:56 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **19** file(s):  - <code>docs/source/locale/zh_CN/LC_MESSAGES/community/contributors.po</code> - <code>docs/source/locale/zh_CN/LC_MESSAGES/community/versioning_policy.po</code> - <code>docs/source/locale/zh_CN/LC_MESSAGES/tutorials/models/DeepSeek-V3.1.po</co…

### #14683 — [[Test] Add GLM-5.2 SFA DCP nightly guard](https://github.com/vllm-project/vllm-ascend/pull/14683)
- **作者**: pisceskkk  **时间**: 2026-08-20 20:51 CST
- **标签**: module:tests
- **摘要**: ## What this PR does  - replaces the obsolete DeepSeek-V3.2 SFA+DCP config with a GLM-5.2 W4A8C8 guard - follows the GLM-5.2 single-node 1M layout: DP1/PP1/TP16/PCP1/DCP16 with block/interleave size 128 - enables SFA C8, LI C8, DSA-CP, and MTP3 to cover the replicated-indexer path - registers the ca…
