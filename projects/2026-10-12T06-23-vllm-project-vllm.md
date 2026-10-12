# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-12 14:23 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🚀 Release
*本期未收到版本发布动态。*

### 🐛 Issue (问题反馈)
近期主要报告了两个严重的运行时上下文异常问题：
1. **跨请求上下文污染** (#61160)：在 PD 分离式部署且默认开启自动前缀缓存的情况下，响应中会间歇性出现来自其他请求的内容。目前只能通过 `--no-enable-prefix-caching` 规避。
2. **并发导致 Agent 会话崩溃** (#61157)：在 GLM hybrid-SSM 模型结合 MTP 推测解码时，当有并发请求到达，长周期的 Agent 会话会急剧退化，出现捏造工具参数、自我失忆等严重问题，且在之前的修复补丁后依然存在。

### 🔧 Pull Request (代码变更)
本期 PR 主要集中在 **MoE/LoRA 修复**、**推测解码与多卡兼容性修复** 以及 **新模型支持**：

**1. MoE 与 LoRA 修复系列 (作者: andakai 等)**
*   **#61166 (NVFP4 MoE)**：修复 Bug，将派生的 NVFP4 MoE 输入全局缩放比例保留为层参数，避免每次加载时重复创建。
*   **#61165 (MoE LoRA)**：修复 Bug，确保仅当 LoRA target_modules 包含专家模块时，才使用 LoRA MoE kernels。
*   **#61164 (MoE LoRA)**：修复 Bug，根据输入形状正确推导 fused MoE LoRA 的输入行。

**2. 推测解码与 MRV2 修复**
*   **#61161 (Spec Decode)**：修复 Bug，避免在块验证轮次中重复使用随机数，消除了跨轮次的采样偏差。
*   **#61162 (MRV2 LoRA)**：修复非 eager 模式下，KV 共享快速 prefill 的 LoRA metadata 绑定及紧凑图捕获问题。

**3. 硬件兼容与性能优化**
*   **#61163 (XPU 性能)**：为 Qwen MoE decode 新增 XPU 编译优化，融合路由专家与 Sigmoid 门控的共享专家，支持 FP16/BF16 激活及在线/静态 FP8 权重。
*   **#61158 (XPU 修复)**：修复 XPU 休眠模式测试在内存释放时导致 `pure virtual method called` 崩溃的问题。
*   **#61156 (EP 优化)**：限制 NIXL EP 每卡 dispatch 容量上限为 1024 个 tokens，并复用同步机制。
*   **#61155 (ROCm 修复)**：修复在开启 AITER 时，DiffusionGemma 26B 模型在 TP4 下启动卡死的问题。

**4. 新模型支持**
*   **#61159**：新增 **TeleOCR** 模型内置支持，复用现有的 Qwen2.5-VL 视觉架构与 Qwen3 文本解码器。

---

## 🐛 Issues

### #61160 — [[Bug]: Cross-request context contamination via shared-prefix KV reuse in PD disaggregation with automatic prefix caching — clean with `--no-enable-prefix-caching`](https://github.com/vllm-project/vllm/issues/61160)
- **作者**: Siquan-Li  **时间**: 2026-10-12 13:28 CST
- **标签**: kv-connector, kv-cache-manager
- **摘要**: ## Summary  With automatic prefix caching enabled (the default) in a NIXL prefill/decode disaggregated deployment serving DeepSeek-V4.1-Flash, responses intermittently contain **content from other requests** — cross-request context contamination ("crosstalk"). We have a scriptable reproducer on `/v1…

### #61157 — [[Bug] GLM hybrid-SSM + MTP: deep-ctx agent session sharply degrades (fabricated tool args, self-amnesia) exactly when a concurrent request arrives — persists after #58454/#58368](https://github.com/vllm-project/vllm/issues/61157)
- **作者**: ming616  **时间**: 2026-10-12 13:13 CST
- **标签**: tool-calling, quantization, glm
- **摘要**: ## Summary  On a hybrid-SSM model (GLM-5.3-Flash) served with MTP speculative decoding + prefix caching, a real long-horizon agent session ran cleanly for ~97 tool-loop rounds and then **abruptly degenerated within a single turn boundary** — empty tool-call arguments, fabricated file paths that neve…

## 🔀 Pull Requests

### #61166 — [[Bugfix][MoE][NVFP4] Keep the derived NVFP4 MoE input global scales as layer parameters](https://github.com/vllm-project/vllm/pull/61166)
- **作者**: andakai  **时间**: 2026-10-12 14:11 CST
- **标签**: bug
- **摘要**: <!-- markdownlint-disable -->  ## Overview  `make_nvfp4_moe_quant_config` recreates `a1_gscale = 1 / a13_scale` (read by `prepare()` for input quantization) on every `process_weights_after_loading`. Because it is not a parameter, weight reload never restores it into the storage captured by CUDA grap…

### #61165 — [[Bugfix][MoE][LoRA] Use LoRA MoE kernels only when target_modules reaches the experts](https://github.com/vllm-project/vllm/pull/61165)
- **作者**: andakai  **时间**: 2026-10-12 14:11 CST
- **标签**: bug
- **摘要**: <!-- markdownlint-disable -->  ## Overview  `FusedMoEConfig.is_lora_enabled` is `lora_config is not None`. With attention-only LoRA (`--lora-target-modules q_proj k_proj v_proj o_proj`) the experts are never LoRA-wrapped, but every MoE layer still switches to its LoRA-capable kernel. This PR applies…

### #61164 — [[Bugfix][LoRA][MoE] Derive the fused MoE LoRA input row from the input shape](https://github.com/vllm-project/vllm/pull/61164)
- **作者**: andakai  **时间**: 2026-10-12 14:11 CST
- **标签**: bug
- **摘要**: <!-- markdownlint-disable -->  ## Overview  The fused MoE LoRA kernels pick the input row of a `(token, top_k)` route with `token_mapping_factor = 1 if mul_routed_weight else top_k`. This ties "apply the routing weight" to "the input has one row per route". An unweighted w2 delta therefore reads the…

### #61163 — [[Perf][XPU] Fuse routed and shared experts for Qwen MoE decode](https://github.com/vllm-project/vllm/pull/61163)
- **作者**: hzjane  **时间**: 2026-10-12 14:04 CST
- **标签**: documentation, intel-gpu, torch.compile, qwen
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Add an opt-in XPU compile pass that fuses routed experts and one sigmoid-gated shared expert for Qwen MoE decode. Support FP16/BF16 activations with online/static per-tensor FP8 weights while preserving the existing gate/top-k routing and tensor-parallel r…

### #61162 — [[Bugfix][MRV2][LoRA] Fix LoRA metadata binding for non-eager KV-sharing fast prefill](https://github.com/vllm-project/vllm/pull/61162)
- **作者**: waizuichougou  **时间**: 2026-10-12 13:56 CST
- **标签**: bug, nvidia, mrv2
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Follow-up to [#59067](https://github.com/vllm-project/vllm/pull/59067), which fixes eager MRV2 LoRA KV-sharing fast prefill. This PR fixes metadata binding and compact graph capture for Gemma3n and Gemma4 under compile-only, compiled PIECEWISE and breakabl…

### #61161 — [[Bugfix][Spec Decode] Avoid reusing random draws across block verification rounds](https://github.com/vllm-project/vllm/pull/61161)
- **作者**: DimensionSTP  **时间**: 2026-10-12 13:34 CST
- **标签**: bug, speculative-decoding, mrv2
- **摘要**: ## Overview  Fixes #61114. Give block verification fresh acceptance and draft random draws when an uncommitted position is revisited in a later round.  ## Claims  - Removes the cross-round sampling bias for deterministic and probabilistic drafts in Model Runner V2. - Keeps standard verification outp…

### #61159 — [[Model] Add TeleOCR support](https://github.com/vllm-project/vllm/pull/61159)
- **作者**: Liuchenbing-2026  **时间**: 2026-10-12 13:26 CST
- **标签**: documentation, new-model, qwen
- **摘要**: ## Overview  Add built-in support for [TeleOCR](https://github.com/caipeng328/TeleOCR), using the existing Qwen2.5-VL vision stack and a Qwen3 text decoder.  ## Claims  - Register `TeleOCRForConditionalGeneration` without replacing the Qwen2.5-VL architecture globally. - Initialize Qwen3 with `hf_te…

### #61158 — [[XPU] refine memory release for sleep mode](https://github.com/vllm-project/vllm/pull/61158)
- **作者**: yma11  **时间**: 2026-10-12 13:23 CST
- **标签**: intel-gpu
- **摘要**: ## Overview  Fix an XPU teardown crash where sleep-mode tests pass their assertions but subsequently abort with `pure virtual method called` during `empty_cache()`. `XpuMemAllocator.release_pools()` previously dropped allocator wrappers after releasing the Python MemPool references. However, live te…

### #61156 — [[EP] Bound NIXL EP dispatch capacity](https://github.com/vllm-project/vllm/pull/61156)
- **作者**: Etelis  **时间**: 2026-10-12 13:07 CST
- **标签**: kv-connector
- **摘要**: Depends on #60900. Follow-up to [@itayalroy's request](https://github.com/vllm-project/vllm/pull/60900#pullrequestreview-5476763020).  Cap NIXL EP dispatch at 1,024 tokens per rank and reuse the synchronized chunking path for larger batches. This bounds RDMA buffers and expert workspaces independent…

### #61155 — [[ROCm][Model][BugFix] Fix DiffusionGemma hang with TP4 with AITER on](https://github.com/vllm-project/vllm/pull/61155)
- **作者**: rasmith  **时间**: 2026-10-12 13:01 CST
- **标签**: bug, rocm
- **摘要**: ## Overview  With `AITER `enabled (`VLLM_ROCM_USE_AITER=1`), `RedHatAI/diffusiongemma-26B-A4B-it-FP8-dynamic` hangs at TP4 during server startup and the server never comes up:  * `vllm serve`:   ```   VLLM_ROCM_USE_AITER=1 vllm serve RedHatAI/diffusiongemma-26B-A4B-it-FP8-dynamic --dtype auto -tp 4 …
