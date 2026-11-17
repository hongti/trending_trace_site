# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-17 16:21 CST

## AI 总结

### vLLM 仓库最近动态摘要

#### 🚀 Release
- **v0.34.0 发布**：本次版本包含来自 412 位贡献者（其中 120 位为新贡献者）的 1,061 次提交。
  - **核心亮点**：新增支持 10 个模型，包括 **Mistral Large 4**、**MiniMax-M3.1-Flash**、**Nemotron-3** 等。

#### 🐛 Issue
1. **LoRA 服务崩溃问题 (#68107)**：当 LoRA 适配器的生成请求与基础模型的 prompt logprobs 请求混合处理时，会导致服务器崩溃。疑似在 prompt-logprob 分块处理时，`LogitsProcessorWithLoRA` 的 punica shrink/expand 路径发生了越界错误。
2. **AMD ROCm 性能优化提案 (#68104)**：针对 MI355X (gfx950) 架构，提出了在 vLLM 上优化 `amd/MiniMax-M3.1-Flash-Quark-MXFP4` 模型推理性能的方案。

#### 🔧 Pull Request
**Bug修复**
- **Mamba 缓存超限报错优化 (#68113)**：针对 Mamba 混合模型，当 `max_num_seqs` 超过可用缓存块时（常见于 24GB 显卡），改为快速失败报错，避免引擎异常。
- **结构化输出推理边界修复 (#68112)**：修复投机解码草稿可能破坏 JSON 结构约束的问题，确保保留最早的推理结束边界。
- **XPU KV 缓存配置修复 (#68108)**：在 XPU 上构建 KV cache 配置时正确设置 vllm config，使滑动窗口等场景下能正常支持用户指定的 `block_size=16`。
- **Mistral Large 4 启动崩溃修复 (#68103)**：将 Mistral Large 4 的 NVIDIA 索引器缓存移植至 `get_kv_cache_config`，修复该模型在 NVIDIA GPU 上初始化即崩溃的问题。

**新特性与性能优化**
- **WekaFS 检查点预取 (#68105)**：模型加载器新增对 WekaFS 文件系统的自动 safetensors 检查点预取支持。
- **Bailing V3 性能提升 (#68106)**：为 Bailing V3 模型的 KDA 层导出 Triton prefill 检查点，提升性能。

**文档与配置更新**
- **NCCL 显存开销说明 (#68111)**：补充了关于分片采样 NCCL 缓冲区显存开销的文档，并指导如何评估 `NCCL_MAX_P2P_NCHANNELS`。
- **v0.34.0 兼容性更新 (#68110, #68109)**：刷新夜间构建版要求；将 DeepSeek-V4.1-Flash 和 Kimi-K3 的 AgentX 预设改为手动开启以防混淆。
- **文档新增 "Ask AI" 助手 (#68102)**：在文档站重新加入 AI 问答小部件，替代已停用的 RunLLM 插件。

---

## 🐛 Issues

### #68107 — [[Bug]: LoRA-enabled server dies when adapter generations mix with base-model prompt logprobs: suspected out-of-bounds in the punica shrink/expand path of LogitsProcessorWithLoRA under prompt-logprob chunking](https://github.com/vllm-project/vllm/issues/68107)
- **作者**: bkgoksel  **时间**: 2026-11-17 15:48 CST
- **摘要**: ## Summary  Start a server with `--enable-lora`. When two kinds of request are in flight together, the engine fails:  - generations addressed to a LoRA adapter, and - `/v1/completions` requests addressed to the base model that ask for prompt logprobs (`echo: true`, `max_tokens: 0`, `logprobs: 128`).…

### #68104 — [[ROCm][AMD] MiniMax-M3.1-Flash gfx950 / MI355X Performance Optimization (vLLM)](https://github.com/vllm-project/vllm/issues/68104)
- **作者**: vorapolsiloai  **时间**: 2026-11-17 15:25 CST
- **标签**: rocm, minimax
- **摘要**: ### Proposal to improve performance  Tracker for the MI355X (gfx950) optimizations of [`amd/MiniMax-M3.1-Flash-Quark-MXFP4`](https://huggingface.co/amd/MiniMax-M3.1-Flash-Quark-MXFP4) on vLLM: text and image input, TP2, NVFP4 KV and indexer caches, with and without DSpark speculative decoding. It li…

## 🔀 Pull Requests

### #68113 — [[Bugfix] Fail fast when max_num_seqs exceeds Mamba cache blocks](https://github.com/vllm-project/vllm/pull/68113)
- **作者**: 123123213weqw  **时间**: 2026-11-17 16:19 CST
- **标签**: bug, kv-cache-manager
- **摘要**: ## Description  On Mamba-hybrid models, when `max_num_seqs` exceeds the available Mamba cache blocks (common with long `max_model_len` + speculative decoding on 24GB-class cards), the engine currently fails only at full-decode CUDA graph capture in `CompilationConfig` — i.e. **after weights are load…

### #68112 — [[Bugfix][Structured Output] Preserve the first reasoning boundary in speculative drafts](https://github.com/vllm-project/vllm/pull/68112)
- **作者**: linnea-lin-00638949  **时间**: 2026-11-17 16:18 CST
- **标签**: bug, documentation, structured-output
- **摘要**: ## Overview  Preserve the earliest reasoning-end prefix when a speculative draft later reopens thinking, so a rejected suffix cannot remove the JSON constraints needed by an accepted prefix. This is a **draft candidate for review** of the existing one-boundary structured-output model; it does not im…

### #68111 — [[Docs] Document batch-sharded sampling NCCL memory overhead](https://github.com/vllm-project/vllm/pull/68111)
- **作者**: mengyuan28  **时间**: 2026-11-17 16:07 CST
- **标签**: documentation
- **摘要**: ## Overview  Document the NCCL buffer memory cost of batch-sharded sampling and how to evaluate `NCCL_MAX_P2P_NCHANNELS`. Addresses the documentation request in #68067.  ## Claims  - Explain the impact on KV cache capacity, show how to set and unset the channel limit, and describe the memory/latency…

### #68110 — [[Recipe] Refresh nightly requirements after v0.34.0](https://github.com/vllm-project/vllm/pull/68110)
- **作者**: esmeetu  **时间**: 2026-11-17 16:04 CST
- **标签**: kimi, recipe
- **摘要**: ## Purpose  v0.34.0 is out, and many recipes still ask for a nightly build, some with `min_vllm_version: "nightly"`, which the renderer reads as version 0. Each one was checked against the release tags (support PR ancestry, or the first tag whose model registry has the checkpoint's architecture).  |…

### #68109 — [[Recipe] Make the DeepSeek-V4.1-Flash and Kimi-K3 AgentX presets opt-in](https://github.com/vllm-project/vllm/pull/68109)
- **作者**: esmeetu  **时间**: 2026-11-17 16:04 CST
- **标签**: deepseek, kimi, recipe
- **摘要**: ## Purpose  Eight tuned presets on DeepSeek-V4.1-Flash and Kimi-K3 are InferenceX AgentX benchmark lanes marked `default: true`, so they are what a reader lands on. For DeepSeek-V4.1-Flash (`default_hardware: gb200`) the page and the JSON API's `recommended_command` opened on the concurrency-4 point…

### #68108 — [[Bugfix][XPU] Set current vllm config when building KV cache layer configs](https://github.com/vllm-project/vllm/pull/68108)
- **作者**: chaojun-zhang  **时间**: 2026-11-17 16:00 CST
- **标签**: bug, intel-gpu, mrv2
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Run the worker-side `get_kv_cache_layer_configs()` under `set_current_vllm_config()`. Follow-up to #62656.  ## Claims  - On XPU, a user-specified `block_size=16` is now honored for sliding-window layers (e.g. gemma-3). Previously it was silently bumped to …

### #68106 — [[Perf][KDA] Export Triton prefill checkpoints for Bailing V3](https://github.com/vllm-project/vllm/pull/68106)
- **作者**: JaredforReal  **时间**: 2026-11-17 15:34 CST
- **标签**: kimi
- **摘要**: ## Overview  Let the KDA layers of Bailing V3 (`BailingMoeV3ForCausalLM`, e.g. Ling-3.0-flash) export align-mode prefill checkpoints, using the per-sequence buffer path #67866 added for GLM-5.3-Flash and Kimi-K3: the Triton chunk recurrence writes each checkpoint's state into a `[N, H, V, K]` buffer…

### #68105 — [[Feature][Model Loader] Enable automatic checkpoint prefetch on WekaFS](https://github.com/vllm-project/vllm/pull/68105)
- **作者**: Super262  **时间**: 2026-11-17 15:25 CST
- **标签**: documentation, ci/build
- **摘要**: ## Overview  Enable automatic safetensors checkpoint prefetching on WekaFS (`wekafs`). Fixes #67170.  ## Claims  - Reuse the existing 90% available-RAM threshold and respect explicit loading strategies. - Update filesystem diagnostics, CLI help, startup documentation, and CI regression coverage.  ##…

### #68103 — [[Bugfix][Model] Port Mistral Large 4 NVIDIA indexer cache to get_kv_cache_config](https://github.com/vllm-project/vllm/pull/68103)
- **作者**: kainoj  **时间**: 2026-11-17 15:17 CST
- **标签**: bug, ready, nvidia, mistral
- **摘要**: ## Overview  Mistral Large 4 fails to start on NVIDIA GPUs on `main`: every worker crashes at model init because the NVIDIA `MistralIndexerCache` is still abstract. #67984 fixed the AMD copy of this class. This PR makes the same change to the NVIDIA copy.  ## Claims  - Fixes the NVIDIA startup crash…

### #68102 — [[Docs] Add Ask AI widget](https://github.com/vllm-project/vllm/pull/68102)
- **作者**: esmeetu  **时间**: 2026-11-17 15:17 CST
- **标签**: documentation
- **摘要**: ## Overview  Add an "Ask AI" assistant to the docs site, replacing the RunLLM widget that was removed in #67092 when that service reached end-of-life. The docs currently have no assistant.  ## Claims  - Add `docs/mkdocs/javascript/ask_ai_widget.js`, a 15-line loader that injects the widget script on…

## 🚀 Releases

### [v0.34.0](https://github.com/vllm-project/vllm/releases/tag/v0.34.0)
- **作者**: khluu  **时间**: 2026-11-17 07:22 CST
- **摘要**: # v0.34.0  **1,061 commits from 412 contributors (120 new).** Thank you!  ## TL;DR  - **10 new models** (Mistral Large 4, MiniMax-M3.1-Flash, Nemotron-3.5-Transcribe, and more!) - **Extensible KV cache is on by default:** vLLM sizes the KV cache from memory actually free after warmup, so most models…
