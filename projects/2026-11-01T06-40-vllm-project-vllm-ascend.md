# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-01 14:40 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
- **#20537 [Bug][KV Cache] Ascend 混合 KV 缓存路径静默忽略 `prefix_cache_retention_interval`**
  - 作者：wuming2027
  - 问题：在 Ascend 混合 KV 缓存路径下，`prefix_cache_retention_interval` 参数被静默忽略，未能按预期执行缓存清理保留策略。

### 🔧 Pull Request 动态

#### ⚡ 性能优化
- **#20544 [KV Pool] 调度器与租约元数据扫描线性化**
  - 优化了 AscendStore 在准备调度器元数据和分层加载时的扫描逻辑。通过将原本对 Python 列表的重复扫描改为线性扫描，解决了大量请求或长前缀恢复时的性能瓶颈。
- **#20541 / #20542 [UVA writes] 避免连续 UVA 写入的逐元素跟踪**（含 v0.32.0rc 适配）
  - 针对 MRV2 UVA 回退前缀写入，取消了逐元素的脏数据跟踪。连续的 `[:n]` 赋值现在直接记录前缀边界，大幅减少了开销。
- **#20536 优化 GLM-5.3-Flash C8 prefill attention**
  - 在 Model Runner V2 上降低了 GLM-5.3-Flash C8 的 prefill 延迟：对符合条件的 A3 TP8 批次采用 projected K/V masked MHA，其他全可见 prefill 场景使用 FIA。

#### 🐛 Bug 修复
- **#20543 [KV Pool] 拒绝不完整的混合批量 KV 加载**
  - 修复了非分层 AscendStore 批量加载在找到键后仍可能失败的问题，避免了因组局部块 ID 无法安全识别无效请求而导致的错误。
- **#20539 / #20540 [BugFix] 修复 SwigluGroupQuant MXFP 尾部存储掩码**（含 v0.32.0rc 适配）
  - 修复了 `SwigluGroupQuant` 中 MXFP 输出存储的尾部掩码问题。原实现因四个连续打包存储使用独立的剩余列计数器导致出错，现已修正。
- **#20538 [KVPool] 针对稀疏索引器模型请求惰性存储初始化**
  - 修正了 KVPoolWorker 的惰性初始化逻辑，使其仅对压缩模型（如 use_compress, DSV4 compress_ratios）请求 `lazy_init`，避免对其他模型造成不必要的影响。

#### 🧪 测试与文档
- **#20535 补充 MiniMax-M3 和 GLM-5.3-Flash 测试**
  - 为 TP2 IPC 权重传输 E2E 测试新增了 MiniMax-M3 和 GLM-5.3-Flash 的覆盖测试。
- **#20534 自动文档翻译**
  - 自动翻译并更新了 66 个文档文件（包含开发者指南中的 `context_parallel.po` 等）。

### 🚀 Release 动态
- 本周期内无新的 Release 版本发布。

---

## 🐛 Issues

### #20537 — [[Bug][KV Cache] prefix_cache_retention_interval is silently ignored on Ascend hybrid KV cache paths](https://github.com/vllm-project/vllm-ascend/issues/20537)
- **作者**: wuming2027  **时间**: 2026-11-01 14:25 CST
- **摘要**: --- title: "[Bug][KV Cache] prefix_cache_retention_interval is silently ignored on Ascend hybrid KV cache paths" ---  ## Environment  | Item         | Value                                                                                                          | | ------------ | -------------------…

## 🔀 Pull Requests

### #20544 — [[Performance][KV Pool] Make scheduler and lease metadata scans linear](https://github.com/vllm-project/vllm-ascend/pull/20544)
- **作者**: Esir98  **时间**: 2026-11-01 14:31 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  AscendStore repeatedly scans Python lists while preparing scheduler metadata and layerwise loads. With many active requests or a long restored prefix, these scans add quadratic CPU overhead before the next execution step.  - Build the scheduled request-ID set…

### #20543 — [[BugFix][KV Pool] Reject incomplete hybrid bulk KV loads](https://github.com/vllm-project/vllm-ascend/pull/20543)
- **作者**: Esir98  **时间**: 2026-11-01 14:30 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  A non-layerwise AscendStore bulk load can fail after its keys were found. For hybrid caches, group-local block IDs cannot safely identify an invalid request through the flat invalid-block fallback. Some paths only log the failure; the synchronous MemCache pat…

### #20542 — [[Performance][v0.32.0rc] Avoid per-element tracking for contiguous UVA writes](https://github.com/vllm-project/vllm-ascend/pull/20542)
- **作者**: Karryking3  **时间**: 2026-11-01 14:28 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Avoid per-element dirty tracking for MRV2 UVA fallback prefix writes. A contiguous `[:n]` assignment now records a prefix bound instead of building a set of n indices and invoking n Python callbacks. Synchronization copies that pinned prefix asynchronously and…

### #20541 — [[Performance] Avoid per-element tracking for contiguous UVA writes](https://github.com/vllm-project/vllm-ascend/pull/20541)
- **作者**: Karryking3  **时间**: 2026-11-01 14:28 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Avoid per-element dirty tracking for MRV2 UVA fallback prefix writes. A contiguous `[:n]` assignment now records a prefix bound instead of building a set of n indices and invoking n Python callbacks. Synchronization copies that pinned prefix asynchronously and…

### #20540 — [[BugFix][v0.32.0rc] Fix SwigluGroupQuant MXFP tail store masks](https://github.com/vllm-project/vllm-ascend/pull/20540)
- **作者**: Karryking3  **时间**: 2026-11-01 14:28 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Fix the MXFP output-store tail masks in SwigluGroupQuant. Four consecutive packed stores previously used four independent remaining-column counters, so each store treated the same tail as valid and could overwrite adjacent rows or UB buffers. Share one counter…

### #20539 — [[BugFix] Fix SwigluGroupQuant MXFP tail store masks](https://github.com/vllm-project/vllm-ascend/pull/20539)
- **作者**: Karryking3  **时间**: 2026-11-01 14:28 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Fix the MXFP output-store tail masks in SwigluGroupQuant. Four consecutive packed stores previously used four independent remaining-column counters, so each store treated the same tail as valid and could overwrite adjacent rows or UB buffers. Share one counter…

### #20538 — [[BugFix][KVPool] Request lazy store init for sparse-indexer models](https://github.com/vllm-project/vllm-ascend/pull/20538)
- **作者**: Dwade3  **时间**: 2026-11-01 14:28 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  KVPoolWorker requests lazy_init from the storage backend only for compressed models (use_compress, DSV4 compress_ratios) — the deferral introduced for that family in #9771. Sparse-indexer models (use_sparse, e.g. GLM-5.x SFA) route their indexer KV through th…

### #20536 — [[Performance] Optimize GLM-5.3-Flash C8 prefill attention](https://github.com/vllm-project/vllm-ascend/pull/20536)
- **作者**: lijiahang226  **时间**: 2026-11-01 14:15 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Reduce GLM-5.3-Flash C8 prefill latency on model runner V2 with projected K/V masked MHA on eligible A3 TP8 batches and FIA for other admitted full-visible prefixes. Skip redundant KPool query selection while retaining cache compression and tail updates. Rest…

### #20535 — [[Test] Add MiniMax-M3 and GLM-5.3-Flash coverage to TP2 IPC weight-transfer tests](https://github.com/vllm-project/vllm-ascend/pull/20535)
- **作者**: Liccol  **时间**: 2026-11-01 14:05 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Temporary development draft split from #18634 to review further changes without repeatedly updating that PR's branch.  - Move the shared IPC weight-transfer E2E entry point to two-card TP2. - Select all model cases for unpacked IPC; use Qwen3.5 for representa…

### #20534 — [[Doc] Translated Doc files 2026-11-01](https://github.com/vllm-project/vllm-ascend/pull/20534)
- **作者**: vllm-ascend-ci  **时间**: 2026-11-01 13:59 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **66** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents/context_parallel.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/develop…
