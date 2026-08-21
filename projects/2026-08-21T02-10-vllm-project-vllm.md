# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-21 10:10 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文简洁摘要：

### 📌 Issue 动态
1. **Xgrammar FSM 推进失败 Bug 遗留** (#53181)：尽管此前的修复 PR (#52805) 已合入，但 "Failed to advance FSM" 错误在 72小时测试中依然触发。
2. **TurboQuant 与 MTP 推测解码冲突** (#53180)：使用 `turboquant_k8v4` KV缓存结合 MTP 推测解码时，在混合 GDN 模型上会产生静默的退化输出（生成质量异常）。
3. **新 RFC：实验性 Recirculation 机制** (#53178)：提议为因果解码器引入 Recirculation 技术，通过将深层残差流激活归一化后混合回浅层，在不改变模型权重的情况下提升预训练模型效果。

---

### 🔀 PR 动态
**🏗️ 架构与核心重构**
* **Model Runner V2 默认全量开启** (#53183)：将 MRV2 作为所有模型的默认运行器，目前正在冲洗排查 CI 兼容问题。
* **多模态 Encoder 路径剥离** (#53176)：将纯编码器（encoder-only）路径从共享 runner 中移出，便于独立运行视觉编码器及发布 embeddings。
* **实验性支持 Recirculation** (#53179)：配合上述 RFC，落地了 Recirculation 执行逻辑的实现。

**🚀 性能优化与新特性**
* **Humming W2 融合 SwiGLU FP8 量化** (#53184)：移植 DeepSeek-v4-pro-engine 的 W2 热路径优化，集成融合 SwiGLU FP8 量化以提升性能。
* **XPU 线性层权重布局优化** (#53185)：确保 XPU 未量化的线性层权重为 N-contiguous 布局，以提升 `torch.nn.functional.linear` 的计算性能。

**🐛 Bug 修复与回滚**
* **回滚 MoE FlashInfer 修复** (#53186)：回滚了 #52989（调整 FlashInfer 专家至调度器 token 限制的修复），因其导致 Nightly 构建失败。
* **修复 Step-3.5 MTP 与结构化输出冲突** (#53174)：修复了 MTP 启动时因 HF 验证覆盖 `layer_types` 导致的与结构化输出不兼容的问题。
* **修复 ROCm Profiler 追踪** (#53182)：在 ROCm 基础 Docker 镜像中内置 rocprofiler-sdk 1.3.2，修复 `torch.profiler` 的 HIP-graph 追踪问题。
* **重新提交 PR 48666** (#53175)：因之前被 CI 误报性能回退而关闭，现查明回退非本 PR 引起，予以重新提交。

**🧪 测试与 CI**
* **ROCm 分页注意力测试增强** (#53177)：为 ROCm 的分页注意力源码测试补充了 `float16` 数据类型及不支持的头维度（head size）异常拦截测试。

---

### 🚀 Release 动态
* **本期无新版本发布**。

---

## 🐛 Issues

### #53181 — [[Bug] Xgrammar "Failed to advance FSM" still fires after #52805 (0.27.2rc1.dev256+geac636a7f) — follow-up to #52852](https://github.com/vllm-project/vllm/issues/53181)
- **作者**: Duckmanjbr  **时间**: 2026-08-21 09:23 CST
- **摘要**: ## Follow-up to #52852 (closed 2026-08-19 as fixed by #52805)  **TL;DR:** The `#52805` fix is verifiably present in the running source, yet the same error line still fires. One occurrence in a 72h window; the request completed (no hard drop).  ### Environment - vLLM `0.27.2rc1.dev256+geac636a7f.d202…

### #53180 — [[Bug]: TurboQuant k8v4 + MTP speculative decoding silently produces degenerate output on hybrid GDN models (v0.27.1, stock)](https://github.com/vllm-project/vllm/issues/53180)
- **作者**: myia-ai-01  **时间**: 2026-08-21 08:36 CST
- **标签**: speculative-decoding, quantization
- **摘要**: # [Bug]: TurboQuant k8v4 + MTP speculative decoding produces silently degenerate output on hybrid GDN models (v0.27.1, Ada)  ## Summary  Combining `--kv-cache-dtype turboquant_k8v4` with MTP speculative decoding (`--speculative-config '{"method":"mtp",...}'`) on a hybrid GDN+attention model produces…

### #53178 — [[RFC]: Experimental Recirculation for causal decoder models](https://github.com/vllm-project/vllm/issues/53178)
- **作者**: mihir-s-05  **时间**: 2026-08-21 08:15 CST
- **摘要**: ## Motivation  [Recirculation](https://arxiv.org/abs/2608.17981) improves a pretrained decoder without changing weights by norm-matching a deep residual-stream activation, mixing it into an earlier residual boundary, and rerunning the upper layers. The first pass supplies the current token's logits;…

## 🔀 Pull Requests

### #53186 — [Revert "[Bugfix][MoE] Tune FlashInfer experts to scheduler token limit" (#52989)](https://github.com/vllm-project/vllm/pull/53186)
- **作者**: vllm-agent  **时间**: 2026-08-21 10:07 CST
- **标签**: bug, nvidia, quantization
- **摘要**: Reverts #52989 ("[Bugfix][MoE] Tune FlashInfer experts to scheduler token limit", merged as `bfb6c1349`).  ## Why  Nightly build [#84887](https://buildkite.com/vllm/ci/builds/84887) (`bfb6c1349`, the merge commit of #52989) turned `:nvidia: (B200) LM Eval PCP` red for the first time. That job was gr…

### #53185 — [[XPU] Ensure unquantized linear weight is N-contiguous](https://github.com/vllm-project/vllm/pull/53185)
- **作者**: zufangzhu  **时间**: 2026-08-21 09:59 CST
- **标签**: intel-gpu, quantization
- **摘要**: The XPU unquantized GEMM path uses torch.nn.functional.linear, which performs better when the (N, K) weight is N-contiguous. Convert the weight layout in process_weights_after_loading for XPU.

### #53184 — [[Perf][Humming] Integrate fused SwiGLU FP8 quantization for W2](https://github.com/vllm-project/vllm/pull/53184)
- **作者**: qianlihuang  **时间**: 2026-08-21 09:55 CST
- **标签**: performance, torch.compile, ci/build, quantization
- **摘要**: ## Purpose  Port the Humming W2 hot-path optimization described in:  https://www.lmsys.org/blog/2026-08-19-deepseek-v4-pro-engine-optimization-h20  This PR is stacked on #46273 and integrates fused SwiGLU + dynamic per-token FP8 quantization into Humming, including DeepSeek-V4 clamp semantics and re…

### #53183 — [[Model Runner V2] Use MRV2 for all models by default](https://github.com/vllm-project/vllm/pull/53183)
- **作者**: njhill  **时间**: 2026-08-21 09:35 CST
- **标签**: nvidia, mrv2
- **摘要**: Currently flushing out any CI issues

### #53182 — [[ROCm] Ship rocprofiler-sdk 1.3.2 in Dockerfile.rocm_base to fix torch.profiler traces](https://github.com/vllm-project/vllm/pull/53182)
- **作者**: Rohan138  **时间**: 2026-08-21 09:32 CST
- **标签**: rocm, ci/build
- **摘要**: # [ROCm] Ship rocprofiler-sdk 1.3.2 in Dockerfile.rocm_base to fix torch.profiler HIP-graph traces  ## Summary The ROCm ≤ 7.2.3 base image (`rocm/dev-ubuntu-22.04:7.2.3-complete`) ships **rocprofiler-sdk 1.1.0**. Since torch 2.12, PyTorch's Kineto GPU backend links `librocprofiler-sdk` (previously r…

### #53179 — [[Core][Model] Add experimental Recirculation support](https://github.com/vllm-project/vllm/pull/53179)
- **作者**: mihir-s-05  **时间**: 2026-08-21 08:16 CST
- **标签**: documentation, performance, llama, qwen, deepseek, gpt-oss, mistral, glm, minimax
- **摘要**: ## Purpose  Add experimental [Recirculation](https://arxiv.org/abs/2608.17981) execution for causal residual-decoder models.  Recirculation feeds a norm-matched deep residual activation into a shallower boundary, returns logits from the normal pass, and reruns the upper stack so later tokens attend …

### #53177 — [[ROCm][CI] Add float16 dtype and unsupported head size tests for paged attention](https://github.com/vllm-project/vllm/pull/53177)
- **作者**: divakar-amd  **时间**: 2026-08-21 08:00 CST
- **标签**: rocm
- **摘要**: - Extended DTYPES in test_attention.py to include torch.float16 (was only torch.bfloat16)   - Added test_paged_attention_unsupported_head_sizes to verify the ROCm kernel correctly rejects head sizes not supported by the native HIP kernel (only 64 and 128 are supported)  The ROCm paged attention kern…

### #53176 — [[Refactor][Model Runner V2][Multimodal] Move the encoder-only path out of the shared runner](https://github.com/vllm-project/vllm/pull/53176)
- **作者**: gty111  **时间**: 2026-08-21 07:33 CST
- **标签**: mrv2
- **摘要**: ## Purpose  An encoder-only instance — `--mm-encoder-only`, or the producer side of encoder-cache disaggregation — runs the vision encoder and publishes the embeddings. It runs no language model, holds no KV cache and samples no token, so most of a step does not apply to it. Today that is expressed …

### #53175 — [Resubmit PR 48666](https://github.com/vllm-project/vllm/pull/53175)
- **作者**: jhaotingc  **时间**: 2026-08-21 07:21 CST
- **标签**: speculative-decoding, ci/build, mrv2
- **摘要**: ## Purpose  Re-submit https://github.com/vllm-project/vllm/pull/48666 due to https://github.com/vllm-project/vllm/pull/52987. Flagged by CI for perf regression, but the regressed perf is resulting from Model emitting special token.   Model emitting special token is due to plain serving with [RedHatA…

### #53174 — [[Bugfix] Fix Step-3.5 MTP and structured outputs](https://github.com/vllm-project/vllm/pull/53174)
- **作者**: yzong-rh  **时间**: 2026-08-21 07:13 CST
- **标签**: bug, tool-calling
- **摘要**: ## Purpose  Two Step-3.5-Flash fixes MTP + structured outputs  ### MTP startup: restore `layer_types` after HF validation  Fixes [#40000](https://github.com/vllm-project/vllm/issues/40000) (stale-closed). Step-3.5 checkpoints ship `layer_types` longer than `num_hidden_layers` (Flash: 48 vs 45). Tran…
