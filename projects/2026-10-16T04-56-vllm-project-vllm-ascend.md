# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-16 12:56 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 📝 Issue（议题）
本次提供的数据中未包含 Issue 相关动态。

### 🔧 Pull Request（代码拉取请求）
近期 PR 主要集中在**废弃旧版 Model Runner V1 (MRV1)**、**多芯片性能优化**、**Bug 修复**以及 **v0.30.0 版本的特性回传**：

- **架构重构与废弃**：
  - **#18964 / #18958**：开始逐步淘汰 Model Runner V1 (MRV1)，分别在 CPP 层和 KVPP 层移除了与 MRV1 相关的启动分析、运行时初始化、虚拟运行和缓存分配等过时代码。
- **性能优化**：
  - **#18963**：为 A5 (Ascend 950) 上的 MiniMax-M3 模型启用 AscendC `MsaIndexScore` 解码路径，使其解码性能与 A3 实现对齐。
  - **#18960**：引入外部优化，更新 A2/A3 架构下的 MSA IndexScore 解码算子，并修复了宽分数 问题。
- **Bug 修复**：
  - **#18959**：修复了 DSV4 模型中 `AscendCompressorStateCache` 上报缓存槽位数量不准确的 Bug（改为以 token 为单位声明 block_size）。
  - **#18956 / #18957**：修复 Mooncake V2 调度器资源清理问题（解决请求 ID 无限期保留、失败通知无限重排队导致的资源泄漏）。
- **版本准备与回传**：
  - **#18961**：将 A5 V1 GLM DFlash2 的 CANN 9.2 FlashAttn 特性回传至 `releases/v0.30.0` 分支。
  - **#18955**：为了夜间验证 以隔离 QKV 布局拷贝问题，在 `main` 分支临时回退了 PR #18723。
- **文档更新**：
  - **#18962**：自动翻译了 31 个中文文档文件。

### 🚀 Release（版本发布）
本次提供的数据中未包含正式的 Release 发布记录。
**版本动态展望**：虽然无正式发布，但从 PR 动态可以看出，项目正在积极向 `v0.30.0` 版本推进。该版本预计将重点包含 **A5 架构上的 GLM DFlash2 注意力机制优化**，以及 **Mooncake V2 调度器的稳定性修复**。

---

## 🔀 Pull Requests

### #18964 — [[Refactor][CPP] Drop Model Runner V1 support](https://github.com/vllm-project/vllm-ascend/pull/18964)
- **作者**: dianWang325  **时间**: 2026-10-16 12:53 CST
- **摘要**: ### What this PR does / why we need it?  Model Runner V1 is approaching end of life on Ascend. CPP still carried MRV1-specific startup profiling, runtime timing, dummy-run, and scheduler output branches that can now be retired.  This PR:  - requires Model Runner V2 whenever `profiling_chunk_config` …

### #18963 — [[Performance][Model] Enable AscendC index-score decode for MiniMax-M3 on A5](https://github.com/vllm-project/vllm-ascend/pull/18963)
- **作者**: xuchi-0808  **时间**: 2026-10-16 12:31 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Enable the bundled AscendC `MsaIndexScore` decode path for MiniMax-M3 on Ascend 950 (A5), aligning A5 decode with the A3 implementation (`MsaIndexScore` + the fused Triton TopK helpers).  This re-lands the A5 decode enablement from #16426, which was reverted …

### #18962 — [[Doc] Translated Doc files 2026-10-16](https://github.com/vllm-project/vllm-ascend/pull/18962)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-16 12:26 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **31** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/slash-commands.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents…

### #18961 — [[v0.30.0][Feature][Attention] CANN FlashAttn for A5 V1 GLM DFlash2 eager drafts](https://github.com/vllm-project/vllm-ascend/pull/18961)
- **作者**: jiajinzhu2  **时间**: 2026-10-16 12:04 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Backport #18717 to `releases/v0.30.0`: use CANN 9.2 device-length FlashAttn and per-step plan reuse for A5 V1 DFlash2 eager drafts paired with GLM MoE DSA targets (`model_type="glm_moe_dsa"`). Keep the existing release V1/V2 validation unchanged.  ### Does th…

### #18960 — [[Performance][Ops] Optimize A2/A3 MSA IndexScore decode and fix wide scores](https://github.com/vllm-project/vllm-ascend/pull/18960)
- **作者**: yingyingzizi  **时间**: 2026-10-16 11:45 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Update the existing MsaIndexScore operator with the A2/A3 optimizations from [ops-transformer-ar](https://gitcode.com/oscillated/ops-transformer-ar), commit `044f648f741bb1f3b1bc5d6adf39afd992fe8a86`.  Small decode batches currently underutilize AICs while ea…

### #18959 — [[BugFix][Model] Declare DSV4 compressor state cache blocks in token units](https://github.com/vllm-project/vllm-ascend/pull/18959)
- **作者**: Absoler  **时间**: 2026-10-16 11:42 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? AscendCompressorStateCache.get_kv_cache_spec reported its slot count (2/8 state slots) in the spec's block_size field, which every cache-manager consumer reads as a token count — one slot holds one compress_ratio-token interval. The engine-wide cache block siz…

### #18958 — [[Refactor][KVPP] Remove Model Runner V1 support](https://github.com/vllm-project/vllm-ascend/pull/18958)
- **作者**: recky-c  **时间**: 2026-10-16 11:33 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Retire KVPP support for Model Runner V1 as MRV1 is being sunset. Remove its KVPP runtime initialization, forward/dummy-forward hooks, cache allocation branch, and runner-specific tests and fixtures. Simplify draft-cache ownership discovery to the V2 speculato…

### #18957 — [[BugFix][v0.30.0] Fix Mooncake V2 scheduler resource cleanup](https://github.com/vllm-project/vllm-ascend/pull/18957)
- **作者**: nwpu-zxr  **时间**: 2026-10-16 11:31 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Mooncake Connector V2 retains completed request IDs indefinitely, requeues failed completion notifications without a limit, and does not stop its scheduler control threads during shutdown.  Use pending delayed-free requests to report completion only once, whi…

### #18956 — [[BugFix][KV Transfer] Fix Mooncake V2 scheduler resource cleanup](https://github.com/vllm-project/vllm-ascend/pull/18956)
- **作者**: nwpu-zxr  **时间**: 2026-10-16 11:31 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Mooncake Connector V2 retains completed request IDs indefinitely, requeues failed completion notifications without a limit, and does not stop its scheduler control threads during shutdown.  Use pending delayed-free requests to report completion only once, whi…

### #18955 — [[Performance][GDN] Revert #18723 on main for nightly validation](https://github.com/vllm-project/vllm-ascend/pull/18955)
- **作者**: czydyy  **时间**: 2026-10-16 11:15 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Temporarily revert #18723 (`2b8da69474a7899a63072e194d502979e7e389df`) on `main` **for nightly validation**, to isolate whether the reduced QKV layout copies for FLA attention are the culprit of the current issue. This branch contains only the #18723 revert, …
