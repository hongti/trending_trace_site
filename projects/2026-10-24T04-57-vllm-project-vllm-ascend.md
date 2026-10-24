# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-24 12:57 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
* **#19686 夜间测试失败告警**：2026-10-24 的 Daily Nightly 测试发生失败。失败环境为 a2-1 卡片，失败用例集中在 `test_npu_ipc_weight_transfer_transac`，需相关同学排查修复。

### 🔀 Pull Request 动态
近期 PR 主要围绕**性能优化、Bug 修复、新特性支持及 CI 构建改进**，具体如下：

**1. 新特性**
* **#19682 增加 DSpine 投机解码支持**：为 Ascend NPU (Model Runner V1) 引入了 DSpine 投机解码能力，这是一种 DFlash 家族的草稿方案。

**2. 性能优化**
* **#19691 优化 Grouped BF16 DSQ 跨核负载均衡**：修复了 `DequantSituQuant` 中每个专家均从 core 0 重新开始行分区导致的负载不均问题，提升了多核利用率。
* **#19685 融合 DCP FlashMLA query 与 KV 准备操作**：引入一个 strided Triton 预处理内核，将原本的两次拷贝和 KV scatter 操作打包融合，减少了数据搬运开销。
* **#19684 Qwen38 MTP 性能修复**：针对 Qwen38 的多 Token 预测进行了性能修复。

**3. Bug 修复**
* **#19692 调度器编码器缓存大小修正**：按单请求驻留的最坏情况调整编码器缓存，修复了上游将缓存大小限制在单个最大多模态项时可能引发的问题。
* **#19683 修复 KVTransfer 无效块 ID 并发更新问题**：修复了 #14150。通过在共享锁下序列化无效 KV 块 ID 的更新，解决了 pool worker 与三个传输线程之间的并发安全问题。

**4. CI 与构建维护**
* **#19690 优化 CI 并行测试**：改进 A2 夜间测试流程，单节点和精度测试不再等待漫长的多节点日志流阶段完成，只要多节点 pod 准备就绪即可并行启动，大幅节省 CI 时间。
* **#19687 / #19689 更新依赖与路径**：将所有 A2/A3/A5 Dockerfile（含 Ubuntu 和 openEuler）中的 `memcache_hybrid` 和 `memfabric_hybrid` wheels 更新至 `20261024.1`，并将 OBS 路径更新至 `release/1.3.0`。
* **#19688 向后移植**：将上述依赖与路径更新 Backport 至 `releases/v0.30.0` 分支。

### 🚀 Release 动态
* 本段时间内未发布新版本 Release。

---

## 🐛 Issues

### #19686 — [Nightly Daily test failure on 2026-10-24 08:00 - please check](https://github.com/vllm-project/vllm-ascend/issues/19686)
- **作者**: czydyy  **时间**: 2026-10-24 10:01 CST
- **摘要**: The nightly Daily test for vllm-ascend reported failures at 08:00 on 2026-10-24.  **Failed on:** a2-1 card-(part 3-7), a2-1 card-(part 6-7)  **Failed test cases:** - test_npu_ipc_weight_transfer_transaction — requests.exceptions.ConnectionError: HTTPSConnectionPool(host='www.modelscope.ai', port=443…

## 🔀 Pull Requests

### #19692 — [[BugFix][Scheduler] Size the encoder cache by worst-case per-request residency](https://github.com/vllm-project/vllm-ascend/pull/19692)
- **作者**: Sfeching  **时间**: 2026-10-24 12:30 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Upstream floors the scheduler's encoder cache at the largest single multimodal item (`compute_mm_encoder_budget`): both the encoder compute budget and the cache capacity are floored at `max_tokens_per_item` (see `vllm/v1/core/encoder_cache_manager.py`).  A re…

### #19691 — [[Performance][codex] Balance grouped BF16 DSQ rows across cores](https://github.com/vllm-project/vllm-ascend/pull/19691)
- **作者**: foraxe  **时间**: 2026-10-24 12:26 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Grouped BF16 `DequantSituQuant` restarts row partitioning at core 0 for every expert. With fourteen singleton groups and a 48-block launch, core 0 processes all fourteen rows serially even though the rows are independent.  Assign grouped BF16 rows by their gl…

### #19690 — [[CI] start single-node tests in parallel once multi-node pods are ready](https://github.com/vllm-project/vllm-ascend/pull/19690)
- **作者**: CuixyHW  **时间**: 2026-10-24 12:13 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it?  Currently the A2 nightly single-node and accuracy jobs wait for the entire `multi-node-tests` job (including the long Stream logs phase) to finish before starting, which serializes independent workloads and wastes time. This change adds an A2-only readiness g…

### #19689 — [[CI][Misc] Update memcache/memfabric to 20261024.1 and OBS path to release/1.3.0](https://github.com/vllm-project/vllm-ascend/pull/19689)
- **作者**: zhangdiago  **时间**: 2026-10-24 10:46 CST
- **摘要**: ### What this PR does / why we need it?  This PR updates the pinned `memcache_hybrid` and `memfabric_hybrid` wheels in all A2/A3/A5 Dockerfiles (both Ubuntu and openEuler variants) to version `20261024.1` (previously `20261022.5`).  Additionally, it refactors the OBS path segment in `MEMFABRIC_URL` …

### #19688 — [[CI][Misc][v0.30.0] Update memcache/memfabric to 20261024.1 and OBS path to release/1.3.0](https://github.com/vllm-project/vllm-ascend/pull/19688)
- **作者**: zhangdiago  **时间**: 2026-10-24 10:19 CST
- **摘要**: ### What this PR does / why we need it?  Backport of #19689 to `releases/v0.30.0`.  This PR updates the pinned `memcache_hybrid` and `memfabric_hybrid` wheels in all A2/A3/A5 Dockerfiles (both Ubuntu and openEuler variants) to version `20261024.1` (previously `20261022.5`).  Additionally, it refacto…

### #19687 — [[CI][Build] Update memcache/memfabric to 20261024.1 and OBS path to release/1.3.0](https://github.com/vllm-project/vllm-ascend/pull/19687)
- **作者**: zhangdiago  **时间**: 2026-10-24 10:18 CST
- **摘要**: ## What this PR does / why we need it?  Updates the pinned `memcache_hybrid` / `memfabric_hybrid` wheels in all A2/A3/A5 Dockerfiles (Ubuntu and openEuler variants):  - `MEMCACHE_DATE` / `MEMFABRIC_DATE`: `20261022.5` → `20261024.1` - OBS path segment in `MEMFABRIC_URL` / `MEMCACHE_URL`: hardcoded `…

### #19685 — [[Performance] Fuse DCP FlashMLA query and current KV preparation](https://github.com/vllm-project/vllm-ascend/pull/19685)
- **作者**: MAyang38  **时间**: 2026-10-24 09:23 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  DCP FlashMLA stages history Q with two copies and current inputs with a KV scatter followed by two Q copies. Add a strided Triton preparation kernel that packs history Q in one launch, and packs current Q while scattering current KV in one launch. Read source…

### #19684 — [Perf/qwen38 mtp fix](https://github.com/vllm-project/vllm-ascend/pull/19684)
- **作者**: sunny-rain-63  **时间**: 2026-10-24 09:16 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization, module:tools, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…

### #19683 — [[BugFix][KVTransfer] Serialize invalid KV block ID updates under the shared lock](https://github.com/vllm-project/vllm-ascend/pull/19683)
- **作者**: wisdom1710  **时间**: 2026-10-24 07:39 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Fixes #14150.  `KVPoolWorker._invalid_block_ids` is shared (same `set` object, same `Lock` object) between the pool worker and the three transfer threads. The worker records block IDs whose KV load failed, and the model runner drains them once per forward pas…

### #19682 — [[Feat][SpecDecode] Add DSpine speculative decoding support](https://github.com/vllm-project/vllm-ascend/pull/19682)
- **作者**: 982945902  **时间**: 2026-10-24 01:24 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Adds support for **DSpine** speculative decoding on Ascend NPU (Model Runner V1). DSpine is a DFlash-family draft scheme whose checkpoint advertises `DFlashDraftModel` plus a `selected_prefix_code_bridge` config; compared to DFlash it adds per-layer selected-…
