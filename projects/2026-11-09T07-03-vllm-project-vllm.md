# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-09 15:03 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
* **环境变量解析异常 (#66493)**：指出 vLLM 中布尔型环境变量存在 6 种不同的解析方式，导致传入 `"true"` 时可能会引发崩溃或被静默忽略。
* **RDNA4 INT4 GEMM 提案 (#66491)**：提交了针对 AMD RDNA4 架构 (gfx1200/gfx1201) 引入 Packed-weight INT4 GEMM (W4A16 / W4A8) 矩阵计算支持的 RFC 提案。

### 🔧 PR 动态
**Bug 修复**
* **前端请求校验 (#66489)**：修复了 `ChatCompletionRequest` 传入空 `messages` 列表时仍能通过校验的问题，现在会直接拒绝空消息，避免无意义的推理计算。
* **量化模型加载 (#66488)**：使 `MoeWNA16Config` 兼容标准 AWQ 检查点的键名（`w_bit` / `q_group_size`），修复了加载 AWQ MoE 模型时的报错。
* **MoE LoRA (#66486)**：修复了在数据并行 (DP) 和专家并行 (EP) 同时开启时，MoE LoRA 错误假设专家仅处理本地 batch 而导致的计算正确性问题。
* **强化学习/权重加载 (#66487)**：在直接重载 (`VLLM_RELOAD_MODE=direct`) 完成时，不再错误地重复绑定词嵌入。
* **XPU 计算 (#66494)**：修复了 XPU MXFP8 线性核心仅支持 2D 输入导致 3D 激活值传入时引擎初始化失败的问题，现会将 N 维输入展平。

**性能优化**
* **ROCm/AMD 显卡 (#66490, #66485, #66484)**：针对 gfx942/gfx950 架构进行多项性能提升：包括在 gfx950 上让 DSv4.1 的 index-source 层以 FP8 indexer cache 整体运行；在 gfx942 上为小 decode batch 启用 64 个 split-KV 段；并针对无 FP4 指令的 gfx942 调优 MXFP4 MoE 矩阵计算参数。
* **CPU 推理 (#66492)**：在非 Triton 的 V1 CPU worker 中，使用 NumPy 替代逐 logit 收集 `int32` 的方式来扩展打包字节，大幅加速结构化输出掩码的应用。

**工具与基准测试**
* **Benchmark (#66483)**：`vllm bench forward` 工具现在默认测量目标采样（拒绝采样）的耗时，并新增 `--exclude-sampler` 参数以便按需排除。

### 🚀 Release 动态
* 近期暂无正式版发布信息。

---

## 🐛 Issues

### #66493 — [[Bug]: Boolean env vars are parsed six different ways - "true" either crashes or is silently ignored](https://github.com/vllm-project/vllm/issues/66493)
- **作者**: zhuzhiyu2026  **时间**: 2026-11-09 15:01 CST
- **标签**: bug
- **摘要**: ### Your current environment  Not environment specific; reproducible on a plain checkout of `main`:  ```console $ python -c "import vllm.envs as envs; print(envs.VLLM_BATCH_INVARIANT)" ```  (No GPU required: `vllm/envs.py` is evaluated on import.)  ### 🐛 Describe the bug  `vllm/envs.py` parses boole…

### #66491 — [[RFC]: Packed-weight INT4 GEMM (W4A16 / W4A8) for RDNA4 (gfx1200/gfx1201)](https://github.com/vllm-project/vllm/issues/66491)
- **作者**: jeffreylowzg  **时间**: 2026-11-09 14:53 CST
- **标签**: rocm, RFC, quantization
- **摘要**: ### Motivation.   On RDNA4 (Radeon AI PRO R9700, gfx1201), W4A16 decode runs on `RDNAHybridW4A16LinearKernel` (#40977): `wvSplitK_int4_g` for up to 5 rows, Triton above. #62708 extends it to an INT4 `lm_head`, and #63562 retunes the gfx12 Triton tiles for small M. I have a set of RDNA4 kernels (HIP,…

## 🔀 Pull Requests

### #66494 — [[XPU] Flatten N-D input in XPU MXFP8 linear kernel](https://github.com/vllm-project/vllm/pull/66494)
- **作者**: Liangliang-Ma  **时间**: 2026-11-09 15:02 CST
- **摘要**: The Transformers modeling backend passes 3D activations to vLLM linear layers, but per_token_group_fp8_quant on XPU only accepts 2D input, causing engine init to fail with 'Expected output_s.dim() == 2'. Reshape the input to 2D and restore the leading dims on the output, matching the other MXFP8 lin…

### #66492 — [[Perf][CPU] Speed up structured-output mask expansion](https://github.com/vllm-project/vllm/pull/66492)
- **作者**: Hang-get  **时间**: 2026-11-09 14:54 CST
- **标签**: cpu
- **摘要**: ## Overview  Speed up grammar-bitmask application in the non-Triton V1 CPU worker by expanding packed bytes with NumPy instead of gathering one `int32` word per logit. Output semantics and public APIs are unchanged.  ## Claims  - Reduce `CPUStructuredOutputsWorker.apply_grammar_bitmask` latency by 3…

### #66490 — [[ROCm][Perf][DSv4.1] Run the index-source layers whole with the FP8 indexer cache](https://github.com/vllm-project/vllm/pull/66490)
- **作者**: ZhengGong-amd  **时间**: 2026-11-09 14:47 CST
- **标签**: rocm, deepseek
- **摘要**: With `--monokernel` on gfx950, the mono decode layer takes an index-source layer whole only when its indexer is aiter's MXFP4 one. With the default indexer cache (`indexer_kv_dtype="auto"`, which resolves to FP8 for DeepSeek-V4.1), the eight index-source layers of DeepSeek-V4.1-Flash (2, 8, 14, 20, …

### #66489 — [[Bugfix][Frontend] Reject empty messages in ChatCompletionRequest](https://github.com/vllm-project/vllm/pull/66489)
- **作者**: Yunt1  **时间**: 2026-11-09 14:45 CST
- **标签**: bug, frontend
- **摘要**: ## Overview  An empty `messages` list passed chat-completions protocol validation and only failed after the engine had produced a full generation: `OpenAIServingChat.get_chat_request_role` does `request.messages[-1]["role"]`, raising `IndexError` that maps to HTTP 500. Reject it at validation time w…

### #66488 — [[Bugfix][Quantization] Accept AWQ key names in MoeWNA16Config](https://github.com/vllm-project/vllm/pull/66488)
- **作者**: Yunt1  **时间**: 2026-11-09 14:45 CST
- **标签**: bug, quantization
- **摘要**: ## Overview  `MoeWNA16Config` reads only the GPTQ key names (`bits` / `group_size`), so it fails on standard AWQ checkpoints (`w_bit` / `q_group_size`) at the exact point where the AutoAWQ MoE fallback ("Falling back to Moe WNA16 kernels") hands it the raw checkpoint config. It raises `ValueError: C…

### #66487 — [[Bugfix][RL] Don't re-tie word embeddings on direct reload](https://github.com/vllm-project/vllm/pull/66487)
- **作者**: rycerzes  **时间**: 2026-11-09 14:35 CST
- **标签**: bug
- **摘要**: ## Overview  Don't re-tie word embeddings when a direct reload (`VLLM_RELOAD_MODE=direct`) finishes. Part of the direct weight reload work (RFC #54477). This is item 4 of the [#64336 review](https://github.com/vllm-project/vllm/pull/64336#discussion_r4429569227).  ## Claims  - A direct reload no lon…

### #66486 — [[Bugfix][LoRA] Fix MoE LoRA correctness under DP + EP](https://github.com/vllm-project/vllm/pull/66486)
- **作者**: jeejeelee  **时间**: 2026-11-09 14:28 CST
- **标签**: bug, cpu, nvidia, mrv2
- **摘要**: With data parallelism and expert parallelism, MoE LoRA runs on tokens dispatched from other DP ranks, but three places assumed the experts only see the local batch:  - LoRAKernelMeta.prepare_tensors returned before copying the token->slot mapping for batches without LoRA, so the EP dispatch sent the…

### #66485 — [[ROCm][Perf] Use 64 split-KV segments for small decode batches on gfx942](https://github.com/vllm-project/vllm/pull/66485)
- **作者**: tc3oliver  **时间**: 2026-11-09 14:24 CST
- **标签**: rocm
- **摘要**: ## Overview  `TritonAttentionMetadataBuilder` already uses 64 instead of 16 split-KV segments when a decode batch under-fills the device, but only on SM120 (#58482). This enables the same rule on gfx942.  ## Claims  - Batch-1 decode with `TRITON_ATTN` on MI325X: TPOT -14% at 8K context, -36% at 32K,…

### #66484 — [[ROCm][Perf] Tune MXFP4 MoE tiles on gfx942](https://github.com/vllm-project/vllm/pull/66484)
- **作者**: tc3oliver  **时间**: 2026-11-09 14:24 CST
- **标签**: rocm, gpt-oss
- **摘要**: ## Overview  gfx942 has no FP4 instructions, so the `triton_kernels` MXFP4 matmul upcasts inside the kernel and its default tiles are slow for gpt-oss. This pins the matmul tiles on gfx942 (`triton_kernels` 3.8), the way the file already does for gfx950 and SM12x. Related: #42578 (open draft, a sepa…

### #66483 — [[Bench] Time target sampling in vllm bench forward](https://github.com/vllm-project/vllm/pull/66483)
- **作者**: WoosukKwon  **时间**: 2026-11-09 14:20 CST
- **标签**: documentation, performance, ready, needs-rebase, agents
- **摘要**: ## Overview  `vllm bench forward` now times target sampling (rejection sampling for verify rows) by default, between the target forward and the drafter, and adds `--exclude-sampler` to drop it. Requests use the model's default sampling parameters instead of hard-coded greedy.  ## Claims  - The captu…
