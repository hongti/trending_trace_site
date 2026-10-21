# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-21 13:11 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 最近的动态摘要：

### 🐛 Issues (问题反馈)
本期共有 2 个重要的 Bug 报告：
1. **ROCm 滑动窗口内核输出 NaN (#62828)**：在 MI355X (gfx950) 环境下，vLLM 的 Triton 滑动窗口注意力内核存在 Bug。当位于窗口外的 KV 槽位持有 NaN 或 Inf 值时，会导致最终输出 NaN。
2. **FlashInfer fp4_gemm 自动调优导致输出改变 (#62820)**：在使用 SM120 架构运行 Qwen3.8-27B NVFP4 模型时，FlashInfer 的 fp4_gemm 自动调优机制会导致 temperature-0（温度为0）下的生成 token 发生改变，影响输出确定性。

### 🔧 Pull Requests (代码合并请求)
本期 PR 主要集中在 **Rust 前端功能对齐**、**性能优化**、**KV Cache 机制** 以及 **ROCm 基础设施升级**：

1. **Rust 前端与 Python 前端行为对齐** (由 BugenZhao 提交的系列 PR)：
   - 支持 `--custom-histogram-buckets` 参数。
   - 支持 `VLLM_LOG_STATS_INTERVAL` 和 `VLLM_DISABLE_REQUEST_ID_RANDOMIZATION` 环境变量。
   - 统一超长 prompt（超过 `max_model_len`）的报错信息与格式，使其与 Python 端一致。
   - 对齐 `response_format` 在 `structured_outputs` 上的应用逻辑。
   - 在 render 过程中透传 `return_token_ids` 参数。

2. **性能与新特性**：
   - **DeepSeek-V4.1 (DSv4.1) Prefill 性能优化 (#62817)**：针对带有 SWA（滑动窗口注意力）的有界重放机制进行优化。对于远离 prompt 末尾的 prefill chunk，仅重放一行数据而非运行全部重放层，从而减少不必要的计算。
   - **KV Cache 分配代次暴露 (#62819)**：暴露 KV 块的分配代次信息，允许异步连接器在物理 slot 释放后拒绝过期的控制完成状态，提升并发安全性。

3. **Bug 修复与依赖升级**：
   - **KV Connector 关闭修复 (#62821)**：修复 LMCache 适配器的关闭钩子转发问题，确保在 EngineCore 退出前正确释放 LMCache 服务。
   - **ROCm UCX 升级 (#62827)**：将 Dockerfile 中的 UCX 版本升级至 v1.23.0-rc1，使 NIXL 能够使用 `rocm_ipc`，并调整了相关内存池配置。
   - **量化依赖升级 (#62816)**：将 `humming-kernels` 升级至 0.1.18，并移除了之前临时使用的 `DeviceInfo` 猴子补丁。

### 🚀 Release (版本发布)
近期无新的正式版本发布动态。

---

## 🐛 Issues

### #62828 — [[Bug][ROCm]: ROCM_ATTN sliding-window kernels output NaN when an out-of-window KV slot holds NaN/Inf](https://github.com/vllm-project/vllm/issues/62828)
- **作者**: zhentaocc  **时间**: 2026-10-21 12:41 CST
- **标签**: rocm
- **摘要**: ### Your current environment  MI355X (gfx950), image `vllm/vllm-openai-rocm:nightly-rocm100-0cbac6cd1` (vLLM 0.30.1rc1.dev558). The kernels involved are vLLM's own Triton kernels, so other ROCm GPUs are affected too.  ### 🐛 Describe the bug  `ROCM_ATTN` can produce NaN attention output for layers wi…

### #62820 — [[Bug]: FlashInfer fp4_gemm autotune changes temperature-0 tokens on Qwen3.8-27B NVFP4 (SM120)](https://github.com/vllm-project/vllm/issues/62820)
- **作者**: jpvelasco  **时间**: 2026-10-21 12:07 CST
- **标签**: quantization
- **摘要**: ### Environment  `collect_env.py` was not captured from the container. The versions below are from the official image logs.  - Image: `vllm/vllm-openai:v0.32.0` - Digest: `sha256:befccf0e7e4089aa23926bb6f887a85c0d46c2a99e6de5c5ffdf5cb7957c5b34` - Engine: vLLM 0.32.0, V1 - FlashInfer: 0.7.0.post1 (au…

## 🔀 Pull Requests

### #62827 — [[ROCm] Bump UCX to v1.23.0-rc1 so NIXL uses rocm_ipc](https://github.com/vllm-project/vllm/pull/62827)
- **作者**: AndreasKaratzas  **时间**: 2026-10-21 12:38 CST
- **标签**: rocm, ci/build, kv-connector
- **摘要**: - Bump `UCX_BRANCH` to `v1.23.0-rc1` in `Dockerfile.rocm` and `Dockerfile.rocm_72` - Drop `UCX_RMA_PPLN_ENABLE=y` - Set `UCX_ROCM_IPC_SIGPOOL_MAX_ELEMS=inf`, matching the existing `rocm_copy` setting  NIXL opens its UCX endpoints with peer error handling, and at the current UCX pin `rocm_ipc` does n…

### #62826 — [[Rust Frontend] Support --custom-histogram-buckets](https://github.com/vllm-project/vllm/pull/62826)
- **作者**: BugenZhao  **时间**: 2026-10-21 12:20 CST
- **标签**: rust
- **摘要**: ## Overview  Python's `--custom-histogram-buckets` (vllm-project/vllm#48867) replaces the default bounds of a bucket family across all of its histograms. The Rust frontend rejects the flag at startup (placeholder from vllm-project/vllm#62431) and hardcodes the bucket lists per histogram. This PR imp…

### #62825 — [[Rust Frontend] Support VLLM_LOG_STATS_INTERVAL and VLLM_DISABLE_REQUEST_ID_RANDOMIZATION](https://github.com/vllm-project/vllm/pull/62825)
- **作者**: BugenZhao  **时间**: 2026-10-21 12:19 CST
- **标签**: rust
- **摘要**: ## Overview  The Python frontend reads two settings only from the environment: `VLLM_LOG_STATS_INTERVAL` paces the periodic stats log, and the deprecated `VLLM_DISABLE_REQUEST_ID_RANDOMIZATION` keeps external request IDs as engine-core request IDs. The Rust frontend ignores both. This PR honors them…

### #62824 — [[Rust Frontend] Report prompts over max_model_len like Python](https://github.com/vllm-project/vllm/pull/62824)
- **作者**: BugenZhao  **时间**: 2026-10-21 12:19 CST
- **标签**: rust
- **摘要**: ## Overview  A prompt longer than `max_model_len` fails on both frontends with a 400, but the Rust message differs from Python's and has no `param`. This PR reports it like Python: `This model's maximum context length is N tokens. However, your prompt contains M input tokens, which exceeds this limi…

### #62823 — [[Rust Frontend] Apply response_format over structured_outputs like Python](https://github.com/vllm-project/vllm/pull/62823)
- **作者**: BugenZhao  **时间**: 2026-10-21 12:19 CST
- **标签**: rust
- **摘要**: ## Overview  Python applies a non-`text` `response_format` on top of `structured_outputs` (`structured_outputs_from_response_format`, `vllm/entrypoints/generate/base/protocol.py`, used by chat and completions). The Rust frontend returns `structured_outputs` as soon as it is present and ignores `resp…

### #62822 — [[Rust Frontend] Carry return_token_ids through render like Python](https://github.com/vllm-project/vllm/pull/62822)
- **作者**: BugenZhao  **时间**: 2026-10-21 12:19 CST
- **标签**: rust
- **摘要**: ## Overview  Since vllm-project/vllm#62449, Python's `/v1/chat/completions/render` and `/v1/completions/render` copy the request's `return_token_ids` onto the rendered `GenerateRequest` (`vllm/entrypoints/scale_out/render/serving.py`). The Rust frontend's `lower_render_request` (`rust/src/server/src…

### #62821 — [[Bugfix][KV Connector] Forward LMCache adapter shutdown](https://github.com/vllm-project/vllm/pull/62821)
- **作者**: MuonChaser  **时间**: 2026-10-21 12:09 CST
- **标签**: bug, kv-connector
- **摘要**: ## Overview  Forward the built-in LMCache connector's shutdown hook to adapters that implement it, releasing LMCache services before EngineCore exits. Fixes #62808.  ## Claims  - Both scheduler and worker connectors delegate cleanup through the adapter's public `shutdown()`. - Native/older adapters …

### #62819 — [feat(kv-cache): expose block allocation generations](https://github.com/vllm-project/vllm/pull/62819)
- **作者**: feichai0017  **时间**: 2026-10-21 12:05 CST
- **标签**: kv-connector, scheduler, kv-cache-manager
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Expose allocator-issued KV block generations through allocation and request-finish connector handoffs, so asynchronous connectors can reject stale control completions after physical slot reuse. Closes #62810.  ## Claims  - `BlockPool` issues a process-loca…

### #62817 — [[Perf][DSv4.1] Replay one row for prefill chunks far from the prompt end](https://github.com/vllm-project/vllm/pull/62817)
- **作者**: ischencheng  **时间**: 2026-10-21 12:01 CST
- **标签**: deepseek, nvidia, mrv2, DSv4.1
- **摘要**: ## Overview  With decoder SWA bounded replay, every DeepSeek-V4.1 prefill chunk runs the replay layers (20-39) on its last 128 rows, but after the prefill only the prompt's last window is read. This PR replays one row for chunks that end far from the prompt's end. A 124K prompt in 16K chunks now ski…

### #62816 — [[Quantization] Bump humming-kernels to 0.1.18 and remove DeviceInfo workaround](https://github.com/vllm-project/vllm/pull/62816)
- **作者**: jinzhen-lin  **时间**: 2026-10-21 11:59 CST
- **标签**: ci/build, quantization
- **摘要**: ## Overview  Bump `humming-kernels[cu13]` to 0.1.18 and remove the temporary `DeviceInfo` monkey patch from #61858. Keep its fullgraph compilation regression test and Buildkite trigger unchanged.  ## Claims  - Use the upstream device-instance caching fix in [humming#149](https://github.com/vllm-proj…
