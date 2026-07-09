# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-09 09:11 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 最近动态的简洁摘要：

### 🐛 Issue (缺陷报告)

近期报告的 Bug 主要集中在**新模型架构支持**与**特定硬件后端（XPU）**上：

1. **Qwen3.5 模型注册失败** (#48060)：`Qwen3_5ForCausalLM` / `Qwen3_5MoeForCausalLM` 虽已定义但未被正确注册，导致纯文本检查点无法加载（此问题与平台无关）。
2. **XPU 后端 Mamba 前缀缓存崩溃** (#48059)：在 XPU 上为混合 GDN/Mamba 模型（如 Qwen3.5/3.6 MoE）启用 `--enable-prefix-caching --mamba-cache-mode align` 时，存储状态指针触发 "Overflow when unpacking long long" 错误导致崩溃。
3. **XPU 后端 FP8 动态量化输出异常** (#48058)：在 Intel Arc Pro B70 (Battlemage) 上运行 compressed-tensors FP8 W8A8（动态 per-token 激活量化）检查点时，模型输出为乱码。

---

### 🔧 PR (合并请求)

PR 动态重点涵盖了**核心硬件适配修复**、**分布式性能优化**及**基础设施维护**：

#### 🚀 核心修复与硬件适配
1. **修复 DeepSeek-V4 在消费级 Blackwell (RTX 5090等) 上的 FP8 NaN 问题** (#48052)：在 SM120/SM121 架构上，`o_proj` fp8 einsum 路径错误采用了 SM100 的 packed int32 scales，现改用类似 SM90 的原始 f32 block scales，解决计算结果输出 NaN 的严重问题。
2. **修复 Rejection Sampler 内核 int32 溢出** (#48055)：修复推测采样相关内核中因 int32 行索引乘法导致的偏移溢出隐患。
3. **修复 ROCm GLM MoE DSA 错误重量化** (#48051)：阻止 ROCm AITER MLA 路径将已量化的 bf16 `kv_b_proj`（如 GLM-5.2-MXFP4）错误地再量化为 fp4/fp8。
4. **修复 Mooncake DP 引擎索引崩溃** (#48061)：在外部/混合 DP 负载均衡下，无头宽 TP 节点使用局部 DP rank 会触发断言崩溃，现改用全局 `data_parallel_index`。
5. **优化 CUDA Graph 捕获线程安全性** (#48053)：将所有 CUDA graph 捕获路径的 `capture_error_mode` 改为 `thread_local`，避免其他线程的 CUDA 工作导致捕获失效。
6. **完善 Pooling 维度验证** (#48057)：修复了非 Matryoshka 嵌入模型中基础数值错误被通用异常掩盖的问题。

#### ⚡ 性能优化
7. **FlashInfer MNNVL Allreduce + RMSNorm 量化融合** (#48064)：为 MNNVL 后端启用 AR + RMSNorm 量化融合（支持 FP4 量化路径），提升分布式推理性能。

#### 📚 测试、文档与依赖
8. **固定 PyNvVideoCodec 版本修复 CI** (#48056)：将 CUDA 依赖 `PyNvVideoCodec` 锁定至已验证的 2.0.4 版本，解决 2.1.0 wheel 缺失导致的 CI 构建阻塞。
9. **增加并行采样测试覆盖** (#48062)：为 `n > 1` 的所有 `RequestOutputKind` 值添加了显式的并行采样测试（覆盖异步与同步引擎路径）。
10. **补充 ModelOpt 加速加载格式文档** (#48063)：在 ModelOpt 量化文档中增加说明，推荐大型 safetensors 检查点使用 `--load-format instanttensor` 加速加载。

---

*(注：本次给定动态中无 Release 版本信息)*

---

## 🐛 Issues

### #48060 — [[Bug] Qwen3_5ForCausalLM / Qwen3_5MoeForCausalLM defined but not registered — text-only checkpoints fail to load](https://github.com/vllm-project/vllm/issues/48060)
- **作者**: Jrojas90  **时间**: 2026-07-09 08:11 CST
- **摘要**: ### Your current environment  <details> <summary>Environment (<code>vllm collect_env</code>, abridged)</summary>  ``` vLLM version              : 0.23.1rc1.dev920+gdd127d82e (main @ dd127d82), VLLM_TARGET_DEVICE=xpu PyTorch version           : 2.12.0+xpu  (XPU build 20250302) Python version         …

### #48059 — [[Bug][XPU] Mamba align-mode prefix caching crashes: "Overflow when unpacking long long" storing state.data_ptr()](https://github.com/vllm-project/vllm/issues/48059)
- **作者**: Jrojas90  **时间**: 2026-07-09 08:11 CST
- **摘要**: ### Your current environment  <details> <summary>Environment (<code>vllm collect_env</code>, abridged)</summary>  ``` vLLM version              : 0.23.1rc1.dev920+gdd127d82e (main @ dd127d82), VLLM_TARGET_DEVICE=xpu PyTorch version           : 2.12.0+xpu  (XPU build 20250302) Python version         …

### #48058 — [[Bug][XPU] compressed-tensors FP8 W8A8 (dynamic) generates garbage output on Intel Arc Pro B70 (Battlemage)](https://github.com/vllm-project/vllm/issues/48058)
- **作者**: Jrojas90  **时间**: 2026-07-09 08:11 CST
- **摘要**: ### Your current environment  <details> <summary>Environment (<code>vllm collect_env</code>, abridged)</summary>  ``` vLLM version              : 0.23.1rc1.dev920+gdd127d82e (main @ dd127d82), VLLM_TARGET_DEVICE=xpu PyTorch version           : 2.12.0+xpu  (XPU build 20250302) Python version         …

## 🔀 Pull Requests

### #48064 — [[Distributed][Perf] Enable FlashInfer MNNVL allreduce RMS quant fusion](https://github.com/vllm-project/vllm/pull/48064)
- **作者**: mmangkad  **时间**: 2026-07-09 08:48 CST
- **标签**: nvidia
- **摘要**: ## Summary  Enable AR + RMSNorm quant fusion to use the MNNVL backend, which current FI already supports  <img width="1632" height="852" alt="image" src="https://github.com/user-attachments/assets/acf40be8-7efb-4360-b179-9141969360ca" />  `QuantType)2` is the FP4 quant path https://github.com/flashi…

### #48063 — [docs: mention accelerated load formats for ModelOpt](https://github.com/vllm-project/vllm/pull/48063)
- **作者**: PSR94  **时间**: 2026-07-09 08:47 CST
- **标签**: documentation
- **摘要**: Summary - Add a note to the ModelOpt quantization docs about using accelerated load formats for large safetensors checkpoints. - Show `--load-format instanttensor` in the ModelOpt serving example context. - Link to the existing InstantTensor and fastsafetensors extension pages for installation and b…

### #48062 — [test: cover parallel sampling output kinds](https://github.com/vllm-project/vllm/pull/48062)
- **作者**: PSR94  **时间**: 2026-07-09 08:40 CST
- **标签**: v1
- **摘要**: Summary - Add explicit parallel sampling coverage for `n > 1` across all `RequestOutputKind` values. - Cover both `AsyncLLM.generate()` and `LLMEngine.add_request()` / `step()` paths. - Assert per-sample indexes, final completion status, and token accounting for DELTA, CUMULATIVE, and FINAL_ONLY out…

### #48061 — [[BugFix][Mooncake] Use global data_parallel_index for the DP engine index](https://github.com/vllm-project/vllm/pull/48061)
- **作者**: ivanium  **时间**: 2026-07-09 08:17 CST
- **标签**: bug, v1, kv-connector
- **摘要**: ## Purpose  `get_mooncake_dp_engine_index` asserted `data_parallel_rank_local is not None` under external/hybrid DP LB. This crashes on headless wide-TP follower nodes: they go through `run_headless` → `MultiprocExecutor` directly, which never populates `data_parallel_rank_local`.  Preferring `data_…

### #48057 — [[Misc] Validate pooling dimensions](https://github.com/vllm-project/vllm/pull/48057)
- **作者**: taneem-ibrahim  **时间**: 2026-07-09 08:06 CST
- **摘要**: ## Purpose PoolingParams.dimensions was partially validated during request verification, but basic numeric errors were masked by the generic non-Matryoshka error on non-Matryoshka embedding models. On` intfloat/e5-small`, `dimensions=0`, `dimensions=-5`, and an obviously oversized value all produced…

### #48056 — [Pin PyNvVideoCodec to tested 2.0.4 wheel](https://github.com/vllm-project/vllm/pull/48056)
- **作者**: brandonpelfrey  **时间**: 2026-07-09 07:40 CST
- **标签**: ci/build, nvidia
- **摘要**: ## Purpose  Pin `PyNvVideoCodec` to `2.0.4` in CUDA requirements.  The CUDA dependency currently points at `PyNvVideoCodec==2.1.0`, which is blocking CI builds while the needed wheel availability catches up. `2.0.4` wheels are now published and we validated the vLLM PyNvVideoCodec video decode path …

### #48055 — [[Bugfix] Fix int32 offset overflow in rejection sampler kernels](https://github.com/vllm-project/vllm/pull/48055)
- **作者**: paulsbrookes  **时间**: 2026-07-09 07:30 CST
- **标签**: bug, v1
- **摘要**: ## Purpose  `rejection_random_sample_kernel` and `sample_recovered_tokens_kernel` index the `[num_tokens, vocab_size]` draft/target probability tensors with an int32 row-index multiply:  - `rejection_random_sample_kernel`: `(start_idx + pos) * vocab_size` - `sample_recovered_tokens_kernel`: `token_i…

### #48053 — [[Core] Use capture_error_mode="thread_local" on all CUDA-graph capture paths](https://github.com/vllm-project/vllm/pull/48053)
- **作者**: kacper-daftcode  **时间**: 2026-07-09 06:49 CST
- **标签**: v1, nvidia
- **摘要**: vLLM captures CUDA graphs with the default capture_error_mode="global", under which CUDA work issued by ANY host thread during capture invalidates the capture (CUDA_ERROR_STREAM_CAPTURE_INVALIDATED) — even when that work is on an unrelated side stream and never touches the captured stream. This brea…

### #48052 — [[Bugfix][Hardware] DeepSeek-V4 o_proj fp8 einsum NaNs on SM12x: use SM90-style raw f32 block scales](https://github.com/vllm-project/vllm/pull/48052)
- **作者**: kacper-daftcode  **时间**: 2026-07-09 06:49 CST
- **标签**: bug, deepseek
- **摘要**: On consumer Blackwell (SM120/SM121, e.g. RTX 5090 / RTX PRO 6000) the DeepSeek-V4 o_proj fp8_einsum path selects the SM100 recipe: (1, 1, 128) with TMA-aligned packed int32 scales. DeepGEMM's SM120 einsum consumes raw row-major f32 block scales (its own SM120 test convention) and produces NaNs on th…

### #48051 — [[ROCm] Don't re-quantize bf16 MLA kv_b_proj to fp4/fp8 for GLM MoE DSA](https://github.com/vllm-project/vllm/pull/48051)
- **作者**: amd-sriram  **时间**: 2026-07-09 06:29 CST
- **标签**: rocm
- **摘要**: ## Purpose  On ROCm, the AITER MLA path assumes a bf16 `kv_b_proj` is unquantized and quantizes it to fp4/fp8. This is wrong for `GlmMoeDsaForCausalLM` (e.g. `amd/GLM-5.2-MXFP4`), where the MLA weights are intentionally bf16 and only the MoE experts are MXFP4.  Quantizing these MLA weights to fp4 ad…
