# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-20 13:10 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🐛 Issue 动态
- **Qwen3.8-27B 启动失败 Bug (#19241)**：
  报告了在单节点 16 NPUs 环境下部署 Qwen3.8-27B 时，若启用前缀缓存（`--enable-prefix-caching`）并设置 `VLLM_PREFIX_CACHE_RETENTION_INTERVAL=4096`，会因为该值不是混合 Mamba 调度器块大小（1536）的整数倍而导致模型启动失败。

### 🔧 PR 动态
**新特性与优化**
- **AscendStoreConnector 支持按组 KV 块计数 (#19249)**：使 KVPoolWorker 兼容 window-capped 按组内存池，改变了原先假设所有 KV cache 张量使用统一块计数的逻辑。
- **优化 NPU RoPE 索引操作 (#19247)**：改为从连续的平面基底获取 RoPE cos/sin 行，解决了在 NPU 上对分层视图进行索引带来的性能问题，优化了 MTP draft step。

**重要修复**
- **DeepSeek-V4 Flash Vision 休眠唤醒修复 (#19248)**：修复了模型在 sleep → wake → 层级更新 → graph replay 流程中的权重重载问题，并新增了单卡 NPU IPC 权重传输通道。
- **MRV2 静态内核初始化报错修复 (#19245, #19246)**：修复了 MRV2 在 profiling 时因缺少 vLLM config 上下文导致的 `AssertionError`（同时覆盖主分支和 v0.30.0 分支）。
- **恢复 MoE-local memory profiling (#19243, #19244)**：还原了之前被替换的 MoE prepare/finalize 实现，避免了通过临时缩小 runtime 内存来进行的开销（同时覆盖主分支和 v0.30.0 分支）。
- **统一 DeepSeek V4 张量并行(TP)限制 (#19240)**：由于 A5 硬件已支持张量并行，移除了此前针对特定硬件的 TP 策略，改为在所有硬件上使用统一的模型级限制。

**文档更新**
- **DeepSeek V4.1 部署指南同步 (#19242)**：将 v0.30.0 版本中的 DeepSeek-V4.1-Flash 中英文部署教程同步至 main 分支。
- **GLM-5.3-Flash 部署文档修复 (#19239)**：将 A2 硬件上的 GLM-5.3-Flash 部署文档修复同步至 main 分支。

### 🚀 Release 动态
- 本周期内无新版本发布。

---

## 🐛 Issues

### #19241 — [[Bug]: Qwen3.8-27B startup fails when VLLM_PREFIX_CACHE_RETENTION_INTERVAL=4096 is not a multiple of the hybrid Mamba scheduler block size (1536)](https://github.com/vllm-project/vllm-ascend/issues/19241)
- **作者**: LZZhang-HW  **时间**: 2026-10-20 11:53 CST
- **标签**: bug
- **摘要**: ### Your current environment  - Deployment: vLLM-Ascend, Qwen3.8-27B, single node with 16 NPUs - Model: Qwen3.8-27B - Feature flags: `--enable-prefix-caching` - Environment variable: `VLLM_PREFIX_CACHE_RETENTION_INTERVAL=4096` - Deployment layouts: both `tp2-dp8` and `tp4-dp4` fail with the same val…

## 🔀 Pull Requests

### #19249 — [[Feature][KV Transfer] AscendStoreConnector: per-group KV block counts (window-capped per-group pools compatibility)](https://github.com/vllm-project/vllm-ascend/pull/19249)
- **作者**: vincentlau2046  **时间**: 2026-10-20 13:09 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  `KVPoolWorker` (AscendStoreConnector) assumed a single uniform block count for every KV cache tensor: `register_kv_caches` validated `tensor_num_blocks % self.num_blocks == 0` with `self.num_blocks = kv_cache_config.num_blocks` (the **maximum** over cache gro…

### #19248 — [[BugFix][WeightTransfer] Fix DeepSeek-V4 Flash Vision reload across sleep and wake](https://github.com/vllm-project/vllm-ascend/pull/19248)
- **作者**: wang2001-hust  **时间**: 2026-10-20 13:06 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Adds a one-card DeepSeek-V4-Flash-Vision NPU IPC weight-transfer lane and fixes the sleep → wake → layerwise update → graph replay path it exercises. Packed updates now load once per complete update, checkpoint-backed routing parameters use the existing Worke…

### #19247 — [[Feature][Ops] Gather RoPE cos/sin rows from the contiguous flat base](https://github.com/vllm-project/vllm-ascend/pull/19247)
- **作者**: dailiang16  **时间**: 2026-10-20 12:53 CST
- **标签**: module:tests, module:ops
- **摘要**: ## Why  Every MTP draft step calls `get_cos_and_sin_mla(positions)` to gather RoPE cos/sin rows. On NPU, indexing the **strided per-half views** (`_cos_cache[positions]`, `_sin_cache[positions]`) makes `aclnnIndex` materialize the **entire cache table** on every lookup, while the effective payload i…

### #19246 — [[BugFix][v0.30.0][MRV2] Use compiler config for static kernel initialization](https://github.com/vllm-project/vllm-ascend/pull/19246)
- **作者**: yiz-liu  **时间**: 2026-10-20 12:41 CST
- **标签**: ready-precise
- **摘要**: ### What this PR does / why we need it?  MRV2 profiling can reach static kernel initialization without an active current-vLLM-config context, causing: `AssertionError: Current vLLM config is not set.`  Use `self.vllm_config`, already supplied by `compute_hash()`, instead of `get_current_vllm_config(…

### #19245 — [[BugFix][MRV2] Use compiler config for static kernel initialization](https://github.com/vllm-project/vllm-ascend/pull/19245)
- **作者**: yiz-liu  **时间**: 2026-10-20 12:41 CST
- **标签**: ready-precise
- **摘要**: ### What this PR does / why we need it?  MRV2 profiling can reach static kernel initialization without an active current-vLLM-config context, causing: `AssertionError: Current vLLM config is not set.`  Use `self.vllm_config`, already supplied by `compute_hash()`, instead of `get_current_vllm_config(…

### #19244 — [[BugFix][v0.30.0][PCP] Restore MoE-local memory profiling](https://github.com/vllm-project/vllm-ascend/pull/19244)
- **作者**: recky-c  **时间**: 2026-10-20 12:00 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Restore the MoE prepare/finalize implementation from #18270 at `fb780a7db6e2118f91f1cc1ea10636b190bde610`, before it was replaced by temporarily shrinking the runner token budget.  During the initial MRV2 memory profile, split TP-local hidden states, router l…

### #19243 — [[BugFix][PCP] Restore MoE-local memory profiling](https://github.com/vllm-project/vllm-ascend/pull/19243)
- **作者**: recky-c  **时间**: 2026-10-20 11:59 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Restore the MoE prepare/finalize implementation from #18270 at `fb780a7db6e2118f91f1cc1ea10636b190bde610`, before it was replaced by temporarily shrinking the runner token budget.  During the initial MRV2 memory profile, split TP-local hidden states, router l…

### #19242 — [[Doc][Model] Sync DeepSeek V4.1 deployment guide from v0.30.0](https://github.com/vllm-project/vllm-ascend/pull/19242)
- **作者**: dragondream-chen  **时间**: 2026-10-20 11:55 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  The DeepSeek-V4.1-Flash tutorial on main is missing deployment documentation already merged into releases/v0.30.0. This ports the English tutorial and Chinese translation catalog from release commit `0a174d6c3` onto main commit `ccaf14ad3`.  - Add the four-se…

### #19240 — [[BugFix] Unify DeepSeek V4 tensor parallel limits across hardware](https://github.com/vllm-project/vllm-ascend/pull/19240)
- **作者**: dragondream-chen  **时间**: 2026-10-20 11:51 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  A5 now supports tensor parallelism. Remove the hardware-specific DeepSeek V4 TP policy introduced in #18418 and use the same model-specific limits on all hardware families:  - DeepSeek V4 Flash and V4.1 Flash: TP <= 8. - DeepSeek V4 Pro: TP <= 16.  Remove the…

### #19239 — [[Doc][GLM-5.3-Flash] Fix A2 deployment guidance on main](https://github.com/vllm-project/vllm-ascend/pull/19239)
- **作者**: calyoung80  **时间**: 2026-10-20 11:38 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  This PR forward-ports the GLM-5.3-Flash A2 deployment documentation fixes from PR #18880 to the `main` branch. It updates `docs/source/tutorials/models/GLM5.3-Flash.md` to: - Remove `--data-parallel-start-rank 0` from the colocated API node so rank 0 is infer…
