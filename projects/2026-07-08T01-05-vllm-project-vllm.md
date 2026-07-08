# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-08 09:05 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 🟥 Issue 动态
近期 Issue 主要集中在新特性/新模型的兼容性 Bug 及性能优化提议：
*   **Bug: DP/EP 下 FlashMLA 的 fp8 KV Cache 异常** (#47935)：在使用 Data/Expert Parallelism 时，fp8 KV Cache 导致 FlashMLA 功能失效。
*   **Bug: DSpark 在 GLM 5.2 FP4 模型上启动失败** (#47934)：使用 `nvidia/GLM-5.2-NVFP4` 作为目标模型时，DSpark 无法正常启动。
*   **Bug: 自动前缀缓存导致 DFlash/DSpark 投机解码接受率崩溃** (#47930)：开启 APC 后，DFlash/DSpark 的 draft 上下文 KV 获取受阻，导致投机接受率大幅下降（该 Bug 已在 PR #47926 中修复）。
*   **Feature: 原生卸载中跨 TP ranks 去重 MLA KV Cache** (#47929)：提议在 MLA 模型的原生 Offloading 场景中，跨张量并行（TP）ranks 去重逻辑上完全相同的 latent KV cache，以节省存储。

### 🟩 PR 动态
近期 PR 涵盖核心性能优化、投机解码修复、前端 API 修复及架构 RFC：

**1. 性能优化与核心架构**
*   **减少 BlockTable 拷贝开销** (#47936)：V1 架构下，稳态解码时 `commit_block_table` 仅将发生修改的“脏行”从 Host 拷贝至 Device，避免全量拷贝，提升性能。
*   **Cudagraph 内存池优化** (#47925)：将 cudagraph 输出暂存缓冲区预分配大小设为最大捕获描述符，缓解池碎片化问题，改善启动体验。
*   **端到端编译 Decode 步骤 (RFC)** (#47924)：提出将 decode 步骤（含 KV-cache 管理/slot-mapping）进行端到端编译（e2e compile）的原型方案，旨在进一步压榨性能。
*   **KV Offload 事件解耦** (#47923)：FS/OBJ 二级层级现在各自负责并发出其 `BlockStored` 事件，完善了 KV 卸载的事件管理机制。

**2. 投机解码修复与增强**
*   **修复 DFlash/DSpark 与 APC 的冲突** (#47926)：将 prefix-cache 恢复的 token 从 DFlash/DSpark 的 draft context 中屏蔽，解决了 Issue #47930 的接受率崩溃问题。
*   **Speculative Slots 计算修正** (#47928)：在空输出行上正确计算已预定的 speculative slots，避免 MTP 等投机解码步骤中的 slot 计算遗漏。

**3. 模型支持与路由修复**
*   **保留 DeepSeek Sigmoid Grouped Routing 元数据** (#47927)：确保 DeepSeekV4/DSA 模型的 sigmoid 路由机制（含 expert-score 校正偏差、n_group/topk_group 等）不被丢失。

**4. 前端 API 及 AMD 环境修复**
*   **Scale-out Token Stream 保留 abort 状态** (#47933)：在 V1 abort 路径中，为 scale-out token-in/out 服务保留 `finish_reason="abort"`。
*   **Derender 返回 tool_calls 原因** (#47931)：修复 `/v1/chat/completions/derender` 接口解析 tool calls 时未正确返回 `tool_calls` finish reason 的问题。
*   **AMD/The Rock 视觉多模态示例修复** (#47932)：为 AMD 环境下的视觉多模态示例添加 `spawn` 多进程方法，避免重初始化报错。

### 🟦 Release 动态
*   本次提供的数据中**无新的版本发布**记录。

---

## 🐛 Issues

### #47935 — [[Bug]: DP/EP with fp8 KV Cache Brokens for FlashMLA](https://github.com/vllm-project/vllm/issues/47935)
- **作者**: robertgshaw2-redhat  **时间**: 2026-07-08 08:26 CST
- **标签**: bug
- **摘要**: ### Your current environment  launch manifest:  ``` apiVersion: leaderworkerset.x-k8s.io/v1 kind: LeaderWorkerSet metadata:   name: wide-ep-llm-d-prefill   labels:     llm-d.ai/inference-serving: "true"     llm-d.ai/guide: "wide-ep-lws"     llm-d.ai/accelerator-variant: "gpu"     llm-d.ai/accelerato…

### #47934 — [[Bug]: DSpark launch failed with FP4 target model on GLM 5.2](https://github.com/vllm-project/vllm/issues/47934)
- **作者**: chungen04  **时间**: 2026-07-08 08:12 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information... ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (x86_64) GCC …

### #47930 — [[Bug]: DFlash/DSpark draft acceptance collapses with automatic prefix caching enabled](https://github.com/vllm-project/vllm/issues/47930)
- **作者**: giorgiopiatti  **时间**: 2026-07-08 07:17 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information... ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (x86_64) GCC …

### #47929 — [[Feature]: Deduplicate replicated MLA KV across TP ranks in native offloading](https://github.com/vllm-project/vllm/issues/47929)
- **作者**: Change72  **时间**: 2026-07-08 07:05 CST
- **摘要**: ### 🚀 The feature, motivation and pitch  #### Problem  For MLA models, the latent KV cache is **logically replicated across TP ranks** — and expected to be byte-identical in homogeneous TP setups without context parallelism. MLA hardcodes `num_kv_heads=1` (`vllm/model_executor/layers/attention/mla_a…

## 🔀 Pull Requests

### #47936 — [[V1]Only copy dirty block-table rows to GPU in commit_block_table](https://github.com/vllm-project/vllm/pull/47936)
- **作者**: jiacao-amd  **时间**: 2026-07-08 08:42 CST
- **标签**: v1
- **摘要**: ## Purpose  At steady-state decode, `BlockTable.commit_block_table` copies the entire `[num_reqs, max_blocks]` block table host->device on every step, even though most rows are unchanged between steps. This is wasted HtoD traffic that scales with batch size and max blocks per request.  This PR track…

### #47933 — [[Bugfix][Frontend] Preserve abort finish_reason for scale-out token streams](https://github.com/vllm-project/vllm/pull/47933)
- **作者**: Sunt-ing  **时间**: 2026-07-08 08:00 CST
- **标签**: bug, frontend
- **摘要**: ## Purpose  `/inference/v1/generate` streams token deltas from the engine for scale-out token-in/token-out serving. The V1 abort path produces a terminal delta with `token_ids=[]` and `finish_reason="abort"`.  Today `ServingTokens` skips every empty `token_ids` delta, so an aborted stream only sends…

### #47932 — [[CI/Build][BugFix][The Rock][AMD] Add spawn method in vision examples to avoid reinitialization](https://github.com/vllm-project/vllm/pull/47932)
- **作者**: rasmith  **时间**: 2026-07-08 07:44 CST
- **标签**: bug, documentation, rocm
- **摘要**: ## Purpose This PR adds `os.environ["VLLM_WORKER_MULTIPROC_METHOD"] = "spawn"`   to the LLM launches in `examples/generate/multimodal/vision_language_multi_image_offline.py` and `examples/generate/multimodal/vision_language_offline.py` to avoid reinitialization of CUDA/HIP.  Without this, the exampl…

### #47931 — [[Bugfix][Frontend] Return tool_calls finish reason from derendered tool calls](https://github.com/vllm-project/vllm/pull/47931)
- **作者**: Sunt-ing  **时间**: 2026-07-08 07:26 CST
- **标签**: bug
- **摘要**: ## Purpose  `/v1/chat/completions/derender` can parse generated token ids into `message.tool_calls` when the caller passes the original `chat_request`. Today that parsed response still forwards the generate-side `finish_reason="stop"`, so a successful response can contain `message.tool_calls` while …

### #47928 — [Account scheduled spec slots on empty output rows](https://github.com/vllm-project/vllm/pull/47928)
- **作者**: alexeldeib  **时间**: 2026-07-08 07:02 CST
- **标签**: v1
- **摘要**: ## Purpose  The async scheduler reserves output slots before a speculative decode step finishes. For MTP with five draft tokens, a decode step can reserve six output slots: one sampled token slot plus five draft slots. Normal rejection sampling later drains the unmaterialized draft slots after the w…

### #47927 — [Preserve DeepSeek sigmoid grouped routing metadata](https://github.com/vllm-project/vllm/pull/47927)
- **作者**: alexeldeib  **时间**: 2026-07-08 07:00 CST
- **标签**: deepseek
- **摘要**: ## Purpose  GLM-style DeepSeekV4/DSA models can use sigmoid routing with an expert-score correction bias, norm_topk_prob=True, n_group/topk_group metadata, and a non-unit routed scaling factor. With no group metadata passed to FusedMoE, the router is classified as RoutingMethodType.Unspecified: the …

### #47926 — [[Bugfix][Spec Decode] Mask prefix-cache-restored tokens out of the DFlash/DSpark draft context](https://github.com/vllm-project/vllm/pull/47926)
- **作者**: giorgiopiatti  **时间**: 2026-07-08 06:38 CST
- **标签**: bug, speculative-decoding, needs-rebase, v1
- **摘要**: ## Purpose  DFlash/DSpark drafters build their context KV from the target's auxiliary hidden states, which only exist for tokens that flow through a target forward pass (`precompute_and_store_context_kv`, written per step for the scheduled tokens in `DFlashSpeculator.propose`). Tokens whose KV is re…

### #47925 — [[Core] Pre-size cudagraph output staging buffers to the max capture descriptor](https://github.com/vllm-project/vllm/pull/47925)
- **作者**: matteso1  **时间**: 2026-07-08 06:14 CST
- **标签**: speculative-decoding, v1, nvidia
- **摘要**: ## Purpose  Context first: capture order came up in the #feat-startup-ux discussions around parallelizing / reordering CUDA-graph capture — @galv raised that capture order affects pool fragmentation, and the in-tree comment in `CudaGraphManager.capture()` already orders PIECEWISE before FULL for poo…

### #47924 — [[RFC+prototype] Compile the decode step e2e incl. KV-cache management (B.1: slot-mapping in the decode cudagraph)](https://github.com/vllm-project/vllm/pull/47924)
- **作者**: bobrenjc93  **时间**: 2026-07-08 06:02 CST
- **标签**: performance, new-model, needs-rebase, v1, gpt-oss, nvidia
- **摘要**: > **Draft / RFC + prototype. Stacked on #46423** (`bobren/vanilla-torch-compile`). > The new work in this PR is `benchmarks/compile/*` (RFC + measurement tool) and the > `VLLM_STOCK_CAPTURE_KV_PREP` prototype (`envs.py`, `stock_cudagraph.py`, > `gpu_model_runner.py`); the rest of the diff is #46423,…

### #47923 — [[kv_offload] Emit tier-owned BlockStored events from FS/OBJ secondary tiers](https://github.com/vllm-project/vllm/pull/47923)
- **作者**: Change72  **时间**: 2026-07-08 05:40 CST
- **标签**: documentation, v1
- **摘要**: ## Purpose  Follow-up to #46544, implementing the direction confirmed there by @orozery: each tier is fully responsible for its own KV events, and "actual secondary tier event implementations should be on a follow-up".  - The `fs` and `obj` secondary tiers each own an event buffer and override `Seco…
