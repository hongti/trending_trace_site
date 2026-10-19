# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-19 13:08 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issues 提交
近期 Issue 主要集中在 **HF3FS KV 连接器** 的缺陷与新特性需求：
1. **内存页泄漏导致保存失败**：HF3FS 元数据服务器无法释放已使用的内存页，页面池耗尽后所有保存操作均会失败。
2. **多形状 KV Cache 适配需求**：请求 `HF3FSKVConnector` 支持具有多种 KV cache 形状的模型（如 MLA + DSA indexer）。目前在使用 `GLM-5.3-NVFP4` 时，会在启动阶段卡死。
3. **配置被忽略的 Bug**：`HF3FSKVConnector` 会无视用户配置的 `hf3fs_metadata_server_url`，强制将两个元数据客户端均指向 `localhost:18000`。

### 🚀 Pull Requests 变更
PR 活动主要围绕性能优化、内核修复以及新特性扩展：
- **新特性与支持**：
  - **Transformers TPSP 支持** (#62385, #62395)：为稠密基线提供 TPSP 支持，引入自动性能分析机制，仅在 token 数量超过阈值时启用 TPSP，以防性能退化。
  - **MoRI OFI 后端** (#62396)：为 MoRI 连接器新增 OFI libfabric 后端支持（原有仅支持 RDMA 和 XGMI）。
  - **DeepSeek-V4.1-Flash 优化** (#62384)：支持解码端的 SWA 有界重放（bounded replay），涵盖 PCP、APC 及前缀命中续写。
- **缺陷修复**：
  - **HF3FS 页面分配** (#62393)：优化页面分配逻辑，仅为未命中的独立缓存分配页面，修复混合批次在页面不足时错误失败的问题。
  - **KV Offload 事件陈旧** (#62394)：在 CPU primary 重置时触发范围性失效，修复残留的过期事件。
  - **ZMQ 线程安全** (#62391)：将 ZMQ socket 的关闭操作收敛至发布线程，避免并发使用导致的冲突。
  - **Hopper 内核缩放因子** (#62386)：修复 `silu_and_mul_per_block_quant` 中未正确处理 `outer_scale_stride` 的问题，确保 TMA 对齐。
- **性能优化**：
  - **避免重复扫描** (#62392)：当 routed-experts 存储满载时，不再从最旧键开始重新扫描未被引用的键，减少额外开销。
- **前端与工具**：
  - 整合前端 setup 工具集 (#62390)。

### 🎉 Release 发布：v0.32.0
**版本亮点**：
- **规模盛大**：包含 **814 个提交**，来自 **363 位贡献者**（其中 **121 位**为首次参与的新贡献者）。
- **重磅新特性**：引入了**结构化决策与通用解析器**，大幅提升了模型推理输出的规范性与灵活性。

---

## 🐛 Issues

### #62389 — [[Bug]: HF3FS metadata server never frees pages; every save fails once the page pool is used up](https://github.com/vllm-project/vllm/issues/62389)
- **作者**: sivabreddy  **时间**: 2026-10-19 11:26 CST
- **摘要**: `hf3fs_metadata_server.py` gives each rank a fixed free-page list (`hf3fs_file_size` / bytes per page). Pages return to the list only for failed writes, for keys that are already complete, or through `POST /clear` (which also removes the rank tables). There is no eviction, so the pool only shrinks. …

### #62388 — [[Feature]: HF3FSKVConnector: support models with more than one KV cache shape (e.g. MLA + DSA indexer)](https://github.com/vllm-project/vllm/issues/62388)
- **作者**: sivabreddy  **时间**: 2026-10-19 11:26 CST
- **摘要**: With `GLM-5.3-NVFP4` (architecture `GlmMoeDsaForCausalLM`; TP8, `--kv-cache-dtype fp8 --block-size 64`, vLLM 0.31.0), the workers stop at start-up in `register_kv_caches` → `_setup_kv_cache_config`: `AssertionError: HF3FS requires uniform KV cache shapes and strides.` (`vllm/distributed/kv_transfer/…

### #62387 — [[Bug]: HF3FSKVConnector ignores hf3fs_metadata_server_url (both metadata clients use localhost:18000)](https://github.com/vllm-project/vllm/issues/62387)
- **作者**: sivabreddy  **时间**: 2026-10-19 11:26 CST
- **摘要**: `HF3FSKVConnector` reads `hf3fs_metadata_server_url` from `kv_connector_extra_config` into `self._metadata_server_url`, but creates both metadata clients with `Hf3fsMetadataClient()` (no argument):  - `vllm/distributed/kv_transfer/kv_connector/v1/hf3fs/hf3fs_connector.py` line 511 (scheduler side) a…

## 🔀 Pull Requests

### #62396 — [Support OFI backend for MORI connector](https://github.com/vllm-project/vllm/pull/62396)
- **作者**: vinaynkaranth  **时间**: 2026-10-19 13:00 CST
- **标签**: kv-connector
- **摘要**: ## Overview  The MoRIIOConnector currently supports only RDMA and XGMI bakends. The changes here adds supports for OFI libfabric as backend.   ## Claims The changes here adds supports for OFI libfabric as backend for MoRIIOConnector. The changes to MORI lib to support OFI libfabric was added via the…

### #62395 — [[Feat][2/2] Add Transformers TPSP dense baseline support](https://github.com/vllm-project/vllm/pull/62395)
- **作者**: hlin99  **时间**: 2026-10-19 12:56 CST
- **标签**: documentation, ci/build, nvidia
- **摘要**: This PR is stacked on top of https://github.com/vllm-project/vllm/pull/62385  Adds automatic profiling to enable TPSP only when the token count exceeds a tuned threshold, avoiding performance regressions on smaller workloads.

### #62394 — [[Bugfix][KV Offload] Emit scoped invalidation on primary reset](https://github.com/vllm-project/vllm/pull/62394)
- **作者**: Alex-ai-future  **时间**: 2026-10-19 12:40 CST
- **标签**: bug, documentation, kv-connector
- **摘要**: ## Overview  Follow-up to [#59855](https://github.com/vllm-project/vllm/pull/59855), which this PR depends on. This change fixes stale KV offload events published after a CPU primary reset by adding a scoped invalidation event and clearing queued CPU raw events.  This is intentionally separate from …

### #62393 — [[Bugfix][KV Connector] Allocate HF3FS pages only for distinct misses](https://github.com/vllm-project/vllm/pull/62393)
- **作者**: kerry0520  **时间**: 2026-10-19 12:38 CST
- **标签**: bug, kv-connector
- **摘要**: ## Overview  Allocate HF3FS metadata pages only for distinct cache misses. A mixed batch containing an already complete key must not fail when the remaining pages are sufficient for its new keys.  Refs vllm-project/vllm#62389. This addresses the near-capacity allocation subproblem; it does **not** i…

### #62392 — [[Perf][AuxOutput] Do not rescan referenced keys in the routed-experts store](https://github.com/vllm-project/vllm/pull/62392)
- **作者**: aoshen02  **时间**: 2026-10-19 12:05 CST
- **摘要**: ## Purpose  Split from #62044 (by @faresobeid), keeping only the store rescan fix.  When the routed-experts store is full, `BlockObjectStore._evict_to_fit` scans the LRU from the oldest key for unreferenced victims. Keys referenced by running requests cannot be evicted and pile up at the front of th…

### #62391 — [[Bugfix][KV Events] Close ZMQ sockets from publisher thread](https://github.com/vllm-project/vllm/pull/62391)
- **作者**: BESzk  **时间**: 2026-10-19 11:34 CST
- **标签**: bug
- **摘要**: ## Overview  Addresses Bug6 in #56251.  Prevent `ZmqEventPublisher.shutdown()` from closing ZMQ sockets while the publisher thread may still be using them.  ## Claims  - Makes the publisher thread the sole owner of PUB and ROUTER socket cleanup. - Preserves the bounded shutdown wait without closing …

### #62390 — [[Frontend] Consolidate setup utils.](https://github.com/vllm-project/vllm/pull/62390)
- **作者**: noooop  **时间**: 2026-10-19 11:32 CST
- **标签**: documentation, performance, frontend, intel-gpu, ci/build, cohere
- **摘要**: ## Overview  Consolidate setup utils.  ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this PR.  - [ ] *…

### #62386 — [[Bugfix][Kernel][Perf] Honor outer_scale_stride in silu_and_mul_per_block_quant for TMA-aligned transposed scales](https://github.com/vllm-project/vllm/pull/62386)
- **作者**: edesoiza  **时间**: 2026-10-19 11:23 CST
- **标签**: bug, torch.compile, quantization
- **摘要**: ## Purpose  On Hopper (`sm90`), `1×128` / `1×64` block-scaled FP8 GEMM consumers (`FlashInfer` `fp8_blockscale_gemm_sm90` `swapAB` and `DeepGEMM` `fp8_gemm_nt` `1d2d`) load activation scaling factors via TMA, which requires **column-major scales with 16-byte (4-element FP32) aligned column strides**…

### #62385 — [[Feat][1/2] Add Transformers TPSP dense baseline support](https://github.com/vllm-project/vllm/pull/62385)
- **作者**: hlin99  **时间**: 2026-10-19 11:15 CST
- **标签**: documentation, ci/build, nvidia
- **摘要**: Design doc: https://github.com/hlin99/vllm/blob/ww43_PR_TPSP_Transformers_1/2/docs/design/tpsp_design.md  # Verified model list - Qwen/Qwen3-8B - Qwen/Qwen2-7B-Instruct - tencent/Hunyuan-7B-Instruct - zai-org/glm-4-9b-chat-hf - meta-llama/Meta-Llama-3.1-8B-Instruct - baidu/ERNIE-4.5-0.3B-PT - mistra…

### #62384 — [[Feature][PCP] Support decoder-side SWA bounded replay with APC for DeepSeek-V4.1-Flash](https://github.com/vllm-project/vllm/pull/62384)
- **作者**: wangyicong52  **时间**: 2026-10-19 11:13 CST
- **标签**: deepseek, mrv2, DSv4.1
- **摘要**: ## Overview  Extend #62183 to support DeepSeek-V4.1-Flash decoder-side SWA bounded replay with PCP and APC, including prefix-hit continuation.  ## Claims  - Retain each request's global trailing window on its original PCP owners while preserving cached compressed KV and indexer keys. - Keep prompt-l…

## 🚀 Releases

### [v0.32.0](https://github.com/vllm-project/vllm/releases/tag/v0.32.0)
- **作者**: khluu  **时间**: 2026-10-18 14:50 CST
- **摘要**: # v0.32.0  ## Highlights  This release features 814 commits from 363 contributors (121 new)!  * **Structured decisions and generic parsers**: the new `POST /v1/systemone` endpoint answers `choice`, `noul` and `score` questions over a structured state (#59299), is served by default (#60677, #60696), …
