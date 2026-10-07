# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-07 14:22 CST

## AI 总结

根据 GitHub 仓库 **vllm-project/vllm-ascend** 的最近动态，以下是中文简洁摘要：

### 🚀 Release (版本发布)
*本期动态中未包含版本发布信息。*

### 💬 Issue (问题讨论)
*本期动态中未包含 Issue 相关信息。*

### 🔧 Pull Request (代码合并)
本次动态共包含 10 个 PR，主要涵盖新特性、Bug 修复、算子重构及 CI 优化：

**1. 新特性**
* **支持 MLA 并行堆叠 (#17940)**：实现了 MLA 的预填充上下文并行（PCP）与解码上下文并行（DCP）的堆叠支持，通过 TP 组收集不同的 query heads 以增强并行处理能力。

**2. Bug 修复**
* **修复 Model Runner V2 重计算采样问题 (#17941)**：在开启 PD decode recompute 时，修复了最后一个 prompt chunk 被误分类为 decode 的问题，确保正确保留真实的 physical prefill 状态。
* **恢复 310P 传统的连续 KV 缓存布局 (#17935)**：保留了旧有的 310P 连续 KV Cache 配置和布局，修复了此前 PR #14340 引入的布局变更问题。

**3. 算子重构**
* **自定义算子重命名以避免冲突 (#17936, #17937)**：为 `kv_quant_sparse_flash_attention` 自定义算子整体添加 `_vllm` 后缀（涉及目录、aclnn API、torch binding、tiling key 及测试等 29 个文件），以防止其 op type 与 A5 产生冲突。#17937 为保留原作者署名的整合 PR。

**4. CI 与性能诊断**
* **增强 NPU 性能诊断能力 (#17942, #17943)**：支持在 YAML 单节点基准测试中捕获受限的 NPU trace，并在吞吐量护栏失败时上传该 trace，专门用于诊断 Qwen3-30B A2 (BF16 和 W8A8) 的吞吐性能回归。
* **触发性能基线重测 (#17938, #17939)**：通过空 commit 触发 Qwen3-30B A2 TP4 的性能护栏重测；并为 GLM-5.2 PD 基线验证创建不变的 CI 基线以供对比。

**5. 文档**
* **文档中文化 (#17944)**：自动翻译了 24 个开发者指南/设计文档至简体中文。

---

## 🔀 Pull Requests

### #17944 — [[Doc] Translated Doc files 2026-10-07](https://github.com/vllm-project/vllm-ascend/pull/17944)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-07 13:49 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **24** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents/add_custom_aclnn_op.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/deve…

### #17943 — [[CI][Diagnostic] Capture Qwen3-30B A2 BF16 and W8A8 NPU profiles](https://github.com/vllm-project/vllm-ascend/pull/17943)
- **作者**: xujiaz2000  **时间**: 2026-10-07 13:26 CST
- **标签**: documentation, ci/build, module:tests
- **摘要**: ### What this PR does / why we need it?  Diagnostic capture for the Qwen3-30B-A3B BF16 and W8A8 A2 throughput regression. This branch includes the profiling support from https://github.com/vllm-project/vllm-ascend/pull/17942 and enables it in the two existing model YAMLs. **Keep this diagnostic PR o…

### #17942 — [[CI] Support bounded NPU profiling in YAML single-node benchmarks](https://github.com/vllm-project/vllm-ascend/pull/17942)
- **作者**: xujiaz2000  **时间**: 2026-10-07 13:23 CST
- **标签**: documentation, ci/build, module:tests
- **摘要**: ### What this PR does / why we need it?  YAML-driven performance tests currently cannot capture a bounded NPU trace while the benchmark is running or upload that trace after a throughput guard fails. This adds an opt-in `profiling` mapping for one named performance benchmark, backed by the existing …

### #17941 — [[BugFix][ModelRunner] Preserve physical prefill during recompute sampling](https://github.com/vllm-project/vllm-ascend/pull/17941)
- **作者**: yongfuFang  **时间**: 2026-10-07 12:28 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  With PD decode recompute enabled in Model Runner V2, the last prompt chunk can be classified as decode to select a uniform decode graph, even though local physical prefill has not finished. On vLLM versions that mask placeholder draft rows using `InputBatch.h…

### #17940 — [[Feature] support MLA PCP and DCP stacking](https://github.com/vllm-project/vllm-ascend/pull/17940)
- **作者**: weiguihua2  **时间**: 2026-10-07 12:23 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it? Support stacking MLA prefill context parallelism (PCP) with decode context parallelism (DCP). The change gathers distinct query heads through the TP group, keeps KV fragments on the DCP communication path, and merges attention outputs through the TP and PCP gr…

### #17939 — [[CI] Empty PR for GLM 5.2 PD baseline verification](https://github.com/vllm-project/vllm-ascend/pull/17939)
- **作者**: Wyz-134  **时间**: 2026-10-07 11:29 CST
- **摘要**: ### What this PR does / why we need it?  Empty PR to provide an unchanged-main CI baseline for `GLM-5.2-w8a8c8-128k-1k-90-50-PD`, for comparison with #17872.  Contains one empty commit on top of `ec570818bc428da1e3dc1820434cbadf2d5cf300`. No files or test settings are changed.  ### Does this PR intr…

### #17938 — [[CI] Retest Qwen3-30B A2 BF16 and W8A8 performance](https://github.com/vllm-project/vllm-ascend/pull/17938)
- **作者**: xujiaz2000  **时间**: 2026-10-07 10:03 CST
- **摘要**: ### What this PR does / why we need it?  Create an empty commit on upstream main `621e741bad97fbb2299e5bebae2ead3e598e9cff` to rerun the existing A2 TP4 performance guards:  - `qwen3-30b-a3b-bf16-a2-performance` - `qwen3-30b-a3b-w8a8-a2-performance`  The previous measurements were approximately 5% b…

### #17937 — [[Ops][Refactor] Integrate custom sparse attention operator rename from #17936](https://github.com/vllm-project/vllm-ascend/pull/17937)
- **作者**: danziheng1024  **时间**: 2026-10-07 01:23 CST
- **标签**: documentation, module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  Integrate #17936 (commit 395d0d24feac0a55e6f54c4ea6c9f71f03dd8478), preserving original author attribution. This draft contains the same changes as that still-open PR, with no additional functional changes.  Rename the custom KvQuantSparseFlashAttention opera…

### #17936 — [[Ops][Refactor] Rename kv_quant_sparse_flash_attention custom op with _vllm suffix](https://github.com/vllm-project/vllm-ascend/pull/17936)
- **作者**: ZT-AIA  **时间**: 2026-10-07 01:06 CST
- **标签**: documentation, module:tests
- **摘要**: What : 自定义算子整体加 _vllm 后缀：目录/内核文件、aclnn API（ aclnnKvQuantSparseFlashAttentionVllm ）、torch binding（ npu_kv_quant_sparse_flash_attention_vllm ）、tiling key、单测/e2e 测试同步改名（29 个文件）。  Why : 自定义算子 op type 与 A5 机型内置 CANN 算子 KvQuantSparseFlashAttention 重名，导致算子注册表冲突，在 aclgraph 捕获阶段 aclnnKvQuantSparseFlashAttent…

### #17935 — [[BugFix][310P] Preserve legacy contiguous KV cache layout](https://github.com/vllm-project/vllm-ascend/pull/17935)
- **作者**: zhaochuang001  **时间**: 2026-10-06 21:55 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Preserve the legacy **310P contiguous KV Cache configuration and layout** that existed before #14340. This replaces the earlier current-token `.contiguous()` proposal in this draft.  #14340 added `super().update_block_size_for_backend()` to the Ascend platfor…
