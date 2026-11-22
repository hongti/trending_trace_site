# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-22 15:20 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期的动态摘要：

### 🐛 Issue 动态
* **v0.34 模型加载回归问题 (#69217)**：有用户反馈在 vllm 0.34 版本中无法加载 `audarai/Audar-ASR-V1-Turbo` 模型（出现验证错误），但在 0.32 版本中可正常加载。

### 🚀 PR 动态

#### 1. 新模型与新特性支持
* **新增 Nemotron 3.6 Academia 模型 (#69214)**：引入了对 NVIDIA Nemotron 3.6 Academia 架构的支持，这是一个结合了 Gated Delta Product (GDP)、稠密注意力和 MLP 层的循环混合模型。
* **MRV2 PCP 支持 LoRA (#69221)**：为 MRV2 预填充上下文并行（PCP）路径增加了 LoRA 支持，涵盖 attention-only LoRA、`lm_head` LoRA 以及 routed-expert LoRA。
* **Phi-4-reasoning-vision 支持 Encoder CUDA Graph (#69223)**：为 `microsoft/Phi-4-reasoning-vision-15B` 的图像输入添加了 `SupportsEncoderCudaGraph` 支持，提升多模态处理效率。

#### 2. 性能优化
* **优化 FA4 H512 在 Hopper 上的性能 (#69218)**：针对 Hopper 架构，确保 FA4 的 head_dim 512 在 BF16/FP16 KV 下使用块大小 64，从而利用更快的 TMA 路径加载分页 K/V，优化 Gemma-4 的 KV 缓存管理。
* **优化 DeepSeek-V4.1 的 CuTe DSL mHC 边界 (#69216)**：在 SM90 架构上，调整了 DeepSeek-V4.1 的内核执行边界，使得在 SM80 CuTe DSL 内核之上运行融合 TileLang 内核，提升计算效率。

#### 3. 关键 Bug 修复
* **修复并行采样挂起与 KV 泄漏 (#69215)**：修复了 MoRIIO 在 WRITE 模式下，当请求 `n > 1`（如 RL rollouts 并行采样）时会导致永久挂起并在 prefill/decode 阶段泄漏 KV 缓存的问题。
* **修复 Executor 陈旧响应 Bug (#69222)**：修复了 `MultiprocExecutor.collective_rpc` 在某 worker 返回 `FAILURE` 时，未读取其他 worker 响应队列就抛出异常，导致后续 RPC 获得陈旧错误响应的问题。
* **修复 Jamba 工具调用丢失前置文本 (#69220)**：修复了使用 Jamba tool parser 流式输出时，如果文本与 `<tool_call>` 标签在同一 delta 中到达，该文本内容会被意外丢弃的问题。

#### 4. CI 与基准测试
* **CI/XPU 测试修复 (#69212)**：恢复了 GLM-5.3-Flash XPU 导入保护，移除了过时的 LoRA deselect，并恢复了 XPU 上的 Engram peer-group 验证测试。
* **修复融合集合通信基准测试 (#69219)**：修复了 `benchmark_fused_collective.py` 中的基线选择和标签问题，确保性能加速比计算准确。

### 📦 Release 动态
* 近期无新的版本发布。

---

## 🐛 Issues

### #69217 — [[Bug]: vllm 0.34 fails to load model: Audar ASR Turbo](https://github.com/vllm-project/vllm/issues/69217)
- **作者**: coder2543  **时间**: 2026-11-22 13:53 CST
- **标签**: bug
- **摘要**: ### Your current environment  Linux  ### 🐛 Describe the bug  The following model fails to load in vllm 0.34.   It does load in vllm 0.32.  vllm serve audarai/Audar-ASR-V1-Turbo  Error message: 1 validation error for ModelConfig   Value error, Unrecognized model type for subconfig: qwen3_asr_text.  #…

## 🔀 Pull Requests

### #69223 — [[MM][CG] Add encoder CUDA graph support for Phi-4-reasoning-vision](https://github.com/vllm-project/vllm/pull/69223)
- **作者**: shuoyue21  **时间**: 2026-11-22 15:11 CST
- **标签**: documentation, multi-modality, nvidia
- **摘要**: ## Overview  Add `SupportsEncoderCudaGraph` support for `Phi4ForCausalLMV` (`microsoft/Phi-4-reasoning-vision-15B`) image inputs. Related to #38175.  `Phi4ForCausalLMV` pairs a NaFlex `Siglip2Model` vision tower (`select_layers=[-2]`) with `LlavaMultiModalProjector`. We precompute bilinearly interpo…

### #69222 — [[Bugfix][Executor] Drain worker replies after a failed collective RPC](https://github.com/vllm-project/vllm/pull/69222)
- **作者**: zyulai  **时间**: 2026-11-22 15:10 CST
- **标签**: bug
- **摘要**: ## Overview  Fixes a stale-response bug in `MultiprocExecutor.collective_rpc`: when one worker returns `FAILURE`, `get_response()` raises before reading the other workers' response queues, so the next RPC on the same executor consumes the leftover reply.  Observed on a 4-worker PCP + EP LoRA server:…

### #69221 — [[Feature][PCP] Support attention, lm_head, and routed-expert LoRA with MRV2 PCP](https://github.com/vllm-project/vllm/pull/69221)
- **作者**: zyulai  **时间**: 2026-11-22 15:10 CST
- **标签**: documentation, deepseek, mrv2
- **摘要**: ## Overview  Adds LoRA support to MRV2 prefill context parallelism (PCP), closing #68890. This PR covers attention-only LoRA, `lm_head` LoRA, and routed-expert LoRA on the PCP-only path (`use_ep=False`). PCP + EP continues to reject adapters that contain expert weights, with an actionable error inst…

### #69220 — [[Bugfix][Tool Calling] Preserve leading content when Jamba tool calls share start delta (#69213)](https://github.com/vllm-project/vllm/pull/69220)
- **作者**: somuai  **时间**: 2026-11-22 14:16 CST
- **标签**: bug, tool-calling
- **摘要**: ## Problem When streaming with Jamba tool parser (`--enable-auto-tool-choice --tool-call-parser jamba`), any text arriving in the same delta as the `<tool_calls>` opening tag was dropped. As described in #69213, this occurs whenever a delta holds several tokens—such as with `stream_interval > 1`, du…

### #69219 — [[Bugfix][Benchmark] Fix fused collective speedup baselines](https://github.com/vllm-project/vllm/pull/69219)
- **作者**: getupandwashits  **时间**: 2026-11-22 13:57 CST
- **标签**: bug, performance
- **摘要**: ## Overview  Fixes #69210.  Fix baseline selection and labeling in `benchmarks/kernels/benchmark_fused_collective.py` so `prepare_results_with_speedups()` matches the standard keys emitted by `run_benchmarks()`, isolates FP8 standard rows from unquantized baselines, and labels whichever standard var…

### #69218 — [[Perf][Attention] Use block size 64 for FA4 H512 on Hopper with BF16/FP16 KV](https://github.com/vllm-project/vllm/pull/69218)
- **作者**: xuanzic  **时间**: 2026-11-22 13:53 CST
- **摘要**: ## Overview  On Hopper, FA4's head_dim 512 forward loads paged K/V with TMA only when `page_size == tile_n` (64). Any other page size takes the slower cp.async path. For Gemma-4, the hybrid KV cache manager gives the head_dim 512 layers blocks twice the base block size. Since the default block size …

### #69216 — [[Perf][Kernel][DSv4.1] Run the CuTe DSL mHC boundary on SM90 above the SM80 kernels' rows](https://github.com/vllm-project/vllm/pull/69216)
- **作者**: nguyenthai-duong  **时间**: 2026-11-22 13:46 CST
- **标签**: ci/build, deepseek
- **摘要**: <!-- markdownlint-disable -->  ## Overview  On SM90, DeepSeek-V4.1's shifted mHC boundary runs the SM80 CuTe DSL kernels (#66497) up to `SM90_MHC_MAX_TOKENS` (48) rows and the fused TileLang kernels above that. This change runs the single-kernel CuTe DSL boundary from #66652 on SM90 for those larger…

### #69215 — [[Bugfix][MoRIIO] Give WRITE parallel sampling siblings their own transfer id](https://github.com/vllm-project/vllm/pull/69215)
- **作者**: amd-dlimpus  **时间**: 2026-11-22 13:43 CST
- **标签**: bug, kv-connector
- **摘要**: ## Overview  With MoRIIO in WRITE mode, any request with `n > 1` (parallel sampling, e.g. RL rollouts) hangs forever and leaks KV on both prefill and decode. All `n` children carry the router's single `transfer_id`, and WRITE mode keys its pairing by `transfer_id` alone. This gives each child its ow…

### #69214 — [[Model] Add Nemotron 3.6 Academia support](https://github.com/vllm-project/vllm/pull/69214)
- **作者**: shan18  **时间**: 2026-11-22 13:22 CST
- **标签**: documentation, new-model, ci/build, nvidia, mrv2
- **摘要**: ## Overview  Adds support for NVIDIA Nemotron 3.6 Academia, a looped hybrid model combining Gated Delta Product (GDP), dense attention, and MLP layers.  ## Claims  - Support the nemotron 3.6 academia architecture with gated delta product layers, attention, and checkpoint-defined recurrent loops. - S…

### #69212 — [[CI][XPU] Restore GLM platform guard and Engram test coverage](https://github.com/vllm-project/vllm/pull/69212)
- **作者**: hlin99  **时间**: 2026-11-22 12:53 CST
- **标签**: intel-gpu, ci/build, glm
- **摘要**: Restore the GLM-5.3-Flash XPU import guard now that LoRA layer tests no longer depend on the GLM package (#68867). Remove the stale LoRA deselect and restore Engram peer-group validation on XPU with the capability mock from #68881. Keep the unsupported-model tensor-schema skips from #68971.
