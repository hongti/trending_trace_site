# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-18 09:04 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文简洁摘要：

### 🚀 Release (版本发布)
本期动态中**未包含版本发布**信息。

### 🐛 Issue (议题)
本期动态中**未包含议题**更新。

### ✨ PR (拉取请求)
近期 PR 主要聚焦于**新模型支持、底层性能优化（特别是冷启动与 Blackwell GPU）以及多项稳定性 Bug 修复**：

**1. 重要新特性与架构优化**
*   **冷启动加速（CRIU 快照恢复）** (#48996)：新增 `vllm snapshot create` 与可选的 CRIU 恢复机制，大幅减少 `vllm serve` 冷启动时的 Python 重复导入开销。
*   **Rust 前端健康检查** (#48992)：在 Rust gRPC 前端引入引擎感知的 `grpc.health.v1` 健康状态报告服务，可同步汇报服务器与引擎状态。
*   **Helion CUDA-graph 捕获路由** (#48995)：在 CUDA-graph 捕获期间，将融合算子（如 RMSNorm+quant, SiLU+mul+quant, QK-norm+RoPE）路由至 Helion 执行。
*   **DeepSeek V4 KV Cache 紧凑化** (#48993)：针对 DSV4 的 MXFP4 indexer 优化 KV Cache 规划器，消除 packed group overlays 中的内存空洞，提升显存利用率。

**2. 新模型与 Attention 后端支持**
*   **新增 MiniCPM-SALA 模型** (#48999)：支持约 9.5B 参数的混合因果语言模型（结合 Lightning Attention 与 GQA）。
*   **MiniMax-M3 稀疏注意力加速** (#48994)：为消费级 Blackwell 架构 (SM120/SM121) GPU 引入 FlashInfer MSA 快速路径，取代原本较慢的 Triton 回退方案。

**3. 关键 Bug 修复**
*   **ROCm GPTQ 量化精度修复** (#48998)：修复 Triton W4A16 在 GPTQ/AutoGPTQ 中因候选形状相同导致的 transpose 判断歧义问题。
*   **AsyncLLM 误报日志消除** (#49000)：修复在有意关闭（SIGTERM/SIGINT）AsyncLLM 时误报 `EngineDeadError` 日志的问题。
*   **RMSNorm 批次不变性修复** (#48997)：修复在 `VLLM_BATCH_INVARIANT=1` 模式下，RMSNorm 输出仍受 `num_tokens` 影响而破坏批次不变性的问题（通过固定 block size）。
*   **ROCm CI 间歇性失败修复** (#49001)：修复分布式测试中因并发 HuggingFace 缓存刷新导致的间歇性错误，增加配置读取重试机制。

---

## 🔀 Pull Requests

### #49001 — [[Bugfix] Retry config read to survive concurrent HF cache refresh](https://github.com/vllm-project/vllm/pull/49001)
- **作者**: peizhang56  **时间**: 2026-07-18 08:54 CST
- **标签**: bug
- **摘要**: ## Purpose  `tests/v1/distributed/test_external_lb_dp.py` intermittently fails on ROCm CI (`Distributed DP Tests (2 GPUs)`) with the `[4]` (`api_server_count=4`) cases erroring as `Exception: Servers failed to start`. The real cause, hidden one level down in the server log, is one API-server process…

### #49000 — [[Bugfix] Suppress spurious EngineDeadError log during intentional AsyncLLM shutdown](https://github.com/vllm-project/vllm/pull/49000)
- **作者**: samikshaaagarwal  **时间**: 2026-07-18 08:53 CST
- **标签**: bug, v1
- **摘要**: <!-- markdownlint-disable -->   ## Purpose      Fixes #48745.      On an intentional shutdown (SIGTERM/SIGINT), AsyncLLM.shutdown() tears down the engine core before cancelling the output_handler task. AsyncMPClient's output queue task (process_outputs_socket) reacts to its own cancellation by pushi…

### #48999 — [[Model] Add MiniCPM-SALA (hybrid Lightning Attention + InfLLM-V2-ready GQA)](https://github.com/vllm-project/vllm/pull/48999)
- **作者**: ArchanaChetan07  **时间**: 2026-07-18 08:21 CST
- **标签**: documentation, performance, new-model, rocm, structured-output, tpu, intel-gpu, ci/build, multi-modality, tool-calling, llama, qwen, deepseek, cpu, gpt-oss, kv-connector, nvidia, mistral, rust
- **摘要**: ## Motivation Adds [openbmb/MiniCPM-SALA](https://huggingface.co/openbmb/MiniCPM-SALA): a ~9.5B, 32-layer hybrid causal LM (24 gated-linear "lightning" attention layers + 8 GQA layers that switch to InfLLM-V2 block-sparse attention at >= 8192 context). This is **PR1 of 2**: model + lightning + dense…

### #48998 — [[ROCm][Bugfix] Fix Triton W4A16 bug in determining if transpose is required for GPTQ/AutoGPTQ ](https://github.com/vllm-project/vllm/pull/48998)
- **作者**: qli88  **时间**: 2026-07-18 08:07 CST
- **标签**: bug, rocm
- **摘要**: ## Purpose this commit #47770 (for issue #47159) introduced a shape-based method to determine if qzeros need transposing, but the shape check is ambiguous when the two candidate shapes are identical. (cyankiwi-MiniMax-M3-AWQ-INT4 with tp=8 as an example). This PR is to fix this issue: 1. Use metadat…

### #48997 — [[Bugfix] Pin RMSNorm block size under batch invariance](https://github.com/vllm-project/vllm/pull/48997)
- **作者**: shivasathishs-rp  **时间**: 2026-07-18 07:46 CST
- **标签**: bug, v1
- **摘要**: ## Summary  Fixes #48271. Under `VLLM_BATCH_INVARIANT=1`, RMSNorm output for a given row still depends on `num_tokens`, so batch invariance is silently violated. For example, scoring a greedy-generated sequence with `prompt_logprobs` disagrees with the tokens that greedy decode actually produced.  *…

### #48996 — [[Frontend] Add vllm snapshot create and opt-in CRIU restore for serve](https://github.com/vllm-project/vllm/pull/48996)
- **作者**: matteso1  **时间**: 2026-07-18 07:33 CST
- **标签**: frontend
- **摘要**: ## Purpose  Cold `vllm serve` pays a large serial Python import bill twice (the API process and the spawned EngineCore child re-import nearly the same module graph) before any engine work starts. This adds the smallest opt-in slice of the imports-snapshot design discussed with simon-mo (design doc w…

### #48995 — [[Helion] Route fusion-only kernels to Helion during CUDA-graph capture](https://github.com/vllm-project/vllm/pull/48995)
- **作者**: yushangdi  **时间**: 2026-07-18 07:31 CST
- **标签**: v1, nvidia
- **摘要**: vLLM's post-grad fusion passes emit native fused ops (RMSNorm+quant, SiLU+mul+quant, QK-norm+RoPE) that have no eager model call site, so the capture-time call-site router used for per_token_group_fp8_quant (#47799) cannot reach them. Instead, add a single post-fusion FX pass that runs after fix_fun…

### #48994 — [[Attention][MiniMax-M3] Add FlashInfer MSA backend for SM120/SM121](https://github.com/vllm-project/vllm/pull/48994)
- **作者**: yichengj0  **时间**: 2026-07-18 06:41 CST
- **标签**: nvidia
- **摘要**: ## Purpose  Give MiniMax-M3 sparse attention a fast path on consumer Blackwell (SM120/SM121). These GPUs currently run the Triton fallback for both the lightning indexer and the block-sparse attend, since the vendored MSA kernels are SM100-only. FlashInfer `vX.Y.Z` <!-- placeholder: first release co…

### #48993 — [[Core][DSV4] Compact MXFP4 indexer KV cache and packed group overlays](https://github.com/vllm-project/vllm/pull/48993)
- **作者**: GirasoleY  **时间**: 2026-07-18 06:33 CST
- **标签**: v1
- **摘要**: ## Context  When DeepSeek V4 uses MXFP4 indexer K values, engine reserves the larger FP8 row for them. Its packed KV planner also buckets layers by page size, leaving avoidable holes when different cache groups have different mixtures of page sizes.  This PR:  - sizes MXFP4 indexer K rows from their…

### #48992 — [[Rust Frontend] Add engine-aware health reporting](https://github.com/vllm-project/vllm/pull/48992)
- **作者**: connorcarpenter15  **时间**: 2026-07-18 06:24 CST
- **标签**: rust
- **摘要**: ## Purpose  Expose the standard `grpc.health.v1.Health` service on the existing Rust frontend gRPC listener.  - Register health on the same listener as `vllm.Generate`. - Report the overall server and `vllm.Generate` as `SERVING` after startup. - Publish the engine client's sticky healthy-to-unhealt…
