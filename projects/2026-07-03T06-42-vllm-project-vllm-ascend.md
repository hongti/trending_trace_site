# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-03 06:42 UTC

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文简洁摘要：

### 🚨 Issue 动态
*   **CI 自动化流程失败**：`main2main` 自动同步/适配流程在多个 commit 范围内均未能完成所有计划步骤，状态为 `failed`，需人工介入审查（#11393, #11387）。
*   **官网文档缺陷**：快速开始页面点击下载 `.md` 源文件无效，且页面右上角按钮释义存在中英文混杂问题（#11392）。

---

### 🛠️ Pull Request 动态

**🌟 新特性与模型支持**
*   **新增 MiniMax-M2 模型支持**：在昇腾 NPU 上完成 MiniMax-M2 模型的适配与注册（#11390）。
*   **DeepSeek-V4 (DSv4) 优化支持**：
    *   **IndexCache 特性**：在 Context-Parallel DSA 路径中支持 `index_cache`，允许跨层复用 indexer 的 top-k indices，避免每层重复计算（#11386）。
    *   **选择性 Prefix-cache 保留**：解决了由于昇腾 `block_size` 语义（物理 slots 数 vs token 数）差异导致缓存无法正确选取的问题（#11383）。
*   **海外版硬件适配**：适配海外版 HDK A2G3 硬件，确保 vllm-ascend 在海外场景下可正常安装与使用（#11382）。
*   **MTP 适配升级**：MTP (Multi-Token Prediction) 特性适配 vLLM v0.23.0（#11381）。

**🐛 重要 Bug 修复**
*   **修复 MoE 算子精度溢出**：修复了 310p 设备上 `MoEGatingTopkSoftmax` 算子在输入 token 维度为 2048 时引发的 UB 溢出问题，该问题导致了 Qwen3.5-MoE 的隐藏精度异常（#11391）。
*   **修复 Model Runner 版本冲突**：解决上游 vLLM 自动启用 v2 runner（基于白名单/Triton等）而 vllm-ascend 仍走 v1 的兼容性问题，改为仅通过环境变量门控 v2 runner（#11389）。
*   **修复 GLM52 DCP 执行异常**：修复了 GLM52 在 SFA 注意力路径中的 DCP 执行问题，处理了异步推测解码清空 `num_computed_tokens_cpu` 的情况，并传递了稀疏 C8 indexer 的量化 scale（#11388）。
*   **修复 MiniMax-M2.5 解析器畸形参数**：修复了增量解析器在 `</parameter>` 被 token 化成碎片时，导致流式输出拼接出畸形 JSON 参数的问题（#11384）。

**⚡ 性能优化**
*   **ChunkedPrefill 拆分调用**：将 ChunkedPrefill 拆分为独立的 decode 和 prefill FIA 调用，以优化执行流程（#11385）。

---

### 📦 Release 动态
*   **近期无新版本发布**。

---

## 🐛 Issues

### #11393 — [main2main manual review required (272c1695)](https://github.com/vllm-project/vllm-ascend/issues/11393)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-03 06:32 UTC
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR:  - Commit range: `a30addc7548a9a8b9b3323a7bc3eb7d7c4895d1c`...`272c16953eac7c46db7719d284d8a0ff19e63446` - Status: `failed`  ## Final Summary  main2main adaptation failed — no steps completed.

### #11392 — [[Doc]:【向我开炮】官网文档问题](https://github.com/vllm-project/vllm-ascend/issues/11392)
- **作者**: fenghaoyu0212  **时间**: 2026-07-03 06:32 UTC
- **标签**: documentation
- **摘要**: ### 📚 The doc issue  https://docs.vllm.ai/projects/vllm-ascend-cn/zh-cn/latest/quick_start.html 中点击“.md”即”下载源文件“，无法下载。右上角四个按钮的释义，中英文混杂。  ### Suggest a potential alternative/fix  _No response_

### #11387 — [main2main manual review required (badddd25)](https://github.com/vllm-project/vllm-ascend/issues/11387)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-03 03:50 UTC
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR:  - Commit range: `a30addc7548a9a8b9b3323a7bc3eb7d7c4895d1c`...`badddd254f744d26b6523b464c596f19015370f1` - Status: `failed`  ## Final Summary  main2main adaptation failed — no steps completed.

## 🔀 Pull Requests

### #11391 — [[BugFix][Ops][310p]:fix the accuracy issue caused by MoEGatingTopkSoftmax](https://github.com/vllm-project/vllm-ascend/pull/11391)
- **作者**: Tflowers-0129  **时间**: 2026-07-03 06:29 UTC
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Based on previous community issues, we had already noticed that Qwen3.5-MoE might have a hidden accuracy issue. The root cause has now been identified: when the input token dimension of the `MoEGatingTopkSoftmax` operator is 2048, UB overflow may occur, which…

### #11390 — [[Core] Add MiniMax-M2 model support](https://github.com/vllm-project/vllm-ascend/pull/11390)
- **作者**: zhaosonghan-2026  **时间**: 2026-07-03 06:28 UTC
- **摘要**: 实现了 MiniMax-M2 模型在昇腾 NPU 上的适配支持。  关联 issue: #7328  修改内容： - 添加了 MiniMax-M2 模型配置和实现 - 注册了模型到 vLLM 模型注册表 - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/a30addc7548a9a8b9b3323a7bc3eb7d7c4895d1c

### #11389 — [[BugFix] fix vllm use v2 but vllm-ascend use v1](https://github.com/vllm-project/vllm-ascend/pull/11389)
- **作者**: wangx700  **时间**: 2026-07-03 03:58 UTC
- **标签**: module:core
- **摘要**: ### What this PR does / why we need it? Add patch_use_v2_model_runner.py — gate v2 runner by env var only. Upstream vLLM enables the v2 model runner based on model architecture whitelists, Triton availability, and feature compatibility — even without VLLM_USE_V2_MODEL_RUNNER set. On Ascend the NPU v…

### #11388 — [[BugFix][Attention] Fix GLM52 DCP](https://github.com/vllm-project/vllm-ascend/pull/11388)
- **作者**: pisceskkk  **时间**: 2026-07-03 03:50 UTC
- **摘要**: ### What this PR does / why we need it?  This PR fixes GLM52 DCP execution in the SFA attention path.  - Uses `num_computed_tokens_of_pcp_dcp` as the SFA CP metadata source when async speculative decoding clears `num_computed_tokens_cpu`. - Propagates sparse C8 indexer quant/dequant scales through C…

### #11386 — [[Feature] Support IndexCache in context-parallel DSA path](https://github.com/vllm-project/vllm-ascend/pull/11386)
- **作者**: GDzhu01  **时间**: 2026-07-03 03:48 UTC
- **摘要**: ## What this PR does / why we need it?  The `index_cache` feature (enabled via `--hf-overrides '{"use_index_cache": true, "index_topk_pattern": "..."}'`) lets DeepSeek-V4 layers reuse the indexer's top-k indices across layers instead of recomputing them every layer. The `index_topk_pattern` string m…

### #11385 — [[Performance][950PR]Split ChunkedPrefill into separate decode and prefill FIA calls](https://github.com/vllm-project/vllm-ascend/pull/11385)
- **作者**: FengDengcai  **时间**: 2026-07-03 03:27 UTC
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/a30addc7548a9a8b9b3323a7bc3eb7d7c4895d1c

### #11384 — [[BugFix][Parser] Avoid partial MiniMax parameter arguments](https://github.com/vllm-project/vllm-ascend/pull/11384)
- **作者**: QwertyJack  **时间**: 2026-07-03 03:26 UTC
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Refs #10758.  MiniMax-M2.5 can tokenize `</parameter>` as pieces such as `</`, `parameter`, and `>`. The current incremental MiniMax-M2 parser on main streams argument JSON while a `<parameter>` is still open, so clients can reconstruct malformed arguments co…

### #11383 — [[Feature][Prefix Caching][DSv4] Support selective prefix-cache retention #43447](https://github.com/vllm-project/vllm-ascend/pull/11383)
- **作者**: Csrayz  **时间**: 2026-07-03 03:25 UTC
- **摘要**: ### What this PR does / why we need it?  Since Ascend's block_size semantics refer to the number of physical slots rather than the number of tokens, selective prefix-cache retention cannot be properly selected during caching. #10517  Previously, alignment_tokens in vLLM-Ascend were defined in mixed …

### #11382 — [HDK A2G3 adapted for the CH version.](https://github.com/vllm-project/vllm-ascend/pull/11382)
- **作者**: ZT-AIA  **时间**: 2026-07-03 03:13 UTC
- **摘要**: ### What this PR does / why we need it? VLLM ASCEND is adapted for the overseas version of HDK A2G3. To ensure that VLLM ASCEND can be properly installed and used in scenarios where the overseas version of HDK is being used.  ### Does this PR introduce _any_ user-facing change? No  ### How was this …

### #11381 — [Mtp vllm23](https://github.com/vllm-project/vllm-ascend/pull/11381)
- **作者**: swimming2007-doge  **时间**: 2026-07-03 03:07 UTC
- **标签**: module:tests, module:ops, module:core, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/a30addc7548a9a8b9b3323a7bc3eb7d7c4895d1c
