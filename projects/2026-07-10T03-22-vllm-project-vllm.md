# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-10 11:22 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 📋 Issue（议题 / RFC）

近期核心议题主要围绕**长序列推理优化、投机解码改进及结构化输出重构**，具体包括：
1. **分层与稀疏 KV Cache 卸载 (#48203)**：提出在长序列推理场景下，将 KV Cache 分层或稀疏化卸载，以突破当前推理瓶颈。
2. **自适应投机解码的逐请求有效提议长度 (#48202)**：计划为每个请求动态计算有效的提议长度，以替代当前批次统一的投机深度，提升解码效率。
3. **GLM-5.x thinking_token_budget 默认值与回归测试 (#48201)**：解决 GLM-5.x 模型开启思考模式后，推理阶段耗尽全部生成预算的问题，以保障 OpenAI 兼容的 Chat/Tool-calling 流程正常可用。
4. **StructuredOutputManager 与投机解码重构 (#48197)**：提议简化调度器、语法后端与结构化输出管理器之间的交互契约，特别是在投机解码场景下。

---

### 🔀 Pull Request（代码变更）

近期的 PR 重点在于**实现上述 RFC、修复混合架构 Bug、性能优化及量化支持改进**：

**📌 RFC 参考实现**
- **#48204**：实现 RFC #48202，引入“逐请求有效提议长度”机制，并以 DSpark confidence head 作为首个应用实例。
- **#48200** (Draft)：实现 RFC #48197，重构 `StructuredOutputManager` 与投机解码逻辑，精简约 200 行非测试代码，并整合了大量测试用例。

**🐛 Bug 修复与测试**
- **#48195**：修复 Hybrid Mamba + KV Connector 在前缀命中出现发散（FA 组浅、Mamba 组深）时导致的异常。
- **#48199**：修复量化警告误报问题——在原生支持 FP4 的 GPU（SM100/SM12x）上，移除了 NVFP4 Marlin 错误提示“GPU 不原生支持 FP4”的日志。
- **#48198**：为 Hybrid-Mamba 前缀缓存导致的隐蔽输出损坏问题 (#43559)，添加了确定性的端到端回归测试（标记为 xfail）。

**⚡ 性能优化与核心改进**
- **#48188**：将 Mamba chunk 元数据计算速度**提升约 6 倍**，主要优化了占大头的逐元素 `.item()` 调用。
- **#48196**：针对稀疏 MLA（Multi-head Latent Attention），优化 DCP 解码注意力输出的合并过程。
- **#48189**：合并并发的外部前缀 KV 加载请求，通过核心注册机制避免重复加载与 GPU block 分配，减少开销。
- **#48190**：使 DeepGEMM JIT 缓存键在不同安装布局下可移植，避免冷启动时重复编译大量内核（如 Qwen3-30B-A3B-FP8 需预热约 800 个用例）。
- **#48192**：清理 FA4 fp8 KV 反量化集成代码。

---

### 🚀 Release（版本发布）

**近期无新版本发布记录。**（当前提供的数据中不包含 Release 信息）

---

## 🐛 Issues

### #48203 — [[RFC]: Layerwise and Sparse KV cache offloading to support longer sequence length](https://github.com/vllm-project/vllm/issues/48203)
- **作者**: zhangsicheng5  **时间**: 2026-07-10 11:02 CST
- **标签**: RFC
- **摘要**: ## Motivation.  In long sequence inference scenario, KV cache size has become one of the inference bottlenecks, arousing great interest in KV cache offloading. We also notice that KV cache offloading is especially efficient for sparse attention based model: Take [DeepSeek-V3.2](https://huggingface.c…

### #48202 — [[RFC]: Per-request effective proposal lengths for adaptive speculative decoding](https://github.com/vllm-project/vllm/issues/48202)
- **作者**: LJX-xixi  **时间**: 2026-07-10 10:55 CST
- **标签**: RFC
- **摘要**: ### Motivation.  ### Current state  vLLM has established support for adapting a batch-uniform speculation depth. Dynamic SD (#32374) profiles position-level acceptance and ITL offline and selects a per-step uniform K by goodput at runtime (currently EAGLE-1), and #45953 extended it to work with full…

### #48201 — [[RFC]: GLM-5.x thinking_token_budget defaults and regression tests](https://github.com/vllm-project/vllm/issues/48201)
- **作者**: AlanJager  **时间**: 2026-07-10 10:49 CST
- **摘要**: ### Motivation  GLM-5.x models can spend the whole completion budget in the reasoning phase when thinking is enabled by default. This makes OpenAI-compatible chat/tool-calling flows difficult to use unless the caller explicitly disables thinking or sets a small `thinking_token_budget`.  In a local v…

### #48197 — [[RFC]: StructuredOutputManager x Speculative Decoding Refactor](https://github.com/vllm-project/vllm/issues/48197)
- **作者**: yzong-rh  **时间**: 2026-07-10 10:13 CST
- **标签**: RFC
- **摘要**: ### Motivation.  Simplify the contract between `Scheduler`, grammar backend, and `StructuredOutputManager`, especially when speculative decoding is used.  `StructuredOutputManager` exists because constrained decoding often doesn't kick in until a model has finished reasoning. With speculative decodi…

## 🔀 Pull Requests

### #48204 — [[Core][Spec Decode] Reference implementation: per-request effective proposal lengths (RFC #48202)](https://github.com/vllm-project/vllm/pull/48204)
- **作者**: LJX-xixi  **时间**: 2026-07-10 11:06 CST
- **标签**: speculative-decoding, v1, qwen
- **摘要**: ## Purpose  Reference implementation for RFC #48202: per-request effective proposal lengths for speculative decoding, with the DSpark confidence head as the first consumer. Draft PR: the metadata ownership question in the RFC is still open, and this branch implements the "independent field on propos…

### #48200 — [[DRAFT] [Refactor]: StructuredOutputManager x Speculative Decoding Refactor](https://github.com/vllm-project/vllm/pull/48200)
- **作者**: yzong-rh  **时间**: 2026-07-10 10:24 CST
- **标签**: structured-output, speculative-decoding, v1
- **摘要**: ## Purpose  See https://github.com/vllm-project/vllm/issues/48197.  ~100 insertions, ~200 deletions of non-test changes.  Then ~400 insertions, ~600 deletions for test consolidation. Moves `tests/v1/spec_decode/test_mtp_structured_output.py` and `tests/v1/structured_output/test_reasoning_structured_…

### #48199 — [[Quantization] Fix misleading NVFP4 Marlin linear warning on FP4-native GPUs](https://github.com/vllm-project/vllm/pull/48199)
- **作者**: waynehacking8  **时间**: 2026-07-10 10:16 CST
- **摘要**: ## Purpose  Fixes #47749  `MarlinNvFp4LinearKernel.process_weights_after_loading` unconditionally logs "Your GPU does not have native support for FP4 computation". That claim is wrong on SM100/SM12x: `ModelOptNvFp4W4A16LinearMethod` pins the Marlin kernel for weight-only `W4A16_NVFP4` checkpoints on…

### #48198 — [[Test] Deterministic e2e regression tests (xfail) for hybrid-Mamba prefix cache corruption (#43559)](https://github.com/vllm-project/vllm/pull/48198)
- **作者**: puririshi98  **时间**: 2026-07-10 10:15 CST
- **标签**: ci/build, v1
- **摘要**: # [Test] Deterministic e2e regression tests (xfail) for hybrid-Mamba prefix cache corruption (#43559)  ## Purpose  #43559 reports silent, hard-to-reproduce output corruption on hybrid-Mamba models when prefix caching and MTP speculative decoding are combined. The maintainer ask in that thread was a …

### #48196 — [[Attention] DCP sparse MLA output-merge optimizations](https://github.com/vllm-project/vllm/pull/48196)
- **作者**: LucasWilkinson  **时间**: 2026-07-10 09:35 CST
- **标签**: v1, nvidia
- **摘要**: ## Summary  Optimizes the DCP decode attention-output merge for sparse MLA. This is one of two PRs split out of #47355 (which is now scoped to just the indexer side-stream overlap). The two are **independent** and target `main` directly — they touch disjoint regions of `forward_impl`, so they can me…

### #48195 — [[BugFix] Fix per-group prefix-hit divergence for hybrid Mamba + KV connector](https://github.com/vllm-project/vllm/pull/48195)
- **作者**: Xuan-yi-yan  **时间**: 2026-07-10 09:33 CST
- **标签**: bug, v1
- **摘要**: Fixes #46453  ## Root Cause  When hybrid models (full-attention + Mamba) with KV connector have per-group cache hits that diverge — FA group shallow, Mamba group deep due to state block survival — `max(per_group_hits)` picks the deeper Mamba value. The connector then accesses FA blocks beyond the va…

### #48192 — [Fa4 fp8 kv dequant integration clean](https://github.com/vllm-project/vllm/pull/48192)
- **作者**: qixiang-99  **时间**: 2026-07-10 07:56 CST
- **标签**: needs-rebase, v1
- **摘要**: ## Purpose  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such as providing test command. - [ ] The …

### #48190 — [[Core] Make the DeepGEMM JIT cache key portable across install layouts](https://github.com/vllm-project/vllm/pull/48190)
- **作者**: matteso1  **时间**: 2026-07-10 07:06 CST
- **标签**: ci/build
- **摘要**: ## Purpose  DeepGEMM's JIT warmup compiles 41 distinct kernels (~797 warmup cases) on a cold Qwen3-30B-A3B-FP8 boot. The results are fully cacheable, and vLLM already pins `DG_JIT_CACHE_DIR` under `VLLM_CACHE_ROOT` — but the cache key defeats reuse across environments. The key (`csrc/jit/compiler.hp…

### #48189 — [[Core] Coalesce concurrent external-prefix KV loads](https://github.com/vllm-project/vllm/pull/48189)
- **作者**: samar1tan  **时间**: 2026-07-10 06:49 CST
- **标签**: v1, kv-connector
- **摘要**: ## Summary  - add a core registry for coalescing concurrent asynchronous KV loads of the same exact external prefix - park followers without allocating GPU blocks or issuing another connector load - publish the owner's blocks through the ordinary GPU prefix cache before releasing followers - opt the…

### #48188 — [[Perf] Speed up Mamba chunk metadata computation by ~6x](https://github.com/vllm-project/vllm/pull/48188)
- **作者**: samuelkim7  **时间**: 2026-07-10 06:42 CST
- **标签**: v1
- **摘要**: ## Purpose  `BaseMambaAttentionMetadataBuilder._compute_chunk_metadata` carries a `# TODO (tdoublep): This code could probably be optimized.` Its cost is dominated by per-element `.item()` calls (3 per prefill request) and a Python loop that iterates once per chunk (e.g. 32 iterations per 8k-token r…
