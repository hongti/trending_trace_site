# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-22 10:01 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
近期 Issue 主要集中在**底层量化精度**与**新硬件兼容性**问题：
*   **FP8 量化计算偏差**：在微小分组（tiny groups）下，UE8M0 `per_token_group_quant_fp8` 的 C++ 实现与 Triton 回退路径算出的缩放系数存在按位级别的差异 (#53339)。
*   **DeepGEMM 内核步幅缺陷**：当 `num_tokens >= 2` 且使用列主序 F32 缩放时，`_fwd_kernel_ep_scatter_2` 忽略了 `recv_x_scale_stride1`，导致计算错误 (#53338)。
*   **TurboQuant KV 缓存异常**：在 sm121 (GB10) 架构评估中，TurboQuant KV 缓存预设导致混合模型初始化失败，且在遇到大幅值异常值时出现 FP16 零点溢出 (#53334)。

### 🔀 Pull Request 动态
PR 活动涵盖新特性、硬件适配、性能优化及多项关键 Bug 修复：

**✨ 新特性与增强**
*   **推理内容支持**：为 `ChatMessage` 协议新增 `reasoning_content` 字段及安全字典序列化器，适配推理模型输出 (#53340)。
*   **流式权重重载**：引入基于 manifest 的流式模型权重重载机制，支持 Dense、量化和 MoE 检查点，且按需懒加载以节省内存 (#53331)。

**🚀 硬件支持与性能优化**
*   **NVIDIA Thor (SM110) 适配**：新增 Mamba `selective_state_update` 的调优配置 (#53337)。
*   **ROCm/gfx950 性能提升**：新增基于 FlyDSL 的 prefill attention 后端 (#53335，依赖上游合并)。
*   **NIXL 延迟优化**：在 P/D 分离架构中，将 NIXL pull 请求推迟到模型启动后提交，显著减少解码阶段的主机耗时（从 5ms+ 偶发 48ms 降至更低） (#53333)。

**🛠️ 关键 Bug 修复**
*   **KV Offload 逻辑**：修复 P2P 层中生产者供应与消费者需求未对齐的问题 (#53330)；修复飞行中（in-flight）主键在请求级层叠时的逻辑缺陷 (#53329)。
*   **量化误报修复**：当检查点为 weight-only 时，移除“GPU 不支持原生 FP4 计算”的误导性警告 (#53328)。
*   **Spec Decoded 修复**：在 KV-cache 布局重构后，重新应用 FlashAttention 元数据的分组几何修复 (#53336)。
*   **CI 稳定性**：强化 MoE 层测试中 DeepEP NVSHMEM 的重新初始化逻辑，修复导致主分支 CI 挂起的问题 (#53332)。

### 🚀 Release 动态
*   本期暂无新版 Release 发布。

---

## 🐛 Issues

### #53339 — [[Bug]: UE8M0 per_token_group_quant_fp8 scales differ between the C++ and Triton fallback paths for tiny groups](https://github.com/vllm-project/vllm/issues/53339)
- **作者**: Suppressor72  **时间**: 2026-08-22 09:24 CST
- **标签**: quantization
- **摘要**: ## Environment  - vLLM nightly `0.26.1rc1.dev926+gb05ae5dc0` (collect_env output attached below   attached below); divergence verified bitwise on hardware   below; both implementations checked against current `main`   (`fp8_utils.py` Triton fallback; `csrc/libtorch_stable/   quantization/w8a8/fp8/pe…

### #53338 — [[Bug]: _fwd_kernel_ep_scatter_2 ignores recv_x_scale_stride1 for column-major f32 scales when num_tokens >= 2](https://github.com/vllm-project/vllm/issues/53338)
- **作者**: Suppressor72  **时间**: 2026-08-22 09:24 CST
- **标签**: quantization
- **摘要**: ## Environment  - vLLM nightly `0.26.1rc1.dev926+gb05ae5dc0` (collect_env output attached below   attached below); bug present in current `main` source of   `deep_gemm_utils.py`. - Hardware: 2× RTX 5090 (SM120), TP=2, CUDA 12.x. - Model: Qwen3.6-35B-A3B-FP8 (DeepGEMM grouped-FP8 contiguous MoE path)…

### #53334 — [TurboQuant KV cache: two observations from an sm121 evaluation: hybrid-model init failure; fp16 zero-point overflow on large value outliers](https://github.com/vllm-project/vllm/issues/53334)
- **作者**: TechPrototyper  **时间**: 2026-08-22 08:13 CST
- **摘要**: (hybrid-model init failure; fp16 zero-point overflow on large value outliers)**  We evaluated the TurboQuant KV-cache presets on a GB10 (sm121) against our production NVFP4-KV recipe and hit two things worth reporting. Full probe code, raw results, and methodology are public.¹  **1. Hybrid linear-at…

## 🔀 Pull Requests

### #53340 — [fix(entrypoints): add reasoning_content field and safe dict serializer to ChatMessage](https://github.com/vllm-project/vllm/pull/53340)
- **作者**: Xayar145  **时间**: 2026-08-22 09:42 CST
- **标签**: frontend
- **摘要**: ## Summary  Adds `reasoning_content` field and serializer safeguards to `ChatMessage` in `vllm.entrypoints.openai.chat_completion.protocol`: - Adds `reasoning_content: str | None = None` to `ChatMessage`, supporting reasoning models (e.g. DeepSeek-R1, QwQ) and downstream OpenAI-compatible proxies th…

### #53337 — [[Kernel] Add tuned Mamba selective_state_update config for NVIDIA Thor (SM110)](https://github.com/vllm-project/vllm/pull/53337)
- **作者**: filipsajdak  **时间**: 2026-08-22 09:19 CST
- **标签**: nvidia
- **摘要**: ## Purpose  Adds a tuned `selective_state_update` config for **NVIDIA Thor (SM110)**. The tree ships 17 tuned Mamba SSU configs (B200, GB200, H100, MI300X/325X/350/355, RTX PRO 6000 Blackwell) and none for any Jetson, so every hybrid-Mamba model load on Thor logs:  ``` [mamba_ssm.py] Using default M…

### #53336 — [[Bugfix][Spec Decode] Reapply group geometry for FlashAttention metadata](https://github.com/vllm-project/vllm/pull/53336)
- **作者**: mgoin  **时间**: 2026-08-22 09:16 CST
- **标签**: bug, speculative-decoding, ready, mrv2, dflash
- **摘要**: ## Summary  Reapply #53002 after it was temporarily reverted in #51718 to unblock the KV-cache layout refactor.  FlashAttention metadata builders use their own attention group as the source of truth for geometry:  - query heads come from the registered attention layers - KV heads and head size come …

### #53335 — [[ROCm][Perf][Attention] Add FlyDSL gfx950 prefill attention backend](https://github.com/vllm-project/vllm/pull/53335)
- **作者**: akii96  **时间**: 2026-08-22 08:44 CST
- **标签**: rocm
- **摘要**: > [!IMPORTANT] > This PR needs https://github.com/ROCm/FlyDSL/pull/1056 merged first, and then picked up by the flydsl install that ships with aiter. Until that lands the backend has no kernel to call and will refuse to start.  ## Motivation  On ROCm the attention kernel that serves long prompts is …

### #53333 — [[Core][KV Connector][NIXL] Defer pull posting until after model launch](https://github.com/vllm-project/vllm/pull/53333)
- **作者**: GirasoleY  **时间**: 2026-08-22 07:37 CST
- **标签**: kv-connector, mrv2
- **摘要**: ## Purpose  In scaled P/D setup, we see decode spend long host time posting NIXL pull requests: i.e. 5ms for 9 remote KV requests fanned out across TP8 (= 72 NIXL handles), occasionally spike to 48.427ms. 72 NIXL handles.   * The time consuming part consists of synchronous descriptor construction: `…

### #53332 — [[CI][BugFix] Harden DeepEP NVSHMEM re-initialization in MoE layer tests](https://github.com/vllm-project/vllm/pull/53332)
- **作者**: khluu  **时间**: 2026-08-22 07:36 CST
- **标签**: bug
- **摘要**: ## Purpose  Harden the DeepEP paths in `tests/kernels/moe/test_moe_layer.py` against an intermittent NVSHMEM re-initialization failure that is currently failing or hanging main-branch CI jobs with no product defect involved.  On CUDA, the test tears down the DeepEP buffer after every DeepEP subtest …

### #53331 — [[Reload] Add manifest-driven streaming modelwise weight reload](https://github.com/vllm-project/vllm/pull/53331)
- **作者**: new-TonyWang  **时间**: 2026-08-22 07:34 CST
- **标签**: documentation, performance, new-model, rocm, structured-output, frontend, intel-gpu, speculative-decoding, ray, torch.compile, needs-rebase, ci/build, multi-modality, tool-calling, llama, qwen, deepseek, cpu, gpt-oss, kv-connector, nvidia, quantization, vllm-ir, mistral, DSv4, dflash, rust, kimi, k3, scheduler, kv-cache-manager, glm, minimax, inkling, cohere
- **摘要**: ## Summary  This PR extends the modelwise reload prototype into a manifest-driven streaming reload path for dense, quantized, and MoE checkpoints.  The implementation:  - lazily materializes only checkpoint bindings needed by received sources; - supports arbitrary checkpoint chunks and rank-sharding…

### #53330 — [[Bugfix][KV Offload] Guard P2P supply against consumer demand](https://github.com/vllm-project/vllm/pull/53330)
- **作者**: almogtavor  **时间**: 2026-08-22 07:05 CST
- **标签**: bug, kv-connector
- **摘要**: ## Purpose  Addresses #53083.  In the PD flow over the P2P secondary tier the producer's supply and the consumer's demand are computed by two independent code paths on two engines, and nothing verifies they agree. When they diverge the consumer's unmatched demand parks until `_LOAD_TIMEOUT_S` and it…

### #53329 — [[Bugfix][KV Offload] Defer request-level cascade of in-flight primary keys](https://github.com/vllm-project/vllm/pull/53329)
- **作者**: almogtavor  **时间**: 2026-08-22 07:05 CST
- **标签**: bug
- **摘要**: ## Purpose  Fixes #53062.  `TieringOffloadingManager._cascade_existing_blocks_to_request_level_tiers` keeps only keys whose primary-tier `lookup` is `HIT`. A `HIT_PENDING` key, present in the primary tier with its write still in flight, is dropped permanently: `prepare_store` already excluded it as …

### #53328 — [[Bugfix][Quantization] Don't claim the GPU lacks FP4 support when the checkpoint is weight-only](https://github.com/vllm-project/vllm/pull/53328)
- **作者**: filipsajdak  **时间**: 2026-08-22 06:22 CST
- **标签**: bug, quantization
- **摘要**: ## Purpose  `prepare_nvfp4_moe_layer_for_marlin()` warns unconditionally:  > Your GPU does not have native support for FP4 computation but FP4 quantization is being used. Weight-only FP4 compression will be used leveraging the Marlin kernel. This may degrade performance for compute-heavy workloads. …
