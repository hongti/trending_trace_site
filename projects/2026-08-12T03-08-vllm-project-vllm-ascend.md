# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-12 11:08 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📌 Issue（问题）

*   **严重兼容性阻塞（#14058）**：在 `v0.26.0rc` 版本中，`hunyuan_vl_processor_compat` 补丁硬性依赖 `transformers 5.14.1`，导致 Kimi-K3 的 P/D 分离部署（16节点）出现 `chat_template 400` 错误及 Cudagraph 捕获崩溃。
*   **安全漏洞（#14057）**：当前依赖的 Ray 版本（2.47.1~2.48.0）存在 CVE-2025-62593 远程代码执行（RCE）漏洞，建议升级至 Ray 2.52.0 或更高版本。

---

### 🔧 Pull Request（拉取请求）

**🚀 性能优化**
*   **Kimi K3 QKV 多流优化（#14063）**：将 QKV 打包与 gate 投影重叠执行，从私有分支移植至主库，提升 Kimi K3 运行性能。
*   **PP 气泡优化（#14054）**：针对流水线并行（Pipeline Parallelism）的气泡问题进行优化。

**♻️ 核心重构**
*   **P/D 分离重构（#14062）**：在 `RecomputeScheduler` 中删除将被抢占请求重新发送至 Prefill 节点进行重计算的逻辑，现统一使用 `RecomputeOffloadConnector`，避免在 Prefill 节点重复计算。
*   **硬件 Profile 扩展（#14059）**：扩展硬件 Profile 抽象，增加语义能力和策略枚举，并集中管理 Ascend 硬件家族的能力矩阵。

**🐛 Bug 修复**
*   **配置读取修复（#14061, #14064）**：修复了 RL/Lora 启动守卫及 `MooncakeConnector` 中的 KV 缓存重格式化路径仍读取已弃用环境变量的问题，现统一从 `additional_config` 读取，与文档保持一致。

**🧪 测试与文档**
*   **Nightly 测试修复（#14052, #14065）**：修复 Nightly 测试运行器异常，并补充缺失的 GLM-5.1 配置文件。
*   **文档整合（#14056）**：将 Layerwise Prefill 与 Sparse Decode KV cache 卸载的用户指南合并，并新增包含缓冲区复用等设计细节的设计文档。
*   **CI 测试（#14060）**：常规 CI 测试运行触发。

---

### 🚀 Release（版本发布）

*   近期**无**新版本发布记录。

---

## 🐛 Issues

### #14058 — [[v0.26.0rc] hunyuan_vl_processor_compat patch hard-requires transformers 5.14.1, blocks kimi-k3 PD (chat_template 400 / cudagraph capture crash)](https://github.com/vllm-project/vllm-ascend/issues/14058)
- **作者**: linnea-lin-00638949  **时间**: 2026-08-12 10:40 CST
- **摘要**: ## Problem  Deploying **kimi-k3 PD disaggregation** (16 nodes, DP8/TP16/EP64) on `releases/v0.26.0rc` (commit ed21138 and latest e592b2e79) with the 0.23.0 kimi-k3-a3 image + 0.26.0 source install hits a chain of issues caused by **`hunyuan_vl_processor_compat` patch hard-requiring `transformers>=5.…

### #14057 — [[Bug]: CVE-2025-62593 Ray AI计算引擎远程代码执行漏洞](https://github.com/vllm-project/vllm-ascend/issues/14057)
- **作者**: gemingjiu  **时间**: 2026-08-12 10:34 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> </details>  ### 🐛 Describe the bug  ``` 当前ray版本为 ray>=2.47.1,<=2.48.0，存在漏洞建议升级到ray 2.52.0或更高版本 ```  <img width="1873" height="900" alt="Image" src="https://github.com/user-attachments/assets/d4565998-fadb-4bc4-be10-1f056da90cc7" />

## 🔀 Pull Requests

### #14065 — [[Test] Fix nightly case runner](https://github.com/vllm-project/vllm-ascend/pull/14065)
- **作者**: chen-commits  **时间**: 2026-08-12 11:05 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389

### #14064 — [[BugFix] Read migrated options from additional_config in RL and LoRA guards](https://github.com/vllm-project/vllm-ascend/pull/14064)
- **作者**: MrlixiangWE  **时间**: 2026-08-12 11:05 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Two startup guards still read the deprecated environment variables directly instead of the migrated `additional_config` keys, so a user who followed the documented migration is rejected on a stale env value:  - `NPUWorker._check_nz_disabled()` reads `VLLM_ASC…

### #14063 — [[Performance][Kimi K3] Overlap QKV packing with gate projections](https://github.com/vllm-project/vllm-ascend/pull/14063)
- **作者**: q664171689  **时间**: 2026-08-12 11:04 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  This PR ports the Kimi K3 QKV multi-stream optimization from [vllm-ascend/vllm-ascend-kimi-k3#58](https://github.com/vllm-ascend/vllm-ascend-kimi-k3/pull/58) to the vLLM Ascend 0.26 release branch.  - Run Q/K/V concatenation on a process-local NPU side stream…

### #14062 — [[Refactor][Core][P/D] Delete recompute resend to Prefill nodes](https://github.com/vllm-project/vllm-ascend/pull/14062)
- **作者**: nwpu-zxr  **时间**: 2026-08-12 11:02 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Delete sending preempted request to prefill nodes for recompute in `RecomputeScheduler`, now only using `RecomputeOffloadConnector` to prevent doing prefill on decode nodes.  ### Does this PR introduce _any_ user-facing change? No.  ### How was this patch test…

### #14061 — [[BugFix] Honour additional_config.enable_transpose_kv_cache_by_block …](https://github.com/vllm-project/vllm-ascend/pull/14061)
- **作者**: MrlixiangWE  **时间**: 2026-08-12 10:56 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  The KV reformat path in `MooncakeConnector` still reads the deprecated `VLLM_ASCEND_FUSION_OP_TRANSPOSE_KV_CACHE_BY_BLOCK` environment variable directly, so `additional_config.enable_transpose_kv_cache_by_block` has no effect for this connector even though th…

### #14060 — [[CI]Run test](https://github.com/vllm-project/vllm-ascend/pull/14060)
- **作者**: shiqiangA  **时间**: 2026-08-12 10:46 CST
- **标签**: ci/build, ready
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14059 — [[Refactor][Device][3/N] Extend hardware profile policies and capabilties](https://github.com/vllm-project/vllm-ascend/pull/14059)
- **作者**: Tflowers-0129  **时间**: 2026-08-12 10:41 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Extends the hardware profile abstraction with semantic capabilities and policy enums, and centralizes the capability matrix for supported Ascend hardware families.  This allows subsequent business logic to select behavior by capability or policy instead of br…

### #14056 — [[Doc] consolidate layerwise and sparse KV cache offloading guidance](https://github.com/vllm-project/vllm-ascend/pull/14056)
- **作者**: ader47  **时间**: 2026-08-12 10:28 CST
- **标签**: documentation
- **摘要**: ## What this PR does  - Consolidates the Layerwise Prefill and Sparse Decode KV cache offloading setup into one user guide. - Adds a design document covering buffer reuse, sparse Decode offloading, connector composition, and unequal Prefill/Decode tensor parallelism. - Documents MemFabric and Memcac…

### #14054 — [pp buble optimize](https://github.com/vllm-project/vllm-ascend/pull/14054)
- **作者**: Tflowers-0129  **时间**: 2026-08-12 10:21 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14052 — [[Test][BugFix] Fix nightly file not found](https://github.com/vllm-project/vllm-ascend/pull/14052)
- **作者**: chen-commits  **时间**: 2026-08-12 09:52 CST
- **标签**: ci/build, module:tests
- **摘要**: ### What this PR does / why we need it?  This PR adds the missing configuration file `GLM-5.1-W8A8C8-A3_128k_90_50.yaml` under `tests/e2e/nightly/multi_node/internal_dp/config/` to fix the "nightly file not found" issue during multi-node end-to-end testing.  ### Does this PR introduce _any_ user-fac…
