# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-28 16:19 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue
- **#70490 [Bugfix][Anthropic] `/v1/messages` 接口意外丢失 system prompt**
  当 Claude Code 的 `x-anthropic-billing-header` 与 system prompt 共享同一文本块时，转换器会错误地剥离该行，导致请求中的 system prompt 被丢弃。

### 🔧 Pull Requests (PR)
- **性能与内核优化**
  - **#70485 [ROCm]** 优化 `prefix_prefill` 内核，跳过空的 query tiles，避免为批次中最长的输入分配过多计算资源，提升性能。
  - **#70483 [Attention]** 为 Blackwell 架构引入 `MXFP8_PREFILL` 注意力后端，支持以 MXFP8 格式进行完整的因果预填充，且不泄漏未来 token。
  - **#70488 [XPU]** 在 XPU 上为 Linear 和 MoE 启用块级 FP8 W8A8 批处理不变性。
- **核心架构修复与增强**
  - **#70491 [KV Offload]** 增强 `BlockPool`，支持为早于请求到达的 KV 数据预留 GPU 块，优化异步加载流程。
  - **#70484 [Core]** 修复 `benchmark_block_pool.py` 脚本，补充必需的 `hash_block_size` 参数以适配现有构造函数。
- **多模态与前端解析**
  - **#70481 [Multimodal]** 在 SM100 架构上为 Qwen3-VL 视觉编码器启用具备批处理不变性的 FA4 前向传播。
  - **#70489 [Rust Frontend]** 修复 Rust 端 Kimi K2 工具解析器错误截断带点号工具名（如 `get-weather.v2`）的问题，使其与 Python 端逻辑保持一致。
- **测试与工具链**
  - **#70486 [Test]** 补充针对 64 段 Triton split-K 注意力机制在长上下文下的测试。
  - **#70482 [Test]** 新增调度器测试，确保多组 KV 异步加载完成后，滑动窗口与全注意力模型能正常恢复推进。
  - **#70487 [Tools]** 在 SPDX header hook 中记录运行环境快照，以便更好地诊断自托管 runner 上的缓存未命中问题。

### 🚀 Release
- *近期无新版本发布。*

---

## 🐛 Issues

### #70490 — [[Bugfix][Anthropic] /v1/messages drops the system prompt when the billing header shares its text block](https://github.com/vllm-project/vllm/issues/70490)
- **作者**: 995114879  **时间**: 2026-11-28 16:15 CST
- **标签**: kimi
- **摘要**: ## Overview  `/v1/messages` strips Claude Code's `x-anthropic-billing-header: ...` line so the per-request `cch` hash does not break prefix caching. The converter currently treats "text starts with the header" as "this whole text block is the header", so when the header shares a block/string with th…

## 🔀 Pull Requests

### #70491 — [[KV Offload] Reserve pool blocks for KV that arrives ahead of its request](https://github.com/vllm-project/vllm/pull/70491)
- **作者**: liteboy-bit  **时间**: 2026-11-28 16:15 CST
- **标签**: kv-cache-manager
- **摘要**: ## Overview  `BlockPool` learns to hold blocks for KV that is copied into the GPU pool *ahead* of the request that will read it: `pin_blocks_by_hash()` reserves, `publish_pinned_hashes()` makes the reservation hittable, `release_pinned()` hands it back readable. Three files, `+266/−0`, nothing calls…

### #70489 — [[Bugfix][Rust Frontend] Keep dotted Kimi K2 tool names whole, like Python](https://github.com/vllm-project/vllm/pull/70489)
- **作者**: Pushkal-Gupta  **时间**: 2026-11-28 15:43 CST
- **标签**: bug, rust, kimi
- **摘要**: ## Overview  The Rust `kimi_k2` tool parser cuts a dotted tool name down to its last segment. Kimi K2 writes a call header as `functions.{name}:{index}`, so a tool named `get-weather.v2` comes out of the model as `functions.get-weather.v2:0`, and Rust returns the name `v2`. The client then gets a ca…

### #70488 — [[XPU][BI] Enable block-wise FP8 W8A8 batch invariance](https://github.com/vllm-project/vllm/pull/70488)
- **作者**: hlin99  **时间**: 2026-11-28 15:42 CST
- **标签**: intel-gpu, ci/build, batch-invariance
- **摘要**: Select the existing Triton block-scaled FP8 W8A8 BI kernel on XPU for Linear and MoE. Admit block-wise Fp8Config checkpoints without hard-coding the weight block size. Reject online overrides because they can change the activation scheme or execution kernel. Without overrides, the existing block FP8…

### #70487 — [tools: record runner state in the SPDX header hook](https://github.com/vllm-project/vllm/pull/70487)
- **作者**: b8zhong  **时间**: 2026-11-28 15:40 CST
- **摘要**: Records a best-effort snapshot of the runner environment once when the SPDX header hook starts, to help diagnose pre-commit cache misses on the self-hosted runners. No change to the check itself.

### #70486 — [[Test] Cover Triton split-K attention at 64 segments with long contexts](https://github.com/vllm-project/vllm/pull/70486)
- **作者**: SRKRZ23  **时间**: 2026-11-28 15:18 CST
- **摘要**: ## Overview  SM120 (#58482) and, with #66485, gfx942 use 64 split-KV segments for small decode batches, but `tests/kernels/attention/test_triton_unified_attention.py` runs the split-K kernel only at 16 segments and with contexts up to about 2K tokens. This adds one reference test at 16 and 64 segmen…

### #70485 — [[ROCm][Perf] Skip empty query tiles in the prefix_prefill kernels](https://github.com/vllm-project/vllm/pull/70485)
- **作者**: SRKRZ23  **时间**: 2026-11-28 15:16 CST
- **标签**: rocm
- **摘要**: ## Overview  `context_attention_fwd` launches `(batch, head, cdiv(max_input_len, BLOCK_M))` programs, so every request gets as many query tiles as the longest chunk in the batch. In `_fwd_kernel` and `_fwd_kernel_alibi`, a tile that starts past its own request's chunk (`block_start_loc >= cur_batch_…

### #70484 — [[Bugfix][Core] Fix BlockPool benchmark hash block size](https://github.com/vllm-project/vllm/pull/70484)
- **作者**: pratik376  **时间**: 2026-11-28 14:54 CST
- **标签**: bug, performance
- **摘要**: ## Overview Fix benchmarks/benchmark_block_pool.py to work with the current BlockPool constructor by providing the required hash_block_size.  ## Claims   1) Fixes benchmark_block_pool.py, which currently fails because BlockPool requires hash_block_size.  2) Adds a configurable --hash-block-size benc…

### #70483 — [[Attention] Add MXFP8_PREFILL: causal MXFP8 prefill on Blackwell](https://github.com/vllm-project/vllm/pull/70483)
- **作者**: tejesh-sarvam  **时间**: 2026-11-28 14:52 CST
- **标签**: documentation, ci/build
- **摘要**: ## Overview  Adds `MXFP8_PREFILL`, an opt-in Blackwell attention backend that runs full, uncached causal prefills with MXFP8 (E4M3 + E8M0) matrix products **without leaking future tokens** through shared 32-token V scales. The KV cache and decode stay native (BF16, FlashAttention).  The normal phase…

### #70482 — [[Test] Cover forward progress after multi-group KV loads](https://github.com/vllm-project/vllm/pull/70482)
- **作者**: nl-alonso  **时间**: 2026-11-28 14:45 CST
- **标签**: scheduler
- **摘要**: ## Overview  Adds one scheduler test: async KV loads admitted in a model with a sliding-window group beside full attention must be able to resume once their receives finish. Tests only, nothing under `vllm/` changes. Related to #54703 and #69444.  ## Claims  - `main` already behaves correctly for th…

### #70481 — [[Multimodal] Enable batch-invariant FA4 for Qwen3-VL vision on SM100](https://github.com/vllm-project/vllm/pull/70481)
- **作者**: HangruiCao2000  **时间**: 2026-11-28 14:40 CST
- **标签**: qwen, batch-invariance
- **摘要**: ## Overview  Refs #69763. Use a fixed FA4 forward configuration in the Qwen3-VL vision encoder on SM100 when batch invariance and FA4 are explicitly selected and the attention shape matches the validated configuration.  ## Claims  - With BF16 and 16 Q/KV heads of dimension 64 per partition, the Qwen…
