# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-17 09:05 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue (问题反馈)
近期主要报告了三个与量化、架构和长上下文相关的严重 Bug：
1. **MLA + FLASHMLA_SPARSE 启动崩溃 (#48896)**：在 v0.25.1 版本中，使用 MLA 与 FLASHMLA_SPARSE 时，`_init_minimal_kv_cache_for_profiling` 阶段发生 Tensor shape 不匹配错误（期望 `[32, 64, 576]` 但输入大小为 1343488），导致服务无法启动。
2. **NVFP4 MoE 权重计算错误 (#48895)**：`moe_wna16_marlin_gemm` 在处理 gpt-oss 的 NVFP4 MoE 模型时，错误应用了 per-row topk 权重（`mul_topk_weights=True`），导致输出损坏（corrupt output）。
3. **EAGLE-3 推测解码长文本崩溃 (#48894)**：使用 EAGLE-3 推测解码时，若 prompt tokens 超过 2048，会在 inductor 编译的 `eagle_head` kernel 中触发 Triton 索引越界断言（`index < 2048`），即使禁用 cudagraph 仍会崩溃，仅 eager 模式可正常运行。

### 🛠️ Pull Request (代码提交)
近期 PR 聚焦于**内核性能优化（尤其是 Helion/ROCm）**、**新特性支持**及**关键 Bug 修复**：

**🚀 性能优化与新特性**
- **Attention / MLA**: 新增基于对称内存的 DCP (Decode Context Parallelism) A2A 直接通信路径（可通过 `VLLM_USE_DIRECT_DCP_A2A=1` 开启），优化 MLA 解码上下文并行 (#48897)。
- **Helion 内核优化**: 
  - 针对 H100 优化了动态量化内核（`dynamic_per_token_scaled_fp8_quant` 等）的缓存淘汰策略，提升数据复用率 (#48893)。
  - 引入基于 batch size (M) 调度的混合 C++ kernel `helion_cutlass_hybrid_scaled_mm`，作为 Helion Linear Backend 的一部分 (#48889)。
  - 为 eager 模式下的 `per_token_group_fp8_quant` 添加调用点路由，减少 host 开销 (#48888)。
- **ROCm 性能提升**: 默认开启 AITER (`VLLM_ROCM_USE_AITER` 默认值改为 True)，以解锁 ROCm 上的最佳性能表现 (#48890)。
- **LoRA 性能优化**: 为 LoRA 内核引入 Zero-slice early-exit（零切片提前退出）机制，自动跳过无权重的 LoRA 计算，减少无用开销 (#48887)。
- **推测解码**: 为 Model Runner V2 添加多模块 MTP (Multi-Token Prediction) 支持 (WIP 状态) (#48892)。

**🔧 关键 Bug 修复**
- **EP 权重加载冲突 (#48891)**：修复了专家并行（EP）权重过滤与多线程 safetensors 加载器同时启用时导致的加载问题。
- **ROCm GLM-5.2 推理失败 (#48886)**：修复了 Quark 量化的 GLM-5.2 Checkpoint 在 ROCm 上运行失败的问题，包括逐通道 FP8 反量化错误及 sparse-MLA 元数据缺失。
- **Responses API 工具历史 (#48885)**：修复了 Responses API 在多轮请求中重新提交自定义工具调用（`custom_tool_call` 等）时的验证处理逻辑。

### 📦 Release (版本发布)
近期暂无新版本发布记录。（注：Issue 中提到的 v0.25.1 启动崩溃问题，预计将在后续版本或热修复中解决）

---

## 🐛 Issues

### #48896 — [[Bug]:v0.25.1 startup crash in _init_minimal_kv_cache_for_profiling: "shape '[32, 64, 576]' is invalid for input of size 1343488" with MLA + FLASHMLA_SPARSE](https://github.com/vllm-project/vllm/issues/48896)
- **作者**: chengyoucai  **时间**: 2026-07-17 08:50 CST
- **标签**: bug
- **摘要**: ### Your current environment  Environment vLLM version: 0.25.1 (image vllm/vllm-openai:v0.25.1) Model: GLM-5.2-NVFP4 (GlmMoeDsaForCausalLM, NVFP4 quantization via ModelOpt) GPUs: 8 × ~140 GiB (tensor-parallel-size=8) Platform: CUDA, NCCL 2.28.9, Python 3.12 Relevant resolved backends: FLASHMLA_SPARS…

### #48895 — [[Bug]: moe_wna16_marlin_gemm applies wrong per-row topk weights (mul_topk_weights=True) at gpt-oss NVFP4 MoE shapes — corrupt output](https://github.com/vllm-project/vllm/issues/48895)
- **作者**: sukritRunara  **时间**: 2026-07-17 08:16 CST
- **标签**: bug
- **摘要**: ### Your current environment  ==============================        PyTorch Info ============================== PyTorch version              : 2.11.0+cu130 CUDA used to build PyTorch   : 13.0  ==============================        CUDA / GPU Info ============================== Is CUDA available     …

### #48894 — [[Bug]: EAGLE-3 + prompts >2048 tokens: device-side assert (Triton `index < 2048`) in inductor-compiled eagle_head kernels; eager works, cudagraph_mode=NONE still crashes (v0.24.0)](https://github.com/vllm-project/vllm/issues/48894)
- **作者**: manojarulmurugan  **时间**: 2026-07-17 08:07 CST
- **摘要**: ### Summary  EAGLE-3 speculative decoding kills the engine with a **device-side assert from an inductor-compiled `eagle_head` kernel** whenever a request's prompt is long enough to cross ~2048 computed tokens. Bisection shows the defect is in the compiled kernels themselves — **not** CUDA-graph capt…

## 🔀 Pull Requests

### #48897 — [[Attention] Add direct symmetric-memory DCP A2A](https://github.com/vllm-project/vllm/pull/48897)
- **作者**: GirasoleY  **时间**: 2026-07-17 09:04 CST
- **标签**: ci/build, v1
- **摘要**: ## Purpose  Add an opt-in direct symmetric-memory A2A path for MLA decode context parallelism. With `VLLM_USE_DIRECT_DCP_A2A=1` and `--dcp-comm-backend a2a`, each rank dispatches its partial output and LSE straight into peer receive buffers, publishes a system-scope completion signal, then waits and…

### #48893 — [[Kernel][Helion] Optimize H100 cache policies for dynamic quant kernels](https://github.com/vllm-project/vllm/pull/48893)
- **作者**: yushangdi  **时间**: 2026-07-17 07:44 CST
- **摘要**: ## Summary  Optimize H100 Helion configs for:  - `dynamic_per_token_scaled_fp8_quant` - `rms_norm_dynamic_per_token_quant`  The updated eviction policies retain input, residual, and weight data reused across multiple passes within a kernel invocation. Large-token configs remain unchanged where cache…

### #48892 — [[WIP][Model Runner V2][Spec Decode] Add multi-module MTP support](https://github.com/vllm-project/vllm/pull/48892)
- **作者**: TheEpicDolphin  **时间**: 2026-07-17 07:10 CST

### #48891 — [[Bugfix] Apply EP weight filter in multithreaded safetensors loader](https://github.com/vllm-project/vllm/pull/48891)
- **作者**: ArsalanShakil  **时间**: 2026-07-17 06:51 CST
- **摘要**: ## Purpose  Fixes #48827.  When expert-parallel (EP) weight filtering and multithreaded safetensors loading are enabled together:  ```bash vllm serve <large MoE checkpoint> \   --enable-expert-parallel \   --enable-ep-weight-filter \   --model-loader-extra-config '{"enable_multithread_load": true}' …

### #48890 — [[ROCm] Enable AITER by default](https://github.com/vllm-project/vllm/pull/48890)
- **作者**: AndreasKaratzas  **时间**: 2026-07-17 06:47 CST
- **标签**: rocm, ready
- **摘要**: Enable AITER by default on ROCm by changing the default value of `VLLM_ROCM_USE_AITER` from `False` to `True`.  AITER unlocks the best performance on ROCm. Enabling it by default allows ROCm users to benefit from AITER without requiring additional environment configuration.

### #48889 — [[HelionLinearBackend][2/N] Add helion_cutlass_hybrid_scaled_mm c++ kernel ](https://github.com/vllm-project/vllm/pull/48889)
- **作者**: xiaohongchen1991  **时间**: 2026-07-17 06:42 CST
- **标签**: nvidia
- **摘要**: ## Purpose As described in the RFC, https://github.com/vllm-project/vllm/issues/46526, the Helion linear backend will dispatch the traffic to different kernel based on the batch size (denoted as "M"): - M in [1, helion_threshold]: use Helion scaled_mm kernel and covered by CUDA graph capture/replay …

### #48888 — [[DO NOT REVIEW][Kernel][Helion] Eager call-site routing for per_token_group_fp8_quant](https://github.com/vllm-project/vllm/pull/48888)
- **作者**: yushangdi  **时间**: 2026-07-17 05:44 CST
- **摘要**: ## Summary  Routes vLLM's `per_token_group_quant_fp8` call site to a Helion kernel in the **eager / non-CUDA-graph path**, where the torch custom-op dispatch boundary is pure per-call host overhead. A generic `FusedCallsiteRouter` caches and replays guarded C++ launchers so warmed calls take a singl…

### #48887 — [[LoRA][Perf] Zero-slice early-exit for LoRA kernels](https://github.com/vllm-project/vllm/pull/48887)
- **作者**: bhoomit  **时间**: 2026-07-17 05:34 CST
- **摘要**: # [LoRA][Perf] Zero-slice early-exit for LoRA kernels  ## Summary  Automatic per-adapter per-slice active mask that skips zero-weight LoRA computations at two levels — a Python layer-level skip and a Triton kernel-level per-CTA skip. The layer-level skip fires for any layer that **no loaded adapter …

### #48886 — [[ROCm] [BugFix] Fix Quark GLM-5.2 Checkpoint inference: indexer wk per-channel FP8 dequant + missing sparse-MLA metadata fields](https://github.com/vllm-project/vllm/pull/48886)
- **作者**: ColinZ22  **时间**: 2026-07-17 05:33 CST
- **标签**: bug, rocm, v1, deepseek
- **摘要**: ## Summary  This PR addresses two independent bugs blocking quark quantized GLM-5.2 checkpoints with Attn quantized to PTPC FP8 from running end-to-end on ROCm (MI355X / gfx950).   ### Per-channel FP8 scale for fused indexer `wk` (`deepseek_v2.py`):  `_try_load_fp8_indexer_wk` dequantizes the FP8 in…

### #48885 — [[Bugfix][Frontend] Handle custom tool history in Responses API](https://github.com/vllm-project/vllm/pull/48885)
- **作者**: tiannianzhu  **时间**: 2026-07-17 05:19 CST
- **标签**: bug, frontend
- **摘要**: ## Purpose  Fix Responses API multi-turn requests that resubmit `custom_tool_call` and `custom_tool_call_output` items.  `ResponsesRequest` validates these input items into `ResponseCustomToolCall` Pydantic models. The non-Harmony message conversion handled function tool models but allowed custom to…
