# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-24 12:57 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要：

### 📝 Issue
- **#63439 [RFC] 降低长上下文 P/D 服务在 Model Runner V2 中的 CPU 开销**
  作者探讨了在 Prefill/Decode 分离场景下部署 DeepSeek-V4.1-Flash 时的性能瓶颈。在 2,048 并发、约 130K token 长上下文的高负载下，GPU 经常受限。作者发起讨论，提出减少 Model Runner V2 的 CPU 开销以提升整体服务吞吐量。

### 🚀 Pull Request
本次提交主要包含**性能优化**与**错误修复**两类：

**性能优化**
- **#63438 [V2] block id 传输优化**：将新请求的 block id 以 int32 数组发送给 V2 model runner workers，弃用 Python list，降低传输与调度开销。
- **#63441 [KV Events] 批量处理 BlockRemoved 事件**：在使用外部 KV store（如 Mooncake P/D prefill）时，按分配策略批量发送 BlockRemoved 事件，避免按块逐个发送造成的巨大开销。
- **#63443 [Pooling] 提前启动离线池化**：优化离线池化（如 `LLM.encode`），在所有 prompt 完成预处理前即开始将请求传入引擎，提高 GPU 利用率。
- **#63444 [MRV2] 批量重置 draft-token**：延迟并批量处理新增请求的 `draft_tokens` 重置操作，避免每新增一个请求就触发一次零散的 GPU 操作和 kernel launch。
- **#63440 [ROCm] FlyDSL monokernel 支持**：在 MI355X 上为 DSpark verify 步骤引入 FlyDSL decode monokernel（通过 `--monokernel` 启用），大幅提升 Muse Spark 的验证性能。

**错误修复**
- **#63448 [Frontend] 补全 `completed_at` 字段**：修复 `/v1/responses` 接口在状态为 `completed` 时遗漏 `completed_at` 字段的问题，解决 OpenAI SDK 客户端读取不到该字段的问题。
- **#63447 [Core] 修复未完成请求计数错误**：修复在 `pause_generation(mode="wait")` 模式下，当被抢占的会话停止时，`get_num_unfinished_requests()` 统计不准确的问题。
- **#63446 [Kernel] 修复 lightning attention prefill 精度问题**：修复序列长度超过单个 prefill block（256 tokens）时 lightning attention 计算错误的问题（RMS 误差达 2.6 倍），通过增加一次 decay 步骤修复。
- **#63445 [Frontend] 聚合数据并行模式下的 ORCA 指标**：修复开启数据并行时，`endpoint-load-metrics` ORCA 仅报告单个引擎负载的问题，现可正确聚合所有引擎的负载指标。
- **#63442 [CPU] 优化 KV cache 内存不足报错**：优化 CPU 后端在节点内存不足时的错误提示，明确引导用户检查 `--gpu-memory-utilization` 参数。

### 📦 Release
近期暂无版本发布信息。

---

## 🐛 Issues

### #63439 — [[RFC]: Reduce CPU overhead in Model Runner V2 for long-context P/D serving](https://github.com/vllm-project/vllm/issues/63439)
- **作者**: gcanlin  **时间**: 2026-10-24 11:26 CST
- **摘要**: ### Background  Serving DeepSeek-V4.1-Flash with P/D disaggregation (2P1D on 16x B200: prefill 2x DEP4, decode DEP8; AgentX traces with ~130K-token prompts at 2,048 concurrent sessions), the GPUs often sit idle waiting for the host: decode was ~17% idle and prefill 15-17% idle. Most of it is per-req…

## 🔀 Pull Requests

### #63448 — [[Bugfix][Frontend] Set completed_at on completed Responses](https://github.com/vllm-project/vllm/pull/63448)
- **作者**: yjlhhh  **时间**: 2026-10-24 12:53 CST
- **标签**: bug, documentation, frontend
- **摘要**: ## Overview  `/v1/responses` omits `completed_at` even when `status` is `completed`, so OpenAI SDK clients always read `None`. Set the field from `ResponsesResponse.from_request`. Fixes #63183.  This does not duplicate an open PR. [PR #63234](https://github.com/vllm-project/vllm/pull/63234) attempte…

### #63447 — [[Bugfix][Core] Fix the PAUSED_NEW unfinished count with a session that stopped while preempted](https://github.com/vllm-project/vllm/pull/63447)
- **作者**: Alex-Wengg  **时间**: 2026-10-24 12:49 CST
- **标签**: bug, scheduler
- **摘要**: ## Overview  During `pause_generation(mode="wait")` (`PAUSED_NEW`), `get_num_unfinished_requests()` counts `kv_holding_waiting` but subtracts every paused streaming session. A session that stopped while preempted (kept since #58778) holds no blocks and sits in `waiting`, so subtracting it cancels ou…

### #63446 — [[Bugfix][Kernel] Apply one more decay step in lightning_attention prefill](https://github.com/vllm-project/vllm/pull/63446)
- **作者**: aniketkrs  **时间**: 2026-10-24 12:07 CST
- **标签**: bug
- **摘要**: ## User impact Sequences longer than one prefill block (256 tokens) computed with lightning attention come out wrong: the reported repro shows ~2.6x the output RMS error on tokens 256+ against a fixed-seed fp64 reference, and even the first token is wrong (error 4.9 vs 1.7e-6) when `kv_history` is n…

### #63445 — [[Bugfix][Frontend] Aggregate per-engine ORCA metrics under data parallel](https://github.com/vllm-project/vllm/pull/63445)
- **作者**: aniketkrs  **时间**: 2026-10-24 12:07 CST
- **标签**: bug, frontend
- **摘要**: ## User impact With data parallel enabled, the `endpoint-load-metrics` ORCA header reports only one engine's load to the gateway: the JSON `named_metrics` dict kept a single value per metric name, so every engine but the last was silently dropped. Gateways doing load-based routing see a skewed pictu…

### #63444 — [[Perf][MRV2] Batch the draft-token reset of newly added requests](https://github.com/vllm-project/vllm/pull/63444)
- **作者**: JobQiu  **时间**: 2026-10-24 12:05 CST
- **标签**: mrv2
- **摘要**: ## Overview  `RequestState.add_request()` zeroes the new request's row of `draft_tokens` immediately, which is one small GPU op and one kernel launch per added request. Defer the reset and do all of them in a single `index_fill_` in `apply_staged_writes()`.  ## Claims  - Removes one GPU op and one k…

### #63443 — [[Perf][Pooling] Start offline pooling before all prompts are preprocessed](https://github.com/vllm-project/vllm/pull/63443)
- **作者**: JobQiu  **时间**: 2026-10-24 12:05 CST
- **标签**: frontend, pooling
- **摘要**: ## Overview  Offline pooling (`LLM.encode`, `embed`, `classify`, `score`) already feeds the engine incrementally, but it preprocessed the entire prompt list before adding the first request, so the GPU sat idle for the whole preprocessing phase. This makes the offline request factory lazy and replace…

### #63442 — [[Bugfix][CPU] Point the short-node KV cache error at --gpu-memory-utilization](https://github.com/vllm-project/vllm/pull/63442)
- **作者**: junqiu-lei  **时间**: 2026-10-24 11:55 CST
- **标签**: bug, cpu
- **摘要**: ## Overview  Since #59405 removed the `CPUWorker.__init__` reservation check, `determine_available_memory` is the only gate on the CPU memory reservation. Its "node short of memory" error only says to free memory elsewhere, so a default `LLM(...)` / `vllm serve` on a busy desktop no longer mentions …

### #63441 — [[Perf][KV Events] Batch BlockRemoved events per allocation](https://github.com/vllm-project/vllm/pull/63441)
- **作者**: gcanlin  **时间**: 2026-10-24 11:36 CST
- **标签**: kv-cache-manager
- **摘要**: ## Overview  Emit one `BlockRemoved` per KV cache group per allocation instead of one per evicted block. When most of a prompt comes from an external KV store (P/D prefill with Mooncake), every allocated block evicts a cached one, and building these events costs the scheduler a few percent of its ti…

### #63440 — [[ROCm][Perf][Muse Spark] FlyDSL decode monokernel for DSpark verify steps on MI355X (--monokernel)](https://github.com/vllm-project/vllm/pull/63440)
- **作者**: azaidy  **时间**: 2026-10-24 11:32 CST
- **标签**: rocm, needs-rebase, ci/build, nvidia, dflash
- **摘要**: ## Overview  With `--monokernel`, Muse Spark Open MXFP4 on 8x MI355X (TP8 + expert parallelism, 7-token DSpark draft) runs decoder layers 1-60 of every verify step of 1-16 requests as persistent FlyDSL launches. One launch, or three, cover the work from a layer's attention output to the next layer's…

### #63438 — [[Perf][V2] Send new requests' block ids to the workers as int32 arrays](https://github.com/vllm-project/vllm/pull/63438)
- **作者**: gcanlin  **时间**: 2026-10-24 11:22 CST
- **标签**: mrv2, scheduler
- **摘要**: ## Overview  Send new requests' block ids to the v2 model runner workers as int32 arrays and stage them without Python lists. Stacked on #59908, which does the same for token ids; only the last commit belongs to this PR.  ## Claims  - P/D decode instance of DeepSeek-V4.1-Flash (DEP8, B200): steps th…
