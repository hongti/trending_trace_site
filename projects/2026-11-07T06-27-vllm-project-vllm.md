# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-07 14:27 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的简洁摘要：

### 🚀 Pull Requests (PR)
本期 PR 主要集中在性能优化、新硬件/新模型支持以及底层架构改进：

**1. 新模型与硬件支持**
*   **MiniMax-M3 优化**：#66227 为 MiniMax-M3 添加了 DeepGEMM NVFP4 MegaMoE 内核的预热机制，避免其在服务启动时触发 JIT 编译导致延迟；#66228 更新了 `fmha_sm100` 代码，引入了 Q8KV4 和 Q8KV8 的 decode indexer 优化。
*   **无损 NF12 量化**：#66221 新增了对自定义 NF12 在线量化的支持，可将 BF16 权重无损转换为 NF12 格式，有效降低模型显存占用。
*   **ROCm / GLM-5.3-Flash 优化**：#66219 在 ROCm 平台上为 GLM-5.3-Flash 保留了融合的 Gated RMSNorm 算子；#66223 优化了 GDN chunk-state 内核的 V tile 选取逻辑，从批次中而非首次自动调优中获取，提升了并行效率。

**2. 核心与性能优化**
*   **投机解码**：#66226 新增配置项 `num_speculative_tokens_under_prefill_load`，在 prefill 负载较重的 decode 步骤中缩短 draft 长度，以提升整体吞吐。
*   **Hopper (SM90) 内核融合**：#66225 引入 FlashTopK，在 SM90 上实现了 MQA-logits 与 top-k 路径的融合，优化了 DSA sparse-indexer 的 prefill 性能。
*   **调度器简化**：#66224 重构了非数据并行模式下的 `prefill_schedule_interval` 调度逻辑，复用现有的 DP 机制，由引擎核心统一控制节奏，降低了代码复杂度。

**3. Rust 前端修复**
*   **HF 模板正则匹配**：#66220 和 #66222 将 Rust 的 HF 解析器切换为使用 PCRE2 进行正则匹配，修复了无法解析带有 look-around 语法的 `response_template` 的问题，使 Qwen3.5+、Mistral、GLM-5 等新模型检查点能正常加载。

### 🐛 Issues
本期提供的数据中未包含相关 Issue 动态。

### 📦 Release
本期提供的数据中未包含新的 Release 信息。

---

## 🔀 Pull Requests

### #66228 — [[Build][MiniMax-M3] Bump fmha_sm100 to 51f4f5c](https://github.com/vllm-project/vllm/pull/66228)
- **作者**: zyongye  **时间**: 2026-11-07 14:04 CST
- **标签**: ci/build, minimax
- **摘要**: ## Overview  Bumps `fmha_sm100` from `25c1447` to `51f4f5c`, the head of `vllm-project/MSA` `dev`. This picks up two commits: the Q8KV4 decode indexer optimizations (vllm-project/MSA#50) and the Q8KV8 prefill indexer rewrite (vllm-project/MSA#48). No open PR in this repo bumps this pin.  ## Claims  …

### #66227 — [[Perf][MiniMax-M3] Warm up DeepGEMM NVFP4 MegaMoE kernels before serving](https://github.com/vllm-project/vllm/pull/66227)
- **作者**: zyongye  **时间**: 2026-11-07 13:43 CST
- **标签**: ready, minimax
- **摘要**: ## Overview  `deep_gemm_warmup` only recognizes DeepSeek-V4 and Kimi K3 MoE layers, so MiniMax M3 `nvfp4_mega_moe` kernels JIT on whichever forward reaches them first. This adds `MiniMaxM3MoE` to the MegaMoE warmup and makes the NVFP4 warmup reach both launch variants of a BLOCK_M.  Follows #60468 (…

### #66226 — [[Feature][Spec Decode] Add spec config to shorten drafts on prefill-heavy steps](https://github.com/vllm-project/vllm/pull/66226)
- **作者**: MichaelLapshin  **时间**: 2026-11-07 12:59 CST
- **标签**: speculative-decoding, torch.compile, kv-connector, nvidia, mrv2, kimi, scheduler
- **摘要**: ## Overview    Add a new speculative config `num_speculative_tokens_under_prefill_load` to shorten draft length on prefill-heavy decode iterations.  ## Claims    TODO: Need to paste speedup data.  ## Validation    The implementation is unit test under the new file ``. Run the test with. ``` TODO: pu…

### #66225 — [[Perf][Kernel] Fused MQA+top-k DSA prefill on SM90 with FlashTopK](https://github.com/vllm-project/vllm/pull/66225)
- **作者**: Ma1oneZhang  **时间**: 2026-11-07 12:46 CST
- **标签**: performance, ci/build
- **摘要**: ## Overview  Run the DSA sparse-indexer prefill's fused MQA-logits + top-k path on Hopper (SM90) with [FlashTopK](https://github.com/Tencent-Hunyuan/FlashTopK), an exact fused indexer-scoring + top-k library (CuTe DSL / Triton, Apache-2.0). It goes behind the existing `fused_mqa_topk` entry points, …

### #66224 — [[Core] Simplify `prefill_schedule_interval` outside data parallelism](https://github.com/vllm-project/vllm/pull/66224)
- **作者**: njhill  **时间**: 2026-11-07 12:22 CST
- **标签**: ready, scheduler
- **摘要**: #54627 applied `prefill_schedule_interval` to non-DP engines with a separate gate and timing state in the scheduler. Replace that with the mechanism DP already uses: the engine core computes the cadence and passes it to `schedule()` as `throttle_prefills`. The base EngineCore now uses `num_executed_…

### #66223 — [[ROCm][Perf] Pick the GDN chunk-state V tile from the batch, not the first autotune](https://github.com/vllm-project/vllm/pull/66223)
- **作者**: ZhengGong-amd  **时间**: 2026-11-07 12:19 CST
- **标签**: rocm
- **摘要**: ## Overview  `chunk_gated_delta_rule_fwd_kernel_h_blockdim64` (the GDN/KDA chunk-state recurrence) is sequential over chunks, so its grid `(cdiv(V, BV), N * H)` is all the parallelism it has. `BV` is autotuned over `{32, 64}` with `key=["H", "K", "V", "BT"]`, which has no `N`: the first launch — the…

### #66222 — [[Rust Frontend] Match response_template regexes with PCRE2 in the hf parser](https://github.com/vllm-project/vllm/pull/66222)
- **作者**: BugenZhao  **时间**: 2026-11-07 12:08 CST
- **标签**: rust, beam
- **摘要**: ## Overview  The `hf` parser rejects `response_template`s whose boundaries use look-around, which includes the templates recently added to Qwen3.5+, Nemotron, Mistral, and GLM-5 checkpoints. This PR matches template regexes with PCRE2 and ports Transformers' `ResponseParser` scan onto it, instead of…

### #66221 — [[Quantization] feat: add support for lossless NF12 online quantization](https://github.com/vllm-project/vllm/pull/66221)
- **作者**: AlpinDale  **时间**: 2026-11-07 11:42 CST
- **标签**: documentation, ci/build, quantization
- **摘要**: ## Overview  This PR adds support for losslessly quantizing BF16 weights to a custom [NF12](https://www.alphaxiv.org/abs/2610.nyoomfloat12) format, which reduces the memory requirements for the model and moderately speeds up inference. The PR uses the [nf12](https://github.com/dphnAI/nf12) package f…

### #66220 — [[Rust Frontend] Support HF response-template lookarounds with PCRE2](https://github.com/vllm-project/vllm/pull/66220)
- **作者**: vyomakesh0728  **时间**: 2026-11-07 11:26 CST
- **标签**: documentation, tool-calling, rust
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Use PCRE2 for Rust HF response-template matching and partial detection, addressing [#65959](https://github.com/vllm-project/vllm/issues/65959). The current DFA/literal-prefix scanner rejects lookaround such as GLM’s `(?=</think>)`.  ## Claims  - Support PC…

### #66219 — [[ROCm][Perf][GLM-5.3-Flash] Keep the fused gated RMSNorm for the KDA o_norm](https://github.com/vllm-project/vllm/pull/66219)
- **作者**: ZhengGong-amd  **时间**: 2026-11-07 10:55 CST
- **标签**: rocm, glm
- **摘要**: ## Purpose  GLM-5.3-Flash's KDA output norm runs as an unfused eager decomposition on ROCm. `FusedRMSNormGated` is a `CustomOp`, and on ROCm `Glm5Next*` is not a default breakable-CUDA-graph architecture, so the model runs in `VLLM_COMPILE` mode with `custom_ops=["none"]` and `o_norm` dispatches `fo…
