# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-23 13:15 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的简体中文摘要：

### 🚀 Pull Request (PR) 动态

近期 PR 活动主要集中在新特性引入、Bug 修复及 CI 性能优化，具体归纳如下：

**1. 新特性与性能优化**
*   **MLA KV Pool 读去重**（#19571）：为非分层 `start_load_kv` 路径实现了 MLA KV Pool 读去重功能，避免在 TP ranks 间重复读取，提升读取效率。
*   **Triton 量化性能优化**（#19575）：在 Triton 量化中引入倒数乘法以提升性能（目前为草稿状态，等待 NPU 性能和精度验证）。
*   **Oscar 系列堆叠切片更新**（#19568, #19569, #19570）：提交了 3 个独立的 PR（系列 2/22, 3/22, 4/22），分别引入了纯 Torch 实现的 INT2 KV 编码器、旋转资产加载器以及 INT2 launcher 扩展源码。

**2. Bug 修复**
*   **STAIR 负载统计类型修复**（#19572, #19574）：修复了 EPLB 中 STAIR 时序分箱隐式将 int32 负载计数提升为 int64 的问题，避免了统计数据负载翻倍及其对 AllReduce 的影响。其中 #19574 为向 `releases/v0.30.0` 的回溯。
*   **多模态重算调度修复**（#19573）：修复了多模态请求在 `RecomputeScheduler.schedule()` 中因解包值过多导致的 `ValueError`，保留了重复的编码器输入。
*   **MRV2 块表保留修复**（#19567）：修复了请求在发生抢占后通过 MRV2 恢复时，AscendStore 未能正确保留其块表的问题（向 `releases/v0.30.0` 的回溯）。

**3. CI 与测试**
*   **单卡 KV Pool 精度检查**（#19576）：为带有 memcache 后端的 AscendStoreConnector 增加了单卡 PR 级别的精度防护，支持参数化非分层和分层模式。

---

### 📝 Issue 动态
*   近期未提供相关的 Issue 动态。

---

### 📦 Release 动态
*   近期未发布新的 Release 版本。

---

## 🔀 Pull Requests

### #19576 — [[CI] Add single-card KV pool accuracy checks](https://github.com/vllm-project/vllm-ascend/pull/19576)
- **作者**: weijinqian0  **时间**: 2026-10-23 12:56 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Add a single-card PR-level accuracy guard for AscendStoreConnector with the memcache backend. Parameterize non-layerwise and layerwise modes, start an uncached baseline and a KV-pool-enabled server, and require identical generated text for a deterministic Qwe…

### #19575 — [[Performance] Use reciprocal multiplication in Triton quantization](https://github.com/vllm-project/vllm-ascend/pull/19575)
- **作者**: zzy-6666  **时间**: 2026-10-23 12:54 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  This is a draft PR because NPU performance results and model-level accuracy validation are still pending.  Inspired by [vllm-project/vllm#61343](https://github.com/vllm-project/vllm/pull/61343), this PR applies reciprocal-multiply optimization to Ascend Trito…

### #19574 — [[BugFix][EPLB][v0.30.0] Preserve int32 load statistics in STAIR bins](https://github.com/vllm-project/vllm-ascend/pull/19574)
- **作者**: freyfwt  **时间**: 2026-10-23 12:32 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Backport of #19572 to `releases/v0.30.0`.  STAIR's temporal bin reduction implicitly promotes int32 load counters to int64, doubling the statistics payload and introducing substantial AllReduce overhead on Ascend. Preserve the input dtype in `sum` and reuse t…

### #19573 — [[BugFix] Preserve duplicate encoder inputs in recompute scheduling](https://github.com/vllm-project/vllm-ascend/pull/19573)
- **作者**: foraxe  **时间**: 2026-10-23 12:28 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  With the main-pinned vLLM API, a multimodal request can terminate `RecomputeScheduler.schedule()` with `ValueError: too many values to unpack (expected 4)`. `_try_schedule_encoder_inputs` returns a fifth value, `duplicate_encoder_inputs`, while both waiting a…

### #19572 — [[BugFix][EPLB] Preserve int32 load statistics in STAIR bins](https://github.com/vllm-project/vllm-ascend/pull/19572)
- **作者**: freyfwt  **时间**: 2026-10-23 12:27 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  STAIR's temporal bin reduction implicitly promotes int32 load counters to int64. The promoted dtype then propagates to logical-load aggregation and AllReduce, doubling the statistics payload and introducing substantial reduction overhead on Ascend.  Pass the …

### #19571 — [[Feature][KV Pool] Add MLA KV pool read deduplication](https://github.com/vllm-project/vllm-ascend/pull/19571)
- **作者**: Suixing5208  **时间**: 2026-10-23 12:22 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Refs #14140.  This PR implements MLA KV Pool read deduplication for the non-layerwise `start_load_kv` path. When the MLA KV cache is replicated across TP ranks and `put_step == tp_size`, each rank reads a disjoint subset of block keys from the AscendStore bac…

### #19570 — [oscar: vendor the INT2 launcher extension sources (r152 fast + r98 bridge) [4/22]](https://github.com/vllm-project/vllm-ascend/pull/19570)
- **作者**: nqz1997  **时间**: 2026-10-23 12:15 CST
- **标签**: documentation
- **摘要**: ## Summary  Standalone slice of stacked series [4/22] — **no dependency on other PRs**; pure vendored sources (3 new files, 1451 insertions: 2 C++ launcher sources + README; no build wiring in this slice, no runtime effect until the bridge lands in 13/22). Directly on `main`. Verification: file-leve…

### #19569 — [oscar: add rotation assets loader with the full three-leg contract [3/22]](https://github.com/vllm-project/vllm-ascend/pull/19569)
- **作者**: nqz1997  **时间**: 2026-10-23 12:15 CST
- **标签**: module:tests
- **摘要**: ## Summary  Standalone slice of stacked series [3/22] — **no dependency on other PRs**; minimal diff (2 files: module + tests) directly on `main`. The +12 rotation-config lines from the stacked version are intentionally NOT here (they extend `vllm_ascend/oscar/config.py`, introduced by 1/22) — they …

### #19568 — [oscar: add the INT2 KV encoder (per-token-head affine, pure Torch) [2/22]](https://github.com/vllm-project/vllm-ascend/pull/19568)
- **作者**: nqz1997  **时间**: 2026-10-23 12:15 CST
- **标签**: module:tests
- **摘要**: ## Summary  Standalone slice of stacked series [2/22] — **no dependency on other PRs**; minimal diff (2 files: module + tests) directly on `main`. Relation to #19552 (1/22, based on `releases/v0.22.1rc`): this encoder is independent of the config surface and is submitted on `main` because the releas…

### #19567 — [[BugFix][KVPool][v0.30.0] Preserve MRV2 block tables after preemption](https://github.com/vllm-project/vllm-ascend/pull/19567)
- **作者**: bowgneo  **时间**: 2026-10-23 12:08 CST
- **标签**: module:tests, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  MRV2 sends a request resumed after preemption through `scheduled_new_reqs` with its full block table. AscendStore rebuilds the tracker there but leaves the request in `_preempted_req_ids`. The next RUNNING step supplies only newly allocated blocks and is inco…
