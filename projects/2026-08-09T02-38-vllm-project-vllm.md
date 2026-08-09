# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-09 10:38 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 🐛 Issue (问题反馈)
近期 Issue 主要集中在 **DeepSeek-V4 模型支持**、**新硬件适配**及**分布式解耦**的问题：
*   **DeepSeek-V4 计算与路由缺陷**：
    *   Decode 阶段无法运行融合算子（触发了 breakable CUDA graph），因为缺少 `@support_torch_compile` 且 inline pybinds 阻碍了 fullgraph 捕获 (#51522)。
    *   融合路由器 `topk_softplus_sqrt` 拒绝非标准专家数（如 144 专家的 REAP 剪枝版本），且 torch fallback 仅限 XPU (#51521)。
*   **新硬件/算子 Bug**：在 SM_120 架构（RTX 5090 笔记本）上，使用 `VLLM_CUTLASS` 后端运行 NVFP4 MoE 模型会导致输出为空并耗尽 `max_tokens` (#51525)。
*   **分布式 KV 传输失败**：使用 MooncakeConnector 进行同节点 P/D 分离部署（8x B200, fp8 MLA）时，NVLink 回退传输报错 "Requested address not found" (#51518)。

---

### 🔀 Pull Request (代码合并)
PR 动态主要涵盖 **性能优化**、**新特性支持** 及 **Bug修复**：

**🚀 性能优化**
*   **TopKTopP 采样加速**：在 log 空间进行采样，跳过全词表 softmax 计算，显著降低采样开销 (#51526)。
*   **RTX 5090 MoE 适配**：为消费级 Blackwell (RTX 5090, sm_120) 添加了 fused-MoE 的调优配置（bf16, E=128, TP=1/4），消除默认配置的启动警告并提升性能 (#51519)。
*   **ROCm MoE 优化**：针对 WNA16 MoE 在小 batch decode 场景，改用 naive block assignment 以避免不必要的 `moe_align_block_size` 开销 (#51515)。

**✨ 新特性与重构**
*   **异构 P/D 前缀缓存**：NIXL KV Connector 支持在 Prefill/Decode 逻辑块大小不同（但 kernel 块大小相同）的混合 Mamba/MLA 部署中拉取前缀缓存 (#51527)。
*   **RL 训练权重传输**：添加基于 NCCL M2N 的分片感知权重传输后端（worker 端），优化 RL 场景推理侧权重更新 (#51520)。
*   **代码重构**：将 `flashinfer_utils` 统一合并到单一文件中，消除重复代码 (#51523 / #51516)。

**🛠️ Bug 修复**
*   **Gemma 4 工具调用**：修复了指定 named `tool_choice` 时模型返回普通文本而非工具调用的问题，现在会正确拒绝不支持的配置 (#51524)。
*   **Anthropic API**：修复了配置多模型别名 (`--served-model-name`) 时，Messages API 响应未保留请求别名的问题 (#51514)。
*   **Inkling 解析器**：支持无工具纯文本场景的增量流式输出，降低流式延迟 (#51517)。

---

### 📦 Release (版本发布)
*   近期**无新的 Release 版本发布**。（注：Issue 中有用户提及 v0.25.0 及 v0.26.0，说明当前主干正在为下一个版本积累重要特性与修复）。

---

## 🐛 Issues

### #51525 — [[Bug]: Explicit VLLM_CUTLASS NVFP4 MoE selection silently produces empty completions on SM_120](https://github.com/vllm-project/vllm/issues/51525)
- **作者**: AubinGil  **时间**: 2026-08-09 08:19 CST
- **标签**: quantization
- **摘要**: ### Summary  On SM_120 (RTX 5090 Laptop), serving an NVFP4 MoE checkpoint with `--moe-backend cutlass` (which resolves to `VLLM_CUTLASS`) makes the server consume the entire `max_tokens` budget and return `content: null` with `finish_reason: "length"` for certain inputs. There is no exception, no wa…

### #51522 — [deepseek_v4: decode runs unfused (breakable CUDA graph) — DeepseekV4ForCausalLM lacks @support_torch_compile; fullgraph capture blocked by inline deep_gemm/tilelang pybinds](https://github.com/vllm-project/vllm/issues/51522)
- **作者**: eisbaw  **时间**: 2026-08-09 05:33 CST
- **摘要**: ### Summary  `DeepseekV4ForCausalLM` (nvidia backend) does not carry `@support_torch_compile`, so vLLM auto-enables `VLLM_USE_BREAKABLE_CUDAGRAPH=1` and runs decode as **unfused eager kernels wrapped in a breakable CUDA graph** (`compilation mode=NONE`). torch.compile/inductor fusion never runs for …

### #51521 — [DeepSeek-V4 (deepseek_v4): fused topk_softplus_sqrt router rejects non-standard expert counts on CUDA (REAP 144-expert ckpts); torch fallback is XPU-gated](https://github.com/vllm-project/vllm/issues/51521)
- **作者**: eisbaw  **时间**: 2026-08-09 05:32 CST
- **摘要**: ### Summary  On CUDA, `DeepseekV4ForCausalLM` cannot serve DeepSeek-V4-Flash checkpoints whose expert count is not one of the values hardcoded in the fused router kernel `topk_softplus_sqrt`. REAP-pruned checkpoints (e.g. `0xSero/DeepSeek-V4-Flash-162B`, 144 experts) fail at load/first-forward with:…

### #51518 — [MooncakeConnector P/D: NVLink fallback transfer fails "Requested address not found" (fp8 MLA, v0.25.0)](https://github.com/vllm-project/vllm/issues/51518)
- **作者**: stewtong  **时间**: 2026-08-09 05:15 CST
- **摘要**: ### Summary  Same-node P/D disaggregation of DeepSeek-V4-Flash-0731 with the MooncakeConnector fails at the first KV transfer on `vllm/vllm-openai:v0.25.0`, on an 8x B200 SXM node, in every configuration tested; each transport fails differently, and none fails at startup.  The primary case: when the…

## 🔀 Pull Requests

### #51527 — [[KV Connector][NIXL] Support prefix caching with heterogeneous P/D logical block sizes](https://github.com/vllm-project/vllm/pull/51527)
- **作者**: ivanium  **时间**: 2026-08-09 09:57 CST
- **标签**: kv-connector
- **摘要**: ## Summary  - Support pull-mode NIXL prefix caching for hybrid Mamba/MLA deployments whose P/D logical block sizes differ but kernel block sizes match. - Align full-attention transfers by token offset, zero untransferred tails, and keep unsupported push, host-buffer, sliding-window, kernel-mismatch,…

### #51526 — [[Perf] Sample in log space in TopKTopPSampler.forward_native, skipping the full-vocab softmax](https://github.com/vllm-project/vllm/pull/51526)
- **作者**: BabyDrangoner  **时间**: 2026-08-09 09:35 CST
- **摘要**: ## Purpose  `TopKTopPSampler.forward_native` computes `probs = logits.softmax(-1)` only to feed the exponential-noise Gumbel trick, `argmax(probs / q)`. The softmax normalizer is constant per row and exp is monotonic, so `argmax(logits - log q)` selects the same token — the full-vocab softmax kernel…

### #51524 — [[Bugfix][Frontend] Reject unsupported named tool_choice for Gemma 4](https://github.com/vllm-project/vllm/pull/51524)
- **作者**: Prudhvivuda  **时间**: 2026-08-09 06:37 CST
- **标签**: bug, tool-calling
- **摘要**: ## Purpose  Fixes #50477.  On v0.26.0, Chat Completions requests with a **named** forced `tool_choice` against `--tool-call-parser gemma4` return HTTP 200 with ordinary prose (`finish_reason: "stop"`, `tool_calls: null`). The forced contract is accepted and then silently not enforced.  `Gemma4Engine…

### #51523 — [feat: Unify flashinfer utils into a single file](https://github.com/vllm-project/vllm/pull/51523)
- **作者**: Himan-D  **时间**: 2026-08-09 06:15 CST
- **标签**: ci/build, nvidia, quantization
- **摘要**: Recreating PR #51516 to comply with the project's PR description guidelines in `AGENTS.md`.  Closes #31414  ### Why this is not a duplicate PR This PR recreates closed PR #51516 because the original was closed due to non-compliance with `AGENTS.md` guidelines for PR descriptions. There are no other …

### #51520 — [[RL] Add nccl-m2n sharding-aware weight transfer (worker side)](https://github.com/vllm-project/vllm/pull/51520)
- **作者**: kwen2501  **时间**: 2026-08-09 05:29 CST
- **标签**: documentation
- **摘要**: ## Purpose  Adds `nccl_m2n`, the inference-side half of a sharding-aware weight transfer backend built on [NCCL M2N](https://github.com/NVIDIA/nccl-extensions), per [RFC #46439](https://github.com/vllm-project/vllm/issues/46439).  The broadcast NCCL backend assumes both sides hold the same layout, s…

### #51519 — [[Perf] Add fused-MoE tuned configs for NVIDIA GeForce RTX 5090 (E=128, bf16, TP=1/TP=4)](https://github.com/vllm-project/vllm/pull/51519)
- **作者**: Rodder5  **时间**: 2026-08-09 05:20 CST
- **标签**: performance, nvidia
- **摘要**: ## Purpose  Requested in #48732: consumer Blackwell (RTX 5090, sm_120) has no fused-MoE tuned configs in-tree for any dtype, and MoE decode runs on default configs with the startup warning "Using default MoE config. Performance might be sub-optimal!". This PR contributes bf16 tuned configs for the Q…

### #51517 — [[Bugfix][Parser] Stream Inkling plain-text answers incrementally](https://github.com/vllm-project/vllm/pull/51517)
- **作者**: Vegetog  **时间**: 2026-08-09 05:09 CST
- **标签**: bug, frontend, tool-calling
- **摘要**: The follow-up @bbrowning asked for in #49876: the no-tools plain-text streaming-latency half, split out of that PR.  Rebased onto `main` now that #49876 has landed, so this is a single independent commit.  ## Purpose  With no tools and a `reasoning_effort` of `none` or `minimal`, Inkling answers in …

### #51516 — [feat: Unify flashinfer utils into a single file (#31414)](https://github.com/vllm-project/vllm/pull/51516)
- **作者**: Himan-D  **时间**: 2026-08-09 04:36 CST
- **标签**: ci/build, nvidia, quantization
- **摘要**: Closes #31414  ### What does this PR do? This PR resolves the confusing duplicate files by completely migrating the contents of `vllm.model_executor.layers.quantization.utils.flashinfer_utils` into `vllm.utils.flashinfer`.   All dependent kernel and test files have been updated to reflect the new im…

### #51515 — [[Kernel][ROCm] Use naive block assignment for WNA16 MoE](https://github.com/vllm-project/vllm/pull/51515)
- **作者**: ciru-ai  **时间**: 2026-08-09 03:39 CST
- **标签**: rocm
- **摘要**: ## Summary  ROCm GPTQ/AWQ WNA16 MoE currently always calls `moe_align_block_size`, even when a very small decode batch activates only a sparse fraction of the expert pool. vLLM's unquantized Triton MoE path already avoids that overhead with a naive assignment mode in this regime.  This PR:  - enable…

### #51514 — [fix(anthropic): preserve requested model alias in messages response](https://github.com/vllm-project/vllm/pull/51514)
- **作者**: safiullah3915  **时间**: 2026-08-09 03:28 CST
- **标签**: frontend
- **摘要**: Co-authored-by: @lazypool   ## Purpose  Fixes #51266.  ### Problem When vLLM is launched with multiple model aliases (--served-model-name alias-a alias-b), the requests to the Anthropic Messages API (POST /v1/messages) specifying a secondary alias (alias-b) received responses where the model field w…
