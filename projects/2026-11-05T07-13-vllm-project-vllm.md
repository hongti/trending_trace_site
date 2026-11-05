# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-05 15:13 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 📋 Issue 动态
- **支持 `vllm.Inference` 扩展功能 (#65841)**：呼吁在 `vllm.Inference` 中支持 `n > 1` 的采样以及多模态处理器参数。此举旨在解决网关在入口处统一分词后调用 worker 时遇到的功能缺失问题。

### 🔧 Pull Request 动态
本期 PR 主要集中在性能优化、特定硬件（ROCm/XPU）适配升级及多个 Bug 修复：

**性能优化**
- **降低 KV 块分配开销 (#65831)**：减少了调度器主线程在 KV 块分配和缓存时的每块 Python 计算成本，该优化随提示词长度线性收益，显著降低 CPU 开销。
- **MiniMax-M3 跨层共享优化 (#65838)**：将原本仅限 AMD 的跨层索引 top-k 共享特性移植到 NVIDIA 路径，开启 `use_index_cache` 时可大幅减少冗余计算。
- **Qwen4Exp 投机解码优化 (#65836)**：为 NVIDIA MTP 实现新增 `get_top_tokens()`，启用本地 argmax 归约优化贪婪草稿 token 的选择。

**Bug 修复**
- **修复注意力机制合并逻辑 (#65839)**：修复 `SlidingWindowMLASpec.merge` 未正确比较 `AttentionSpec` 声明字段的问题。
- **修复 Gemma 4 纯文本模型加载 (#65835)**：修复 Gemma 4 在无视觉塔配置（`vision_config: null`，如纯文本微调模型）下构建失败报错的 Bug。

**硬件适配与优化（ROCm / XPU）**
- **ROCm / DeepSeek V4.1 优化**：
  - #65837：使用 AITER 的融合双 RMSNorm 对共享 query 进行一次性量化。
  - #65833：在 ROCm 上将 `SiluAndMulWithClamp` 接入 AITER 的融合 clamped SwiGLU。
  - #65832：修复 CSA 注意力路径中多流 GEMM token 阈值未生效的问题。
- **XPU 测试覆盖 (#65840)**：解除 Kimi-K3 KDA 元数据测试对 CUDA 的硬编码限制，使其能在 XPU 设备上运行。

**文档与其他**
- **修复 API 文档构建 (#65834)**：补全 `decode_attention_fwd_verify` 缺失的参数类型声明，修复导致 Read the Docs 严格构建中断的问题。

### 🚀 Release 动态
- 本次更新数据中未包含版本发布（Release）信息。

---

## 🐛 Issues

### #65841 — [[Feature]: Support n > 1 sampling and multimodal processor kwargs in vllm.Inference](https://github.com/vllm-project/vllm/issues/65841)
- **作者**: ahg-g  **时间**: 2026-11-05 15:12 CST
- **摘要**: ### 🚀 The feature, motivation and pitch  Gateways that tokenize once at ingress and call workers over `vllm.Inference` (`rust/proto/inference.proto`, llm-d/llm-d-router#3758) hit two feature gaps against the OpenAI HTTP routes:  1. **No `n > 1` multi-choice sampling.** OpenAI Chat (`/v1/chat/complet…

## 🔀 Pull Requests

### #65840 — [[XPU][Kimi-K3] Run KDA metadata tests on XPU](https://github.com/vllm-project/vllm/pull/65840)
- **作者**: Liangliang-Ma  **时间**: 2026-11-05 15:10 CST
- **标签**: intel-gpu, needs-rebase, kimi
- **摘要**: The Kimi-K3 KDA metadata builder only launches Triton kernels, but its tests were gated on torch.cuda.is_available() with a hard-coded "cuda" device, and the kernel wrappers asserted tensor.is_cuda. On XPU this skipped 20 cases that pass there.  Gate the tests on a CUDA-alike or XPU device, use the …

### #65839 — [[Bugfix][Core] Compare all attention fields in SlidingWindowMLASpec.merge](https://github.com/vllm-project/vllm/pull/65839)
- **作者**: yzeyu71  **时间**: 2026-11-05 14:54 CST
- **标签**: bug, kv-cache-manager
- **摘要**: ## Overview  `SlidingWindowMLASpec.merge` is the only attention `merge` that does not compare the fields declared on `AttentionSpec`. `FullAttentionSpec.merge`, `MLAAttentionSpec.merge` and `SinkFullAttentionSpec.merge` each end with the same loop over `fields(AttentionSpec)`, and `RSWASpec.merge` d…

### #65838 — [[Perf][MiniMax-M3] Port cross-layer index top-k sharing to the NVIDIA path](https://github.com/vllm-project/vllm/pull/65838)
- **作者**: grYe99  **时间**: 2026-11-05 14:50 CST
- **标签**: ci/build, nvidia, minimax
- **摘要**: ## Summary  Adds opt-in **cross-layer sharing** of the lightning-indexer top-k selection to the NVIDIA MiniMax-M3 path, porting #47269 (AMD-only until now). When `use_index_cache` is set, only 1 of every `index_topk_freq` sparse-attention layers recomputes the indexer score + top-k; the rest reuse t…

### #65837 — [[ROCm][DSV4.1] Quantize the shared query once with AITER's fused dual RMSNorm](https://github.com/vllm-project/vllm/pull/65837)
- **作者**: xudonlyu  **时间**: 2026-11-05 14:46 CST
- **标签**: rocm, deepseek, quantization
- **摘要**: ## Purpose  DeepSeek V4.1 normalizes the q-lora and kv halves of the fused down-projection together. Where both query projections (`wq_b` and the indexer's `wq_b`) take MXFP8 input, it quantizes the normalized Q **once** and hands both projections the same `QuantizedActivation` (`fused_q_kv_rmsnorm_…

### #65836 — [[Spec Decode][Qwen4Exp] Enable local argmax reduction for NVIDIA MTP](https://github.com/vllm-project/vllm/pull/65836)
- **作者**: irenjj  **时间**: 2026-11-05 14:40 CST
- **标签**: documentation, qwen, nvidia
- **摘要**: ## Overview  Add `get_top_tokens()` to the NVIDIA `Qwen4ExpMTP` implementation, enabling the existing `use_local_argmax_reduction` option for greedy draft-token selection. The adapter reuses `LogitsProcessor.get_top_tokens()` and the same head/hidden-state pair as `compute_logits()`; the MTP guide d…

### #65835 — [[Bugfix][Model] Gemma 4: support checkpoints without a vision tower](https://github.com/vllm-project/vllm/pull/65835)
- **作者**: kunal-prodigal  **时间**: 2026-11-05 14:39 CST
- **标签**: bug, multi-modality
- **摘要**: ## Overview  `Gemma4ForConditionalGeneration` treats `audio_config` as optional but always builds the vision tower, so Gemma 4 checkpoints saved with `vision_config: null` (text-only fine-tunes) fail to load, even with `--language-model-only`. This PR handles `vision_config` the same way as `audio_c…

### #65834 — [[Docs] Fix missing MLA verify parameter types in API docs](https://github.com/vllm-project/vllm/pull/65834)
- **作者**: Liamup777  **时间**: 2026-11-05 14:38 CST
- **标签**: ready
- **摘要**: ## Overview  Fix strict API documentation builds by adding the missing Google-style argument types to `decode_attention_fwd_verify`, introduced in #62725.  The 14 Griffe warnings abort Read the Docs builds, including [#64689](https://app.readthedocs.org/projects/vllm/builds/35661546/) and [#51808](h…

### #65833 — [[ROCm] Run SiluAndMulWithClamp through AITER's fused clamped SwiGLU](https://github.com/vllm-project/vllm/pull/65833)
- **作者**: xudonlyu  **时间**: 2026-11-05 14:38 CST
- **标签**: rocm
- **摘要**: ## Purpose  On ROCm, `SiluAndMulWithClamp.forward_hip` uses the `_C` `silu_and_mul_with_clamp` kernel for the default `alpha=1, beta=0` configuration, and the native implementation otherwise. The `_C` restriction exists because that kernel rounds its SiLU intermediate to the activation dtype (the pr…

### #65832 — [[ROCm][DSV4.1] Honor the multi-stream GEMM token threshold in the CSA attention path](https://github.com/vllm-project/vllm/pull/65832)
- **作者**: xudonlyu  **时间**: 2026-11-05 14:38 CST
- **标签**: rocm, ci/build, deepseek
- **摘要**: ## Purpose  `DeepseekV41ROCMAiterMLAAttention.forward` forks the CSA input chains (Q/KV projections, compressor, indexer inputs) onto the aux streams for **every** batch. The shared path it parallels, `DeepseekV4Attention._run_parallel_input_projections`, only overlaps its input GEMMs while  ```pyth…

### #65831 — [[Perf][Core] Reduce per-block CPU cost of KV block allocation and caching](https://github.com/vllm-project/vllm/pull/65831)
- **作者**: njhill  **时间**: 2026-11-05 14:36 CST
- **标签**: kv-connector, scheduler, kv-cache-manager, nixl
- **摘要**: ## Overview  Reduces the per-block Python work on the scheduler's main thread when an async KV load starts, is promoted to running, and finishes. These costs grow linearly with prompt length, and on a DP decode instance a slow `schedule()` on one rank holds up the per-step all-reduce.  Related: #617…
