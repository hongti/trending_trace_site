# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-30 09:01 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📌 Issue（问题）
- **CI 自动化流程故障**：`main2main` 自动化流程连续两次（#13126, #13125）在完成所有计划步骤前中断，状态为 `failed`，目前均需要人工介入审查。

### 🚀 Pull Request（拉取请求）
PR 活动主要集中在关键 Bug 修复、算子新增与性能优化，并大量涉及向各发布分支的代码前向移植：

**1. 核心缺陷修复**
- **AscendStore 保存与异步加载修复**（#13115 - #13120）：
  - **恢复同步保存**：修复了此前非分层保存改为异步后引入的问题，在 `AscendStore` 的 `wait_for_save()` 中恢复了 `request_queue.join()` 屏障。
  - **异步 KV 加载失败上报**：修复了 `load_async=true` 时，`KVCacheStoreRecvingThread` 缺少 worker 共享无效块状态导致异步 KV 加载失败无法上报给调度器的问题。
  - *注：以上两项修复已被同步移植至 `main`、`v0.24.0rc` 和 `v0.25.1rc` 分支。*
- **AscendInputBatch Token 分布修复**（#13123）：修复了 `make_dummy` 中的 token 分布逻辑，使其与上游 vLLM 实现对齐，解决了剩余 token 分配问题。

**2. 新特性与算子支持**
- **新增 Ascend 310P 算子**（#13122）：为 Ascend 310P NPU 平台引入了自定义 AscendC 算子实现 `ChunkGatedDeltaRuleComputeWy`。
- **Dspark 修复**（#13124）：修复了 Dspark 中 sparse cuda capturing 的问题。

**3. 性能优化**
- **移除过时的 GDN prefill 元数据**（#13121）：将 Qwen3.5 GDN prefill 切换至 `aclnn chunk_gated_delta_rule` 算子，移除了仅为旧版 Triton 实现保留的元数据构建逻辑，提升了执行效率。

### 📦 Release（发布）
- 近期**无全新版本发布**，但多项关键 Bug 修复（AscendStore 同步保存恢复、异步 KV 加载失败上报）已作为重要补丁，被集中前向移植至 **`v0.24.0rc`** 和 **`v0.25.1rc`** 这两个候选发布分支，以保障后续版本的稳定性。

---

## 🐛 Issues

### #13126 — [[main2main] main2main manual review required ()](https://github.com/vllm-project/vllm-ascend/issues/13126)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-30 03:35 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR:  - Commit range: `d02df748bf9efd99022f1a062597dc3cb3808485`...`` - Status: `failed`  ## Final Summary  main2main adaptation failed — no steps completed.

### #13125 — [[main2main] main2main manual review required ()](https://github.com/vllm-project/vllm-ascend/issues/13125)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-30 00:34 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR:  - Commit range: `d02df748bf9efd99022f1a062597dc3cb3808485`...`` - Status: `failed`  ## Final Summary  main2main adaptation failed — no steps completed.

## 🔀 Pull Requests

### #13124 — [Dspark full fixed sparse cuda capturing ](https://github.com/vllm-project/vllm-ascend/pull/13124)
- **作者**: StanislavII  **时间**: 2026-07-29 23:57 CST
- **摘要**: - vLLM version: v0.25.1 - vLLM main: https://github.com/vllm-project/vllm/commit/d02df748bf9efd99022f1a062597dc3cb3808485

### #13123 — [[MRV2][BugFix] Fix token distribution in make_dummy for AscendInputBatch](https://github.com/vllm-project/vllm-ascend/pull/13123)
- **作者**: zouzy5137  **时间**: 2026-07-29 22:44 CST
- **摘要**: ### What this PR does / why we need it?  This PR aligns the dummy input generation logic in `AscendInputBatch.make_dummy` with the upstream vLLM implementation. Previously, any remaining tokens (`num_tokens % num_reqs`) were accumulated entirely onto the last request. This PR distributes the extra t…

### #13122 — [[Ops][Feature] Add chunk_gated_delta_rule_compute_wy operator for Ascend 310P- #11941](https://github.com/vllm-project/vllm-ascend/pull/13122)
- **作者**: vladimirevmenoff  **时间**: 2026-07-29 22:30 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? This PR introduces a custom AscendC operator implementation for `ChunkGatedDeltaRuleComputeWy`, specifically targeting the Ascend 310P NPU platform. This operator speeds up the prefill phase by 40%.  **Why the changes are needed:** Previously, this specific co…

### #13121 — [[Performance][Ops] Remove obsolete GDN prefill metadata](https://github.com/vllm-project/vllm-ascend/pull/13121)
- **作者**: shaopeng-666  **时间**: 2026-07-29 22:08 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  This PR switches Qwen3.5 GDN prefill to the aclnn `chunk_gated_delta_rule` operator and removes the metadata construction that existed only for the old Triton chunk pipeline.  Related to vllm-project/vllm-ascend#12607.  - Remove host `cu_seqlens`, chunk indic…

### #13120 — [[BugFix] Restore synchronous AscendStore save](https://github.com/vllm-project/vllm-ascend/pull/13120)
- **作者**: Pz1116  **时间**: 2026-07-29 21:54 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it?  Forward-port #13024 to `main`.  Restore the `request_queue.join()` barrier in AscendStore `wait_for_save()`.  #12783 made non-layerwise saves asynchronous. Its full rollback together with #12819 restores the expected DeepSeek V4 prefix hit length, but that ro…

### #13119 — [[v0.25.1rc][BugFix] Restore synchronous AscendStore save](https://github.com/vllm-project/vllm-ascend/pull/13119)
- **作者**: Pz1116  **时间**: 2026-07-29 21:54 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it?  Forward-port #13024 to `releases/v0.25.1rc`.  Restore the `request_queue.join()` barrier in AscendStore `wait_for_save()`.  #12783 made non-layerwise saves asynchronous. Its full rollback together with #12819 restores the expected DeepSeek V4 prefix hit lengt…

### #13118 — [[v0.24.0rc][BugFix] Restore synchronous AscendStore save](https://github.com/vllm-project/vllm-ascend/pull/13118)
- **作者**: Pz1116  **时间**: 2026-07-29 21:54 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it?  Forward-port #13024 to `releases/v0.24.0rc`.  Restore the `request_queue.join()` barrier in AscendStore `wait_for_save()`.  #12783 made non-layerwise saves asynchronous. Its full rollback together with #12819 restores the expected DeepSeek V4 prefix hit lengt…

### #13117 — [[v0.25.1rc][BugFix] Report async KV load failures to scheduler](https://github.com/vllm-project/vllm-ascend/pull/13117)
- **作者**: Pz1116  **时间**: 2026-07-29 21:54 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it?  Forward-port #13099 to `releases/v0.25.1rc`.  When AscendStore runs with `load_async=true`, `KVCacheStoreRecvingThread` was created without the worker's shared invalid-block set and lock. The receive thread therefore recorded backend load failures in a privat…

### #13116 — [[BugFix] Report async KV load failures to scheduler](https://github.com/vllm-project/vllm-ascend/pull/13116)
- **作者**: Pz1116  **时间**: 2026-07-29 21:54 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it?  Forward-port #13099 to `main`.  When AscendStore runs with `load_async=true`, `KVCacheStoreRecvingThread` was created without the worker's shared invalid-block set and lock. The receive thread therefore recorded backend load failures in a private set, while `…

### #13115 — [[v0.24.0rc][BugFix] Report async KV load failures to scheduler](https://github.com/vllm-project/vllm-ascend/pull/13115)
- **作者**: Pz1116  **时间**: 2026-07-29 21:52 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it?  Forward-port #13099 to `releases/v0.24.0rc`.  When AscendStore runs with `load_async=true`, `KVCacheStoreRecvingThread` was created without the worker's shared invalid-block set and lock. The receive thread therefore recorded backend load failures in a privat…
