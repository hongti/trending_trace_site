# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-14 11:09 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 🐛 Issue 动态
1. **DeepSeek-V4 分词器 Bug** (#52253)：在 `deepseek_v4` 模式下，当 assistant 回合无推理内容时，编码器会错误地输出空的 `<think>` 块。该问题无需 GPU 或模型权重即可复现。
2. **EngineCore 死锁 Bug** (#52247)：当 GPU kernel 无法终止时，`AsyncModelRunnerOutput` 中的 `copy_event.synchronize()` 会无限期阻塞（缺乏超时机制），导致 EngineCore 永久卡死。

### 🔧 PR 动态
**✨ 新特性与优化**
- **Prometheus 指标增强** (#52249)：为延迟直方图添加 `operation` 标签，以符合 OpenTelemetry GenAI 语义规范。
- **AMD/ROCm DCP 支持** (#52248)：启用 ROCm AITER MLA decode 后端的 Decode Context Parallelism (DCP) 支持，包括 A2A 通信路径。
- **DSpark 自适应验证** (#52242)：解除了 logprobs 与 `enable_adaptive_verification` 的互斥限制，支持在设备端按请求分配 draft 数量。
- **调试统计** (#52251)：WIP 状态，添加引擎调试统计信息。

**🛠️ 关键 Bug 修复**
- **Anthropic 接口错误码修复** (#52246)：修复了客户端输入错误（如 stop sequences 超出限制）时返回 5xx 的问题，现在正确返回 4xx 状态码。
- **V1 前缀缓存修复** (#52244)：修复了在 MTP 投机解码下，混合 GDN 前缀缓存命中失效的问题（影响如 Qwen3.5-122B-A10B 等模型）。
- **Kernel 舍入语义修复** (#52243)：修正了 `vllm_c fused_add_rms_norm` 的舍入顺序，使其与原生 IR 的舍入语义保持一致。
- **PD 连接器存活追踪** (#52245)：在 D 侧记录远程节点的最新活动时间，修复了 liveness 追踪问题。

**🧹 其他改进**
- **CI 超时调整** (#52252)：将扩展生成测试的超时时间由 65 分钟延长至 80 分钟，避免误杀。
- **文档修正** (#52250)：修正 README 中的拼写错误，将 `Mixture-of-Expert` 更正为标准的 `Mixture-of-Experts`。

### 🚀 Release 动态
近期无新版 Release 发布。

---

## 🐛 Issues

### #52253 — [[Bug]: DeepSeek-V4 encoder emits an empty `<think></think>` block for assistant turns without reasoning](https://github.com/vllm-project/vllm/issues/52253)
- **作者**: flexwang  **时间**: 2026-08-14 11:05 CST
- **摘要**: ### Your current environment  Affects the `deepseek_v4` tokenizer mode on current `main`. Reproduces with the encoder alone — no GPU or model weights needed.  ### 🐛 Describe the bug  `vllm/tokenizers/deepseek_v4_encoding.py` renders an assistant turn that carries no `reasoning` as a literal empty th…

### #52247 — [[Bug]: EngineCore blocks forever (no timeout) in AsyncModelRunnerOutput copy_event. synchronize() when a GPU kernel never terminates](https://github.com/vllm-project/vllm/issues/52247)
- **作者**: leka-k  **时间**: 2026-08-14 10:21 CST
- **标签**: bug
- **摘要**: ### environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  Collecting environment information... ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (x86_64) GCC version              …

## 🔀 Pull Requests

### #52252 — [[CI] Increase extended generation test timeout](https://github.com/vllm-project/vllm/pull/52252)
- **作者**: LucasWilkinson  **时间**: 2026-08-14 10:58 CST
- **标签**: ci/build
- **摘要**: ## Summary  Raise the **Language Models Test (Extended Generation)** timeout from 65 to 80 minutes. This keeps the existing test coverage and single-H200 resource shape unchanged.  ## Why  PR #48186 reduced this timeout from 110 to 65 minutes based on one successful 48.9-minute nightly and a 15-minu…

### #52251 — [WIP: zhongxin: add debug stat](https://github.com/vllm-project/vllm/pull/52251)
- **作者**: wenjinhust  **时间**: 2026-08-14 10:46 CST
- **标签**: mrv2
- **摘要**: add engine stat     ## Purpose  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such as providing test…

### #52250 — [fix(docs): correct "Mixture-of-Expert" to "Mixture-of-Experts" in README](https://github.com/vllm-project/vllm/pull/52250)
- **作者**: arpingblue  **时间**: 2026-08-14 10:46 CST
- **标签**: documentation
- **摘要**: ## What does this PR do?  Fixes a typo in `README.md`: `Mixture-of-Expert` → `Mixture-of-Experts`.  The standard term is "Mixture of Experts" (MoE), plural. The singular form `Mixture-of-Expert` is a typo in the model architecture list.  ## Changes  - `README.md` line 56: `Mixture-of-Expert LLMs` → …

### #52249 — [[Metrics] Add operation label to Prometheus latency histograms](https://github.com/vllm-project/vllm/pull/52249)
- **作者**: ashraf-bhuiyan  **时间**: 2026-08-14 10:41 CST
- **标签**: documentation, frontend
- **摘要**: ## Problem  The [OpenTelemetry GenAI Semantic Conventions](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-metrics.md) require `gen_ai.operation.name` as a **Required** attribute on server-side metrics like `gen_ai.server.request.duration`. vLLM's Prometheus…

### #52248 — [[AMD] [Rocm]Enable DCP support in vLLM for the A2A communication path](https://github.com/vllm-project/vllm/pull/52248)
- **作者**: haic0  **时间**: 2026-08-14 10:36 CST
- **标签**: rocm
- **摘要**: ## Summary  - Enable the ROCm AITER MLA decode backend to participate in decode context parallelism (DCP), including the A2A communication path. - Size persistent decode metadata for the DCP-gathered query-head count instead of the local TP shard count. For Kimi-K3 TP8/DCP8 this changes the decode v…

### #52246 — [[Bugfix][Anthropic] Return 4xx for client-caused errors in /v1/messages](https://github.com/vllm-project/vllm/pull/52246)
- **作者**: SayHelloToWorld  **时间**: 2026-08-14 10:16 CST
- **标签**: bug, frontend
- **摘要**: ## Purpose  Closes #52088.  Invalid user input that passes the Anthropic request schema but fails the Anthropic→OpenAI conversion (e.g. more than `VLLM_MAX_STOP_STRINGS` stop sequences) currently surfaces as an **HTTP 500**: the `except Exception` in `vllm/entrypoints/anthropic/api_router.py` swallo…

### #52245 — [PD][PushConnector] Record last activity of remotes on the D side](https://github.com/vllm-project/vllm/pull/52245)
- **作者**: snadampal  **时间**: 2026-08-14 10:07 CST
- **标签**: kv-connector
- **摘要**: Refresh `_engine_last_active` in `start_load_kv` for each request being received, mirroring pull-mode. Gate the update on the engine already being present in `_remote_agents` so we never leave a liveness entry without a matching agent entry, which would trip the `_remote_agents` invariant in `_clean…

### #52244 — [[Bugfix][V1] Restore hybrid GDN prefix-cache hits under MTP spec decoding- #42](https://github.com/vllm-project/vllm/pull/52244)
- **作者**: Y-aang  **时间**: 2026-08-14 10:02 CST
- **标签**: bug
- **摘要**: ## Purpose  On **Qwen3.5-122B-A10B** with MTP speculative decoding, a replay of a cached prompt never reaches the depth the prefix cache could give it, and prompts whose length is a multiple of the hash unit get **no hit at all**. Measured live before this change, with a 1072-token GDN page and `--p…

### #52243 — [[Kernel] Fix vllm_c fused_add_rms_norm to match native IR rounding semantics](https://github.com/vllm-project/vllm/pull/52243)
- **作者**: lw-liuwang  **时间**: 2026-08-14 09:57 CST
- **摘要**: # PR body — Fix vllm_c fused_add_rms_norm rounding order to match native IR  ## Title `[Kernel] Fix vllm_c fused_add_rms_norm to match native IR rounding semantics`  ## Body  ### Summary  Fixes #52104. The CUDA `vllm_c` implementation of `fused_add_rms_norm` (vectorized and generic kernel specializa…

### #52242 — [[Feature][DSpark]: Logprobs adaptive verification](https://github.com/vllm-project/vllm/pull/52242)
- **作者**: therealnaveenkamal  **时间**: 2026-08-14 09:53 CST
- **标签**: documentation, mrv2
- **摘要**: Solves #51873   ## Purpose  Lifts the restriction blocking logprobs with enable_adaptive_verification, implementing the TODO from #47808.  Adaptive verification assigns per-request draft counts on device; the CPU-side `cu_num_logits_np` only holds an evenly-distributed stand-in. Using it as `cu_num_…
