# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-08 14:40 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期的动态摘要：

### 🐛 Issues
本期共收到 2 个 Bug 报告：
1. **后台响应请求卡死** (#66351)：在使用 `/v1/responses` 发起 `background: true` 的请求时，如果生成过程中失败，响应状态会永远卡在 "queued"，且流式场景未被现有 PR 覆盖。
2. **Granite 模板报错** (#66349)：`granite_20b_fc` 的聊天模板在渲染包含系统消息的对话时会抛出 `TemplateError`。

### 🚀 Pull Requests
本期共有 10 个 PR，涵盖性能优化、新特性支持及多个关键 Bug 修复：

**性能优化与新特性**
- **Kimi-K3 解析性能大幅提升** (#66354)：将流式推理和工具输出的解析复杂度从 O(n²) 降至 O(delta)。
- **DSA 性能优化** (#66352)：在 SM100 架构上，默认使用 DeepSelect 对 BF16 prefill indexer logits 进行排序。
- **在线量化新参数** (#66356)：为量化配置新增 `requantize_from` 参数，支持基于 dtype 选择需要重新量化的检查点层。
- **新增推理预设配方** (#66343)：为 Kimi-K3.1-Pro 添加了 MI355X 的 concurrency-1 预设。

**Bug 修复**
- **Granite 模板修复** (#66348)：修复了 `granite_20b_fc` 无法渲染系统消息的问题（对应 Issue #66349）。
- **Rust 前端卡死修复** (#66353)：修复了 `vllm-mock-engine` 在处理超过 64 tokens 的 prompt 或带工具的聊天请求时挂起的问题（接收辅助 ZMQ 帧）。
- **ROCm 兼容性修复** (#66350)：修复了 FP8 MoE oracle 在 AITER 无法服务某层时会抛出异常的问题，现在会自动回退，解决了 MI355 上的阻塞问题。
- **Llama4 工具解析修复** (#66346)：修复了 `llama4_pythonic` 模板将工具调用参数值错误地作为带引号字符串输出的问题。
- **MoE 基准测试修复** (#66347)：修复了 `benchmark_moe.py` 读取 GLM-5.3-Flash 模型 MoE shape 错误的问题。

**测试改进**
- **解耦测试环境依赖** (#66355)：使多 GPU 配置测试不再依赖宿主机的物理 GPU 数量。

### 📦 Release
本期无 Release 活动。

---

## 🐛 Issues

### #66351 — [[Bug]: Background /v1/responses requests that fail during generation stay "queued" forever (streaming case not covered by open PRs)](https://github.com/vllm-project/vllm/issues/66351)
- **作者**: SohamDutta02  **时间**: 2026-11-08 12:53 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Your output of `python collect_env.py` here ``` ==============================         System Info ============================== OS                           : Ubuntu 24.04.5 LTS (x8…

### #66349 — [[Bug]: granite_20b_fc chat template fails to render any conversation with a system message](https://github.com/vllm-project/vllm/issues/66349)
- **作者**: pra2107tham  **时间**: 2026-11-08 12:38 CST
- **摘要**: ### Your current environment  <details> <summary>Environment (the repro needs no GPU, weights or network)</summary>  ```text macOS 15 arm64, Python 3.12, torch 2.13.0 (CPU/MPS, CUDA unavailable) jinja2 3.1.6 vLLM: current main ```  </details>  ### Describe the bug  `examples/tool_chat_template_grani…

## 🔀 Pull Requests

### #66356 — [[Online quantization] Add requantize_from to select re-quantized checkpoint layers based on dtype](https://github.com/vllm-project/vllm/pull/66356)
- **作者**: mikekg  **时间**: 2026-11-08 14:15 CST
- **标签**: documentation, quantization
- **摘要**: ## Overview  Adds `requantize_from` to `--quantization-config`. It selects which checkpoint layers online (re-)quantization applies to: quantized layers by checkpoint weight format, unquantized layers by dtype. Builds on the MoE re-quantization API from #60276. For example, this runs the experts of …

### #66355 — [[Tests] Make multi-GPU config tests independent of the host GPU count](https://github.com/vllm-project/vllm/pull/66355)
- **作者**: jjzz888  **时间**: 2026-11-08 13:47 CST
- **标签**: kv-connector
- **摘要**: ## Overview  `create_scheduler` (`tests/v1/core/utils.py`) and `test_kv_connector` (`tests/v1/kv_connector/unit/test_config.py`) build a `ParallelConfig` with world size > 1 and no executor backend. On CUDA hosts with fewer local GPUs than the world size, these tests fail in config validation before…

### #66354 — [[Perf][Kimi-K3] Parse streamed reasoning and tool output in O(delta)](https://github.com/vllm-project/vllm/pull/66354)
- **作者**: gaby  **时间**: 2026-11-08 13:24 CST
- **标签**: tool-calling, kimi
- **摘要**: ## Overview  The Python Kimi K3 reasoning parser and tool parser search the whole accumulated text again on every streamed delta, so parsing one request costs O(n²) in its output length. Both now resume from where the previous call stopped, so a delta costs O(delta).  Related: [vllm-project/vllm#508…

### #66353 — [[Bugfix][Rust Frontend] Accept auxiliary frames in the mock engine](https://github.com/vllm-project/vllm/pull/66353)
- **作者**: hokwangchoi  **时间**: 2026-11-08 12:57 CST
- **标签**: bug, rust
- **摘要**: ## Overview  `vllm-mock-engine` hangs on any prompt over about 64 tokens, and on chat requests with tools, because it rejects the auxiliary ZMQ frame that #62702 introduced for large prompt token ids. This accepts trailing frames on add requests.  ## Claims  - Long prompts and chat requests with too…

### #66352 — [[Perf][DSA] Rank BF16 prefill indexer logits with DeepSelect by default on SM100](https://github.com/vllm-project/vllm/pull/66352)
- **作者**: guanzheh  **时间**: 2026-11-08 12:55 CST
- **标签**: ci/build
- **摘要**: ## Overview  With the default `sparse_indexer_topk_backend="auto"`, the sparse-indexer prefill converts the BF16 MQA logits to FP32 and ranks them with `top_k_per_row_prefill`. This PR makes `"auto"` rank them with DeepSelect on SM100 and SM103, as `"deep_select"` does since #62240. Decode, explicit…

### #66350 — [[Bugfix][ROCm] Let the FP8 MoE oracle fall back when the global AITER flag can't serve the layer](https://github.com/vllm-project/vllm/pull/66350)
- **作者**: AndreasKaratzas  **时间**: 2026-11-08 12:51 CST
- **标签**: bug, rocm
- **摘要**: ## Overview  With only `VLLM_ROCM_USE_AITER=1`, the FP8 MoE oracle now prefers AITER and falls back when AITER can't serve the layer, instead of raising. Unblocks the MI355 Nemotron-3-Super FP8 ReplaySSM mirror in #66318.  ## Claims  - `oracle/fp8.py`: only an explicit `VLLM_ROCM_USE_AITER_MOE` (set…

### #66348 — [[Bugfix] granite_20b_fc: render conversations that carry a system message](https://github.com/vllm-project/vllm/pull/66348)
- **作者**: pra2107tham  **时间**: 2026-11-08 12:38 CST
- **标签**: bug, documentation
- **摘要**: ### Purpose  `examples/tool_chat_template_granite_20b_fc.jinja`, the template `docs/features/tool_calling.md` recommends for `ibm-granite/granite-20b-functioncalling` (`--tool-call-parser granite-20b-fc --chat-template examples/tool_chat_template_granite_20b_fc.jinja`), fails to render any conversat…

### #66347 — [[Benchmark] benchmark_moe: read GLM-5.3-Flash's MoE shape from its text config](https://github.com/vllm-project/vllm/pull/66347)
- **作者**: anhalu  **时间**: 2026-11-08 12:22 CST
- **标签**: performance, glm
- **摘要**: ## Overview  `benchmarks/kernels/benchmark_moe.py --model <GLM-5.3-Flash>` tunes the wrong shape and writes a file the runtime never loads. `Glm5NextForConditionalGeneration` has no branch in `get_model_params()`, and its text config sets `num_local_experts`, so it reaches the default branch, which …

### #66346 — [[Bugfix][Tool Parser] llama4_pythonic: render argument values as literals, not quoted strings](https://github.com/vllm-project/vllm/pull/66346)
- **作者**: pra2107tham  **时间**: 2026-11-08 11:44 CST
- **标签**: bug, documentation, tool-calling, llama
- **摘要**: ### Purpose  `examples/tool_chat_template_llama4_pythonic.jinja`, the template `docs/features/tool_calling.md` recommends for Llama 4, wraps every tool call argument in literal quote characters and renders the value with `"%s" | format(...)`, which is Python `str()`. Replaying a call from conversati…

### #66343 — [[Recipe] Kimi-K3.1-Pro: add MI355X concurrency-1 preset, align with InferenceX#4887](https://github.com/vllm-project/vllm/pull/66343)
- **作者**: seungrokj  **时间**: 2026-11-08 10:51 CST
- **标签**: ready, kimi, recipe
- **摘要**: ## Overview  Aligns the Kimi-K3.1-Pro recipe's MI355X section with [SemiAnalysisAI/InferenceX#4887](https://github.com/SemiAnalysisAI/InferenceX/pull/4887), the per-concurrency retune of the K3.1 MI355X vLLM AgentX lane on the recipe's own `vllm/vllm-openai-rocm:kimi-k31` image (`sha256:5f47cc10…`).…
