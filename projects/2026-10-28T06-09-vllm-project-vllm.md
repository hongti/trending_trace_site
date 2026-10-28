# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-28 14:09 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issues (问题)
1. **Qwen3.5 模型 AOT 编译缓存加载失败 (#64204)**：在 Windows 构建版本中，Qwen3-Next / Qwen3.5 模型始终无法加载 AOT（提前编译）缓存，报错找不到内核索引。
2. **多模态视频帧数超限被拒 (#64202)**：当输入的 `video/jpeg` 视频帧数超过 `num_frames` 限制（默认 32 帧），且请求中带有对应的 `frames_indices` 时，请求会被错误拒绝。

---

### 🔧 Pull Requests (代码提交)
**Bug 修复**
* **#64210**：修复 Qwen3.5 AOT 编译缓存问题（对应 #64204）。将融合的 QK-norm/RoPE/gate Triton 内核注册为自定义操作，使其能在编译图中正确运行。
* **#64201**：修复多模态视频帧验证逻辑（对应 #64202）。调整截断逻辑，校验 `frames_indices` 与实际发送帧数，避免超限请求被拒。
* **#64203**：修复 Engram 模型在配置的 tokenizer 版本与模型版本不一致时的 token 映射构建错误。
* **#64211**：修复 ROCm 环境下 MLA 后端正确性测试的随机种子依赖问题，并允许 ROCm 上 GLM 模型存在 1 ULP 的 bf16 精度误差。

**性能优化**
* **#64205 (Attention)**：将 FA3（FlashAttention-3）混合批次拆分为独立的解码和预填充调用，避免预填充查询长度抑制长解码的 split-KV，提升了解码效率。
* **#64200 (DeepSeek-V4.1)**：在 CUDA eager 预填充阶段，将 KV exchange 与缓存发布和 rank-local attention 及查询准备重叠执行，提升并行处理性能。
* **#64207 (ROCm/GLM-5.3-Flash)**：将 mHC Sinkhorn 操作移出 MonoKernel 门控的关键路径，使并发 1 和 4 时的中位解码延迟（ITL）降低约 6.7%~6.9%。

**新特性与内核支持**
* **#64209 (Quantization)**：支持 `compressed-tensors` 格式的 NVFP4 MoE 模型使用 `--quantization-config.moe.activation=nvfp4_per_token` 参数进行激活覆盖配置。
* **#64208 (ROCm/DeepSeek-V4)**：为 DeepSeek-V4 / V4.1 的路由添加了 Wave64 `topk_softplus_sqrt` 快速路径内核。
* **#64206 (WIP/ROCm/MiniMax-M3)**：将路由缩放因子折叠到 AITER top-k 权重中，优化未共享专家融合场景下的 MoE 路由性能。

---

### 🚀 Release (版本发布)
* 近期无新版本发布记录。

---

## 🐛 Issues

### #64204 — [[Bug]: AOT compile cache never loads for Qwen3.5 models: "Kernel index 1 not found in id_to_kernel"](https://github.com/vllm-project/vllm/issues/64204)
- **作者**: sataz-ehl  **时间**: 2026-10-28 13:04 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>Output of <code>vllm collect-env</code></summary>  ``` ==============================         System Info ============================== OS                           : Microsoft Windows 11 Enterprise GCC version                  : Could not collect Cl…

### #64202 — [[Bug]: video/jpeg frames_indices rejected when more frames than num_frames are sent](https://github.com/vllm-project/vllm/issues/64202)
- **作者**: MarinaMackay  **时间**: 2026-10-28 12:55 CST
- **摘要**: ### Your current environment  - vLLM main `274a5e19a` (precompiled wheel), Python 3.12.16, torch 2.14.0+cu130, transformers 5.20.2 - 1x NVIDIA H200 NVL (driver 615.71.09) for the server check; the minimal repro below runs on CPU  ### 🐛 Describe the bug  Sending pre-extracted frames as `video/jpeg` t…

## 🔀 Pull Requests

### #64211 — [[CI][Bugfix][ROCm] Seed each MLA backend correctness case; allow one bf16 ULP for ROCm glm](https://github.com/vllm-project/vllm/pull/64211)
- **作者**: AndreasKaratzas  **时间**: 2026-10-28 13:50 CST
- **标签**: bug, rocm, glm
- **摘要**: - `test_mla_backends.py`: seed each case at the start of `_run_backend_correctness`, so its random inputs no longer depend on which tests ran before it - `test_mla_backends.py`: on ROCm with a bf16 KV cache and `v_head_dim=256` (glm), allow one bf16 ULP of the largest reference value as `atol`  `tes…

### #64210 — [[Bugfix] Register fused_qk_rmsnorm_rope_gate as a custom op to fix AOT compile cache loading ( #64204 fix )](https://github.com/vllm-project/vllm/pull/64210)
- **作者**: harsHdokyc  **时间**: 2026-10-28 13:44 CST
- **标签**: bug, torch.compile
- **摘要**: ## Overview  Fixes #64204. AOT compile cache never loads for Qwen3-Next / Qwen3.5 because the fused QK-norm/RoPE/gate Triton kernel is launched directly inside the compiled graph. This registers it as a vLLM custom op. Also fixes the UTF-8 encoding issue in `_compute_code_hash` noted in the same iss…

### #64209 — [[Quantization] Support nvfp4_per_token MoE activation override for compressed-tensors](https://github.com/vllm-project/vllm/pull/64209)
- **作者**: zhangyimi  **时间**: 2026-10-28 13:26 CST
- **标签**: documentation, quantization
- **摘要**: ## Overview  Make compressed-tensors NVFP4 MoE checkpoints honor `--quantization-config.moe.activation=nvfp4_per_token`, which today only works for ModelOpt checkpoints (#56740).  ## Claims  - Problem: on a compressed-tensors NVFP4 MoE checkpoint, the per-token activation override never reaches `Com…

### #64208 — [[ROCm][Kernel] Wave64 topk_softplus_sqrt fast path for DeepSeek-V4 routing](https://github.com/vllm-project/vllm/pull/64208)
- **作者**: amd-dlimpus  **时间**: 2026-10-28 13:21 CST
- **标签**: rocm, deepseek, DSv4
- **摘要**: ## Purpose  DeepSeek-V4 / V4.1 routing calls `topk_softplus_sqrt` once per MoE layer per step (40 times per decode step on DeepSeek-V4.1-Flash). On ROCm it runs the generic `topkGatingSoftplusSqrt<6,384,4,8,64,...>` kernel, which keeps its row in a dynamically indexed private array (32 B/lane of scr…

### #64207 — [[ROCm][Perf][GLM-5.3-Flash] MonoKernel: take the mHC Sinkhorn off the gate's critical path (on #61662)](https://github.com/vllm-project/vllm/pull/64207)
- **作者**: amd-dlimpus  **时间**: 2026-10-28 13:19 CST
- **标签**: rocm, ci/build, glm
- **摘要**: ## Overview  Takes the mHC Sinkhorn off the critical path of the GLM-5.3-Flash MonoKernel's gate. Median decode ITL drops 6.7% at concurrency 1 and 6.9% at concurrency 4 (TP2, MI355X, real-text prompts), with bit-identical outputs.  **Stacked on #61662**, which has not landed, so the diff against ma…

### #64206 — [[WIP][ROCm][MiniMax-M3] Fold routed scaling into the AITER top-k weights](https://github.com/vllm-project/vllm/pull/64206)
- **作者**: micah-wil  **时间**: 2026-10-28 13:10 CST
- **标签**: rocm, minimax
- **摘要**: Without shared-expert fusion, have the router fold routed_scaling_factor into the top-k weights instead of scaling the bf16 MoE output, and pass the factor to AITER's biased_grouped_topk, which applies it in-kernel, rather than multiplying the weights afterwards. Removes one elementwise kernel per M…

### #64205 — [[Perf][Attention] Split FA3 mixed batches into decode and prefill calls](https://github.com/vllm-project/vllm/pull/64205)
- **作者**: zksainx  **时间**: 2026-10-28 13:07 CST
- **标签**: performance
- **摘要**: ## Overview  Split eligible FA3 mixed batches into single-token decode and prefill calls, so prefill query length does not suppress split-KV for long decodes. Fixes #64058.  ## Claims  - On Qwen3-30B-A3B TP4 long+short serving, short-request median TTFT falls from 7.99–11.08 s to 103–106 ms and medi…

### #64203 — [[Bugfix][Model] Honor tokenizer revision for Engram token maps](https://github.com/vllm-project/vllm/pull/64203)
- **作者**: RuixiangMa  **时间**: 2026-10-28 12:58 CST
- **标签**: bug, deepseek, DSv4.1
- **摘要**: ## Overview  Fix Engram token-map construction when the configured tokenizer revision differs from the model revision.   ## Claims - Load the Engram tokenizer using `model_config.tokenizer_revision`. - Add regression coverage for distinct model and tokenizer revisions.  ## Validation  - pytest -q te…

### #64201 — [[Bugfix][Multimodal] Validate video/jpeg frames_indices against the frames sent](https://github.com/vllm-project/vllm/pull/64201)
- **作者**: MarinaMackay  **时间**: 2026-10-28 12:51 CST
- **标签**: bug, multi-modality
- **摘要**: ## Overview  A `video/jpeg` frame sequence that has more frames than the `num_frames` limit (default 32) is rejected when the request also sends `frames_indices` for those frames. `load_base64` truncates the frames to the limit and then compares the original `frames_indices` with the truncated count…

### #64200 — [[Perf][DSv4.1][PCP] Overlap KV exchange with query preparation](https://github.com/vllm-project/vllm/pull/64200)
- **作者**: wangyicong52  **时间**: 2026-10-28 12:12 CST
- **标签**: deepseek, DSv4.1
- **摘要**: ## Overview  Overlap DeepSeek-V4.1 PCP KV exchange and cache publication with rank-local attention and indexer query preparation on CUDA eager prefills.  ## Claims  - Keep SWA KV, compressor-tail and compressed-latent collectives on the calling stream in their existing order while preparing queries …
