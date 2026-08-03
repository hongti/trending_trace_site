# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-03 09:06 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 📌 Issue（问题）
共 2 个 Issue，均与 v0.26.0 版本的内存/缓冲区分配缺陷有关：
*   **FlashInfer MLA 解码缓冲区溢出**（#50781）：在 MLA（多头潜在注意力）结合解码上下文并行（DCP）场景下，FlashInfer 工作区缓冲区过小导致溢出（环境：v0.26.0 + B200 + GLM-5.2）。
*   **CUDA Graph 内存分析器计费错误**（#50780）：自 v0.26.0 起，分析器将 `mnbt-linear` 的性能分析临时内存错误计为“graph memory”，导致混合 GDN 模型的 KV 缓存池被大幅压缩 18%-24%。

---

### 🔧 Pull Request（代码合并）
共 10 个 PR，涵盖核心架构演进、Bug 修复、性能优化及文档改进：

**🚀 核心架构与新特性**
*   **可扩展（可增长）KV 缓存**（#50779）：提出可动态增长的 KV 缓存架构，目前为 Draft 状态，依赖 KV 布局标准化 PR。
*   **XPU FP8 量化路由优化**（#50787）：自动将块量化的 FP8 权重路由至 W8A8 内核，无需再手动开启 opt-in 标志。
*   **Voxtral 实时转写时间戳**（#50783）：为 `/v1/realtime` 接口增加可选的分段/词级时间戳，解决此前做说话人分离或字幕时只能靠猜测对齐时间的问题。

**🐛 重要 Bug 修复**
*   **Kimi 多轮工具调用 ID 丢失**（#50782）：修复使用 Responses API 配合 Kimi 模型时，多轮 Agentic 会话中原生 tool-call ID 间歇性丢失的问题。
*   **Elastic EP 永久 503 错误**（#50778）：修复扩展失败时未清除全局 `_scaling_elastic_ep` 标志位，导致 HTTP 请求陷入永久 503 状态的问题。
*   **Gemma3 模型参数默认值**（#50777）：修复 `Gemma3Model.forward` 的 `intermediate_tensors` 参数缺少默认值的问题，使其与其他模型保持一致。

**⚡ 性能优化**
*   **Triton Prefill 注意力优化**（#50776）：在窗口化 Triton prefill 中，直接跳过完全掩码的 key blocks，避免加载无效数据，减少无用计算。

**📖 文档与维护**
*   **多模型网关提示**（#50786）：补充 OpenAI 客户端通过 `base_url` 对接多模型网关的用法说明。
*   **WSL2 故障排除**（#50784）：在故障排除指南中新增 WSL2 专区，涵盖锁页内存限制和残留 worker 进程问题。
*   **MkDocs TOC 对齐**（#50785）：修复文档目录锚点生成与 markdownlint 规则不一致的问题。

---

### 🚀 Release（版本发布）
*   近期无新的 Release 版本发布记录。

---

## 🐛 Issues

### #50781 — [FlashInfer MLA decode workspace buffer overflow with decode-context-parallel](https://github.com/vllm-project/vllm/issues/50781)
- **作者**: flexwang  **时间**: 2026-08-03 07:11 CST
- **摘要**: ## Bug: FlashInfer workspace buffer too small for MLA + DCP  ### Environment - vLLM version: 0.26.0 - FlashInfer version: 0.6.14.dev20260705 - GPU: B200 x8 - Model: GLM-5.2   ### Configuration ``` --tensor-parallel-size 8 --decode-context-parallel-size 8 --max-num-seqs 24 --max-num-batched-tokens 16…

### #50780 — [[Bug]: CUDA graph memory profiler charges an mnbt-linear profiling transient as "graph memory" → KV cache pool −18–24% on hybrid GDN models since v0.26.0 (the real FULL graphs cost ~24 MiB each)](https://github.com/vllm-project/vllm/issues/50780)
- **作者**: jhsmith409  **时间**: 2026-08-03 06:55 CST
- **摘要**: ## Summary  Since v0.26.0, `profile_cudagraph_memory()` reports ~4.2 GiB of "CUDA graph memory" on a hybrid-GDN model (Qwen3.5-122B-A10B, int4 AutoRound, SM120) where 0.23.x reported 0.56 GiB for the *same graph set* (PIECEWISE=7, FULL=5). The KV cache pool shrinks by the difference: **1,489,281 → 1…

## 🔀 Pull Requests

### #50787 — [[XPU] Route block-quantized FP8 weights to the W8A8 kernel](https://github.com/vllm-project/vllm/pull/50787)
- **作者**: chaojun-zhang  **时间**: 2026-08-03 08:49 CST
- **标签**: intel-gpu, quantization
- **摘要**: ## Purpose   This PR fix after #43645 and routes block-quantized weights to the W8A8 kernel unconditionally (no opt-in flag `--linear-backend xpu` required), while keeping the existing opt-in behavior for per-tensor/per-channel weights.

### #50786 — [docs(serving): note OpenAI client base_url for multi-model gateways](https://github.com/vllm-project/vllm/pull/50786)
- **作者**: seven7763  **时间**: 2026-08-03 08:42 CST
- **标签**: documentation
- **摘要**: ## Summary  The OpenAI-compatible server docs already show the OpenAI Python client via `base_url`.  This PR adds a one-line tip that the same client pattern works with OpenAI-compatible multi-model gateways when not running vLLM locally, using [DaoXE](https://daoxe.com) (`https://api.daoxe.com/v1`)…

### #50785 — [[Doc] Align MkDocs TOC slugify with markdownlint](https://github.com/vllm-project/vllm/pull/50785)
- **作者**: frank-suwen  **时间**: 2026-08-03 08:13 CST
- **标签**: frontend
- **摘要**: ## Purpose  Fixes #49662.  This PR aligns MkDocs' in-page TOC slug generation with the PyMdown slugifier already used elsewhere in the docs configuration. This makes rendered heading anchors match the anchor style validated by markdownlint MD051.  It also updates the generated CLI docs source link f…

### #50784 — [[Doc] Troubleshooting: add a WSL2 section (pinned memory gates, lingering workers)](https://github.com/vllm-project/vllm/pull/50784)
- **作者**: manunicholasjacob  **时间**: 2026-08-03 08:02 CST
- **标签**: documentation
- **摘要**: ## Purpose  Adds a short WSL2 section to the troubleshooting guide covering two vLLM-specific behaviors that are currently undocumented, plus brief pointers to the general WSL caveats that come up in vLLM reports (#31124, #49861, #41933, #41619, #39093).  **Pinned host memory.** `VLLM_WSL2_ENABLE_PI…

### #50783 — [[Frontend][Voxtral] Add opt-in segment timestamps to /v1/realtime](https://github.com/vllm-project/vllm/pull/50783)
- **作者**: AndriiPasternak31  **时间**: 2026-08-03 07:32 CST
- **标签**: documentation, frontend, mistral
- **摘要**: ## Purpose  Relates to #39735. `/v1/realtime` streams transcription text with no timing, so clients doing diarization or subtitling have to guess word timing from message arrival.  Voxtral realtime emits one token per 80 ms frame, so a token's index is an index into the audio and `[STREAMING_WORD]` …

### #50782 — [[Bugfix][Frontend] Preserve Kimi native tool-call IDs across Responses API multi-turn round trip (Fixes #50768)](https://github.com/vllm-project/vllm/pull/50782)
- **作者**: thegoldenflow  **时间**: 2026-08-03 07:15 CST
- **标签**: bug, frontend, tool-calling, kimi
- **摘要**: ## Purpose  Fixes #50768. On `/v1/responses` with a Kimi model and `--tool-call-parser kimi_k2`, multi-turn agentic sessions intermittently lose the model's tool call and the turn comes back empty — the reporter's "performs work but never answers", at a rate that climbs sharply with the number of tu…

### #50779 — [[Core] Extensible (growable) KV cache](https://github.com/vllm-project/vllm/pull/50779)
- **作者**: njhill  **时间**: 2026-08-03 06:36 CST
- **标签**: frontend, kv-connector, mrv2
- **摘要**: > **Stacked on #44458** (`lwilkinson/kv-layout/core-standardize`). Only the commits above that PR's head belong to this one; review #44458 first. > This is a **draft** — it cannot merge until #44458 lands, after which the base will be retargeted to `main`.  ## Summary  Opt-in growable KV cache (`--e…

### #50778 — [[Bugfix][Elastic EP] Clear scaling flag on failure to prevent permane…](https://github.com/vllm-project/vllm/pull/50778)
- **作者**: gagandhakrey  **时间**: 2026-08-03 06:31 CST
- **标签**: bug
- **摘要**: ## Purpose  ### Issue  `ScalingMiddleware` returns `503` for all HTTP requests while the process-wide `_scaling_elastic_ep` flag is set.  `AsyncLLM._scale_elastic_ep()` only cleared this flag on the success path. If `engine_core.commit_elastic_ep()` raised an exception or the task was cancelled, the…

### #50777 — [[Bugfix] Default Gemma3 Model intermediate_tensors to None](https://github.com/vllm-project/vllm/pull/50777)
- **作者**: taneem-ibrahim  **时间**: 2026-08-03 06:25 CST
- **标签**: bug
- **摘要**: ## Purpose  Currently `Gemma3Model.forward` declares `intermediate_tensors` as a required positional parameter:  ```python intermediate_tensors: IntermediateTensors | None,   # no default ```  Every other pooling-capable model defaults it, e.g. `BertModel`:  ```python intermediate_tensors: Intermedi…

### #50776 — [[Kernel] Skip fully masked key blocks in windowed Triton prefill](https://github.com/vllm-project/vllm/pull/50776)
- **作者**: almogtavor  **时间**: 2026-08-03 05:45 CST
- **摘要**: ## Purpose  `_fwd_kernel` in `triton_prefill_attention.py` walks every key block from 0 to `end_n` and masks out-of-window keys after loading them. The masking is correct, but the work is not skipped, so a sliding-window pass still costs O(seq_len^2).  For a query block covering `[start_m * BLOCK_M,…
