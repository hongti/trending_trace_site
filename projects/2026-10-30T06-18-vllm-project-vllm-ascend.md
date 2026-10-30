# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-30 14:18 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期的动态摘要：

### 🐛 Issue 动态
- **DeepSeek-V4.1-Flash-1017 部署死锁问题 (#20329)**
  有用户反馈在 A5 (950DDT) 环境部署 DeepSeek-V4.1-Flash-1017 时，一旦请求输出长度超过 200 token，就会触发进程死锁。

### 🔀 Pull Request 动态
**1. 新特性与性能优化**
- **Qwen3.8-Next 模型支持优化**：
  - 新增 PLE/ngram embedding 的 host offload 功能，支持 BF16 精度，优化显存使用 (#20326)。
  - 提升 Qwen3.8 Flash Next MRV2 的解码性能：允许 CPU 在前一次迭代仍在运行时，提前提交下一次解码迭代并捕获目标 metadata，减少 CPU 等待时间 (#20327)。

**2. Bug 修复**
- **Triton RoPE 空维度报错**：修复了 NoPE 层（如 `qk_rope_head_dim == 0`）调用 Triton RoPE 时因计算 `next_power_of_2(0)` 导致的问题，现在会直接跳过 (#20331)。
- **大 logprobs 请求处理**：将大请求拆分为 `BLOCK_SIZE` 块进行处理，而不是使用单一向量，修复了相关问题 (#20325)。
- **KVPool 参数报错**：移除了 `KVPoolWorker` 中已废弃的 `memcache_dp_init_barrier` 参数传递 (#20321)。
- **KV Transfer 端点隔离修复回移**：将 dense DP replica endpoints 的隔离修复 Cherry-pick 到 v0.30.0 发布分支 (#20324)。

**3. CI 与文档更新**
- **CI 与依赖更新**：更新了 A5 环境中的 cann、fla、cannbot 等依赖，并在 Dockerfile 中添加了版本打印信息 (#20330)。
- **文档维护**：移除了 DeepSeek-V4-Flash 教程中过时的 MTP 部署示例 (#20323)，并更新了依赖版本匹配说明 (#20322)。
- **日常测试**：创建了用于 nightly 验证的空 PR (#20328)。

### 🚀 Release 动态
- **v0.30.0 稳定版发布 (2026.10.29)**
  - **版本亮点**：vLLM Ascend v0.30.0 正式稳定版发布。该版本基于 v0.29 发行线构建，并与上游 **vLLM v0.30.0** 版本完全对齐，为 Ascend 硬件用户提供最新稳定的推理服务支持。

---

## 🐛 Issues

### #20329 — [[Bug]: DeepSeek-V4.1-Flash-1017在A5(950DDT)部署，请求输出长度超出200token就会进程死锁](https://github.com/vllm-project/vllm-ascend/issues/20329)
- **作者**: JiangnanRomeo  **时间**: 2026-10-30 13:55 CST
- **标签**: bug, llm-model, deepseek
- **摘要**: ### Your current environment  Collecting environment information... PyTorch version: 2.10.0+cpu Is debug build: False  OS: Ubuntu 22.04.5 LTS (aarch64) GCC version: (Ubuntu 11.4.0-1ubuntu1~22.04.3) 11.4.0 Clang version: 15.0.7 CMake version: version 4.4.4 Libc version: glibc-2.35  Python version: 3.…

## 🔀 Pull Requests

### #20331 — [[BugFix][Ops] Skip Triton RoPE when rope_dim is zero](https://github.com/vllm-project/vllm-ascend/pull/20331)
- **作者**: holykie  **时间**: 2026-10-30 14:17 CST
- **摘要**: ### What this PR does / why we need it? NoPE layers (e.g. `qk_rope_head_dim == 0`) call `rope_forward_triton` with `rope_dim == 0`. The host wrapper computes `pad_rope_dim = triton.next_power_of_2(0) == 0`, and the kernel then evaluates `tl.arange(0, 0)`, raising:  ValueError: arange's end argument …

### #20330 — [[CI] update cann, fla, cannbot in a5, commen on triton, torch-npu, memcache, memfabric, add version print in Dockerfile main](https://github.com/vllm-project/vllm-ascend/pull/20330)
- **作者**: xqchen7  **时间**: 2026-10-30 14:12 CST
- **标签**: image-build
- **摘要**: ### What this PR does / why we need it? update cann, fla, cannbot in a5, commen on triton, torch-npu, memcache, memfabric, add version print in Dockerfile  ### Does this PR introduce _any_ user-facing change? NA  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / ed…

### #20328 — [[Verify] Nightly test on main](https://github.com/vllm-project/vllm-ascend/pull/20328)
- **作者**: LQDLove  **时间**: 2026-10-30 13:29 CST
- **标签**: documentation
- **摘要**: Auto-created empty PR for nightly verification. - vLLM main: https://github.com/vllm-project/vllm/commit/544f182d9f0c9bfe8bd7645525951d5e3692a542

### #20327 — [[Performance][MRV2] Queue steady Qwen decode and capture target metadata](https://github.com/vllm-project/vllm-ascend/pull/20327)
- **作者**: Hyx098130  **时间**: 2026-10-30 12:49 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Allow the CPU to submit the next Qwen3.8 Flash Next MRV2 decode iteration while the preceding iteration is still running. The current path waits on the CPU for a corrected-length snapshot before preparing the next batch, and a sparse capture grid can leave QS…

### #20326 — [[Feat][Model] Add Qwen3.8-Next PLE host offload](https://github.com/vllm-project/vllm-ascend/pull/20326)
- **作者**: Karryking3  **时间**: 2026-10-30 12:42 CST
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Add opt-in BF16 PLE/ngram embedding host offload for Qwen3.8-Next (Qwen4Exp) on Ascend, based on current main after #16981.  Keep vocabulary shards in CANN-registered host memory and gather the selected rows directly on NPU with a masked Triton kernel. Preser…

### #20325 — [[Bugfix][Sample] Walk large logprobs requests in blocks instead of one vector](https://github.com/vllm-project/vllm-ascend/pull/20325)
- **作者**: tgrhn  **时间**: 2026-10-30 12:31 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Fixes #19352.  `_topk_log_softmax_kernel` scans the vocabulary in `BLOCK_SIZE` chunks twice, then gathers the requested token ids in a single vector sized `next_power_of_2(topk)`. That vector must fit unified buffer, so once a caller asks for more than a few …

### #20324 — [[Cherry-pick][releases/v0.30.0][BugFix][KV Transfer] Isolate dense DP replica endpoints (from #19744)](https://github.com/vllm-project/vllm-ascend/pull/20324)
- **作者**: ckeyever  **时间**: 2026-10-30 12:12 CST
- **标签**: documentation, module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  Backport #19744 (b006d843bcd1f5a75d5d7299a2c9dc10192ffba5) to `releases/v0.30.0`.  Dense-model engines reset their compute `data_parallel_rank` to zero while preserving the serving `data_parallel_index`. Using the compute rank for connector endpoints makes in…

### #20323 — [[Doc] Remove obsolete MTP deployment examples from DeepSeek-V4-Flash tutorial](https://github.com/vllm-project/vllm-ascend/pull/20323)
- **作者**: pgzddxx  **时间**: 2026-10-30 12:02 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Remove the four obsolete A2/A3 MTP deployment tabs from Sections 5.1 and 5.2 of the DeepSeek-V4-Flash tutorial. Keep the existing DSpark examples and rename their tabs to `A2 series` / `A3 series`.  Remove stale MTP weight, image, model-path and comparison re…

### #20322 — [[Doc] Update the version matching information in the document](https://github.com/vllm-project/vllm-ascend/pull/20322)
- **作者**: MrZ20  **时间**: 2026-10-30 11:57 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Adjust the supporting dependency relationships of the version.  Due to the abnormal CI domain name `cache-service.nginx-pypi-cache.svc.cluster.local`, the installation check is temporarily disabled, and this issue will be fixed later. ### Does this PR introduc…

### #20321 — [[BugFix][KVPool] Remove stale Memcache init barrier arguments](https://github.com/vllm-project/vllm-ascend/pull/20321)
- **作者**: LifeOfCoffee  **时间**: 2026-10-30 11:52 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  `KVPoolWorker.__init__` no longer accepts `memcache_dp_init_barrier`, but the AscendStore factory still forwards it as a keyword and `LayerwiseKVPoolWorker` still forwards it positionally. Bulk construction therefore raises an unexpected-keyword `TypeError`, …

## 🚀 Releases

### [v0.30.0](https://github.com/vllm-project/vllm-ascend/releases/tag/v0.30.0)
- **作者**: zzzzwwjj  **时间**: 2026-10-29 22:07 CST
- **摘要**: ## v0.30.0 - 2026.10.29  This is the stable release of vLLM Ascend v0.30.0, built on the v0.29 release line and aligned with upstream vLLM v0.30.0 via the main2main pipeline. Building on v0.30.0rc1, this release integrates FlashMLA decode with DCP and DSpark, extends KV Pool layerwise pooling and PC…
