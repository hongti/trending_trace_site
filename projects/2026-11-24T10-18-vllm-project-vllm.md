# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-24 18:18 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🚀 Pull Requests (PR)
本次更新主要集中在性能优化、快速启动机制增强以及对 Kimi-K3 系列模型的全面适配与修复。

**1. 性能与启动优化**
*   **Ada GPU 并行采样加速** (#69628)：针对 Ada GPU 上的单 rank 全注意力解码器，优先使用 16-token 的 FlashAttention KV 块，修复了并行采样中额外的 prompt 计算开销问题。
*   **Mamba 缓存性能优化** (#69626)：降低了落后于当前对话最新缓存状态（`t-2` 及更早）的 Mamba/GDN/PLE 前缀缓存状态的优先级，仅保留直接前驱（`t-1`），从而提升处理效率。
*   **快速启动支持 Engram 卸载** (#69627)：为预加载守护进程 (`vllm preload`) 增加了 CPU 卸载支持，使得重启的引擎可以直接映射宿主机表用于 GPU 查找，加速 Qwen4Exp 模型的启动。

**2. Kimi-K3 / K3.1-Pro 模型适配与 Bug 修复**
*   **采样与工具调用** (#69624)：配置 Kimi-K3.1-Pro 使用模型卡默认的采样参数，并为 Kimi-K3 / K3.1-Pro 增加了可选的严格工具调用模式。
*   **前端思维链控制** (#69623)：前端现在支持 Chat Completions 接口中顶层的 `thinking` 对象，并对不支持的努力等级返回 400 错误。
*   **Schema 兼容性修复** (#69622)：修复了前端拒绝带有自身属性树 `$ref` 的严格工具 Schema 导致 HTTP 400 报错的问题。
*   **透明图像处理修复** (#69621)：修复了 vLLM 在预处理时直接丢弃 Alpha 通道的问题，现在会将 RGBA 等透明图像正确合成到检查点背景上。
*   **ROCm 适配** (#69625)：在 gfx950 上记录 SiTUv2 MoE 激活模式，并修正了相关的配置说明。

**3. 文档与代码清理**
*   **文档修复** (#69620)：移除了预加载文档中不存在的 `--weight-cache-vmm-max-allocation-bytes` 参数说明。
*   **代码去重** (#69619)：移除了 `Ling3Parser` 中冗余的 `extract_reasoning` 方法，该方法与其父类实现完全一致。

---
*注：本次提供的动态数据中未包含 Issue 和 Release 相关信息。*

---

## 🔀 Pull Requests

### #69628 — [[Perf] Reduce parallel sampling prefill on Ada GPUs](https://github.com/vllm-project/vllm/pull/69628)
- **作者**: Zhangz1ye  **时间**: 2026-11-24 17:53 CST
- **标签**: documentation, ci/build
- **摘要**: ## Overview  Prefer 16-token FlashAttention KV blocks for single-rank, full-attention decoders using local prefix caching on Ada. This addresses the extra prompt computation in #69408 following the default change in #63231. The related #69455 adds a benchmark result analyzer.  ## Claims  - On an RTX…

### #69627 — [[Fast Start] Add engram cpu offloading support for preload daemon](https://github.com/vllm-project/vllm/pull/69627)
- **作者**: Isotr0py  **时间**: 2026-11-24 17:48 CST
- **标签**: documentation, needs-rebase, qwen
- **摘要**: ## Overview  Support Qwen4Exp's private pinned Engram (PLE) tables with `vllm preload` and `--load-format ipc_cache`. A restarted engine maps the daemon's host table and uses it for GPU lookups without allocating another copy.  ## Claims  - With the BF16 Qwen3.8-Flash-Next checkpoint on four GB200s,…

### #69626 — [[Core][Perf] Deprioritize superseded Mamba prefix-cache states](https://github.com/vllm-project/vllm/pull/69626)
- **作者**: ZJY0516  **时间**: 2026-11-24 17:47 CST
- **标签**: kv-cache-manager
- **摘要**: ## Overview  Deprioritize Mamba/GDN/PLE prefix-cache states that are two or more turns behind a conversation's latest cached tip (`t-2` and older), while retaining the immediate predecessor (`t-1`) in normal LRU as a one-turn rollback fallback and placing deprioritized blocks in a middle eviction ti…

### #69625 — [[ROCm][Kimi-K3] Log the SiTUv2 MoE activation mode and correct the recipe note](https://github.com/vllm-project/vllm/pull/69625)
- **作者**: AndreasKaratzas  **时间**: 2026-11-24 17:47 CST
- **标签**: rocm, quantization, kimi, recipe
- **摘要**: ## Overview  On gfx950 the Kimi-K3 MXFP4 experts run on AITER's SiTUv2 kernels, and `VLLM_ROCM_USE_AITER_MOE_SITUV2` picks their activation precision. The startup log line does not say which mode is active, and the Kimi-K3 recipe's AMD note describes a default and an environment variable that no lon…

### #69624 — [[Recipe] Kimi-K3.1-Pro: serve the model card's sampling defaults; Kimi-K3 / K3.1-Pro: opt-in strict tool calls](https://github.com/vllm-project/vllm/pull/69624)
- **作者**: AndreasKaratzas  **时间**: 2026-11-24 17:47 CST
- **标签**: kimi, recipe
- **摘要**: ## Overview  Recipe-only change for Kimi-K3.1-Pro and Kimi-K3, from running the [Kimi Vendor Verifier](https://github.com/MoonshotAI/Kimi-Vendor-Verifier) suite on 8x MI355X (#66699, #65632). Kimi-K3.1-Pro now serves the model card's sampling values as server defaults, and both recipes get an opt-in…

### #69623 — [[Frontend][Kimi-K3] Honor the top-level `thinking` object in Chat Completions and return 400 for efforts the encoder rejects](https://github.com/vllm-project/vllm/pull/69623)
- **作者**: AndreasKaratzas  **时间**: 2026-11-24 17:47 CST
- **标签**: documentation, frontend, kimi
- **摘要**: ## Overview  Two fixes to thinking controls in the Python frontend, found while serving Kimi K3 and K3.1 Pro (context: #66699, #65632).  1. A top-level `thinking` object (`{type, effort, keep}`) on `/v1/chat/completions` was accepted and ignored. A client that sends only that object could not disabl…

### #69622 — [[Bugfix][Frontend] Kimi K3: accept strict tool schemas with a `$ref` into their own property tree](https://github.com/vllm-project/vllm/pull/69622)
- **作者**: AndreasKaratzas  **时间**: 2026-11-24 17:47 CST
- **标签**: bug, tool-calling, kimi
- **摘要**: ## Overview  A strict Kimi K3 tool whose parameters use a `$ref` into their own property tree (for example `#/properties/config/properties/limit`) is rejected with HTTP 400 on the Python frontend. This PR makes such schemas compile and be enforced.  One commit, `ba2e60a4ee`, on base `ab1c06419c`. No…

### #69621 — [[Bugfix][Kimi-K3] Composite transparent images over the checkpoint background](https://github.com/vllm-project/vllm/pull/69621)
- **作者**: AndreasKaratzas  **时间**: 2026-11-24 17:47 CST
- **标签**: bug, multi-modality, kimi
- **摘要**: ## Overview  Kimi-K3 / K3.1 images with transparency (RGBA, LA, palette with a transparent colour) are preprocessed differently from the checkpoint's own image processor. vLLM drops the alpha channel; the checkpoint blends the image over an 8 px chessboard. This is a regression from #59278, which mo…

### #69620 — [[Docs][Preload] Remove reference to a non-existent --weight-cache-vmm-max-allocation-bytes flag](https://github.com/vllm-project/vllm/pull/69620)
- **作者**: wjunLu  **时间**: 2026-11-24 17:42 CST
- **标签**: documentation
- **摘要**: ## Overview  Drop a sentence in `docs/features/preload.md` that tells users to set `--weight-cache-vmm-max-allocation-bytes`, a CLI flag that does not exist.  ## Claims  - `--weight-cache-vmm-max-allocation-bytes` appears nowhere in the codebase (only in that doc line). The VMM packing cap is a code…

### #69619 — [[Parser] Remove redundant extract_reasoning override in Ling3Parser](https://github.com/vllm-project/vllm/pull/69619)
- **作者**: wjunLu  **时间**: 2026-11-24 17:40 CST
- **标签**: tool-calling
- **摘要**: ## Overview  Remove `Ling3Parser.extract_reasoning`, which shadows an identical method on its parent class.  ## Claims  - `Ling3Parser.extract_reasoning` duplicates `Glm47MoeParser.extract_reasoning` byte for byte (modulo tuple parenthesization, which is semantically identical). - `Ling3Parser` subc…
