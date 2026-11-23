# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-23 16:03 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库最近的动态摘要：

### 🐛 Issue 动态
*   **#22898 [性能优化]**：提议在 Ascend A2 架构上跨两个核心组（Core Groups）并行化稀疏 KV gather 和 attention 操作。作者在分析 Ascend 910B3 上的 `SparseFlashMla` 算子后，提出改进方案以打破当前单 Cube 核加双 Vector 核的限制，从而提升整体性能。

### 🔀 PR 动态

**新特性**
*   **#22893 / #22894 [EPLB 增强]**：为 Model Runner V2 (MRv2) 引入了静态 EPLB（专家负载均衡）部署功能。支持版本化的离线导出（#22893，含 HTTP 端点和关停时自动导出），并支持在启动时安装静态部署文件，同时在服务期间关闭动态收集与重排（#22894）。
*   **#22892 [投机解码]**：在 Ascend MRV2 中支持 DFlash 系列 P/D（Prefill/Decode）分离架构下的 `prefill_context_kv_only` 配置，增强了投机解码能力。

**Bug 修复**
*   **#22888 [引擎启动崩溃修复]**：修复了 `AscendMultiConnector` 在启动时因 `engine_id` 参数重复传递导致的 `TypeError` 崩溃问题（关联 Issue #22081）。
*   **#22896 [EPLB 权重名称]**：在 W4A4MXFP 量化场景下，将 EPLB 适配器查找专家权重的名称更新为 CANN Mega MoE 的标准命名。
*   **#22895 [生命周期隔离]**：在 MRv2 中将目标模型与 MTP 草稿模型的 EPLB 处理生命周期解耦，并明确在草稿模型配置上禁用 EPLB。

**文档与测试**
*   **#22897 [文档修复]**：修复了 Ascend 文档中的多处事实错误，包括 API 方法名错误和描述不准确。
*   **#22889 / #22890 / #22891 [CI 探针]**：提交了多个用于 CI 环境的只读探针测试（如 Ascend 310P3 NPU 设备身份探测、RBAC 权限探测等），主要用于验证底层硬件与权限配置。

### 🚀 Release 动态
*   近期无新版本发布。

---

## 🐛 Issues

### #22898 — [[Performance]: Parallelize sparse KV gather and attention across two core groups on Ascend A2](https://github.com/vllm-project/vllm-ascend/issues/22898)
- **作者**: Dunnn-Smith  **时间**: 2026-11-23 15:58 CST
- **标签**: performance
- **摘要**: ### Proposal to improve performance  ### Background  I profiled `SparseFlashMla` as a standalone operator on Ascend 910B3. The current mixed core group consists of one Cube core and two Vector cores (`1C2V`). Before the long-context KV matrix multiplication starts, the Cube has to wait until both Ve…

## 🔀 Pull Requests

### #22897 — [[Doc]  Fix factual errors in documentation](https://github.com/vllm-project/vllm-ascend/pull/22897)
- **作者**: axx-ty911  **时间**: 2026-11-23 15:56 CST
- **标签**: documentation, merge-conflicts
- **摘要**: ### What this PR does / why we need it? Fix multiple factual errors in Ascend documentation, including incorrect API method names, inaccurate descriptions and other document content mistakes. These corrections help users avoid confusion when following the docs.  ### Does this PR introduce _any_ user…

### #22896 — [[BugFix][EPLB] Use W4A4MXFP CANN Mega MoE weight names](https://github.com/vllm-project/vllm-ascend/pull/22896)
- **作者**: xbwcj  **时间**: 2026-11-23 15:48 CST
- **摘要**: ## What this PR does Uses the CANN Mega MoE per-expert list tensor names when the W4A4MXFP EPLB adaptor looks up expert weights.  ## Changes - Update EPLB_EXPERT_WEIGHT_NAMES for QuantType.W4A4MXFP transposed weights. - Add a release-note draft.  ## How tested - Python compilation of the changed ada…

### #22895 — [[BugFix][SpecDecode] Keep EPLB target and MTP draft lifecycle separate](https://github.com/vllm-project/vllm-ascend/pull/22895)
- **作者**: xbwcj  **时间**: 2026-11-23 15:48 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: Depends on #22893 and #22894.  ## What this PR does Separates EPLB handling for the target model and MTP draft model in Model Runner V2.  ## Changes - Disable EPLB on the MTP draft configuration and clear draft redundant experts. - Skip draft registration for dynamic EPLB in PD producer mode. - Prev…

### #22894 — [[Feature] Install static MRv2 EPLB deployment](https://github.com/vllm-project/vllm-ascend/pull/22894)
- **作者**: xbwcj  **时间**: 2026-11-23 15:47 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: Depends on #22893.  ## What this PR does Adds startup-time installation of a previously exported static MRv2 EPLB deployment, disables dynamic collection/rearrangement while serving, and adds optional lightweight load observability.  ## Changes - Install the deployment once before profiling and grap…

### #22893 — [[Feature] Export committed MRv2 EPLB deployment](https://github.com/vllm-project/vllm-ascend/pull/22893)
- **作者**: xbwcj  **时间**: 2026-11-23 15:47 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ## What this PR does Adds versioned Model Runner V2 EPLB deployment files and a collective offline export operation. It also exposes an HTTP export endpoint and DP-aware auto-export on shutdown.  ## Changes - Add portable EplbPlacement schema with metadata, provenance, canonical digest, validation, …

### #22892 — [[Spec Decode][MRV2] Support prefill_context_kv_only for DFlash-family P/D prefill](https://github.com/vllm-project/vllm-ascend/pull/22892)
- **作者**: Liuchenbing-2026  **时间**: 2026-11-23 15:46 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  This PR adds Ascend Model Runner V2 (MRV2) support for `speculative_config.prefill_context_kv_only` in P/D disaggregation with DFlash-family speculative decoding (requiring `--kv-transfer-config` with `kv_role` set to `kv_producer` or `kv_both`). When enabled…

### #22891 — [test-pr-probe](https://github.com/vllm-project/vllm-ascend/pull/22891)
- **作者**: xbwcj  **时间**: 2026-11-23 15:43 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: probe - vLLM main: https://github.com/vllm-project/vllm/commit/c410543d9ffff03f5f6c4a8c0ae1d3ad082c4a15

### #22890 — [c211-132055 read-only SA/RBAC job-interface probe c211-132055-9e8ce70347f9c50e](https://github.com/vllm-project/vllm-ascend/pull/22890)
- **作者**: picxbed  **时间**: 2026-11-23 15:32 CST
- **摘要**: Nonce-tagged, strictly read-only service-account/RBAC presence probe (no secret values, no object creation) for the CR-02 chain. nonce c211-132055-9e8ce70347f9c50e - vLLM main: https://github.com/vllm-project/vllm/commit/c410543d9ffff03f5f6c4a8c0ae1d3ad082c4a15

### #22889 — [c211-132012 read-only Ascend 310P3 NPU card/chip-id identity probe (310p-2)](https://github.com/vllm-project/vllm-ascend/pull/22889)
- **作者**: picxbed  **时间**: 2026-11-23 15:31 CST
- **摘要**: Read-only per-device Ascend NPU identity check on the linux-aarch64-310p-2 runner pool (310P3 cards on BDFs 0000:08:00.0 / 0000:09:00.0, image tag 9.2.1-310p-ubuntu22.04-py3.12). Each job prints nonce/hostname/uid/RUNNER_NAME and only local device surfaces: full `npu-smi info`, `npu-smi -v`, `npu-sm…

### #22888 — [[BugFix] Honor child engine_id in AscendMultiConnector](https://github.com/vllm-project/vllm-ascend/pull/22888)
- **作者**: four1er  **时间**: 2026-11-23 15:28 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Fixes #22081. Upstream `MultiConnector` crashes at engine startup with `TypeError: KVTransferConfig() got multiple values for keyword argument 'engine_id'` when a child connector config explicitly sets `engine_id`, so any MultiConnector deployment with per-ch…
