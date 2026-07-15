# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-15 09:00 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🚀 Release (版本发布)
* **v0.25.1**：这是一个小版本补丁发布，主要亮点是包含了两项针对性的 Bug 修复，由 2 位贡献者（含 1 位新增贡献者）完成。

### 🐛 Issue (问题追踪)
* **Idefics3/SmolVLM 图像处理尺寸不匹配 (#48667)**：当图像未被分块处理（`num_patches=0`）时，`pixel_values` 尺寸计算错误，但 prompt 仍会预留 `image_seq_len` 数量的占位符 token，导致二者不一致。

### 🔀 Pull Request (代码合并)
**1. 核心 Bug 修复**
* **SentencePiece logprobs 冲突 (#48674)**：修复了 SentencePiece 解码时去除前导空格（`add_dummy_prefix` 逆操作）导致的 token 字符串冲突问题（例如 "▁true" 和 "true" 被错误解析为同一字符串）。
* **Harmony 模型空响应 (#48672)**：在 OpenAI 兼容路径中，若无 tool-call parser，现在会拒绝 `tool_choice="required"` 或指定工具名的请求，避免产生空响应。
* **投机解码 QK 融合异常 (#48671)**：修复了当目标模型与草案模型具有不同注意力几何结构时，QK Norm+RoPE 融合失败的问题。
* **离线多进程堆管理 (#48675)**：修复了 V1 架构下离线多进程前端（运行 `step()` 的进程）的 heap 冻结与解冻逻辑，确保在 teardown 时正确解冻。
* **DCP 错误提示优化 (#48665)**：当使用不兼容的注意力 backend 设置 DCP（Decode Context Parallel）时，现在会在报错信息中明确列出兼容的 backend，便于排查。

**2. 新特性与内核支持**
* **Gemma-4 FA4 FP8 内核 (#48666)**：新增对 Gemma-4 `full_attention` 层（`head_dim=512`）的支持，在 SM90 (H100/H200) 上自动从 FA3 升级至 FlashAttention-4 cuteDSL 内核，并支持 fp8-KV-dequant 变体。

**3. 架构优化、指标与测试**
* **请求验证规范化架构分析 (#48669)**：针对工具调用等问题，提出在 vLLM 上游引入“硬拦截”请求验证与规范化模块的架构方案，以从根本上解决 DSML 泄漏、乱码和空响应问题。
* **V1 Prefix-cache 指标丢失 (#48668)**：修复了在零输出 step 中 prefix-cache 命中/查询计数器被清空但未记录的问题，确保指标统计不遗漏。
* **量化权重绑定测试与状态确认 (#48673, #48670)**：为 ModelOpt 量化路径中 `lm_head` 与 `embed_tokens` 权重绑定添加了回归测试；同时确认 Gemma4 dense NVFP4 之前因 `tie_weights` 抛出 `NotImplementedError` 的崩溃已在 main 分支修复，仅属历史版本滞后问题。

---

## 🐛 Issues

### #48667 — [Idefics3/SmolVLM: num_patches=0 for non-tiled images mis-sizes pixel_values while the prompt still reserves image_seq_len placeholder tokens](https://github.com/vllm-project/vllm/issues/48667)
- **作者**: ErenAta16  **时间**: 2026-07-15 07:33 CST
- **摘要**: ### Description  For Idefics3/SmolVLM, when an image doesn't get tiled (`do_image_splitting=False`, or splitting enabled but the image is already small enough that no split is needed), `Idefics3ProcessingInfo` computes `num_patches=0` for that image and uses it to size the image's `pixel_values`/`pi…

## 🔀 Pull Requests

### #48675 — [fix(v1): freeze offline multiprocess front-end heap (#48229)](https://github.com/vllm-project/vllm/pull/48675)
- **作者**: aryanyadav0402  **时间**: 2026-07-15 08:15 CST
- **标签**: frontend, v1
- **摘要**: fix(v1): freeze offline multiprocess front-end heap and unfreeze on teardown (#48229)  ## Summary  In multiprocess mode the offline `LLM` / `LLMEngine` front-end process (the one that runs `step()`) is never handed to `gc.freeze()`, even though every comparable long-lived heap in vLLM already is:  -…

### #48674 — [[Bugfix] Fix logprobs token-string collision from SentencePiece space…](https://github.com/vllm-project/vllm/pull/48674)
- **作者**: aoshen02  **时间**: 2026-07-15 08:10 CST
- **标签**: bug
- **摘要**: … stripping  Fixes #44319  SentencePiece's decode() strips the leading space from the first token (add_dummy_prefix inverse), causing distinct tokens like "▁true" (id=1565) and "true" (id=3009) to both decode to "true". When the Legacy Completions API builds top_logprobs as dict[str, float], these c…

### #48673 — [test(quantization): cover tied lm_head/embed_tokens when lm_head excluded from ModelOpt](https://github.com/vllm-project/vllm/pull/48673)
- **作者**: pjdurden  **时间**: 2026-07-15 07:59 CST
- **摘要**: ### Summary Adds regression coverage for tying an excluded `lm_head` to `embed_tokens` through the quantization method path (`QuantizeMethodBase.tie_weights`), the behavior that fixed the original NVFP4 crash on main.  Test only. Related to #48238

### #48672 — [fix: reject required/named tool_choice for Harmony models without a tool parser](https://github.com/vllm-project/vllm/pull/48672)
- **作者**: pjdurden  **时间**: 2026-07-15 07:55 CST
- **标签**: gpt-oss
- **摘要**: ### Summary In the OpenAI-compatible chat path, when no tool-call parser is configured, `tool_choice="required"` and named tool choices were accepted for Harmony (gpt-oss) models and then produced empty or incorrect tool-call results.  ### Fix Only exempt Harmony for `tool_choice="auto"` (best effor…

### #48671 — [[Bugfix][Spec Decode] Support heterogeneous QK fusion geometry](https://github.com/vllm-project/vllm/pull/48671)
- **作者**: aoshen02  **时间**: 2026-07-15 07:41 CST
- **标签**: bug, ready
- **摘要**: ## Purpose  Fix QK Norm+RoPE fusion when speculative decoding uses target and draft models with different attention geometries.  `QKNormRoPEFusionPass` discovers attention metadata from `CompilationConfig.static_forward_context`, which contains modules from both the target and draft models. It previ…

### #48670 — [[Bug]: Gemma4 dense NVFP4 (nvidia/Gemma-4-31B-IT-NVFP4) fails on v0.24.0 with NotImplementedError in quant_method.tie_weights (lm_head/embed_tokens weight tying)](https://github.com/vllm-project/vllm/pull/48670)
- **作者**: pjdurden  **时间**: 2026-07-15 07:35 CST
- **标签**: bug, documentation, nvidia
- **摘要**: # Issue #48238 — Gemma4 dense NVFP4 fails in `quant_method.tie_weights`  **Headline: the crash is already fixed on `main`, and the fix is present in this tree.** The bug is a release-lag problem, not a live `main` defect. The only real gap left in `main` is that the fix landed with no regression tes…

### #48669 — [[Bug]: Analysis for an Automated Request Validation and Normalization Module at the Upstream of vLLM (Hard Interception) to Fundamentally Solve DSML Leakage, Garbled Output, and Empty Responses](https://github.com/vllm-project/vllm/pull/48669)
- **作者**: pjdurden  **时间**: 2026-07-15 07:35 CST
- **标签**: bug, documentation
- **摘要**: # Issue #48207 — request validation for tool calling  Issue #48207 is filed as a broad architectural proposal ("add a hard-interception request validation/normalization module upstream of vLLM"). Most of what it asks for already exists, so this change fixes the one concrete hole in the existing vali…

### #48668 — [[V1][Metrics] Preserve prefix-cache stats on zero-output steps](https://github.com/vllm-project/vllm/pull/48668)
- **作者**: puririshi98  **时间**: 2026-07-15 07:33 CST
- **标签**: v1
- **摘要**: ## Purpose  `Scheduler.make_stats()` drains the prefix-cache hit/query counters every step and attaches them to a (possibly empty) `EngineCoreOutputs`. But `LLMEngine.step()` only records stats when the step produced request outputs:  ```python if (     self.logger_manager is not None     and output…

### #48666 — [[Kernel] Gemma-4 FA4 FP8 Kernel](https://github.com/vllm-project/vllm/pull/48666)
- **作者**: jhaotingc  **时间**: 2026-07-15 07:28 CST
- **标签**: v1
- **摘要**: <!-- markdownlint-disable -->  ## Purpose  Gemma-4's `full_attention` layers use `head_dim=512`, which on SM90 (H100/H200) auto-upgrades from FA3 to the FlashAttention-4 cuteDSL kernel. FA4's fp8-KV-dequant variant — it takes fp16 Q plus paged **fp8 e4m3 K/V**, dequantizes K/V in-kernel, and compute…

### #48665 — [[Bugfix] Surface DCP-compatible backends in the DCP error message](https://github.com/vllm-project/vllm/pull/48665)
- **作者**: raghavchitkara36  **时间**: 2026-07-15 07:14 CST
- **标签**: bug, v1
- **摘要**: ## Purpose  Fixes #28407.  When `--decode-context-parallel-size` (DCP) is set with an attention backend whose impl can't return the softmax LSE for decode, `check_attention_cp_compatibility` in `vllm/v1/worker/cp_utils.py` raised an assertion telling the user to "try a different backend" without say…

## 🚀 Releases

### [v0.25.1](https://github.com/vllm-project/vllm/releases/tag/v0.25.1)
- **作者**: khluu  **时间**: 2026-07-14 16:51 CST
- **摘要**: # vLLM v0.25.1  ## Highlights  This release features 2 commits from 2 contributors (1 new)!  v0.25.1 is a patch release containing two targeted bug fixes on top of v0.25.0.  ### Bug Fixes * **Avoid blocking model launching when no system FFmpeg is available for TorchCodec** (#47888). Previously `imp…
