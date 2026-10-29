# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-29 14:18 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要：

### 📋 Issues (问题与提案)
近期提案主要聚焦于 **MRV2（Model Runner V2）的 DBO（Decode Bo Overlap）** 机制与特定硬件的性能优化：
*   **MRV2 DBO 架构扩展** (#64440, #64439)：提议在 MRV2 中实现并验证 scoped eager decode overlap 路径，分别针对“仅有 attention 的 LoRA”以及“混合 attention/循环 MoE 模型”，旨在降低解码延迟。
*   **SM90 性能评估** (#64446)：提议在 SM90 架构、TP2 配置下，针对 DeepSeek-V4.1 的 mHC 边界融合（旋转权重场景）进行基准测试优先的性能优化。

### 🔀 Pull Requests (代码与变更)
PR 活动密集，主要围绕 **DeepSeek-V4.1 模型支持**、**推测解码** 以及多项核心 Bug 修复：

**🚀 新特性与性能优化**
*   **DeepSeek-V4.1 多模态支持** (#64435)：在 MRV2 PCP 上支持 DeepSeek-V4.1 的图像输入，利用现有编码器缓存和 PCP 段映射实现。
*   **SM100 性能优化** (#64447)：跨层共享 prefill indexer 的 logits schedule，避免在每层重复构建，提升 SM100 上的计算效率。
*   **DSpark 采样加速** (#64442)：通过逆 CDF 将 DeepSeek-V4.1 的 Markov 采样循环融合，在 NVIDIA GPU 上提升草稿采样性能。
*   **推测解码增强** (#64445)：新增 MRV2 语法感知 draft proposal bridge，支持验证 token 和结构化输出，并为未来的 EAGLE-4 插件铺路。

**🐛 关键 Bug 修复**
*   **DeepSeek-V4.1 路由修复** (#64443)：修复了解码侧 SWA bounded replay 中的图填充行未被正确从专家路由中屏蔽的问题。
*   **KV Offload 竞态修复** (#64444)：修复了 store 触发的无效化中的竞态问题，成功存储后会发布已知探测结果以更新 lookup state。
*   **MoE 索引与 LoRA 修复** (#64441, #64437)：规范化 batch-invariant 模式下的稀疏索引 top-k 排序；为 Gemma4 启用 3D MoE LoRA 包装器，修复其无法加载 PEFT 格式 adapter 的问题。

**🛠️ 工具与测试**
*   **MoE 基准测试扩展** (#64438)：使 `benchmark_moe.py` 能够支持调优 Qwen4Exp 架构（如 Qwen3.8-Flash-Next）。
*   **测试废弃** (#64434)：关于零 max-concurrency 的测试已被 #64390 取代。

### 🚢 Releases (版本发布)
*   本期动态中未包含版本发布信息。

---

## 🐛 Issues

### #64446 — [[Performance]: Evaluate SM90 TP2 mHC boundary fusion under rotating weights](https://github.com/vllm-project/vllm/issues/64446)
- **作者**: link-tools-01  **时间**: 2026-10-29 14:09 CST
- **摘要**: ### Proposal to improve performance  I'd like to propose a benchmark-first contribution for the DeepSeek-V4.1 attention-to-FFN mHC boundary on **SM90, TP2, M=1, hidden=5120, hc_mult=4**. The target is the complete all-reduce + delayed mHC post/projection + first-consumer boundary, including completi…

### #64440 — [[Feature]: [MRV2][DBO] Scoped eager decode overlap with attention-only LoRA](https://github.com/vllm-project/vllm/issues/64440)
- **作者**: algebra-MCX  **时间**: 2026-10-29 13:44 CST
- **摘要**: ### 🚀 The feature, motivation and pitch  **I (@algebra-MCX) propose to implement and validate a scoped LoRA + DBO path in MRV2, subject to maintainer feedback.** This is the standalone F02 feature track under [#64100](https://github.com/vllm-project/vllm/issues/64100). Implementation, selection of a…

### #64439 — [[Feature]: [MRV2][DBO] Scoped eager decode overlap for hybrid attention/recurrent MoE models](https://github.com/vllm-project/vllm/issues/64439)
- **作者**: algebra-MCX  **时间**: 2026-10-29 13:44 CST
- **标签**: quantization
- **摘要**: ### 🚀 The feature, motivation and pitch  **I (@algebra-MCX) propose to implement and validate a scoped hybrid attention/recurrent MoE DBO path in MRV2, subject to maintainer feedback.** This turns the previously investigated F01 task into a standalone feature issue under [#64100](https://github.com/…

## 🔀 Pull Requests

### #64447 — [[Perf][DSA] Share the prefill indexer's logits schedule across layers on SM100](https://github.com/vllm-project/vllm/pull/64447)
- **作者**: Juntian777  **时间**: 2026-10-29 14:14 CST
- **摘要**: ## Overview  On SM100, the sparse indexer's prefill builds DeepGEMM's per-SM `mqa_logits` schedule once per prefill chunk and passes it to `fp8_fp4_mqa_logits` in every indexer layer that scores the chunk. Without a schedule, DeepGEMM runs its unbalanced kernel variant, which is slow when rows' KV r…

### #64445 — [[Feature][Spec Decode] Add MRV2 grammar-aware draft proposal bridge](https://github.com/vllm-project/vllm/pull/64445)
- **作者**: larryli2-amd  **时间**: 2026-10-29 14:08 CST
- **标签**: speculative-decoding, mrv2
- **摘要**: ## Overview  Add an opt-in MRV2 draft-proposal grammar bridge for speculative methods, including future EAGLE-4 plugins.  The bridge gives a speculator verified tokens and serializable structured-output request metadata before the next proposal. A speculator that cannot install an exact device-side …

### #64444 — [[Bugfix][KV Offload] Publish successful stores to lookup state](https://github.com/vllm-project/vllm/pull/64444)
- **作者**: Alex-ai-future  **时间**: 2026-10-29 14:01 CST
- **标签**: bug
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Builds on the store-triggered invalidation introduced in [#61187](https://github.com/vllm-project/vllm/pull/61187) and closes its in-flight probe race. Successful stores publish a known-present verdict to the lookup cache in both filesystem and object-stor…

### #64443 — [[Bugfix][DSv4.1] Mask replay-graph padding rows out of expert routing](https://github.com/vllm-project/vllm/pull/64443)
- **作者**: gcanlin  **时间**: 2026-10-29 13:55 CST
- **标签**: bug, deepseek, DSv4.1
- **摘要**: ## Overview  DeepSeek-V4.1's decoder-side SWA bounded replay pads its sub-batch to the replay CUDA graph size (the largest DP rank's under DP), but the replay forward context passes `is_padding` only under PCP. The padding rows go through expert routing; with garbage hidden states they all pick expe…

### #64442 — [[Perf][DSpark][DSv4.1] Fuse the Markov sampling loop on NVIDIA by inverse CDF](https://github.com/vllm-project/vllm/pull/64442)
- **作者**: Juntian777  **时间**: 2026-10-29 13:52 CST
- **标签**: ci/build, deepseek, nvidia, dflash, DSv4.1
- **摘要**: ## Overview  Runs DSpark's sequential Markov sampling for DeepSeek-V4.1 on NVIDIA as the existing bias GEMM plus two Triton kernels per draft step, through the draft model's `sample_sequential` hook (as the ROCm model does). Sampled draft tokens are drawn by inverse CDF of the cached draft logits in…

### #64441 — [[Bugfix] Canonicalize sparse indexer top-k order in batch-invariant mode](https://github.com/vllm-project/vllm/pull/64441)
- **作者**: hlin99  **时间**: 2026-10-29 13:49 CST
- **标签**: bug, documentation, ci/build
- **摘要**: ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this PR.  - [ ] **Design Fit:** Minimizes…

### #64438 — [[Misc][MoE] Support Qwen4Exp in benchmark_moe.py](https://github.com/vllm-project/vllm/pull/64438)
- **作者**: cdreetz  **时间**: 2026-10-29 13:32 CST
- **标签**: performance, qwen
- **摘要**: ## Overview  `benchmarks/kernels/benchmark_moe.py` cannot tune `Qwen/Qwen3.8-Flash-Next` (`Qwen4ExpForConditionalGeneration`): `get_model_params()` falls through to the default branch and fails on the nested text config.  ## Claims  - The tuner runs on `Qwen/Qwen3.8-Flash-Next-FP8` and writes a conf…

### #64437 — [[Bugfix][LoRA] Use 3D MoE LoRA wrapper for Gemma4](https://github.com/vllm-project/vllm/pull/64437)
- **作者**: gc-fu  **时间**: 2026-10-29 13:30 CST
- **标签**: bug
- **摘要**: ## Overview  Fixes #64436. Gemma4 MoE expert LoRA adapters produced by PEFT use the fused 3D layout, but Gemma4 does not declare `is_3d_moe_weight`, so loading them with the default configuration raises a `ValueError` inside EngineCore and kills the engine. This PR declares `is_3d_moe_weight` on bot…

### #64435 — [[DSv4.1][PCP] Support image input](https://github.com/vllm-project/vllm/pull/64435)
- **作者**: xijiaat  **时间**: 2026-10-29 13:22 CST
- **标签**: documentation, ci/build, deepseek, mrv2, dflash, DSv4.1
- **摘要**: ## Overview  Enable DeepSeek-V4.1 image inputs with Model Runner V2 PCP on NVIDIA GPUs, including fixed-length DSpark.  ## Claims  - Use the existing encoder cache and PCP segment mapping for image inputs. - Skip empty PCP windows so idle ranks do not request invalid image slices.  ## Validation  H2…

### #64434 — [WIP: tests for zero max-concurrency (superseded by #64390)](https://github.com/vllm-project/vllm/pull/64434)
- **作者**: MayaJShetty  **时间**: 2026-10-29 13:20 CST
- **标签**: rust
- **摘要**: ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this PR.  - [ ] **Design Fit:** Minimizes…
