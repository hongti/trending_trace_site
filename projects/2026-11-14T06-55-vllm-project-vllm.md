# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-14 14:55 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
- **Blackwell 平台 MoE 加载失败 (#67603)**：报告了在 NVIDIA B300 (Blackwell) 上使用 Quark W4A4 (`w_mxfp4_a_mxfp4`) 加载 gpt-oss MoE 模型时失败。根本原因是 CUTLASS W4A4 内核尚不支持 expert biases，但在校验环节被错误选中，导致 bias 数据被静默丢弃。

### 🔧 PR 动态
**核心修复**
- **MoE 内核选择修复 (#67609)**：针对上述 Issue，禁用带有 expert biases 的层使用 `CUTLASS W4A4 MXFP4 MoE` 内核，避免在 SM100 架构上静默丢失 bias 数据。
- **多模态请求重试机制 (#67604)**：将 KV cache 和 encoder cache 加载失败标记为“可重试”，使客户端能够自动重试失败的多模态请求。
- **基准测试修正 (#67608)**：修复了 `vllm bench serve` 在高并发场景下首字延迟 (TTFT) 报告过高的问题，仅在有代理时才信任环境变量。

**新特性与功能增强**
- **v1 Worker CUDA Graph API (#67602)**：在 v1 GPU worker 层面引入了 CUDA graph 的释放和重新捕获 API (`release_cuda_graphs` 和 `recapture_cuda_graphs`)，提升弹性执行能力。
- **推测解码指标增强 (#67610)**：引入了“每位置接受率”统计指标，用于分析短于 `num_speculative_tokens` 的 drafts 在各位置的接受情况。
- **性能预设更新 (#67600)**：为 Kimi-K3.1-Pro 新增了覆盖所有六种并发设置的 MI355X AgentX 预设方案。

**CI 与测试稳定性修复**
- **ROCm 平台 (#67605, #67606)**：修复了 Canary WER 正确性测试在 AMD MI355 上的 flaky 问题（改用 fp32 运行）；新增 AMD CI 测试超时时的子进程堆栈转储功能，便于排查引擎挂起问题。
- **KV Offload 端口冲突 (#67607)**：修复了 P2P 生命周期测试中因端口预留竞争导致的间歇性失败。
- **XPU 平台 OOM 修复 (#67601)**：限制了 DeepStack 测试的 profiling 行为，避免在仅校验 384×384 图像时因按模型最大尺寸分配内存（256 GiB）而导致的 OOM 问题。

### 🚀 Release 动态
本次提供的时间段内暂无新的 Release 版本发布。

---

## 🐛 Issues

### #67603 — [[Bug]: Quark W4A4 (`w_mxfp4_a_mxfp4`) gpt-oss MoE fails to load on Blackwell, and the CUTLASS W4A4 kernel silently drops expert biases](https://github.com/vllm-project/vllm/issues/67603)
- **作者**: kannakAWS  **时间**: 2026-11-14 13:17 CST
- **标签**: bug, gpt-oss, nvidia
- **摘要**: ### Your current environment vLLM `main` @ `3d091b6f0` (nightly wheel `0.34.0rc1.dev460+g3d091b6f0`), `amd-quark` 0.13, torch 2.14.0, FlashInfer 0.7.4, 1x NVIDIA B300 (SM 10.3). <!-- collect_env.py -->  ### 🐛 Describe the bug On SM100, a Quark `w_mxfp4_a_mxfp4` gpt-oss checkpoint selects `CUTLASS_MX…

## 🔀 Pull Requests

### #67610 — [[Spec Decode] Per-position acceptance rate over the drafts that reached each position](https://github.com/vllm-project/vllm/pull/67610)
- **作者**: LantaoJin  **时间**: 2026-11-14 14:17 CST
- **标签**: documentation, performance, speculative-decoding, rust
- **摘要**: ## Overview  The per-position acceptance rate divides the accepted tokens at position `i` by the number of drafts. When drafts are shorter than `num_speculative_tokens`, for example under dynamic speculative decoding, that under-reports every later position, and a position no draft reached shows `0.…

### #67609 — [ [Bugfix][MoE] Don't select CUTLASS W4A4 MXFP4 MoE for layers with expert biases](https://github.com/vllm-project/vllm/pull/67609)
- **作者**: kannakAWS  **时间**: 2026-11-14 13:51 CST
- **标签**: bug, nvidia
- **摘要**: ## Overview  `CutlassExpertsMxfp4` (`CUTLASS_MXFP4_MXFP4`) has no bias epilogue but accepts `has_bias=True` layers, so the MXFP4 oracle picks it for gpt-oss W4A4 (Quark `w_mxfp4_a_mxfp4`) on SM100. That fails to load today and drops the expert biases if the missing converter branch is added. This PR…

### #67608 — [[Bugfix][Benchmark] Trust the environment only when a proxy applies](https://github.com/vllm-project/vllm/pull/67608)
- **作者**: oscar030406  **时间**: 2026-11-14 13:48 CST
- **标签**: bug, performance, rust
- **摘要**: ## Overview  `vllm bench serve` reports TTFT that is too high at high concurrency. Against a server whose time to the first token is fixed at 51.1 ms, `vllm bench serve --max-concurrency 256` reports a mean TTFT of 73.7 ms for the requests after the first burst. With this PR it reports 51.5 ms.  The…

### #67607 — [[KV Offload][Bugfix] Hold P2P e2e test ports until the tiers bind them](https://github.com/vllm-project/vllm/pull/67607)
- **作者**: AndreasKaratzas  **时间**: 2026-11-14 13:45 CST
- **标签**: bug, nixl
- **摘要**: ## Overview  Fixes a port-reservation race that makes `tests/v1/kv_offload/tiering/test_p2p_lifecycle_e2e.py` fail intermittently in the `:amd: (MI355 DPX) V1 KV Offload` CI job with `ZMQError: Address already in use`. This changes only the test file.  ## Claims  - The P2P lifecycle e2e tests no lon…

### #67606 — [[ROCm][Bugfix] Run the Canary WER correctness test in fp32](https://github.com/vllm-project/vllm/pull/67606)
- **作者**: AndreasKaratzas  **时间**: 2026-11-14 13:44 CST
- **标签**: bug, rocm
- **摘要**: ## Overview  Fixes the flaky `test_wer_correctness[...-model_config2]` (nvidia/canary-1b-v2) in `:amd: (MI355 DPX) Entrypoints Integration (Speech to Text)`. The test now runs Canary in fp32 and compares against the deterministic fp32 baseline. Related: #64215, which added the Canary case and its 12…

### #67605 — [[ROCm][CI] Dump child process stacks when an AMD CI test times out](https://github.com/vllm-project/vllm/pull/67605)
- **作者**: AndreasKaratzas  **时间**: 2026-11-14 13:43 CST
- **标签**: rocm, ci/build
- **摘要**: ## Overview  AMD CI jobs (e.g. V1 Metrics in [build 102446](https://buildkite.com/vllm/ci/builds/102446)) sometimes hang while an EngineCore starts and end with `Timeout (0:25:00)!`, but the dump only shows the pytest process. This makes the `amd_ci_fast_exit` plugin also dump the stacks and kernel …

### #67604 — [[Bugfix][Inference]_Mark_cache_load_failures_retryable](https://github.com/vllm-project/vllm/pull/67604)
- **作者**: sudhanshu112233shukla  **时间**: 2026-11-14 13:19 CST
- **标签**: bug, scheduler
- **摘要**: Fixes #67430: Mark KV and encoder-cache load failures as retryable so clients can retry failed multimodal requests

### #67602 — [feat(v1): worker-level CUDA graph release and re-capture APIs](https://github.com/vllm-project/vllm/pull/67602)
- **作者**: akshaver  **时间**: 2026-11-14 13:17 CST
- **标签**: needs-rebase, nvidia
- **摘要**: ## Overview  Move CUDA-graph release and re-capture out of the elastic-EP executor and onto the v1 GPU worker: `Worker.release_cuda_graphs()` and `Worker.recapture_cuda_graphs()`. Elastic EP delegates to them (no behavior change); runtime features that invalidate captured graphs get a supported path…

### #67601 — [[CI][XPU] Bound DeepStack test profiling to avoid OOM](https://github.com/vllm-project/vllm/pull/67601)
- **作者**: hlin99  **时间**: 2026-11-14 12:55 CST
- **标签**: intel-gpu, multi-modality
- **摘要**: The test validates DeepStack correctness using images resized to at most 384×384, but profiling still uses the model’s maximum image size. This creates an unnecessary 256 GiB allocation with FP32 SDPA.  Bound profiling to the test’s image dimensions and batch size. This fixes a platform-independent …

### #67600 — [[Recipe] Kimi-K3.1-Pro: MI355X AgentX preset covering all six InferenceX#5043 concurrencies](https://github.com/vllm-project/vllm/pull/67600)
- **作者**: seungrokj  **时间**: 2026-11-14 12:24 CST
- **标签**: ready, kimi, recipe
- **摘要**: ## Overview  Replace the two Kimi-K3.1-Pro MI355X AgentX presets from #67487 (`mi355x_agentx` at concurrency 48 and `mi355x_agentx_c1` at concurrency 1) with one `mi355x_agentx` preset whose points cover every concurrency of the MI355X vLLM AgentX lane in [InferenceX #5043](https://github.com/SemiAn…
