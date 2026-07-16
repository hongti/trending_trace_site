# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-16 09:04 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库最近的动态摘要：

### 🚀 Release（版本发布）
近期暂无新版本发布动态。

### 🐛 Issue（问题反馈）
近期暂无新增 Issue 动态。

### 🔀 Pull Request（代码合并）
本期 PR 动态主要集中在**Bug修复**、**代码重构与清理**、**文档更新**及**新特性支持**四个方面：

**🛠️ Bugfix（重要修复）**
- **Mooncake 线程绑定修复**：修复了 Mooncake receiver 的 `ThreadPoolExecutor` 工作线程未正确绑定到注册 KV cache 的 NPU 设备的问题，该修复同时应用于标准和混合连接器，并已合入 `v0.23.0` 分支（#12126, #12127）。
- **数据切片修复**：修复了 `storekv_aicpu` 操作中数据切片不正确的问题（#12124）。
- **算子精度修复**：修复了在特定维度（`x.shape=[2,192]`）下 `npu_dequant_swiglu_quant` 算子的精度问题（#12123）。

**♻️ Refactor（重构与清理）**
- **KV Pool 简化**：重构了 `AscendStore` KV Pool 内部实现，集中了 KV cache layout 的推理逻辑，外部行为保持不变（#12125）。
- **移除废弃优化项**：清理并移除了 `enable_async_exponential` 配置及相关的异步指数采样路径（#12120）；移除了 `VLLM_ASCEND_ENABLE_MATMUL_ALLREDUCE` 配置及 `fuse_allreduce_rms` 优化路径（#12119）。

**📖 Doc（文档更新）**
- **GLM5.2 长上下文配置**：更新了 GLM5.2 模型的使用文档，新增了 1M（百万级）上下文配置指南及 DCP 相关的配置与部署说明，并同步合入 `v0.23.0` 分支（#12121, #12122）。

**✨ Feature（新特性）**
- **DFlash 支持 DCP/PCP**：为 DFlash 启用了 DCP（Disaggregated Context Processing）及 PCP 的支持（#12118）。

---

## 🔀 Pull Requests

### #12127 — [[v0.23.0][Bugfix] Bind Mooncake receiver threads to KV cache device](https://github.com/vllm-project/vllm-ascend/pull/12127)
- **作者**: maoxx241  **时间**: 2026-07-16 03:18 CST
- **标签**: module:tests
- **摘要**: ## What this PR does  Backport of #12126 to `releases/v0.23.0`.  - Bind every Mooncake receiver `ThreadPoolExecutor` worker to the NPU device of the registered KV cache. - Apply the fix to both the standard and hybrid Mooncake connectors. - Add regression tests that force two executor workers and ve…

### #12126 — [[Bugfix] Bind Mooncake receiver threads to KV cache device](https://github.com/vllm-project/vllm-ascend/pull/12126)
- **作者**: maoxx241  **时间**: 2026-07-16 03:11 CST
- **标签**: module:tests
- **摘要**: ## What this PR does  - Bind every Mooncake receiver `ThreadPoolExecutor` worker to the NPU device of the registered KV cache. - Apply the fix to both the standard and hybrid Mooncake connectors. - Add regression tests that force two executor workers and verify each worker calls `torch.npu.set_devic…

### #12125 — [[Refactor] Simplify AscendStore KV pool internals](https://github.com/vllm-project/vllm-ascend/pull/12125)
- **作者**: Pz1116  **时间**: 2026-07-16 03:09 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This PR simplifies the AscendStore KV Pool implementation without changing its external behavior.  - Centralizes KV cache layout inference shared by the scheduler and worker. - Uses the current vLLM KV cache spec registry and manager APIs directly instead of …

### #12124 — [[BugFix]Fix the issue of incorrect data slicing in storekv_aicpu](https://github.com/vllm-project/vllm-ascend/pull/12124)
- **作者**: ZT-AIA  **时间**: 2026-07-16 00:33 CST
- **摘要**: ### What this PR does / why we need it? Fix the issue of incorrect data slicing in storekv_aicpu  ### Does this PR introduce _any_ user-facing change? no ### How was this patch tested? test  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af6…

### #12123 — [[BugFix]Fix the precision issue of the npu_dequant_swiglu_quant operator when x.shape=[2,192]](https://github.com/vllm-project/vllm-ascend/pull/12123)
- **作者**: ZT-AIA  **时间**: 2026-07-15 23:39 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Fix the precision issue of the npu_dequant_swiglu_quant operator when x.shape=[2,192]  ### Does this PR introduce _any_ user-facing change? No ### How was this patch tested? test  - vLLM version: v0.24.0 - vLLM main: https://github.com/vllm-project/vllm/commit…

### #12122 — [[cherry-pick][v0.23.0] [Doc](GLM5.2, DCP): update GLM5.2 docs with 1M Context Configuration](https://github.com/vllm-project/vllm-ascend/pull/12122)
- **作者**: pisceskkk  **时间**: 2026-07-15 23:06 CST
- **标签**: documentation
- **摘要**: ## Summary - Cherry-picks the GLM5.2 docs update into releases/v0.23.0. - Keeps the release documentation aligned with the main PR: https://github.com/vllm-project/vllm-ascend/pull/12121  ## Validation - Not run; docs-only change.  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project…

### #12121 — [[Doc](GLM5.2, DCP): update GLM5.2 docs with 1M Context Configuration](https://github.com/vllm-project/vllm-ascend/pull/12121)
- **作者**: pisceskkk  **时间**: 2026-07-15 23:06 CST
- **标签**: documentation
- **摘要**: ## Summary - Updates the GLM5.2 tutorial docs with 1M context configuration guidance. - Adds the DCP-related configuration and deployment notes for the GLM5.2 documentation. - Cherry-picked from f080f60cdd7c3f737dac54a092091d3e0fb1251b.  ## Validation - Not run; docs-only change.  - vLLM version: v0…

### #12120 — [[Refactor] Remove enable_async_exponential optimization techniques](https://github.com/vllm-project/vllm-ascend/pull/12120)
- **作者**: ningjingbengxiaohai  **时间**: 2026-07-15 22:49 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ## What this PR does / why we need it?  Remove the enable_async_exponential configuration and the async-exponential sampling path. This deletes the related config field, model runner hook, sampler-side async event logic, the dedicated e2e test, and the additional_config documentation entries.  ## Do…

### #12119 — [[Refactor] Remove matmulallreduce and pass optimization techniques](https://github.com/vllm-project/vllm-ascend/pull/12119)
- **作者**: ningjingbengxiaohai  **时间**: 2026-07-15 22:34 CST
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: ## What this PR does / why we need it?  Remove the VLLM_ASCEND_ENABLE_MATMUL_ALLREDUCE configuration and the fuse_allreduce_rms optimization path. This deletes the matmul_allreduce_add_rmsnorm custom op, graph fusion pass, linear matmul-allreduce dispatch path, config/env migration entries, related …

### #12118 — [Enabling DCP/PCP for DFlash](https://github.com/vllm-project/vllm-ascend/pull/12118)
- **作者**: Laasap  **时间**: 2026-07-15 22:14 CST
- **标签**: module:ops
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.24.0 - vLLM main: https://github.com/vllm-project/vllm/commit/85c09e9885e346ea1612da30ebff5a75f67d2350
