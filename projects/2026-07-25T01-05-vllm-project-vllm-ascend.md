# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-25 09:05 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文简洁摘要：

### 📌 Issue 动态
- **CI 流程异常**：`main2main` 自动化同步/移植流程在执行过程中失败并中断，目前状态标记为 `failed`，需要人工审核介入（#12824）。

### 🚀 PR 动态
近期 PR 主要集中在**算子清理、Bug修复、性能优化与文档更新**，且多处涉及跨版本（v0.23.0 / v0.24.0 / main）的代码同步与 Backport：

- **🛠️ BugFix (缺陷修复)**：
  - **修复 MRotaryEmbedding 硬编码问题**：修正了 `AscendMRotaryEmbedding.forward_oot` 中强制硬编码 Neox 风格（`rotary_mode="half"`）旋转的 Bug（#12820）。
  - **修复 AscendStore 跳过与保留查找逻辑**：针对 v0.23.0 分支，修复了 Scheduler 提供的原始 token 数量与压缩缓存（如 c4、c128）迭代方式不匹配导致的查找错误（#12819）。

- **⚡ Performance (性能优化)**：
  - **优化 AscendStore Key 构建与 Miss 路径**：将 v0.23.0 的优化适配至 main 分支。通过直接生成序列化非分层 key、缓存不可变前缀以及优化分组选择，提升了 KVPool miss 路径的执行效率。该优化由 #12818 及取代旧实现（#12136）的 #12814 共同推进。

- **🗑️ Misc / Ops (算子清理)**：
  - **移除自定义 Top-K Top-P AscendC 实现**：跨多个分支（v0.23.0, v0.24.0 及主线）全面移除了自定义的 `apply_top_k_top_p_custom` 算子实现及其相关的 kernels、tiling 和 acl 代码，回归通用实现（#12821, #12822, #12823）。

- **📖 Doc / Feature (文档与特性说明)**：
  - **更新 EPLB（专家并行负载均衡器）文档**：在主线和 v0.24.0rc 分支中补充了重要限制说明：**Ascend A5 硬件不支持 EPLB 与特定量化或 DeepSeek V4 模型配合使用**（#12816, #12817）。

- **🆕 新增支持**：
  - **CSA 多流特性**：引入了 CSA（Communication Stream Aggregation等）多流相关支持，以提升并发或通信效率（#12825，具体细节未完整展示）。

### 📦 Release 动态
- 近期**无新版本发布**记录。

---

## 🐛 Issues

### #12824 — [[main2main] main2main manual review required ()](https://github.com/vllm-project/vllm-ascend/issues/12824)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-25 00:37 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR:  - Commit range: `fe784ff22e630a31fd798f392b01e0a75c18f047`...`` - Status: `failed`  ## Final Summary  main2main adaptation failed — no steps completed.

## 🔀 Pull Requests

### #12825 — [csa multi-stream](https://github.com/vllm-project/vllm-ascend/pull/12825)
- **作者**: nklkj-hw  **时间**: 2026-07-25 08:51 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #12823 — [[v0.24.0][Misc][Ops] Remove custom top k top p AscendC Implementation](https://github.com/vllm-project/vllm-ascend/pull/12823)
- **作者**: linfeng-yuan  **时间**: 2026-07-24 22:38 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it? Backports #12232 to releases/v0.24.0  This pull request removes the custom `apply_top_k_top_p_custom` operator implementation, including its kernels, tiling, aclnn wrappers, and PyTorch bindings.  ### Does this PR introduce _any_ user-facing change? No user-fa…

### #12822 — [[v0.23.0][Misc][Ops] Remove custom top k top p AscendC Implementation](https://github.com/vllm-project/vllm-ascend/pull/12822)
- **作者**: linfeng-yuan  **时间**: 2026-07-24 22:34 CST
- **标签**: documentation, module:tests, ready
- **摘要**: ### What this PR does / why we need it? Backports #12232 to releases/v0.23.0  This pull request removes the custom `apply_top_k_top_p_custom` operator implementation, including its kernels, tiling, aclnn wrappers, and PyTorch bindings.  ### Does this PR introduce _any_ user-facing change? No user-fa…

### #12821 — [Remove custom top k top p 0240](https://github.com/vllm-project/vllm-ascend/pull/12821)
- **作者**: linfeng-yuan  **时间**: 2026-07-24 22:30 CST
- **标签**: documentation, ci/build, module:tests, module:ops, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?   - vLLM version: v0.25.1 - vLLM main: https://github.com/vllm-project/vllm/commit/fe784ff22e630a31fd798f392b01e0a75c18f047

### #12820 — [ [BugFix] Fix MRotaryEmbedding issue #12765](https://github.com/vllm-project/vllm-ascend/pull/12820)
- **作者**: JaYung136  **时间**: 2026-07-24 20:32 CST
- **标签**: module:ops
- **摘要**: ### What this PR does / why we need it?  Fixes #12765   **Bug:** `AscendMRotaryEmbedding.forward_oot` hardcodes Neox-style (chunk-half) rotation via `rotary_mode="half"` when calling `torch_npu.npu_mrope`, completely ignoring the `self.mrope_interleaved` flag.  This causes GPT-J style models (e.g. G…

### #12819 — [[v0.23.0][BugFix] Fix AscendStore skip and retention lookup](https://github.com/vllm-project/vllm-ascend/pull/12819)
- **作者**: Pz1116  **时间**: 2026-07-24 20:02 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it?  This is a follow-up to #12783.  - Scheduler-provided AscendStore skip bounds are raw token counts, while compressed cache families such as `c4` and `c128` iterate store chunks in the physical cache domain. Convert each group chunk range back to raw token posi…

### #12818 — [[Performance] Optimize AscendStore key construction and miss path](https://github.com/vllm-project/vllm-ascend/pull/12818)
- **作者**: Pz1116  **时间**: 2026-07-24 19:51 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This ports #12783 from `releases/v0.23.0` to the latest `main`.  - Generates serialized non-layerwise keys directly, caches immutable prefixes, and selects grouped chained hashes lazily. - Centralizes token/chunk iteration and removes duplicated key-building …

### #12817 — [[v0.24.0rc][Doc][Feature] Add info for eplb](https://github.com/vllm-project/vllm-ascend/pull/12817)
- **作者**: Spicy-Stick  **时间**: 2026-07-24 18:21 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  This PR updates the Expert Parallelism Load Balancer (EPLB) documentation to include: - A note that Ascend A5 does not support using EPLB with specific quantization types ("W4A8MXFP4", "W4A16", "W4A16MXFP4"). - Usage recommendations and scenarios where EPLB i…

### #12816 — [[main][Doc][Feature] Add info for eplb](https://github.com/vllm-project/vllm-ascend/pull/12816)
- **作者**: Spicy-Stick  **时间**: 2026-07-24 18:17 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? This PR updates the Expert Parallelism Load Balancer (EPLB) documentation to include:  A note that Ascend A5 does not support using EPLB with DeepSeek V4. Documentation for the new eplb_heat_collection_stage parameter. A warning that Static EPLB is scheduled f…

### #12814 — [[Performance] Optimize AscendStore key construction and miss path](https://github.com/vllm-project/vllm-ascend/pull/12814)
- **作者**: zmc1997  **时间**: 2026-07-24 17:54 CST
- **标签**: ci/build, module:tests, ready
- **摘要**: ### What this PR does / why we need it?  This supersedes #12136 and adapts the AscendStore key-construction and KVPool miss-path optimizations to `main` with a smaller, unconditional implementation.  - Generates serialized non-layerwise keys directly, caches immutable prefixes, and computes grouped …
