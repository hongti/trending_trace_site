# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-24 18:18 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 最近动态的中文摘要：

### 📦 Release
*本次提供的数据中未包含 Release 相关动态。*

### 💬 Issue
*本次提供的数据中未包含 Issue 相关动态。*

### 🔨 Pull Request (PR)
本次共有 9 个 PR，主要涵盖新特性集成、性能优化、缺陷修复及日常维护：

**🚀 新特性与集成**
*   **Kimi K3.1 解码集成**（#23054）：在 Ascend 主分支集成了 localTP、C8 草稿预处理、KDA 提交及 MegaKDA/MegaDSA 解码等特性，针对 A5、DP64、TP1 等环境进行了性能优化。
*   **EPLB 负载记录融合**（#23056）：在 A3/V2 融合 MC2 路径上启用 EPLB 记录时，将负载记录操作融合进 MoE 计算（`dispatch_ffn_combine`）中，避免了单独启动 Record 操作带来的开销。

**⚡ 性能优化**
*   **AscendStore KV 传输优化**（#23060）：为层级 AscendStore 加载新增可选调度器查找路径，用于在各 worker 间去重公共 KV 键并获取租约，减少冗余查询。
*   **Mooncake 传输优化**（#23058）：当 DCP stripe 无本地块时，跳过空的 Mooncake WRITE 任务并推迟 DONE 等待，提升 KV 传输效率。

**🐛 缺陷修复**
*   **DeepSeek V4.1 Engram 形状校验**（#23059）：修复了本地加载器在源 scale 张量形状为 `[4, 1]` 时，将单个 scale 错误复制到每组行的问题。现已在分配表前加入形状校验。
*   **原生批量求和回退修复**（#23055）：修复了非位打包 DeepSeek-V4.1 索引器在有效 token 数超 524,288 时，将 INT32 目标传给 `torch.sum` 导致的输出缓冲区问题。

**🛠️ 维护与其他**
*   **PD 分离式推理日志增强**（#23063）：在 PD 分离式服务中，当 Decode 端无法分配 KV 块导致请求被退回 Prefill 端重算时，新增了重计算路径的日志记录，便于追踪排查。
*   **调度器代码同步**（#23057）：将 Ascend 本地复制的调度器（`BalanceScheduler`, `DyntraLbscheduler`, `RecomputeScheduler`）与上游 vLLM 代码进行同步。
*   **自动化测试**（#23061, #23062）：创建了用于 nightly 验证的自动化空 PR。

---

## 🔀 Pull Requests

### #23063 — [[Misc] Add recompute-path logging for PD disaggregated serving](https://github.com/vllm-project/vllm-ascend/pull/23063)
- **作者**: Anndy965  **时间**: 2026-11-24 18:04 CST
- **摘要**: ### What this PR does / why we need it? In PD disaggregated serving, when the decode (D) side cannot allocate KV blocks, the request is aborted and sent back to the prefill (P) side to be recomputed. Today this path is almost silent: the client only sees a normal completion, and the proxy sees a sin…

### #23062 — [[Verify] Nightly test on main](https://github.com/vllm-project/vllm-ascend/pull/23062)
- **作者**: LQDLove  **时间**: 2026-11-24 17:51 CST
- **标签**: documentation
- **摘要**: Auto-created empty PR for nightly verification. - vLLM main: https://github.com/vllm-project/vllm/commit/c410543d9ffff03f5f6c4a8c0ae1d3ad082c4a15

### #23061 — [[Verify] Nightly test on main](https://github.com/vllm-project/vllm-ascend/pull/23061)
- **作者**: LQDLove  **时间**: 2026-11-24 17:50 CST
- **标签**: documentation
- **摘要**: Auto-created empty PR for nightly verification. - vLLM main: https://github.com/vllm-project/vllm/commit/c410543d9ffff03f5f6c4a8c0ae1d3ad082c4a15

### #23060 — [[Perf][KV Transfer] Add opt-in scheduler lookup for layerwise AscendStore loads](https://github.com/vllm-project/vllm-ascend/pull/23060)
- **作者**: bowgneo  **时间**: 2026-11-24 17:38 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  MemCache layerwise loads can query the same public KV keys and acquire leases on each worker. This draft adds an opt-in scheduler path that deduplicates public keys across requests in a step, acquires read leases, and queries the pinned global virtual address…

### #23059 — [[BugFix] Validate DeepSeek V4.1 Engram checkpoint shapes before table allocation](https://github.com/vllm-project/vllm-ascend/pull/23059)
- **作者**: brandonRong  **时间**: 2026-11-24 17:37 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  On `ed07ebefa`, the local DeepSeek V4.1 Engram loader copied a single scale into every group of a row when the source scale tensor had shape `[4, 1]` and the destination had shape `[4, 8]`. A `[4, 7]` source also passed preflight, then failed during the copy …

### #23058 — [[Performance][KV Transfer] Skip empty Mooncake push tasks and defer DONE waits](https://github.com/vllm-project/vllm-ascend/pull/23058)
- **作者**: nwpu-zxr  **时间**: 2026-11-24 17:34 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  MooncakeConnectorV2 layerwise push submits a WRITE task even when a DCP stripe has no local blocks. That task only waits on a device event and returns without transferring data. The model step also waits for all DONE/ACK exchanges after the actual WRITEs have…

### #23057 — [[Misc][Scheduler] Sync Ascend copied schedulers with upstream vLLM](https://github.com/vllm-project/vllm-ascend/pull/23057)
- **作者**: wei-yubo  **时间**: 2026-11-24 17:34 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Closes #22967  #### Background & Motivation  `BalanceScheduler`, `DyntraLBScheduler`, and `RecomputeScheduler` maintain local copies of upstream `Scheduler.schedule()`. Those copies had drifted behind the pinned vLLM main commit (`c410543d9ffff03f5f6c4a8c0ae1…

### #23056 — [[Feature] Fuse EPLB load recording into A3 fused MC2](https://github.com/vllm-project/vllm-ascend/pull/23056)
- **作者**: Liamup777  **时间**: 2026-11-24 17:24 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  When EPLB recording is enabled on the supported A3/V2 fused MC2 path, `dispatch_ffn_combine` currently finishes MoE computation and then launches a separate Record operator to accumulate the local experts' token counts. This draft extends the W8A8, W4A8, and …

### #23055 — [[BugFix] Allow output buffers on native batch-invariant sum fallbacks](https://github.com/vllm-project/vllm-ascend/pull/23055)
- **作者**: ttqiao1617  **时间**: 2026-11-24 17:19 CST
- **标签**: module:tests, module:core, ci-run
- **摘要**: ### What this PR does / why we need it?  The non-bitpacked DeepSeek-V4.1 indexer switches to a separate valid-token count above 524,288 elements. It passes an INT32 destination to `torch.sum`. With batch invariance enabled, the AscendC wrapper raises `TypeError` before writing lengths.  ### Does thi…

### #23054 — [[Feature][Kimi] Integrate localTP, C8 draft preprocessing, KDA commit and MegaKDA/MegaDSA decode](https://github.com/vllm-project/vllm-ascend/pull/23054)
- **作者**: Dawn952  **时间**: 2026-11-24 17:12 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  Integrate the Kimi K3.1 decode changes on Ascend main. The integration started at `b776261e32`. The performance scope below is **A5, DP64, TP1, four concurrent requests per NPU**.  | Change | Implementation and dispatch | | --- | --- | | **LocalTP weight shar…
