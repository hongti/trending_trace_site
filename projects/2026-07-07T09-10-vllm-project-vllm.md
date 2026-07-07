# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-07 17:10 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的简洁摘要：

### 📌 Issue（议题）
1. **[Feature] Model Runner v2 支持 prefill 上下文并行** (#47846)：建议在 v2 中支持 prefill 阶段的上下文并行（CP）及 `cp8+tp8=worldsize8` 组合，以更好地运行 glm-5.2、deepseek-v4 等大模型。
2. **[RFC] 打包可变长投机解码** (#47839)：提出在投机解码中，按请求动态选择验证不同数量的 draft token，从而降低验证开销。

### 🛠 Pull Request（代码变更）
**1. 重要安全修复**
- **限制 Completions Prompt 列表长度** (#47845)：修复 `/v1/completions` 接口允许无限长 prompt 列表的漏洞，防止恶意客户端通过构造大量列表项引发引擎请求风暴。
- **加固 P2P KV-offload** (#47843)：阻止客户端通过篡改 `kv_transfer_params` 中的 peer 地址来劫持 ZMQ 控制连接，消除潜在 SSRF 风险。

**2. 核心功能与性能优化**
- **优先级感知的 KV Cache 淘汰** (#47837)：为核心调度器引入基于优先级的 KV Cache 淘汰机制，保障高优先级请求的资源需求。
- **ROCm Qwen GDN 性能优化** (#47842)：重构 Qwen GDN `_output_projection` 逻辑，直接产出扁平化张量，移除了 Inductor 生成的多余 reshape/copy 内核。

**3. 前端与 Bugfix**
- **Rust 前端：配置化 HTTP Body 限制** (#47840)：新增环境变量 `VLLM_HTTP_MAX_JSON_BODY_SIZE`，打破原有 32MiB 硬限制，解决长上下文或批量请求失败的问题。
- **Rust 前端：修复 `continue_final_message`** (#47844)：通过引入 renderer sentinel 使该参数真正作用于 HuggingFace chat template，不再沦为空操作。
- **修复 `bad_words` 空白字符崩溃** (#47841)：拦截 `SamplingParams` 中仅含空白字符（如空格、换行）的 `bad_words`，防止引发 `IndexError` 崩溃。
- **修复 `top_logprobs: null` 处理** (#47838)：当客户端显式请求 logprobs 但传入 `top_logprobs: null` 时，确保正常保留输出 logprobs。
- **修复 derender 验证硬编码** (#47834)：将 derender 验证中硬编码的 `top_logprobs` 上限（20）替换为服务端配置的 `max_logprobs`。

**4. 依赖升级**
- **升级 tpu-inference 至 v0.24.0** (#47835)：同步最新稳定版依赖。

### 🚀 Release（版本发布）
- 本次动态周期内**无新版本发布**信息。

---

## 🐛 Issues

### #47846 — [[Feature]: model runner v2 support prefill context parallelism & cp8+tp8=worldsize8](https://github.com/vllm-project/vllm/issues/47846)
- **作者**: yiminghub2024  **时间**: 2026-07-07 17:08 CST
- **标签**: feature request
- **摘要**: ### 🚀 The feature, motivation and pitch  [Feature]: model runner v2 support prefill context parallelism & cp8+tp8=worldsize8  1.why request this  because i find that  sglang with glm-5.2 and deepseek-v4 with follow --enable-nsa-prefill-context-parallel --nsa-prefill-cp-mode round-robin-split --attn-…

### #47839 — [[RFC]: Packed Variable Length Speculative Decoding](https://github.com/vllm-project/vllm/issues/47839)
- **作者**: benchislett  **时间**: 2026-07-07 16:04 CST
- **标签**: RFC
- **摘要**: ### Motivation.  In speculative decoding, we can save verification cost by selectively verifying a different number of draft tokens per request in each step. There are various techniques of selecting and tuning which / how many tokens to submit for verification, but all share a common benefit: reduc…

## 🔀 Pull Requests

### #47845 — [fix(security): bound completion prompt list to prevent unbounded engine fan-out](https://github.com/vllm-project/vllm/pull/47845)
- **作者**: jperezdealgaba  **时间**: 2026-07-07 17:08 CST
- **摘要**: The /v1/completions endpoint accepted prompt as a list of arbitrary length without any outer prompt-count limit. Each list element becomes a separate engine request, allowing an authenticated client to turn one HTTP request into an attacker-chosen number of backend subrequests, starving other tenant…

### #47844 — [[Rust Frontend] Handle `continue_final_message` with renderer sentinel](https://github.com/vllm-project/vllm/pull/47844)
- **作者**: BugenZhao  **时间**: 2026-07-07 16:55 CST
- **标签**: rust
- **摘要**: ## Purpose  Previously, we only passed the `continue_final_message` value to the chat template, but in practice, almost no Hugging Face chat template actually observes this value, so it was a no-op.  This PR ports Transformers v5 semantics to the Rust frontend: it appends the last message with a sen…

### #47843 — [fix(security): harden P2P KV-offload against client-supplied peer add…](https://github.com/vllm-project/vllm/pull/47843)
- **作者**: jperezdealgaba  **时间**: 2026-07-07 16:53 CST
- **标签**: frontend, v1
- **摘要**: The P2P secondary tier for KV offloading trusted remote_host and remote_port from client-supplied kv_transfer_params, allowing an authenticated API client to steer outbound ZMQ control connections to arbitrary endpoints (SSRF). Defense in depth with two layers: 1. Strip remote_host/remote_port from …

### #47842 — [[ROCm][Perf] Avoid extra reshape kernel in Qwen GDN output projection](https://github.com/vllm-project/vllm/pull/47842)
- **作者**: mjkvaak-amd  **时间**: 2026-07-07 16:44 CST
- **标签**: rocm, qwen
- **摘要**: ## Purpose  Removes an extra Inductor-generated reshape/copy kernel, by reworking Qwen GDN `_output_projection` method. `RMSNormGated` logic now directly produces the flattened `[num_tokens, hidden]` tensor consumed by `out_proj` which allows Inductor to pick.  ## Test Plan  Run existing RMSNormGate…

### #47841 — [[Bugfix] Reject whitespace-only bad_words in SamplingParams](https://github.com/vllm-project/vllm/pull/47841)
- **作者**: anxkhn  **时间**: 2026-07-07 16:31 CST
- **标签**: bug
- **摘要**: ## Purpose  A whitespace-only `bad_words` entry (for example `bad_words=[" "]`, `"\t"`, or `"\n"`) passes `SamplingParams` validation and then crashes request setup with an uncaught `IndexError`, which the server surfaces to the client as HTTP 500 rather than a 400.  Root cause: `SamplingParams._ver…

### #47840 — [[Rust Frontend] Make HTTP request body limit configurable via VLLM_HTTP_MAX_JSON_BODY_SIZE](https://github.com/vllm-project/vllm/pull/47840)
- **作者**: ivanium  **时间**: 2026-07-07 16:06 CST
- **标签**: rust
- **摘要**: ## Purpose  The Rust frontend caps HTTP request bodies at a fixed 32 MiB, introduced in #46582. The Python frontend (uvicorn) has no body-size limit, so long-context or batched requests that work fine against the Python server fail with a terminal client error on the Rust one (the length-limit rejec…

### #47838 — [[Bugfix][Frontend] Preserve logprobs when top_logprobs is null](https://github.com/vllm-project/vllm/pull/47838)
- **作者**: Sunt-ing  **时间**: 2026-07-07 16:03 CST
- **标签**: bug, frontend
- **摘要**: ## Purpose  `/v1/chat/completions` and `/v1/responses` both accept `top_logprobs: null`. When a client explicitly requests output logprobs and serializes this optional field as JSON null, current main returns 200 but silently omits generated-token logprobs.  The request conversion uses `top_logprobs…

### #47837 — [[Core] Priority-aware KV cache eviction for priority scheduling](https://github.com/vllm-project/vllm/pull/47837)
- **作者**: chaunceyjiang  **时间**: 2026-07-07 15:53 CST
- **标签**: v1
- **摘要**: ## Purpose fix https://github.com/vllm-project/vllm/issues/47802  ## Test Plan Priority-aware KV cache eviction for priority scheduling ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue…

### #47835 — [Upgrade tpu-inference to v0.24.0](https://github.com/vllm-project/vllm/pull/47835)
- **作者**: CienetStingLin  **时间**: 2026-07-07 15:40 CST
- **标签**: ci/build
- **摘要**: ## Purpose  Upgrade tpu-inference to latest stable release v0.24.0  ## Test Plan  Verified on tpu-inference CI.  ## Test Result  Success.  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existi…

### #47834 — [fix: use configured max_logprobs instead of hardcoded 20 in derender validation](https://github.com/vllm-project/vllm/pull/47834)
- **作者**: jperezdealgaba  **时间**: 2026-07-07 15:22 CST
- **标签**: frontend, ready
- **摘要**: ## Summary  - Replace the hardcoded `top_logprobs` limit of `20` in `_validate_derender_bounds()` with `self.model_config.max_logprobs`, so the derender endpoints respect the server's `--max-logprobs` setting. - The default value (`20`) comes from the [OpenAI Chat Completions API spec](https://platf…
