# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-19 10:04 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue (问题反馈)
1. **流式输出静默中断 (xgrammar FSM 报错)**：在使用 `--enable-auto-tool-choice` 和 xgrammar 后端时，Qwen 3.8 27B NVFP4 模型的流式输出会缺失 `finish_reason` 或 `[DONE]` 标记，部分情况由 "Failed to advance FSM" 错误触发，部分则完全静默无报错 (#52852)。
2. **DeepSeek V4 工具调用参数缓冲延迟**：DeepSeek V4 在处理长字符串工具参数时，会一直缓冲直到遇到 `</parameter>` 才输出，该问题仅需解析器即可复现 (#52846)。
3. **MoE 基准测试崩溃**：`benchmark_moe.py` 在 NVIDIA 新架构显卡（如 RTX 5080, sm_120）上运行 `int4_w4a16` 精度调优时，因 `BLOCK_SIZE_K` 与 `group_size` 不匹配而崩溃 (#52845)。

---

### 🔀 Pull Request (代码合并)

**核心修复与稳定性：**
- **Blackwell MXFP4 修复**：为 Consumer Blackwell 架构强制设置 MXFP4 持久化内核约束，修复 `matmul_ogs` 运行问题 (#52850)。
- **Rust 前端修复**：修复 Rust 前端 `/inference/v1/generate` 路由静默丢弃 `n > 1` 参数的问题，改为显式拒绝 (#52844)。
- **MiniMax-M3 修复**：将解码索引 tile 补齐至 2 的幂次方，修复 `_decode_index_score_kernel` 的 lane tile 问题 (#52847)。
- **DeepSeek-V4 测试修复**：完善 Fused Shared Expert (FSE) 测试夹具，修复主分支回归问题 (#52842)。

**新特性与性能优化：**
- **Rust 前端增强**：新增 ERNIE 4.5 的工具与推理解析器 (#52841)；通过 gRPC API 新增 LoRA 生命周期控制（加载/卸载/列表） (#52840)。
- **ROCm 性能优化**：为 MiniMax-M3 的 MTP 和 dense 层启用 AITER PA gluon decode，使 EAGLE3 投机解码不再需要回退到原生 unified_attention (#52849)。
- **核心调度优化**：为共享内存（shm）广播忙循环添加可配置的休眠机制，允许降低 CPU 轮询占用 (#52848)。

**文档与 CI：**
- **文档修正**：澄清 `inter_token_latency_seconds` 指标是按引擎步长（而非单个 token）计算的，以避免在投机解码场景下产生误解 (#52853)。
- **CI 改进**：添加无依赖的本地 OpenTelemetry (OTel) 追踪辅助脚本，并将可变运行时状态移至临时目录 (#52851)。

---

### 🚀 Release (版本发布)
*近期暂无新版发布动态。*

---

## 🐛 Issues

### #52852 — [[Bug] Streaming completions end with no finish_reason / no [DONE] — xgrammar "Failed to advance FSM" is one logged trigger, but some drops are fully silent (no engine error)](https://github.com/vllm-project/vllm/issues/52852)
- **作者**: Duckmanjbr  **时间**: 2026-08-19 09:24 CST
- **摘要**: ### Summary  While serving a Qwen 3.8 27B NVFP4 checkpoint with `--enable-auto-tool-choice` (xgrammar FSM backend for tool-call constrained decoding), the engine logs a `Failed to advance FSM` ERROR for some requests, and the HTTP SSE stream for those requests terminates **without a final `finish_re…

### #52846 — [[Bug]: DeepSeek V4 buffers long string tool arguments until </parameter>](https://github.com/vllm-project/vllm/issues/52846)
- **作者**: QwertyJack  **时间**: 2026-08-19 08:24 CST
- **摘要**: ### Your current environment  This is reproducible in the parser without a model or accelerator.  - Reproduced on vLLM `v0.25.1` (`752a3a504485790a2e8491cacbb35c137339ad34`). - Confirmed by source inspection on current `main` at `f1178f3a06fa30a0cc282376924210cedad08c44`: `deepseek_v4_config()` stil…

### #52845 — [[Bug]: benchmark_moe.py --tune --dtype int4_w4a16 crashes on NVIDIA (BLOCK_SIZE_K vs group_size)](https://github.com/vllm-project/vllm/issues/52845)
- **作者**: JeremiahM37  **时间**: 2026-08-19 08:23 CST
- **标签**: rocm
- **摘要**: ### Your current environment  ``` vLLM 0.27.1 (also on main) | torch 2.13.0+cu130 | Triton 3.7.1 NVIDIA GeForce RTX 5080 (sm_120), Ubuntu 24.04 (WSL2) ```  ### 🐛 Describe the bug  `benchmark_moe.py --tune --dtype int4_w4a16` aborts on the first config it evaluates on NVIDIA:  ```bash python benchmar…

## 🔀 Pull Requests

### #52853 — [[Doc] Clarify inter_token_latency_seconds is per engine step, not per token](https://github.com/vllm-project/vllm/pull/52853)
- **作者**: haasnani  **时间**: 2026-08-19 09:32 CST
- **标签**: documentation
- **摘要**: The design doc equated vllm:inter_token_latency_seconds with TPOT in two places. Since speculative decoding can emit several tokens per engine step and the recorded gap is not divided by tokens emitted, the metric's mean exceeds true per-token latency by a workload-dependent factor. Add a note expla…

### #52851 — [[CI] Add repository-local OTel tracing helpers](https://github.com/vllm-project/vllm/pull/52851)
- **作者**: khluu  **时间**: 2026-08-19 09:02 CST
- **标签**: ci/build
- **摘要**: ## Summary  - add the dependency-free command and pytest OTel helpers under `.buildkite/scripts/ci-otel` - keep mutable runtime state in a temporary directory rather than writing into the checkout - move the helper unit and fail-open tests alongside the runtime code  This PR only makes the helpers a…

### #52850 — [[Bugfix] Set the MXFP4 persistent-kernel constraint on consumer Blackwell](https://github.com/vllm-project/vllm/pull/52850)
- **作者**: guoriyue  **时间**: 2026-08-19 08:48 CST
- **标签**: bug, gpt-oss, quantization
- **摘要**: ## Purpose  `matmul_ogs` requires the persistent kernel whenever `has_native_mxfp()` is true:  ```python if w_scale is not None and w_scale.storage.layout.name is not None \         and not opt_flags.is_persistent and target_info.has_native_mxfp():     raise NotImplementedError(         "Must use pe…

### #52849 — [[ROCm][PERF] Enable AITER PA gluon decode for MiniMax-M3 MTP and dense layers](https://github.com/vllm-project/vllm/pull/52849)
- **作者**: ukannika  **时间**: 2026-08-19 08:34 CST
- **标签**: rocm
- **摘要**: ## Purpose The gluon paged-attention decode kernel handles multi-token query lengths, so EAGLE3 speculative decoding no longer has to fall back to native vllm unified_attention.   ## Test Plan Server cmd to run EAGLE3 speculative decoding   ``` vllm serve amd/MiniMax-M3-MXFP4  \ --served-model-name …

### #52848 — [[Core] Add configurable sleep to shm broadcast busy loop](https://github.com/vllm-project/vllm/pull/52848)
- **作者**: AlexLJC  **时间**: 2026-08-19 08:32 CST
- **摘要**: ## Summary  This adds a configurable, bounded sleep to the shared-memory reader's existing busy branch:  - `VLLM_SHM_BROADCAST_BUSY_SLEEP_S=0` is the default and preserves continuous polling. - A positive finite value sleeps for that duration after `sched_yield()` on each busy-path check. - Negative…

### #52847 — [[Bugfix] Pad the MiniMax-M3 decode index tile to a power of two](https://github.com/vllm-project/vllm/pull/52847)
- **作者**: guoriyue  **时间**: 2026-08-19 08:27 CST
- **标签**: bug
- **摘要**: ## Purpose  Fixes #49157.  `_decode_index_score_kernel` derives its lane tile directly from the head count:  ```python BLOCK_SIZE_HQ: tl.constexpr = num_idx_heads * BLOCK_SIZE_Q hq_offsets = tl.arange(0, BLOCK_SIZE_HQ) ```  `tl.arange` requires a power-of-two extent, so decode fails to compile whene…

### #52844 — [[Bugfix][Frontend] Reject n > 1 in the Rust /inference/v1/generate route](https://github.com/vllm-project/vllm/pull/52844)
- **作者**: qgallouedec  **时间**: 2026-08-19 08:23 CST
- **标签**: bug, rust
- **摘要**: ## Purpose  Closes #52843  With `VLLM_USE_RUST_FRONTEND=1`, `/inference/v1/generate` silently ignores `n > 1`: `vllm_text::SamplingParams` has no `n` field, so the key is dropped at deserialization and a single choice is returned. The Rust OpenAI routes already reject `n > 1` explicitly; this makes …

### #52842 — [[CI][Bugfix] Complete DeepSeek-V4 FSE test fixture contract](https://github.com/vllm-project/vllm/pull/52842)
- **作者**: khluu  **时间**: 2026-08-19 07:44 CST
- **标签**: bug, deepseek
- **摘要**: ## Purpose  Complete the test-fixture fix for the DeepSeek-V4 fused-shared-expert regression on `main`.  This builds directly on #52829 by @tdoublep. Their exact commit `038d308c4` is preserved as the first commit in this PR, retaining its original authorship. Buildkite #84493 showed that the origin…

### #52841 — [[Rust Frontend] Add ERNIE 4.5 tool and reasoning parsers](https://github.com/vllm-project/vllm/pull/52841)
- **作者**: JonSnow1807  **时间**: 2026-08-19 07:37 CST
- **标签**: rust
- **摘要**: ## Purpose  Add the `ernie45` tool parser and `ernie45` reasoning parser to the Rust frontend, closing two items of the parser parity checklist in #44280 (claimed [here](https://github.com/vllm-project/vllm/issues/44280#issuecomment-5334022394)).  ERNIE 4.5 thinking models (`baidu/ERNIE-4.5-21B-A3B-…

### #52840 — [feat(grpc): add Rust frontend LoRA lifecycle control](https://github.com/vllm-project/vllm/pull/52840)
- **作者**: connorcarpenter15  **时间**: 2026-08-19 06:46 CST
- **标签**: rust
- **摘要**: ## Purpose  Add runtime LoRA lifecycle operations to the Rust frontend's existing gRPC APIs.  - Add load, unload, and list RPCs to the existing `Control` service. - Let unary and streaming generation select a loaded adapter by name. - Reuse the same `LoraManager` registry for HTTP and gRPC so adapte…
