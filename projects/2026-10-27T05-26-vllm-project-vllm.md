# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-27 13:26 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issues (问题反馈)
近期主要报告了多方面的问题，涉及模型推理、底层缓存及性能：
1. **模型与解码异常**：
   - Kimi K3 在非流式输出时，会意外泄露被截断的 XTML 标记片段（#63941）。
   - Model Runner V2 (MRV2) 的推测解码在启动 `profile_run` 时未覆盖拒绝采样，导致 KV cache 容量估算偏小（#63942）。
   - EmbeddingGemma 2 使用 `FLEX_ATTENTION` 时，CUDAGraph 的输出在编译模式下被错误覆盖（#63945）。
2. **内核与架构 Bug**：
   - 在混合模型中，`int8_per_token` 的 scale-view 填充操作意外覆盖了 Gated DeltaNet 的状态（#63944）。
   - KV offload P2P 层在 prefill 节点上执行 `reset_cache()` 时触发断言失败（#63948）。
3. **性能问题**：
   - Triton MoE 的 naive block 分配逻辑重复读取了常驻设备的共享专家权重，导致 GB10 芯片上的解码速度下降 8% 至 2 倍不等（#63943）。

### 🔧 Pull Requests (代码合并请求)
PRs 主要集中在关键 Bug 修复、KV Connector 架构优化以及内核性能提升：
1. **重要 Bug 修复**：
   - **Kimi K3 / FunctionGemma 修复**：修复了 Kimi K3 非流式输出泄露 XTML 标记的问题（#63946）；修复了 FunctionGemma 将转义分隔的字符串参数误解析为 JSON 的问题（#63947）。
   - **KV Offload 安全修复**：避免将因报错而提前终止的请求的“未完成 prefill 块”进行卸载，防止其错误哈希污染后续请求的 prefix 缓存（#63937）。
   - **多模态防死循环**：在 `split_audio` 中拒绝小于一个采样点的音频块，防止切割逻辑陷入死循环（#63939）。
2. **性能优化**：
   - **DCP fp8 优化**：在解码上下文并行（DCP）下，MLA chunked prefill 直接以 fp8 格式聚合上下文，避免了原本“反量化为 bf16 再 all-gather”的性能开销（#63935）。
   - **调度器减负**：跳过运行中请求的无操作 KV 分配过程（#63934）；将 NIXL 的缓存块计数与拉取块 ID 收集操作相融合，消除重复遍历（#63933）。
3. **KV Connector 架构增强**：
   - 在 residual SJF（最短作业优先）调度中复用缓存的连接器命中估算，避免重复扫描块表（#63940）。
   - 通过 MultiConnector 转发预调度查找句柄（#63938）。
   - 引入 `enable_early_recv` 标志，允许 NIXL 在下一个模型输出到达调度器前提前上报已完成的 pulls（#63936）。

### 🚀 Release (版本发布)
近期无新的版本发布记录。

---

## 🐛 Issues

### #63948 — [[Bug]: KV offload P2P tier: reset_cache() fails an assert on a prefill node with stores waiting for the decoder](https://github.com/vllm-project/vllm/issues/63948)
- **作者**: Vedant-Agarwal  **时间**: 2026-10-27 13:22 CST
- **摘要**: ### Your current environment  Not environment specific: reproduces in a CPU-only unit test on main (checked at 5d35ce393), using the P2P tier test helpers and the real `TieringOffloadingManager` + CPU primary tier.  ### 🐛 Describe the bug  On a P2P producer (prefill) node, `TieringOffloadingManager.…

### #63945 — [[Bug]: FLEX_ATTENTION fails on EmbeddingGemma 2 (encoder-only): create_block_mask compiled with mode="reduce-overhead" returns CUDAGraph outputs that get overwritten](https://github.com/vllm-project/vllm/issues/63945)
- **作者**: vuquoctrungbk  **时间**: 2026-10-27 13:11 CST
- **摘要**: ### Your current environment  - vLLM `0.32.1rc1.dev505+g00b6c4c56` (`vllm/vllm-openai:nightly-00b6c4c565a8732c131e1f452d4a17803bb0842b`, digest `sha256:76a9614e…`), torch 2.14.0+cu130, triton 3.8.0, transformers 5.20.2 - Also reproduced earlier on `0.31.1rc1.dev166+g51ef23976` (wheel), so it is not …

### #63944 — [[Bug]: int8_per_token_head scale-view fill overwrites Gated DeltaNet state in hybrid models](https://github.com/vllm-project/vllm/issues/63944)
- **作者**: F1xGOD  **时间**: 2026-10-27 13:02 CST
- **摘要**: ### Environment  - vLLM 0.32.0 built from the sdist for sm_70 (Tesla V100 PCIe 32 GB, driver   580.196.07, CUDA 12.9), torch 2.13.0+cu126. The code path below is the same   on every architecture, and `main` still contains it. - `RedHatAI/Qwen3.8-27B-INT4`, a Qwen3.5-family hybrid: 48 Gated DeltaNet …

### #63943 — [[Performance]: Triton MoE naive block assignment re-reads shared experts for device-resident weights (8% to 2x slower decode on GB10)](https://github.com/vllm-project/vllm/issues/63943)
- **作者**: ks-huang  **时间**: 2026-10-27 12:57 CST
- **标签**: performance
- **摘要**: ### Proposal to improve performance  ### Summary  `_prepare_expert_assignment` takes the low-latency "naive block assignment" path when `expert_map is None` and `num_tokens * top_k * 4 <= num_experts` ([fused_moe.py#L1600-L1609](https://github.com/vllm-project/vllm/blob/5d35ce3934a7804b298559d2d99ee…

### #63942 — [[Bug]: [MRV2] Spec-decode rejection sampling is not covered by profile_run](https://github.com/vllm-project/vllm/issues/63942)
- **作者**: Alex-Wengg  **时间**: 2026-10-27 12:44 CST
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information... uv is set ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (x8…

### #63941 — [[Bug]: Kimi K3 non-streaming content leaks truncated XTML marker fragments](https://github.com/vllm-project/vllm/issues/63941)
- **作者**: xiaohaifeng1999  **时间**: 2026-10-27 12:44 CST
- **标签**: tool-calling, kimi, k3
- **摘要**: ### Your current environment  <details open> <summary>The output of <code>python collect_env.py</code></summary>  ```text vLLM main @ cd9aa5b5ee (2026-10-26; still in main's history — current main tip 6fe3d447ac, 2026-10-27, is 105 commits ahead and does not touch kimi_k3_tool_parser.py). Parser-lev…

## 🔀 Pull Requests

### #63947 — [[Bugfix] Preserve FunctionGemma escape-delimited string arguments](https://github.com/vllm-project/vllm/pull/63947)
- **作者**: MarinaMackay  **时间**: 2026-10-27 13:22 CST
- **标签**: bug, documentation, tool-calling
- **摘要**: ## Overview  Preserve FunctionGemma `<escape>`-delimited argument values as strings instead of decoding them as JSON scalars or containers. Fixes #63906 for streaming and non-streaming chat completions.  ## Claims  - Escaped numeric, boolean, null, JSON-container and quoted text retains its literal …

### #63946 — [[Bugfix] Kimi K3: withhold truncated XTML marker fragments in content](https://github.com/vllm-project/vllm/pull/63946)
- **作者**: xiaohaifeng1999  **时间**: 2026-10-27 13:14 CST
- **标签**: bug, tool-calling, kimi, k3
- **摘要**: ## Overview  Non-streaming Kimi K3 content leaked raw XTML control-marker fragments when generation was cut mid-marker (`max_tokens`); the streaming path already kept them out of content, so the two paths returned different content for the same output. Gives `_strip_response_content` the streaming p…

### #63940 — [[Core][KV Connector] Use cached connector hit estimates in residual SJF](https://github.com/vllm-project/vllm/pull/63940)
- **作者**: Sy0307  **时间**: 2026-10-27 12:37 CST
- **标签**: documentation, ci/build, kv-connector, scheduler, kv-cache-manager, mooncake
- **摘要**: ## Overview  Use completed connector prefix-hit estimates when ranking residual SJF requests, and cache local hit estimates to avoid repeating block-table scans on every queue selection.  **Stacked dependency draft:** local residual SJF comes from immengzi's #51375; input-thread lookup comes from Da…

### #63939 — [[Bugfix][Multimodal] Reject audio chunk sizes smaller than one sample](https://github.com/vllm-project/vllm/pull/63939)
- **作者**: pbb2007  **时间**: 2026-10-27 12:34 CST
- **标签**: bug, documentation, multi-modality
- **摘要**: ## Overview  `split_audio` can loop without advancing when `int(sample_rate * max_clip_duration_s)` is zero or negative. Reject these chunk sizes before entering the splitting loop.  ## Claims  - Raise `ValueError` for computed chunk sizes below one sample, including positive durations that truncate…

### #63938 — [[KV Connector] Forward preschedule lookup handles through MultiConnector](https://github.com/vllm-project/vllm/pull/63938)
- **作者**: Sy0307  **时间**: 2026-10-27 12:28 CST
- **标签**: documentation, ci/build, kv-connector, mooncake
- **摘要**: ## Overview  **Stacked on #61292** (Dao Le, original head `83bc03ec3`, rebased prerequisite `62d9876d9` on main `3d81a510f`). Forward its input-thread lookup hook through MultiConnector and keep each child's handle isolated throughout scheduler callbacks, including nested connectors and request comp…

### #63937 — [[Bugfix][KV Offload] Do not offload the partially computed block of a request that finished with an error](https://github.com/vllm-project/vllm/pull/63937)
- **作者**: sohom-cs  **时间**: 2026-10-27 12:26 CST
- **标签**: bug, kv-connector
- **摘要**: ## Overview  When a request finishes with an error before its prefill is complete, the block it was computing is stored under the hash of the full block. A later request with that prefix hits the offloaded block and loads KV for tokens that were never computed. One-expression fix in `_build_store_jo…

### #63936 — [[KV Connector][NIXL] Report completed pulls before the next model output](https://github.com/vllm-project/vllm/pull/63936)
- **作者**: Sy0307  **时间**: 2026-10-27 12:24 CST
- **标签**: documentation, ci/build, kv-connector, scheduler
- **摘要**: ## Overview  Draft for design review. Head `435cb2b53`, base `3e75ccba2`.  Report completed NIXL pulls before a subsequent model output reaches the scheduler. Enable with `enable_early_recv` in the decoder's KV transfer config; disabled by default.  ## Claims  - Count early and ordinary notification…

### #63935 — [[Perf][Attention][DCP] Gather the MLA chunked prefill context in fp8 under decode context parallelism](https://github.com/vllm-project/vllm/pull/63935)
- **作者**: mispa-ms  **时间**: 2026-10-27 12:22 CST
- **标签**: rocm, kimi, k3
- **摘要**: ## Overview  With decode context parallelism (DCP), a plain fp8 KV cache and an fp8 prefill query, the MLA chunked-context loop dequantizes each rank's cached rows to bf16, all-gathers the bf16 rows across the DCP group, and reorganizes them with a Python loop (one slice per request and rank, plus `…

### #63934 — [[Perf][Core] Skip no-op KV allocation passes for running requests](https://github.com/vllm-project/vllm/pull/63934)
- **作者**: Sy0307  **时间**: 2026-10-27 12:15 CST
- **标签**: kv-cache-manager
- **摘要**: ## Overview  Skip the per-group `allocate_slots()` passes when a running request needs no freeing, allocation or cache registration. Each supported manager explicitly opts in; other manager types retain the existing path.  ## Claims  - Reduce decode scheduler CPU work between block boundaries while …

### #63933 — [[Perf][NIXL] Fuse cached-block counting and pull block-ID collection](https://github.com/vllm-project/vllm/pull/63933)
- **作者**: Sy0307  **时间**: 2026-10-27 12:14 CST
- **标签**: ci/build, kv-connector
- **摘要**: ## Overview  Fuse NIXL pull block-ID collection and per-group cached-block counting after trimming each group's null prefix. This removes duplicate scheduler-thread traversal when D admits long-context sliding-window requests.  ## Claims  - Preserve interior-null filtering, interleaved hash states, …
