# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-27 13:26 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🛑 Issue 动态
* **性能瓶颈问题 (#19910)**：报告了在 Ascend 910B 架构下，`QuantLightningIndexerV2` 的 consumer mode（`candidate_mode=2`）存在严重的性能浪费。当前逻辑会先计算整个 2048 宽 tile 的分数再进行掩码，导致约 99% 的 `aiv_time` 被消耗在无用数据的排序上。

### 🛠️ PR 动态
**1. 性能优化**
* **#19911**：针对上述 Issue #19910 提出修复，为 `QuantLightningIndexerV2` 添加了候选跳过快速路径（mode 4），避免对掩码数据的无效排序计算。
* **#19909**：优化 FlashMLA DCP 投机解码，引入自动预算 overlap cores 机制，解决小 pack 情况下 Vector 资源受限的问题。

**2. Bug 修复**
* **#19912 / #19913**：修复了升级至 Triton 3.6.0 后引发的 N-gram 投机解码兼容性报错问题（修正了 `tl.load` 中的 `care_padding` 参数）。其中 #19913 为向 `v0.30.0` 版本回溯的修复补丁。
* **#19905**：修复了版本发布时文档测试配置错误的问题。在 CANN 镜像可用但 vLLM Ascend wheel 尚未就绪时，强制使用源码安装进行 doctests。

**3. 文档与 CI 维护**
* **#19904 / #19903**：澄清并完善了 GLM-5.2 / 5.3 及 GLM-5.3-Flash 模型的部署指南与前置条件，统一了 A3 设备描述（#19903 为 `v0.30.0` 专属补丁）。
* **#19908**：完善了 `v0.30.0` 版本中 Qwen3.8-2.4T-A95B 模型的部署指南，统一使用 TiB 表示权重大小。
* **#19907**：自动翻译了 29 个文档文件至简体中文。
* **#19906**：将 CI 流程改进（依据 PR 基础分支选择 vLLM ref 用于 E2E 和 CPU UT）cherry-pick 至 `releases/v0.31.0rc` 分支。

### 🚀 Release 动态
* 本期数据中无新的版本发布记录。

---

## 🐛 Issues

### #19910 — [[Perf] QuantLightningIndexerV2 consumer mode spends ~99% aiv_time sorting masked full-tile data](https://github.com/vllm-project/vllm-ascend/issues/19910)
- **作者**: qys4123  **时间**: 2026-10-27 13:09 CST
- **摘要**: ## Description  `VllmAscendQuantLightningIndexerV2` (arch22, Ascend 910B) in `candidate_mode=2` (consumer) computes scores for the **full 2048-wide tile** and then only *masks* out-of-candidate scores to `NEG_HUGE` right before the sort. The AIV `SortAll(2048) + MergeSort` cascade still processes th…

## 🔀 Pull Requests

### #19913 — [[Bugfix] Fix N-gram Triton 3.6.0 compatibility on v0.30.0](https://github.com/vllm-project/vllm-ascend/pull/19913)
- **作者**: AuroraEmiya  **时间**: 2026-10-27 13:18 CST
- **标签**: module:ops, ready-precise
- **摘要**: ### What this PR does / why we need it?  Apply the N-gram speculative decoding compatibility fix from #19912 to `releases/v0.30.0`.  After upgrading to Triton 3.6.0, the affected environment rejects the `care_padding=False` argument passed to `tl.load` in `ngram_spec_decode_kernel`, causing kernel c…

### #19912 — [[Bugfix] Fix N-gram speculative decoding compatibility with Triton 3.6.0](https://github.com/vllm-project/vllm-ascend/pull/19912)
- **作者**: AuroraEmiya  **时间**: 2026-10-27 13:17 CST
- **标签**: module:ops, ready-precise
- **摘要**: ### What this PR does / why we need it?  Fix an N-gram speculative decoding failure observed after upgrading to Triton 3.6.0.  `ngram_spec_decode_kernel` passes `care_padding=False` to three `tl.load` calls. The `tl.load` implementation in the affected environment does not accept this argument, caus…

### #19911 — [[Perf][Csrc] Add candidate skip fast path (mode 4) for QuantLightningIndexerV2](https://github.com/vllm-project/vllm-ascend/pull/19911)
- **作者**: qys4123  **时间**: 2026-10-27 13:10 CST
- **标签**: module:core
- **摘要**: ## What this PR does / why we need it?  Fixes #19910.  `QuantLightningIndexerV2` in `candidate_mode=2` (consumer) computes scores for the full 2048-wide tile and only *masks* out-of-candidate scores right before the sort, so the AIV `SortAll(2048) + MergeSort` cascade spends ~99% of `aiv_time` sorti…

### #19909 — [[Performance] Automatically budget FlashMLA DCP overlap cores](https://github.com/vllm-project/vllm-ascend/pull/19909)
- **作者**: pisceskkk  **时间**: 2026-10-27 12:42 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  In FlashMLA DCP speculative decoding, current-chunk MIX attention overlaps history output/LSE packing. Both consume Vector resources. With a small pack, unrestricted overlap can delay attention enough to erase most of the overlap benefit.  Automatically deriv…

### #19908 — [[Doc][Misc]Clarify Qwen3.8-2.4T-A95B deployment guide for v0.30.0](https://github.com/vllm-project/vllm-ascend/pull/19908)
- **作者**: Jingluo-nan  **时间**: 2026-10-27 12:29 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Backports the documentation improvements from #19900 to `releases/v0.30.0`:  - Use TiB consistently for Qwen3.8-2.4T-A95B weight sizes. - Explain how to check startup logs during weight loading and graph capture. - Use the configured `SERVICE_PORT` in API and…

### #19907 — [[v0.30.0][Doc] Translated Doc files 2026-10-27](https://github.com/vllm-project/vllm-ascend/pull/19907)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-27 12:23 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **29** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/getting_started/installation.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/getting_started/quick_start.…

### #19906 — [[Cherry-pick][releases/v0.31.0rc][CI] Select vLLM ref by PR base branch for E2E and CPU UT (from #19902)](https://github.com/vllm-project/vllm-ascend/pull/19906)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-27 11:58 CST
- **标签**: documentation, ci/build
- **摘要**: Cherry-pick of PR #19902 onto `releases/v0.31.0rc`.  Original PR: #19902 Original author: @zhangxinyuehfad  ---  ### What this PR does / why we need it?  Resolve the vLLM ref from the PR target branch instead of hardcoding the verified main commit, so release branches no longer need a manual CI edit…

### #19905 — [[BugFix] Use source installation doctests for release version updates](https://github.com/vllm-project/vllm-ascend/pull/19905)
- **作者**: MrZ20  **时间**: 2026-10-27 11:54 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Release configuration updates in `mkdocs.yml` currently select pip installation doctests. When the CANN image is available but the vLLM Ascend wheel has not been published yet, these tests still run and fail during installation.  Select source installation in…

### #19903 — [[v0.30.0][Doc][Misc] Clarify GLM deployment prerequisites and references](https://github.com/vllm-project/vllm-ascend/pull/19903)
- **作者**: Wyz-134  **时间**: 2026-10-27 11:51 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Clarify the remaining issues from a review of the GLM-5.2/5.3 and GLM-5.3-Flash guides on releases/v0.30.0. Align A3 logical-device descriptions, document tokenizer installation and launcher arguments, clarify sequence limits and Mooncake ports, and add readi…

### #19904 — [[Doc][Misc] Clarify GLM deployment prerequisites and references](https://github.com/vllm-project/vllm-ascend/pull/19904)
- **作者**: Wyz-134  **时间**: 2026-10-27 11:51 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Clarify the remaining issues from a review of the GLM-5.2/5.3 and GLM-5.3-Flash deployment guides. Align A3 device descriptions with the logical devices used by serving commands, document tokenizer installation and launcher arguments, clarify sequence limits …
