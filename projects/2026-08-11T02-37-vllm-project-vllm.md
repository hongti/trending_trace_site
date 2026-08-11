# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-11 10:37 CST

## AI 总结

# vLLM 仓库近期动态摘要

---

## 📌 Issue

- **#51752 [Bug]**：在流水线并行中，不拥有注意力层的 rank 会跳过混合 block-size 对齐，导致潜在的对齐错误。环境为 2×RTX 3090，已在 main 分支确认。
- **#51751 [RFC]**：提议扩展 `--kv-cache-dtype` 参数，使其支持可插拔的 KV Cache 数据类型，当前仅支持 `auto`/`float16`/`bfloat16`/`fp8` 等有限选项，灵活性不足。

---

## 🔀 Pull Request

### 新特性 / 增强
- **#51754 [MoE][NVIDIA]**：为 TRTLLM FP8 添加 **共享专家融合**，将共享专家表示为常驻路由专家槽位，启用 DeepSeek-V3.2 / GLM-5.2 模型的该优化路径。
- **#51747 [可观测性]**：为 GPU-less render server 的所有端点添加 **OpenTelemetry 追踪**，支持 W3C trace context 关联。
- **#51746 [前端]**：显式化 Model Info 缓存准备流程，缓存命中后模型类检查从 ~10s 降至 ~0.6s。

### Bug 修复
- **#51749 [Bugfix]**：修复 **sliding-window KV cache 块未清零** 问题，补全 `KVBlockZeroer` 对滑动窗口注意力组的处理。
- **#51742 [Bugfix]**：修复混合模型 + DFlash 场景下 **KV cache group 爆炸** 问题（如 Qwen3.6-35B-A3B + DFlash drafter 导致注意力组数异常膨胀）。
- **#51741 [Bugfix]**：FlashInfer 采样内核构建失败时，**回退到原生采样器**，避免运行时崩溃。

### 安全修复
- **#51748 [Security]**：HF3FS 外部缓存 key 仅基于 token ID，缺少 `cache_salt`，多引擎共享元数据服务时可能发生 **缓存别名/碰撞**，现已加入 salt。

### 代码质量
- **#51753 [Misc]**：将评分输入验证中的 `ValueError` 迁移为 `VLLMValidationError`，统一错误处理。

### 回退
- **#51750**：**回退** PR #51430（"Narrow DeepSeek V4 eager CUDA graph region"），因导致 Nightly CI 构建失败。

---

## 🚀 Release — v0.27.0

**版本亮点：**

- 共 **561 个提交**，来自 **242 位贡献者**（其中 64 位新贡献者）
- 🏆 **新增 Kimi K3 模型全栈支持**
- 其余详细特性需参考完整 Release Notes

> 此次版本规模较大，涉及大量模型支持、性能优化和 bug 修复，建议升级用户关注完整变更日志。

---

## 🐛 Issues

### #51752 — [[Bug]: Hybrid block-size alignment is skipped on pipeline-parallel ranks that own no attention layer](https://github.com/vllm-project/vllm/issues/51752)
- **作者**: nickus  **时间**: 2026-08-11 10:07 CST
- **标签**: kimi
- **摘要**: ### Your current environment  ``` Hardware: 2 × NVIDIA RTX 3090 (sm_86), single node vLLM: built from source @38a267cdd; bug re-verified by code inspection on main @1a1727330a Distributed executor: mp, --pipeline-parallel-size 2, --tensor-parallel-size 1 Model: Kimi-K3 (hybrid: KDA linear attention …

### #51751 — [[RFC]: Pluggable KV Cache Data Types](https://github.com/vllm-project/vllm/issues/51751)
- **作者**: lcfenglinwan  **时间**: 2026-08-11 10:06 CST
- **标签**: RFC, quantization
- **摘要**: ### Motivation.  The `--kv-cache-dtype` CLI parameter currently accepts only the values listed in the `CacheDType` literal:  ```python CacheDType = Literal[     "auto", "float16", "bfloat16", "fp8", "fp8_e4m3", "fp8_e5m2",     "fp8_inc", "fp8_ds_mla", "turboquant_k8v4", "turboquant_4bit_nc",     "tu…

## 🔀 Pull Requests

### #51754 — [[MoE][NVIDIA] Add shared expert fusion for TRTLLM FP8](https://github.com/vllm-project/vllm/pull/51754)
- **作者**: WoosukKwon  **时间**: 2026-08-11 10:23 CST
- **标签**: deepseek, nvidia, quantization
- **摘要**: ## Summary  - Extend the FusedMoE abstraction so shared experts can be represented as always-on routed expert slots on NVIDIA GPUs. - Enable the path for the NVIDIA DeepSeek-V3.2/GLM-5.2 model when routed and shared expert storage formats match. - Run fused FP8 shared experts through the modular Fla…

### #51753 — [[Misc] Use VLLMValidationError in scoring input validation](https://github.com/vllm-project/vllm/pull/51753)
- **作者**: frank-suwen  **时间**: 2026-08-11 10:21 CST
- **标签**: frontend
- **摘要**: ## Purpose  Part of #48227.  Migrate four client-caused validation errors in `vllm/entrypoints/pooling/scoring/utils.py` from raw `ValueError` to `VLLMValidationError`:  - unsupported multimodal input - incompatible input lengths - empty first input - empty second input  This preserves the existing …

### #51750 — [Revert "[Perf] Narrow DeepSeek V4 eager CUDA graph region" (#51430)](https://github.com/vllm-project/vllm/pull/51750)
- **作者**: vllm-agent  **时间**: 2026-08-11 10:06 CST
- **标签**: deepseek, nvidia
- **摘要**: Reverts vllm-project/vllm#51430 — "[Perf] Narrow DeepSeek V4 eager CUDA graph region" (merge commit `79c865b`).  ## Why  Nightly CI build [#83211](https://buildkite.com/vllm/ci/builds/83211) (commit `635dd6a`) failed `MoE Refactor Integration Test (B200 - TEMPORARY)`:  ``` FAILED evals/gsm8k/test_gs…

### #51749 — [[Bugfix] Zero sliding-window KV cache blocks](https://github.com/vllm-project/vllm/pull/51749)
- **作者**: mgoin  **时间**: 2026-08-11 09:52 CST
- **标签**: bug, ready
- **摘要**: ## Summary  - Record newly allocated `SlidingWindowSpec` block IDs when worker-side KV zeroing is required. - Include sliding-window attention groups in `KVBlockZeroer`. - Add focused scheduler and worker regressions.  ## Root cause  `KVCacheConfig.needs_kv_cache_zeroing` is enabled for hybrid Mamba…

### #51748 — [[Security] Include cache_salt in HF3FS external cache keys](https://github.com/vllm-project/vllm/pull/51748)
- **作者**: infinityscroll  **时间**: 2026-08-11 09:48 CST
- **标签**: kv-connector
- **摘要**: ## Summary  HF3FS external cache keys were derived from token IDs alone and omitted the per-request `cache_salt`. Engines sharing an HF3FS metadata service could therefore alias the same exact prefix across different salts and expose cached-prefix membership through hit observability.  This change: …

### #51747 — [[Observability] Add OpenTelemetry tracing to GPU-less render server endpoints](https://github.com/vllm-project/vllm/pull/51747)
- **作者**: mschulist  **时间**: 2026-08-11 09:36 CST
- **标签**: frontend
- **摘要**: ## Summary  Add OpenTelemetry tracing to the GPU-less render server (`vllm launch render`): every endpoint the command opens now emits a `SpanKind.SERVER` span linked to the caller's W3C trace context.  - **`vllm/entrypoints/cli/launch.py`** — `run_launch_fastapi` built `VllmConfig(model_config=...)…

### #51746 — [[Frontend] Add explicit model info cache preparation](https://github.com/vllm-project/vllm/pull/51746)
- **作者**: matteso1  **时间**: 2026-08-11 09:35 CST
- **标签**: documentation, new-model, frontend
- **摘要**: ## Purpose  PR #23558 by @manoelmarques added the source-hash-validated runtime ModelInfo cache and reported model-class inspection at 10.1034 seconds uncached versus 0.6127 seconds cached, n=1 per state. During review, @hmellor objected to mirroring common runtime requirements into the build. The m…

### #51745 — [Kunshang/whl rls](https://github.com/vllm-project/vllm/pull/51745)
- **作者**: jikunshang  **时间**: 2026-08-11 09:12 CST
- **标签**: documentation, needs-rebase, ci/build
- **摘要**: ## Purpose  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such as providing test command. - [ ] The …

### #51742 — [Fix KV cache group explosion for hybrid models with DFlash](https://github.com/vllm-project/vllm/pull/51742)
- **作者**: xyang16  **时间**: 2026-08-11 08:16 CST
- **摘要**: ## Purpose  For Qwen/Qwen3.6-35B-A3B-FP8 + z-lab/Qwen3.6-35B-A3B-DFlash, the target model layer buckets are [30 mamba, 10 full]. When a DFlash drafter adds [5 sliding, 1 full], the added full-attention layer that can't merge with the target's 10 full attention layers (different spec fields). The cur…

### #51741 — [[Bugfix] Fall back to native sampler when FlashInfer sampling kernel fails to build](https://github.com/vllm-project/vllm/pull/51741)
- **作者**: Dhruv235  **时间**: 2026-08-11 07:25 CST
- **标签**: bug, mrv2
- **摘要**: > **AI assistance was used on this PR.** The fix and its design are mine; an AI assistant > (Claude Opus 5) helped with the rebase onto current `main`, running and interpreting the > test/control runs, and drafting this description. Attribution is in the commit trailer. > I have reviewed every chang…

## 🚀 Releases

### [v0.27.0](https://github.com/vllm-project/vllm/releases/tag/v0.27.0)
- **作者**: khluu  **时间**: 2026-08-11 05:18 CST
- **摘要**: # vLLM v0.27.0 Release Notes  ## Highlights  This release features 561 commits from 242 contributors (64 new)!  * **Kimi K3 support** with a full stack landing in one release: core model files and kernels (#50089, #50000), Python (#50093) and Rust (#50104) frontends, AttnRes kernels (#50090), DeepGE…
