# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-04 14:56 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issues (问题反馈)
1. **Bug #20904: Adler32 请求 ID 碰撞导致 KV 缓存错误复用**
   - 在开启 `use_fused_overlap` 的稀疏 KV 卸载场景下，两个不同请求的 ID 如果 Adler32 哈希值相同，会被误判为同一缓存所有者。这会导致新请求错误复用旧的（过时的）KV 缓存。
2. **Bug #20898: 每日夜间测试失败**
   - 2026-11-04 的夜间测试在多个环境（包括 310p-1/2/4 卡、cpu-0 卡等配置）下报告失败，需要开发者介入排查。

### 🚀 Pull Requests (代码合并)
#### ✨ 新特性与性能优化
1. **#20906 Mooncake Prefill 层级 KV Buffer 复用**：为 Mooncake 新增可选的层级 KV buffer 复用功能，通过有界环形设备缓冲区在分块 prefill 期间跨层保存和恢复数据。
2. **#20903 降低 Mooncake V2 地址构建开销**：优化了传输层性能，避免在仅需单一布局时为每个请求重复构建单例块列表和 NumPy 连续组。
3. **#20902 QuantLightningIndexerV2 引入 Split-K**：将长 K 序列分布到多个可用核心上（支持多请求批处理），优化内核性能，同时保留不适配该机制任务的元数据调度。
4. **#20896 稀疏 KV 卸载引入复用优先 LRU 策略**：新增可选的复用优先 LRU 策略，改变原有将新加载条目放置在末尾的机制，提升缓存复用率。
5. **#20895 Sleep Mode 轻量级 HCCL 生命周期**：为 Ascend 的睡眠模式引入 HCCL 挂起/恢复机制，避免在多卡任务中销毁并重建通信组带来的高昂耗时。

#### 🐛 修复
1. **#20905 修复 OPP 重定位后的算子缓存失效问题**：解决移动生成的 OPP 目录会导致算子缓存 key 改变的问题，确保编译输入相同时能正确复用缓存。
2. **#20900 修复 RMSNorm 算子路由错误**：修正了带有 residual 的无偏置 RMSNorm 错误调用非原生算子的问题，将其正确路由至原生 CANN 算子 `npu_add_rms_norm_bias`。

#### 📝 文档与其他
1. **#20901 DeepSeek-V4-Flash 部署文档对齐**：更新 A2 DSpark 2P2D 部署指南，对齐已验证的 1M 上下文标准配置。
2. **#20897 MiniMax-M3.1 Flash 部署文档**：新增 A3 W8A8 三节点 1P2D 的 1M 上下文 PD 部署指南及性能数据。
3. **#20899 文档自动翻译**：自动翻译了 14 个文档文件。

### 📦 Release (版本发布)
- 本期动态中未包含版本发布信息。

---

## 🐛 Issues

### #20904 — [[Bug]: Adler32 request-ID collisions cause stale KV reuse in fused-overlap sparse offload](https://github.com/vllm-project/vllm-ascend/issues/20904)
- **作者**: Esir98  **时间**: 2026-11-04 14:38 CST
- **摘要**: ## Summary  With sparse KV offload and `use_fused_overlap=true`, two distinct request IDs with the same Adler32 value can be treated as the same resident-cache owner. The new request preferentially reuses the previous request's physical row, skips request-change invalidation, and treats the previous…

### #20898 — [Nightly Daily test failure on 2026-11-04 12:00 - please check](https://github.com/vllm-project/vllm-ascend/issues/20898)
- **作者**: czydyy  **时间**: 2026-11-04 14:00 CST
- **摘要**: The nightly Daily test for vllm-ascend reported failures at 12:00 on 2026-11-04.  **Failed on:** 310p-1 card-(part 2-2), 310p-1 card-(part 1-2), cpu-0 card-(part 1-1), 310p-4 card-(part 1-1), 310p-2 card-(part 1-1), a3-4 card-(part 2-5), Kimi-K2.6-w4a8-A3  **Failed test cases:** - test_qwen35_lora_w…

## 🔀 Pull Requests

### #20906 — [[Feature] Add Mooncake prefill layerwise KV buffer reuse](https://github.com/vllm-project/vllm-ascend/pull/20906)
- **作者**: Eric-dot  **时间**: 2026-11-04 14:54 CST
- **摘要**: ### What this PR does / why we need it?  Add opt-in Mooncake-backed layerwise KV buffer reuse on prefill producers. A bounded ring of device buffers is restored and saved per layer across chunked prefills, including incomplete/unhashed pages and aligned recurrent state. Full blocks retain the existi…

### #20905 — [[BugFix][Build] Reuse operator cache after generated OPP relocation](https://github.com/vllm-project/vllm-ascend/pull/20905)
- **作者**: AuroraEmiya  **时间**: 2026-11-04 14:47 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Moving the generated OPP directory currently changes the operator cache key even when its compiler inputs are identical. Documentation and production-unused tests can also invalidate the operator namespace.  This follow-up is limited to three changes:  1. **G…

### #20903 — [[Performance][KV Transfer] Reduce Mooncake V2 address construction overhead](https://github.com/vllm-project/vllm-ascend/pull/20903)
- **作者**: nwpu-zxr  **时间**: 2026-11-04 14:30 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Mooncake V2 currently builds both singleton block lists and NumPy contiguous groups for every request in a transfer bucket, even when the layout only uses one representation. Long-context transfers allocate many short-lived Python objects before submitting th…

### #20902 — [[Performance][Kernel] Add split-K to QuantLightningIndexerV2](https://github.com/vllm-project/vllm-ascend/pull/20902)
- **作者**: Hirozy  **时间**: 2026-11-04 14:29 CST
- **标签**: module:tests
- **摘要**: Distribute long K sequences across available cores for candidate-consumer and candidate-disabled modes, including multi-request batches. Preserve metadata scheduling for workloads that do not benefit from split-K and retain Top-K selection semantics during shard merging.  Add correctness, graph repl…

### #20901 — [[v0.30.0][Doc][Misc] DeepSeek-V4-Flash: align A2 DSpark PD settings with the validated 1M configuration](https://github.com/vllm-project/vllm-ascend/pull/20901)
- **作者**: chenzhongsong93  **时间**: 2026-11-04 14:25 CST
- **标签**: documentation
- **摘要**: ## Summary  - Align the A2 DSpark 2P2D deployment with the validated standard:   - P: `--max-model-len 1048576`, `--max-num-batched-tokens 4096`, `--max-num-seqs 16`, GMU `0.95`.   - D: `--max-model-len 1048576`, `--max-num-batched-tokens 720`, `--max-num-seqs 42`, DSpark draft 5, GMU `0.95`, `--asy…

### #20900 — [[BugFix][Ops] Route biasless RMSNorm with residual to the native CANN operator](https://github.com/vllm-project/vllm-ascend/pull/20900)
- **作者**: wzk-jazk  **时间**: 2026-11-04 14:08 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  `AscendRMSNorm.forward_oot` reached for `torch.ops._C_ascend.npu_add_rms_norm_bias` whenever a residual was supplied, in two situations where that operator is not the right target.  For a biasless checkpoint `self.bias` is `None`, so `beta=None` was handed to…

### #20899 — [[Doc] Translated Doc files 2026-11-04](https://github.com/vllm-project/vllm-ascend/pull/20899)
- **作者**: vllm-ascend-ci  **时间**: 2026-11-04 14:07 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **14** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/contribution/doc_writing.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/getting_started/…

### #20897 — [[Doc] Add MiniMax-M3.1 Flash 1M-context PD deployment](https://github.com/vllm-project/vllm-ascend/pull/20897)
- **作者**: yingyingzizi  **时间**: 2026-11-04 13:45 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Add Section 5.4 to the MiniMax-M3.1-Flash guide, following the MiniMax-M3 1M-context deployment format. It provides an A3 W8A8 three-node 1P2D profile, complete role/proxy commands, and exact 1,048,576-token retrieval verification.  The functional profile use…

### #20896 — [[Feat][KVCache] Add reuse-prioritized LRU for sparse KV offload](https://github.com/vllm-project/vllm-ascend/pull/20896)
- **作者**: wangbiao0814  **时间**: 2026-11-04 13:07 CST
- **标签**: documentation, module:core
- **摘要**: ### What this PR does / why we need it?  Adds an optional reuse-prioritized LRU policy for sparse KV offload, based directly on `releases/v0.26.0rc`.   The existing policy places newly loaded entries after all surviving non-hit entries. Scans of one-time accesses can therefore evict previously reuse…

### #20895 — [[Feature][Sleep Mode] Add HCCL suspend/resume lightweight lifecycle](https://github.com/vllm-project/vllm-ascend/pull/20895)
- **作者**: JINITAIMEI121  **时间**: 2026-11-04 13:03 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Sleep Mode extra cleanup on Ascend currently destroys every HCCL process group and recreates it on wake-up. Recreating communicators is slow on multi-rank jobs and can fail when ranks wake at different times.  This PR adds a capability-gated **suspend/resume*…
