# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-07 14:27 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🚀 Pull Requests (PR) 动态
近期仓库共更新了 10 个 PR，主要涵盖新特性、性能优化、Bug修复以及文档与测试完善：

**1. 新特性**
*   **Engram 分布式分片**：引入了跨 owner 分片共享 HIXL 表的功能（#21246），以及基于原生 CANN VMM 在 A3 上跨 P/D 实例分片宿主机表的功能（#21245），提升多节点分布式处理与内存管理能力。

**2. 性能优化**
*   **MRV2 SWA 性能提升**：通过复用 V4.1 SWA 的重放可见长度，避免了冗余的计算操作，提升了整体运行效率（#21244）。

**3. Bug 修复**
*   **KV Transfer 修复**：修复了当 D 节点仍在进行前向计算时，提前到达的分层 P/D 完成信号导致接收线程异常的问题（#21247）。
*   **DeepSeek V4.1 报错优化**：针对不支持的 block size，将原本晦涩的报错改为了抛出明确的错误提示，提升排障体验（#21243）。

**4. 文档更新**
*   **部署指南更新**：基于双节点 Atlas 800 A2 服务器的验证结果，将 DeepSeek-V4.1-Flash A2 的部署教程更新为 W4A8 检查点和 Model Runner V2（#21241），并同步 cherry-pick 到了 `v0.32.0rc` 分支（#21242）。
*   **文档本地化**：自动翻译了 7 个开发指南文档（如多节点测试指南）（#21248）。

**5. 测试与 CI/CD**
*   **单元测试补充**：为 Ascend KV cache 配置构建器添加了仅在 CPU 上运行的单元测试（#21239）。
*   **CI 构建优化**：将 310P 的 entrypoint 脚本从 Dockerfile 中抽离为独立文件，大幅提升了可读性与可维护性（#21240）。

---
*注：近期暂无开放的 Issue 与 Release 动态。*

---

## 🔀 Pull Requests

### #21248 — [[Doc] Translated Doc files 2026-11-07](https://github.com/vllm-project/vllm-ascend/pull/21248)
- **作者**: vllm-ascend-ci  **时间**: 2026-11-07 13:51 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **7** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/contribution/multi_node_test.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/getting_start…

### #21247 — [[BugFix][KV Transfer] Retain early layerwise receive completions](https://github.com/vllm-project/vllm-ascend/pull/21247)
- **作者**: bowgneo  **时间**: 2026-11-07 12:46 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  A layerwise P/D completion can arrive while D is still finishing an earlier forward, before `start_load_kv` registers the new request. The receive thread currently drains all completed IDs, then the worker filters out IDs absent from `request_map`; an early n…

### #21246 — [[Feature][Engram] Shard shared HIXL tables across owners](https://github.com/vllm-project/vllm-ascend/pull/21246)
- **作者**: foraxe  **时间**: 2026-11-07 12:17 CST
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Extends the HIXL shared Engram backend in #20612 to distribute each immutable checkpoint table across a configurable number of owners. Each owner loads only its contiguous row range; consumers partition requested IDs, fetch the corresponding packed rows, dequ…

### #21245 — [[Feature][Engram] Shard shared host tables across P/D instances on A3 (VMM)](https://github.com/vllm-project/vllm-ascend/pull/21245)
- **作者**: foraxe  **时间**: 2026-11-07 11:47 CST
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Parallel draft to #19528, following the host-backed Engram work in #16544 and #19528; this draft uses native CANN VMM as the reference backend for static multi-owner sharing on A3. It does not replace or require merging #19528.  This draft implements static m…

### #21244 — [[Performance][MRV2] Reuse V4.1 SWA replay visible lengths](https://github.com/vllm-project/vllm-ascend/pull/21244)
- **作者**: zyz111222  **时间**: 2026-11-07 11:31 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Follow-up to #19362. SWA cache groups can share the same request coordinates and replay floor, but `_prepare_query_inputs` still launches `build_replay_swa_visible_lens` for every group. Reuse its output through the existing batch-local `common_v41_batch_meta…

### #21243 — [[BugFix] Raise clear error for unsupported DeepSeek V4.1 block size](https://github.com/vllm-project/vllm-ascend/pull/21243)
- **作者**: liuxi620  **时间**: 2026-11-07 11:02 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  `AscendDeepseekV41SWACache` builds the V4.1 SWA cache spec with:  ```python block_size = DSV4_BLOCK_SIZES[cache_config.block_size][0][1] ```  `DSV4_BLOCK_SIZES` only contains the 32/64/128 entries. An unsupported `cache_config.block_size` therefore fails with…

### #21242 — [ [Cherry-pick][releases/v0.32.0rc][Doc] Update DeepSeek-V4.1-Flash A2 deployment guide to W4A8 (from #21241)](https://github.com/vllm-project/vllm-ascend/pull/21242)
- **作者**: hansonchenmin  **时间**: 2026-11-07 10:59 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?      Cherry-pick of #21241 to the `releases/v0.32.0rc` branch.      Update the A2 part of the DeepSeek-V4.1-Flash tutorial from the W8A8   deployment to W4A8 with model runner V2, matching the configuration that   has been validated on two Atlas 800 A2 servers…

### #21241 — [[Doc] Update DeepSeek-V4.1-Flash A2 deployment guide to W4A8](https://github.com/vllm-project/vllm-ascend/pull/21241)
- **作者**: hansonchenmin  **时间**: 2026-11-07 10:50 CST
- **标签**: documentation
- **摘要**: Update the A2 part of the DeepSeek-V4.1-Flash tutorial to the W4A8 checkpoint and model runner V2, based on the configuration validated on two Atlas 800 A2 servers (8 x 64 GB per node).  - Rewrite the A2 PD section for the two-node W4A8 layout: Prefill DP2/TP1/PCP4 with 4096-token chunks per DP rank…

### #21240 — [[CI] Extract 310P entrypoint scripts out of the Dockerfiles](https://github.com/vllm-project/vllm-ascend/pull/21240)
- **作者**: wjunLu  **时间**: 2026-11-07 09:53 CST
- **摘要**: The two 310P Dockerfiles generated `/usr/local/bin/entrypoint.sh` with a ~13-line `RUN echo` chain, which is hard to read and review.  - add `docker/entrypoint_310p_ubuntu.sh` and `docker/entrypoint_310p_openeuler.sh` with the same content the chain produced (the two variants differ only in the `/li…

### #21239 — [[Test][KVCache] Add unit tests for the Ascend KV cache config builder](https://github.com/vllm-project/vllm-ascend/pull/21239)
- **作者**: Mr-qiji  **时间**: 2026-11-07 09:04 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Adds CPU-only unit tests for `vllm_ascend.core.kv_cache_config_builder`, locking the three-branch dispatch of `AscendKVCacheConfigBuilder._get_pool_bytes_per_block`:  - GLM-Next layout probed first; when present, its helper result is forwarded verbatim (inclu…
