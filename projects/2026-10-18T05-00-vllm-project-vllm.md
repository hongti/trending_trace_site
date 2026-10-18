# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-18 13:00 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要。本次更新主要围绕新模型支持、KV Connector 兼容性修复、性能优化及多硬件后端适配等方面展开。

### 1. Issue 动态
*   **[Feature] 支持 GUI-Actor 指针头模型的 `box` 和 `ground` 决策原语 (#62247)**：提出通过 `/v1/systemone` 结构化决策 API，支持 GUI-Actor 模型的 box 和 ground 原语，以增强模型的状态交互能力。
*   **[RFC]: Mooncake P2P KV 连接器的模块布局 (#62243)**：提议重构 Mooncake P2P 连接器的代码结构。当前其全部实现集中在一个超过 3000 行的单文件中，亟待模块化拆分以提高可维护性。

### 2. PR 动态
本次合并的 PR 涵盖了新特性、性能优化与多项关键 Bug 修复：

*   **新特性与模型支持**
    *   **GUI-Actor 模型架构支持 (#62248)**：实现了上述 Issue 的需求，新增 `box` 和 `ground` 决策原语及对应的 `GuiActorForConditionalGeneration` 架构。
    *   **XPU 自适应验证 (#62244)**：为 XPU 路径启用了无需 confidence head 的投机解码自适应验证功能。
*   **性能优化**
    *   **SPLADE 批处理池化 (#62250)**：将 SPLADE 的 MLM 头和激活投影进行批处理（控制在 64 MiB logits 预算内），并在 max-pooling 后应用 `log1p(relu)`，有效减少了小 logits 的计算开销。
    *   **DeepSelect BF16 排序 (#62240)**：在 DSA 的 prefill 路径中，直接使用 BF16 格式对 MQA logits 进行排序，避免了原先向 FP32 转换的额外开销。
*   **重要 Bug 修复**
    *   **混合 Mamba 模型 PD 分离部署修复 (#62251)**：修复了 Mooncake 连接器此前要求 Prefill 和 Decode 引擎必须使用相同张量并行（TP）大小的问题，现已支持不等长 TP。
    *   **结构化输出后端冲突修复 (#62246)**：修复了混合请求场景下，所有请求都被迫使用首个 `auto` 模式选中的后端进行编译的问题，现改为正确遵守每个请求各自验证过的后端。
    *   **输入长度限制修复 (#62249)**：使 Clef SystemOne 的读取逻辑遵守引擎的 `max_model_len` 限制，超长提示词将被拒绝，而不再是被静默截断。
    *   **DecodeBenchConnector 组报告修复 (#62242)**：修复了 `DecodeBenchConnector` 仅将前缀组报告为已加载，导致自身块无法在所有 KV 缓存组中正确填充的问题。
*   **硬件与平台兼容性**
    *   **RedHat FP8 Xeon 支持修复 (#62245)**：通过现有的 `supported_hardware` 字段，将 RedHatAI 的 FP8/NVFP4 配方变体限制在已验证的硬件上运行。
    *   **ROCm 通信器释放 (#62241)**：修复了使用外部启动器（如 torchrun）关闭时，vLLM 的模型并行组未被正确销毁的问题，行为现已与 `MultiprocExecutor` 对齐。

### 3. Release 动态
*   本次提供的动态数据中**无新版本发布**信息。

---

## 🐛 Issues

### #62247 — [[Feature] Support box and ground decision primitives with GUI-Actor pointer-head models](https://github.com/vllm-project/vllm/issues/62247)
- **作者**: ningyx25  **时间**: 2026-10-18 12:11 CST
- **摘要**: # [Feature] Support `box` and `ground` decision primitives with GUI-Actor pointer-head models  ## Motivation  The structured decisions API (`/v1/systemone`) answers questions about a state by scoring a fixed set of options: `choice`, `noul`, `score`. That covers classification and routing, but not w…

### #62243 — [[RFC]: Module layout for the Mooncake P2P KV connector](https://github.com/vllm-project/vllm/issues/62243)
- **作者**: CAICAIIs  **时间**: 2026-10-18 11:40 CST
- **摘要**: ### Motivation.  `vllm/distributed/kv_transfer/kv_connector/v1/mooncake/mooncake_connector.py` holds the entire Mooncake P2P connector in one module. Measured on `main` (3077 lines):  | Concern | Location | Lines | | --- | --- | --- | | Module header, constants, `ReqId`/`TransferId` | 1–107 | 107 | …

## 🔀 Pull Requests

### #62251 — [[Bugfix][KV Connector] Support unequal P/D TP for hybrid Mamba models](https://github.com/vllm-project/vllm/pull/62251)
- **作者**: zyulai  **时间**: 2026-10-18 12:53 CST
- **标签**: bug, kv-connector, mooncake
- **摘要**: PD disaggregation with the Mooncake connector previously assumed that the prefill and decode engines use the same tensor-parallel size for hybrid Mamba models (Mamba1, GDN). With unequal TP, a Mamba cache page is packed: conv state followed by SSM state, and for GDN the conv state itself packs Q|K|V…

### #62250 — [[Perf][Pooling] Batch SPLADE max-pooling projections](https://github.com/vllm-project/vllm/pull/62250)
- **作者**: taneem-ibrahim  **时间**: 2026-10-18 12:51 CST
- **标签**: pooling
- **摘要**: ## Purpose  SPLADE runs its MLM head and activation separately for every sequence. This groups the projection within a 64 MiB logits budget and applies `log1p(relu)` after max pooling, reducing small launches and intermediates. Sum pooling stays unchanged. One file, +32/-8; no added tests or files. …

### #62249 — [[Bugfix] Respect max_model_len for Clef SystemOne reads](https://github.com/vllm-project/vllm/pull/62249)
- **作者**: webcodes-cz  **时间**: 2026-10-18 12:33 CST
- **标签**: bug, documentation, frontend
- **摘要**: ## Overview  Make Clef SystemOne pooling reads respect the engine's resolved `max_model_len` and reject oversized complete prompts instead of silently truncating the state.  ## Claims  - `ClefFlashStrategy` has no additional 16,384-token cap. The complete prompt,   including expanded media, question…

### #62248 — [[Model] Support box and ground decisions with GUI-Actor](https://github.com/vllm-project/vllm/pull/62248)
- **作者**: ningyx25  **时间**: 2026-10-18 12:16 CST
- **标签**: documentation, new-model, frontend, multi-modality, pooling
- **摘要**: ## Overview  Adds `box` and `ground` as structured-decision primitives, answered by GUI-Actor's pointer head, and the `GuiActorForConditionalGeneration` architecture that serves it. `/v1/systemone` gains two question types that locate a target in an image instead of picking an option.  The primitive…

### #62246 — [[Bugfix][Structured Output] Honor resolved backends in mixed requests](https://github.com/vllm-project/vllm/pull/62246)
- **作者**: webcodes-cz  **时间**: 2026-10-18 12:06 CST
- **标签**: bug, structured-output
- **摘要**: ## Overview  Use each structured-output request's validated backend instead of compiling every request with the first backend selected by `auto`. Related to #43920 and #55152.  ## Claims  - Cache compiler instances by the already-resolved backend name; do not change explicit backend configuration or…

### #62245 — [fix for redhat fp8 xeon support issue](https://github.com/vllm-project/vllm/pull/62245)
- **作者**: louie-tsai  **时间**: 2026-10-18 11:50 CST
- **摘要**: ## Overview  Scope RedHatAI FP8/NVFP4 recipe variants to validated hardware using the existing `supported_hardware` field.  Port the changes from [[vLLM Recipes PR #1142](https://github.com/vllm-project/recipes/pull/1142)](https://github.com/vllm-project/recipes/pull/1142) into `vllm/recipes/models/…

### #62244 — [[XPU] Enable spec-decode adaptive verification without a confidence head](https://github.com/vllm-project/vllm/pull/62244)
- **作者**: yma11  **时间**: 2026-10-18 11:48 CST
- **标签**: intel-gpu, speculative-decoding
- **摘要**: ## Overview  PR #52228 provides support adaptive verification without a confidence head and this PR enable it for XPU path. We identified perf gap caused by FA kernel and can be addressed by PR [647](https://github.com/vllm-project/vllm-xpu-kernels/pull/647). With these 2 PRs, we got below bench res…

### #62242 — [[Bugfix][KV Connector] Report every transfer group as loaded in Decod…](https://github.com/vllm-project/vllm/pull/62242)
- **作者**: Juntian777  **时间**: 2026-10-18 11:25 CST
- **标签**: bug, kv-connector
- **摘要**: ## Purpose  `DecodeBenchConnector` fills a new request's own blocks in every KV cache group, like a P/D transfer, but it inherits the base `get_loaded_kv_cache_group_ids`, which reports only the prefix-cacheable groups (what a hash-addressed store holds). A bounded-replay sliding-window group (DeepS…

### #62241 — [[ROCm][Bugfix] Release vLLM's communicators on external-launcher shutdown](https://github.com/vllm-project/vllm/pull/62241)
- **作者**: AndreasKaratzas  **时间**: 2026-10-18 10:30 CST
- **标签**: bug, rocm
- **摘要**: ## Overview  `ExecutorWithExternalLauncher` (torchrun, `distributed_executor_backend="external_launcher"`) now destroys vLLM's model-parallel groups on shutdown, like `MultiprocExecutor`'s workers do. Without it, `:amd: (MI355) Distributed DP + EP` intermittently aborts at exit on the ROCm 10 image …

### #62240 — [[Perf][DSA] Rank the prefill indexer logits with DeepSelect in BF16](https://github.com/vllm-project/vllm/pull/62240)
- **作者**: Juntian777  **时间**: 2026-10-18 09:59 CST
- **摘要**: <!-- markdownlint-disable -->  ## Overview  With `sparse_indexer_topk_backend="deep_select"`, the sparse-indexer prefill path still upcasts DeepGEMM's BF16 MQA logits to FP32 and ranks them with `top_k_per_row_prefill`. This PR ranks the BF16 logits with DeepSelect directly, as decode already does s…
