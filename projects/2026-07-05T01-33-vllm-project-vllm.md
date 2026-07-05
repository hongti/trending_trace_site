# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-05 01:33 UTC

## AI 总结

## vllm-project/vllm 近期动态摘要

---

### 🐛 Issue

- **#47620** — `collect_env.py` 在 Windows/macOS 及无 pip 的 uv 环境下启动崩溃。脚本中的版本获取函数假定仅 Linux 运行，导致 `AssertionError`；pip 包列表获取在 uv 环境中因缺失 `pip` 模块而崩溃。

- **#47614** — Voxtral 实时语音转写（STT）陷入"空白 token 自循环"：模型在真实语音上持续输出空白 token（id 32），静默长达 1–20 分钟后自行恢复，严重影响实时转录体验。

---

### 🔀 Pull Request

#### 🔧 Bug 修复

| PR | 核心修复 |
|---|---|
| **#47621** | 修复 #47620：`get_pkg_version()` 在非 Linux 平台回退为"Could not collect"而非抛异常；`get_pip_packages()` 在无 pip 环境中优雅降级。 |
| **#47618** | 修复 #42890 引入的 DeepSeek V4 Flash 在 SM121 上的 KV cache 形状不匹配（期望 584，实际 512）。根因：跳过层在无量化时获得了未量化的形状，但 spec 仍传 `"auto"`。 |
| **#47610** | 修复 DeepSeek V4 SM10 FP8 FlashMLA 启动路径：保留 attention spec 中的 cache dtype 信息，并对普通 FP8 KV cache 避免走 SM10 FlashMLA sparse 路径。 |
| **#47617** | 修复推测解码 + 结构化输出 + reasoning 模式下，推理结束检测失败的问题：`num_computed_tokens` 提前推进导致新接受 token 未及时写入 `all_token_ids`。 |
| **#47616** | 修复 ngram 推测解码中，stop string 在接受批次中间完成时，超出 stop 边界的 token/logprobs 残留在输出中未裁剪。 |
| **#47612** | 修复 `llama4_pythonic` 工具解析器：部分 Llama-4 checkpoint 以 JSON 格式发出工具调用而非 pythonic 形式，此前被原样留在 `content` 中，现在正确解析。 |
| **#47611** | 兼容旧版 FlashInfer FP8 MoE：新版 TRTLLM MoE 签名新增 `gemm1_alpha/beta/clamp_limit` 可选参数，封装层现在按实际签名决定是否传入 SwiGLU 参数。 |

#### ✨ 新特性 / 增强

| PR | 核心内容 |
|---|---|
| **#47615** | Voxtral 实时 STT 新增可选"空白惩罚"机制，打破模型空白 token 自循环静默状态（缓解 #47614）。 |
| **#47613** | Anthropic `/v1/messages` 端点补齐采样参数透传：`seed`、`frequency_penalty`、`presence_penalty` 等此前被静默丢弃，现已正确映射到 `ChatCompletionRequest`。 |
| **#47619** | Rust Chat Frontend 新增 DeepSeek V3.2 往返测试 fixture，覆盖 DSML 工具调用路径的 render → stream parse → rerender 全流程。 |

---

### 📦 Release

本次抓取范围内无新版本发布。

---

## 🐛 Issues

### #47620 — [[Bug]: collect_env.py crashes on Windows/macOS and pip-less uv environments](https://github.com/vllm-project/vllm/issues/47620)
- **作者**: raman118  **时间**: 2026-07-05 00:19 UTC
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Crashes on startup on Windows/macOS. ```  </details>  ###  Describe the bug  When running the diagnostic script `python vllm/collect_env.py` on Windows or macOS, the execution crashes…

### #47614 — [[Bug]: Realtime STT (Voxtral realtime): self-sustained blank-token rut mutes live transcription for minutes over real speech](https://github.com/vllm-project/vllm/issues/47614)
- **作者**: damienlaine  **时间**: 2026-07-04 21:07 UTC
- **摘要**: ## Summary  A realtime transcription session (`/v1/realtime`, Voxtral Mini 4B Realtime, realtime STT path from #45833) can fall into a self-sustained decoding rut: the model emits the blank token (id 32) at every frame (12.5 tok/s) over real speech, for 1 to 20 minutes, then recovers on its own. The…

## 🔀 Pull Requests

### #47621 — [[Bugfix] Fix collect_env.py crash on Windows/macOS and pip-less uv environments](https://github.com/vllm-project/vllm/pull/47621)
- **作者**: raman118  **时间**: 2026-07-05 01:08 UTC
- **标签**: bug
- **摘要**: Fixes #47620.  - get_pkg_version() previously asserted linux-only, causing get_env_info() to raise AssertionError on Windows/macOS when unconditionally calling Intel XPU version helpers. Now falls back to 'Could not collect' on non-linux platforms. - get_pip_packages() previously crashed with Attrib…

### #47619 — [[Rust Frontend] Add DeepSeek V3.2 roundtrip fixture](https://github.com/vllm-project/vllm/pull/47619)
- **作者**: reidliu41  **时间**: 2026-07-04 23:52 UTC
- **标签**: deepseek, rust
- **摘要**: ## Purpose  Adds a DeepSeek V3.2 roundtrip case for the Rust chat frontend.   The new case uses `deepseek-ai/DeepSeek-V3.2-Exp` with the existing   DeepSeek V3.2 renderer and auto-selected parser, exercising the DSML   tool-call path through render, stream parse, and rerender.   ## Test Plan  ``` ca…

### #47618 — [Fix Deepseek v4 flash error due to #42890](https://github.com/vllm-project/vllm/pull/47618)
- **作者**: sychen52  **时间**: 2026-07-04 23:09 UTC
- **标签**: v1, deepseek
- **摘要**: ## Purpose    1. DeepSeek V4 Flash on SM121: `Expected packed SM120 DSV4 swa_kv_cache head dim 584, got 512` (reported in #42890 comments)    **Root cause:** #42890 passes `"auto"` to `get_kv_cache_shape()` when `spec.kv_quant_mode == NONE` (so skipped layers get the unquantized shape). But specs wh…

### #47617 — [Fix: detect reasoning end with accepted MTP tokens](https://github.com/vllm-project/vllm/pull/47617)
- **作者**: karthiksenv  **时间**: 2026-07-04 22:58 UTC
- **标签**: structured-output, v1
- **摘要**: ## Summary  Fixes reasoning-end detection when structured output is used together with reasoning mode and MTP/speculative decoding.  In the speculative decoding path, `num_computed_tokens` can be advanced before the newly accepted output tokens are appended to `request.all_token_ids`. Because `Struc…

### #47616 — [[Bugfix][Spec Decode] Trim token_ids/logprobs left past a stop string under speculative decoding](https://github.com/vllm-project/vllm/pull/47616)
- **作者**: Sunt-ing  **时间**: 2026-07-04 21:28 UTC
- **标签**: bug, v1
- **摘要**: ## Purpose  With `ngram` speculative decoding, a single decode step can accept several tokens at once. When a stop string completes in the middle of that accepted batch, V1 truncates `output_text` to the stop boundary but leaves the tokens generated after the stop in `RequestOutput.outputs[0].token_…

### #47615 — [[Core][Model] Voxtral realtime: opt-in blank-run penalty to break self-sustained silence ruts](https://github.com/vllm-project/vllm/pull/47615)
- **作者**: damienlaine  **时间**: 2026-07-04 21:07 UTC
- **标签**: documentation, performance, frontend, v1, multi-modality, mistral
- **摘要**: Mitigates #47614.  Stacked on #45833 (the realtime STT path this applies to); please review the last commit only, the rest is the RFC branch.  ## What  A realtime session can fall into a self-sustained decoding rut: the model emits its blank/silence token at every frame over real speech, for minutes…

### #47613 — [[Frontend] Pass sampling params through Anthropic /v1/messages](https://github.com/vllm-project/vllm/pull/47613)
- **作者**: fenghourun  **时间**: 2026-07-04 21:02 UTC
- **标签**: frontend
- **摘要**: ## Summary The Anthropic `/v1/messages` endpoint silently dropped several sampling parameters that the engine and the OpenAI endpoint already support, so clients couldn't set them. This wires them through to the converted `ChatCompletionRequest`:  - `seed` - `frequency_penalty` - `presence_penalty` …

### #47612 — [[Bugfix][Tool Parser] llama4_pythonic: accept JSON tool calls in streaming and non-streaming](https://github.com/vllm-project/vllm/pull/47612)
- **作者**: pablopupo  **时间**: 2026-07-04 20:53 UTC
- **标签**: bug, tool-calling, llama
- **摘要**: ## Purpose  Fixes #46863.  Some Llama-4 checkpoints (e.g. served with the Llama-3.1 JSON chat template) emit tool calls as JSON, like `{"type": "function", "name": "Bash", "parameters": {...}}`, instead of the pythonic form `llama4_pythonic` expects. Those responses were left verbatim in `content` i…

### #47611 — [[Bugfix][MoE] Handle older FlashInfer FP8 MoE signatures](https://github.com/vllm-project/vllm/pull/47611)
- **作者**: LucasWilkinson  **时间**: 2026-07-04 19:43 UTC
- **标签**: bug, nvidia
- **摘要**: ## Summary  Handle installed FlashInfer builds whose TRTLLM FP8 MoE Python signatures do not accept the optional `gemm1_alpha`, `gemm1_beta`, and `gemm1_clamp_limit` kwargs.  The wrapper now checks the FlashInfer function signature and:  - passes the SwiGLU params when the installed FlashInfer suppo…

### #47610 — [[Bugfix][DeepSeek V4] Avoid SM10 FlashMLA cache shape mismatch](https://github.com/vllm-project/vllm/pull/47610)
- **作者**: LucasWilkinson  **时间**: 2026-07-04 19:43 UTC
- **标签**: bug, v1, deepseek
- **摘要**: ## Summary  Fix the DeepSeek V4 SM10 FP8 startup path by preserving attention-spec cache dtype information during v1 KV cache shape computation and avoiding the SM10 FlashMLA sparse path for plain FP8 KV cache.  The branch does two things:  - Uses `cache_dtype_str` from attention specs, including `M…
