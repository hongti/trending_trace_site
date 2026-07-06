# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-06 13:22 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 📋 Issue 动态

1. **#47689 [提案] HeartFlow 认知引擎与 vLLM 结合**
   - 提议将 vLLM 的 LLM *推理*优化能力与 HeartFlow（包含 68 个模块的 AI 认知引擎）的 *判断/认知*能力相结合，扩展 vLLM 的功能边界。
2. **#47684 [RFC] FlashInfer 在 pre-SM100 GPU 上支持 NVFP4 KV 服务**
   - 讨论在 Ampere/Hopper 等 pre-SM100 架构 GPU 上，通过 FlashInfer 实现 `--kv-cache-dtype nvfp4` 的可行性，主要目标是优化大上下文模型的 KV cache 服务。
3. **#47683 [特性请求] ModernBERT 支持 W8A8-FP8 动态量化**
   - 请求为 ModernBERT 模型适配 llmcompressor 的 W8A8-FP8-Dynamic 量化方案。当前该模型的 Linear 层未正确传递 `quant_config`，导致加载量化检查点时出错。

---

### 🔧 Pull Request 动态

**🛠️ 重要 Bug 修复**
- **#47690 [LoRA]** 修复 `_get_lora_device` 无法识别 `ark_linear` 基础层的问题，避免在使用 INC WNA16/AutoRound 方案时启用 LoRA 导致崩溃。
- **#47681 [Anthropic API]** 修复内联系统消息的合并逻辑。之前会错误地认为只要模板渲染成功模型就能处理内联系统消息（如 DeepSeekV4-Flash），现默认将其合并以防出错。
- **#47680 [V1/V2]** 修复 `prompt_logprobs` 未遵循 `logprobs_mode` 配置的问题，使其在 V1/V2 GPU runner 中均能按配置正确返回结果（而非始终返回 `log_softmax`）。
- **#47678 [聊天模板]** 补齐发布包中缺失的 `tool_chat_template_gemma4.jinja`，修复 Gemma-4 函数调用时报 ValueError 的问题。

**⚡ 性能优化与架构改进**
- **#47686 [Speculative V2]** 将 V2 `PPHandler` 中每步的 3 个 NCCL 操作（sampled/counts/draft broadcast）打包压缩为 **1 个 NCCL 操作**，显著提升通信效率。
- **#47679 [KV Offload]** 将 KV Offload 延迟监控指标拆分为 `sync` 和 `async` 两个独立的 histogram，提升监控与性能分析的粒度。

**🔧 硬件适配与回归修复 (ROCm / XPU / CPU)**
- **#47685 [ROCm]** 修复 V2 runner 中 encoder-decoder 模型在不同 attention backend 共享同一 KV cache 分配时物理布局不一致导致的回归问题。
- **#47688 [XPU]** 针对不支持 FA4 的 Intel XPU，将 mm_prefix（prefix-LM 双向掩码）模型路由至 Triton attention backend 以保证兼容性。
- **#47682 [XPU]** 在 XPU 的 `test_lmeval.py` 测试中限制 `max-num-seqs` 为 64，避免默认值导致的设备内存错误。
- **#47687 [CPU/CI]** 移除 CPU 构建中的全局 extra index，解决依赖包下载不稳定的问题。

---

### 🚀 Release 动态

*本期统计范围内无新版本发布信息。*

---

## 🐛 Issues

### #47689 — [[Proposal] HeartFlow - vLLM Inference + Cognitive Engine](https://github.com/vllm-project/vllm/issues/47689)
- **作者**: yun520-1  **时间**: 2026-07-06 12:46 CST
- **摘要**: ## Proposal: HeartFlow × vLLM  ### Synergy  vLLM optimizes LLM *inference*. HeartFlow adds LLM *judgment*.  ### What is HeartFlow?  HeartFlow is an **AI cognitive engine** (68 modules) that gives models decision-making capability:  ``` ├── Three-Layer Memory    # Context persistence ├── Decision Rou…

### #47684 — [[RFC]: FlashInfer NVFP4 KV serving on pre-SM100 GPUs](https://github.com/vllm-project/vllm/issues/47684)
- **作者**: lesj0610  **时间**: 2026-07-06 11:39 CST
- **标签**: RFC
- **摘要**: ### Motivation.  This RFC tracks the work needed to make `--kv-cache-dtype nvfp4` usable with FlashInfer on pre-SM100 GPUs, mainly Ampere/Hopper systems.  The practical target is serving large-context Qwen3.6 and Gemma 4 models with FlashInfer-backed NVFP4 KV cache without relying on the Blackwell-o…

### #47683 — [[Feature]: Support llmcompressor W8A8-FP8-Dynamic quantization for ModernBERT](https://github.com/vllm-project/vllm/issues/47683)
- **作者**: H0radricCube  **时间**: 2026-07-06 11:38 CST
- **标签**: feature request
- **摘要**: ### 🚀 The feature, motivation and pitch  ## Summary  `vllm/model_executor/models/modernbert.py` currently does not thread `quant_config` through its Linear layers. Loading a ModernBERT checkpoint produced by `llmcompressor` with the `FP8_DYNAMIC` recipe fails with:  ``` File ".../vllm/model_executor…

## 🔀 Pull Requests

### #47690 — [[Bugfix][LoRA] Support ark_linear base layer in _get_lora_device](https://github.com/vllm-project/vllm/pull/47690)
- **作者**: AlejandroParedesLT  **时间**: 2026-07-06 12:50 CST
- **标签**: bug
- **摘要**: ## Summary  Fixes bug 1 from #47650: `_get_lora_device` doesn't recognize the `ark_linear` base-layer shape produced by `INCARKLinearMethod` (INC WNA16 / AutoRound scheme), so `--enable-lora` crashes at model load with `ValueError: Unsupported base layer` for any AutoRound/INC ark-backed checkpoint.…

### #47688 — [[XPU] Route mm_prefix models to Triton attention backend](https://github.com/vllm-project/vllm/pull/47688)
- **作者**: zhenwei-intel  **时间**: 2026-07-06 12:36 CST
- **标签**: intel-gpu
- **摘要**: ## Purpose mm_prefix (prefix-LM bidirectional mask) is only supported by FA4, which is unavailable on XPU, so fall back to Triton attention.  ## Test Plan `pytest -s -v tests/models/multimodal/generation/test_mm_prefix_lm.py::test_mm_prefix_lm_e2e`  ## Test Result  --- <details> <summary> Essential …

### #47687 — [[CI/Build][CPU] Remove global extra index](https://github.com/vllm-project/vllm/pull/47687)
- **作者**: bigPYJ1151  **时间**: 2026-07-06 12:30 CST
- **标签**: documentation, ready, ci/build, cpu
- **摘要**: ## Purpose  Remove global extra index to avoid flaky package downloads.  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - …

### #47686 — [[Spec][V2] Pack PP sampled/counts/draft broadcast into one NCCL op per step](https://github.com/vllm-project/vllm/pull/47686)
- **作者**: eastwood-c  **时间**: 2026-07-06 12:24 CST
- **标签**: rocm, intel-gpu, v1, qwen, deepseek, nvidia
- **摘要**: > Stacked on #46994 — only the last two commits are new; review those.  ## Purpose  Reduce the V2 `PPHandler` deferred broadcast from 3 NCCL ops per step to 1.  With the draft-token relay added in #46994, each broadcast-eligible step issues three LL broadcasts on the sibling group (sampled tokens, c…

### #47685 — [[ROCm] Align mixed encoder-decoder KV cache views in V2 runner](https://github.com/vllm-project/vllm/pull/47685)
- **作者**: AndreasKaratzas  **时间**: 2026-07-06 11:59 CST
- **标签**: rocm, ready, v1
- **摘要**: Fixes regression in encoder-decoder models that share one KV cache allocation across attention backends with different physical layouts, introduced by: - #47035  That PR normalized the legacy model runner path, but the failing Buildkite job uses the V2 model runner. In the V2 path, `attn_utils._resh…

### #47682 — [[XPU] limit max-num-seqs in test_lmeval.py for XPU](https://github.com/vllm-project/vllm/pull/47682)
- **作者**: mayuyuace  **时间**: 2026-07-06 11:23 CST
- **标签**: intel-gpu, ready
- **摘要**: Default number of max-num-seqs would cause the UT failure on XPU. Limit it to 64 to avoid device error.

### #47681 — [[Bugfix][Anthropic API] Default to merging inline system messages](https://github.com/vllm-project/vllm/pull/47681)
- **作者**: lazypool  **时间**: 2026-07-06 11:13 CST
- **标签**: bug, frontend
- **摘要**: ## What & why  `_detect_merge_inline_system` treats "template renders without error" as "model handles inline system messages." This is wrong -- DeepSeekV4-Flash's template renders fine, but its 128-token SWA window means an inline system message at the end overwrites the user's query, producing gar…

### #47680 — [[Bugfix][V1/V2] Fix prompt_logprobs to respect logprobs_mode](https://github.com/vllm-project/vllm/pull/47680)
- **作者**: aoshen02  **时间**: 2026-07-06 11:07 CST
- **标签**: bug, v1
- **摘要**: ## Summary  Fix `prompt_logprobs` to respect the `logprobs_mode` configuration setting across **both** V1 and V2 GPU model runners.  Previously, `prompt_logprobs` always returned `log_softmax` results regardless of `logprobs_mode`. When a user set `logprobs_mode="raw_logits"`, output logprobs correc…

### #47679 — [[KV Offload] Split tiering_lookup_delay into sync/async histograms](https://github.com/vllm-project/vllm/pull/47679)
- **作者**: Srinivasoo7  **时间**: 2026-07-06 10:48 CST
- **标签**: v1, kv-connector
- **摘要**: ## Summary  Splits the planned `vllm:kv_offload_tiering_lookup_delay_seconds` metric into two histograms on `TieringOffloadingManager`:  - `vllm:kv_offload_tiering_lookup_sync_delay_seconds` : cost of a single secondary-tier lookup call. - `vllm:kv_offload_tiering_lookup_async_delay_seconds` : total…

### #47678 — [[Bugfix] Ship missing tool_chat_template_gemma4.jinja in packaged chat_templates](https://github.com/vllm-project/vllm/pull/47678)
- **作者**: HAN-oQo  **时间**: 2026-07-06 10:26 CST
- **标签**: bug
- **摘要**: ## Purpose  Fixes #47600  Passing `--chat-template examples/tool_chat_template_gemma4.jinja` (the exact invocation documented for Gemma-4 function calling) fails with `ValueError: The supplied chat template string ... appears path-like` on any installation that isn't a git checkout of the vLLM repo …
