# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-08 14:33 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要：

### 🐛 Issue 动态
共 3 个重点 Issue，主要涉及新架构设计提案及运行时错误：
* **TPSP 融合设计 (RFC #60559)**：提出将 AllReduce 分解为 ReduceScatter (RS) + AllGather (AG)，并将其与相邻的计算操作（如 Norm、路由、GEMM）融合，以隐藏通信开销，提升性能。
* **Prefix Cache 重置崩溃 (Bug #60552)**：当 `OffloadingConnector` 正在从 CPU 加载 KV Cache 时，如果调用 `/reset_prefix_cache?reset_running_requests=true` 会触发 HTTP 500 错误。
* **外部投机采样崩溃 (Bug #60551)**：在 GLM-5.3 NVFP4 上使用 `kv_cache_dtype=nvfp4_ds_mla` 时，外部推测模型（DSpark Qwen3 draft, DFlash2）会崩溃，原因是 draft-model 代码错误假设了 split k/v 结构。

### 🔧 PR 动态
共 10 个 PR，主要集中在前端与 API 修复、调度与底层优化、以及 CI 修复：
**1. 前端与 API 修复**
* **保留内联系统消息 (#60555)**：修复了原生编码器（如 DeepSeek V4.1）在 `/v1/messages` 中对话中途的 `system` 消息被错误提升到开头的 Bug。
* **流式输出修复 (#60548)**：修复了 `stream_interval` 未对 `CUMULATIVE` 请求输出进行批处理的 Bug（之前 `sent_tokens_offset` 仅在 DELTA 模式下推进）。
* **在线束搜索清理 (#60550)**：当在线束搜索失败时，会取消并等待兄弟候选任务，确保引擎请求在错误抛给调用方前完成清理。
* **Rust 前端增强 (#60546, #60557)**：在 Rust 前端中透传推理控制参数（`reasoning_ended` 等）；并将 `run_rust_frontend` 迁移至 launchers 模块。

**2. 调度与底层组件修复**
* **修复编码器缓存死锁 (#60553)**：修复了在使用 EAGLE / MTP drafter look-ahead 时导致的编码器缓存死锁问题。
* **TorchCodec 异常处理 (#60554)**：将 TorchCodec 原生库加载引发的 `OSError` 视为后端不可用，使导入和自动解码器选择能平稳回退。
* **NIXL 接收处理优化 (#60549)**：修改 KVConnector 的 NIXL 接收后处理逻辑，使其直接在 block 的物理字节上运行。

**3. CI 与测试修复**
* **Intel CI 修复 (#60556, #60558)**：修复了因测试文件路径移动导致的 stale ignore path 问题，并在 Intel CI 上取消选择跨 API 服务器的权重同步指标测试。

### 🚀 Release 动态
* 本次提供的动态中 **未包含 Release 信息**。重点变更主要集中在提升投机采样稳定性、完善 Rust 前端功能以及修复多处边界条件导致的死锁和崩溃。

---

## 🐛 Issues

### #60559 — [[RFC]: TPSP Fusion Design](https://github.com/vllm-project/vllm/issues/60559)
- **作者**: hlin99  **时间**: 2026-10-08 14:29 CST
- **标签**: RFC
- **摘要**: ### Motivation.  # RFC: TPSP Fusion Design  ## 0. Goal  TPSP decomposes AllReduce into ReduceScatter (RS) + AllGather (AG), fusing RS/AG with adjacent compute (Norm, routing, GEMM) to hide communication behind computation. This brings roughly **10% TTFT improvement on long prefills**, which is espec…

### #60552 — [[Bug]: `POST /reset_prefix_cache?reset_running_requests=true` returns HTTP 500 while an OffloadingConnector load is in flight](https://github.com/vllm-project/vllm/issues/60552)
- **作者**: Yunzez  **时间**: 2026-10-08 13:42 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text ((vllm-30) ) gpu-a6-7:~$ python -m vllm.collect_env Collecting environment information... ==============================         System Info ============================== OS         …

### #60551 — [External speculators (DSpark Qwen3 draft, DFlash2) crash with kv_cache_dtype=nvfp4_ds_mla on GLM-5.3 NVFP4: draft-model code assumes split k/v](https://github.com/vllm-project/vllm/issues/60551)
- **作者**: renancloudwalk  **时间**: 2026-10-08 13:34 CST
- **标签**: glm
- **摘要**: Title: External speculators (DSpark Qwen3 draft, DFlash2) crash with kv_cache_dtype=nvfp4_ds_mla on GLM-5.3 NVFP4: draft-model code assumes split k/v  **Environment**: vLLM v0.31.0, GLM-5.3 NVFP4 (GlmMoeDsaForCausalLM), 8x B200, TP4/TP8 P/D (NixlConnector), --kv-cache-dtype nvfp4_ds_mla --block-size…

## 🔀 Pull Requests

### #60558 — [[XPU] [CI] Fix stale ignore path for weight_transfer_metrics tests on Intel CI](https://github.com/vllm-project/vllm/pull/60558)
- **作者**: chaojun-zhang  **时间**: 2026-10-08 14:21 CST
- **标签**: intel-gpu, ci/build
- **摘要**: ## Purpose  Introduced by #57849, which moved `test_weight_operation_metrics.py` into `tests/entrypoints/rl/weight_transfer_metrics/` (and added `test_across_api_servers_metrics.py` in the same directory) without updating the Intel CI `--ignore` path, which still pointed at the old flat `entrypoints…

### #60557 — [[Frontend] mv run_rust_frontend to launchers.](https://github.com/vllm-project/vllm/pull/60557)
- **作者**: noooop  **时间**: 2026-10-08 14:19 CST
- **标签**: frontend, rust
- **摘要**: <!-- markdownlint-disable -->  ## Overview  1. mv run_rust_frontend to launchers. 2. using contextlib.ExitStack  ## Claims  <!-- State the accomplishments in a few concrete bullets: speedups, feature support, bug fixes, etc. -->  ## Validation  <!-- Show evidence that the claims are met by this chan…

### #60556 — [[CI] Deselect test_weight_sync_metrics_aggregate_across_api_servers on Intel CI](https://github.com/vllm-project/vllm/pull/60556)
- **作者**: chaojun-zhang  **时间**: 2026-10-08 14:05 CST
- **标签**: intel-gpu, needs-rebase, ci/build
- **摘要**: ## Purpose  `tests/entrypoints/serve/dev/rlhf/test_weight_operation_metrics.py` contains a single test, `test_weight_sync_metrics_aggregate_across_api_servers`, which exercises the IPC weight-transfer backend (`--weight-transfer-config '{"backend": "ipc"}'`) and hard-codes `trainer = AutoModelForCau…

### #60555 — [[Bugfix][Anthropic] Preserve inline system messages for native encoders](https://github.com/vllm-project/vllm/pull/60555)
- **作者**: divyuuu  **时间**: 2026-10-08 13:57 CST
- **标签**: bug, frontend, DSv4.1
- **摘要**: ## Overview  Preserve mid-conversation `system` messages in `/v1/messages` for native encoders (DeepSeek V4.1) instead of hoisting them into the leading system prompt when `--chat-template` is omitted. Fixes #60537.  ## Claims  - `deepseek_v41` deployments without `--chat-template` no longer merge  …

### #60554 — [Handle TorchCodec native library load failures](https://github.com/vllm-project/vllm/pull/60554)
- **作者**: Asthenia0412  **时间**: 2026-10-08 13:57 CST
- **标签**: multi-modality
- **摘要**: Treat OSError from optional TorchCodec libraries as an unavailable backend so imports and automatic decoder selection can fall back cleanly.    ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` s…

### #60553 — [[Bugfix][Scheduler] Fix encoder cache deadlock with drafter look-ahead](https://github.com/vllm-project/vllm/pull/60553)
- **作者**: gty111  **时间**: 2026-10-08 13:51 CST
- **标签**: bug, scheduler
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Fixes the encoder-cache deadlock with a drafter look-ahead (EAGLE / MTP) reported in the second half of #40707 ([comment](https://github.com/vllm-project/vllm/issues/40707#issuecomment-5760731067)). When two multimodal items only fit in the encoder cache o…

### #60550 — [[Bugfix][Frontend] Clean up failed online beam searches](https://github.com/vllm-project/vllm/pull/60550)
- **作者**: galleonli  **时间**: 2026-10-08 13:31 CST
- **标签**: bug, frontend
- **摘要**: ## Overview  Cancel and await sibling candidate tasks when online beam search fails, so their engine requests finish cleanup before the error reaches the caller.  ## Claims  - Clean up outstanding candidates when one candidate raises an exception or is cancelled. - Preserve the original exception an…

### #60549 — [[KVConnector] Run NIXL receive post-process on the block's physical bytes](https://github.com/vllm-project/vllm/pull/60549)
- **作者**: Dao007forever  **时间**: 2026-10-08 13:16 CST
- **标签**: kv-connector
- **摘要**: ## Purpose  The NIXL receive post-process helpers in `kv_connector/utils.py` — `kv_postprocess_layout_on_receive`, `kv_postprocess_blksize_on_receive`, `kv_postprocess_blksize_and_layout_on_receive` — assume the tensor they receive is in physical memory order: they `index_select` the received blocks…

### #60548 — [[Bugfix][Frontend] Honor stream_interval for CUMULATIVE request outputs](https://github.com/vllm-project/vllm/pull/60548)
- **作者**: Justee21  **时间**: 2026-10-08 13:12 CST
- **标签**: bug
- **摘要**: ## Overview  Fixes #60547. `stream_interval` never batched outputs for `RequestOutputKind.CUMULATIVE` (the `SamplingParams` default) because `sent_tokens_offset` was only advanced in DELTA mode. This advances it for every streaming output kind.  ## Claims  - With `stream_interval=k`, CUMULATIVE requ…

### #60546 — [[Rust Frontend] Pass reasoning controls through render -> generate](https://github.com/vllm-project/vllm/pull/60546)
- **作者**: BugenZhao  **时间**: 2026-10-08 12:53 CST
- **标签**: rust
- **摘要**: ## Overview  Mirror https://github.com/vllm-project/vllm/pull/60062 for the Rust frontend: carry `reasoning_ended` and `reasoning_parser_kwargs` through `/v1/chat/completions/render` → `/inference/v1/generate`, so the engine gates structured outputs the same way as on `/v1/chat/completions`.  ## Cla…
