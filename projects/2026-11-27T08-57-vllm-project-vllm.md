# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-27 16:57 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
- **SM120 架构上 MLA 出现间歇性内存错误 (#70328)**
  在 RTX PRO Blackwell (SM120) 显卡搭配 cuBLAS 13.5.x / 13.6.0 时，MLA（Multi-Head Latent Attention）中的 nvjet GEMM 会对交错的 `W_UK` / `W_UV` `torch.bmm` 操作数发生过度读取，导致间歇性 IMA 错误。作者已提供了可独立复现的代码。

### 🔧 PR 动态

**1. Rust Frontend 完善与对齐 Python 行为**
由 BugenZhao 集中提交了多项优化，主要使 Rust 前端在错误处理和日志记录上与 Python 前端保持一致：
- **错误分类优化**：将输出流错误像提交错误一样进行分类处理（400 / 503 / 500），而非统一报 500；对于提示词中缺少占位符的媒体文件或格式错误的消息内容，明确返回 400 请求错误及具体原因。
- **日志与 ID 规范**：为 `/inference/v1/generate` 请求加上 `generate-tokens-` 前缀 ID；在 KV cache 日志中准确打印引擎设备名（而非统称 GPU）；支持按 Python 格式解码并记录每次迭代的详细信息。

**2. 性能优化与新模型/特性支持**
- **Qwen4Exp 性能提升 (#70329)**：重构 HC up-gate kernel 并提前触发 PDL，将 SM100 架构的起始阈值从 M=9 下调，提升整体性能。
- **Qwen4Exp 长上下文支持 (#70326)**：在 NVIDIA 实现中引入实验性的纯文本 prefill context parallelism (PCP)，针对长输入部署进行优化。
- **HY4 Flash 磁盘表支持 (#70324)**：为 CUDA 上的 HY4 Flash 引入可选的基于磁盘的 Engram 表，同时保留原有的 pinned-RAM 默认设置。

**3. CI 修复**
- **修复 LoRA 夜间测试 (#70327)**：由于先前 PR 将 embedding 和 `lm_head` LoRA 改为了可选启用，导致 `test_mixtral_lora` 测试失败。本 PR 为该测试重新启用了相关模块。

*(注：本期数据未包含 Release 版本发布信息)*

---

## 🐛 Issues

### #70328 — [[Bug]: Intermittent IMA in MLA on SM120 with cuBLAS 13.5.x / 13.6.0: nvjet GEMM over-reads the interleaved W_UK / W_UV `torch.bmm` operands (deterministic repro, possibly #64266)](https://github.com/vllm-project/vllm/issues/70328)
- **作者**: JimpleMa  **时间**: 2026-11-27 16:50 CST
- **摘要**: ### Your current environment  The repro below is standalone: it needs torch and cuda-python, not vLLM.  - GPU: SM120 (compute capability 12.0), an RTX PRO Blackwell class card - Driver: 580.95.05, with CUDA 13.3 forward compatibility - torch: 2.13.0a0 from NGC `nvcr.io/nvidia/pytorch:26.06-py3` (CUD…

## 🔀 Pull Requests

### #70329 — [[Perf][Qwen4Exp] HC up-gate kernel restructure, early PDL trigger, SM100 down from M = 9](https://github.com/vllm-project/vllm/pull/70329)
- **作者**: gau-nernst  **时间**: 2026-11-27 16:52 CST
- **标签**: ci/build, qwen
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Follow-up to [#70160](https://github.com/vllm-project/vllm/pull/70160) (stacked on it; the diff shrinks to the last two commits once it merges and this is rebased). Three changes to the Qwen4Exp hyperconnection chain:  1. Restructure the SM100 `hc_up_gate_…

### #70327 — [[CI][LoRA] Enable embedding module LoRA in test_mixtral_lora](https://github.com/vllm-project/vllm/pull/70327)
- **作者**: bogdan-petkovic  **时间**: 2026-11-27 16:48 CST
- **标签**: ready, mistral
- **摘要**: <!-- markdownlint-disable -->  ## Overview  `lora/test_mixtral.py::test_mixtral_lora` fails in the nightly since #69354 made embedding and `lm_head` LoRA opt-in. This enables them for the test, the same way #69354 did for `tests/lora/test_llama_tp.py`.  ## Claims  - `(MI355) Distributed` passes agai…

### #70326 — [[Model] Enable experimental Qwen4Exp prefill context parallelism on NVIDIA](https://github.com/vllm-project/vllm/pull/70326)
- **作者**: erictanjn  **时间**: 2026-11-27 16:35 CST
- **标签**: documentation, ci/build, qwen, nvidia, mrv2
- **摘要**: ## Overview  Add experimental text-only prefill context parallelism (PCP) to the Qwen4Exp NVIDIA implementation, validated with Qwen3.8-Flash-Next. This targets long-input deployments using PCP4 with expert parallelism instead of TP4 with expert parallelism.  This change enables whole-model PCP and …

### #70325 — [[Rust Frontend] Prefix /inference/v1/generate request IDs with generate-tokens- like Python](https://github.com/vllm-project/vllm/pull/70325)
- **作者**: BugenZhao  **时间**: 2026-11-27 16:30 CST
- **标签**: rust
- **摘要**: ## Overview  Name `/inference/v1/generate` requests `generate-tokens-<base>` in the Rust frontend, as Python's token-in/token-out route does (since vllm-project/vllm#24261) and as its docs tell clients to abort by. First, every route resolves its prefixed request ID in one place.  ## Claims  - The e…

### #70324 — [[Model][HY4 Flash] Support disk-backed Engram tables](https://github.com/vllm-project/vllm/pull/70324)
- **作者**: 123123213weqw  **时间**: 2026-11-27 16:30 CST
- **标签**: documentation
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Prototype for #70306, related to #69789. Enable opt-in disk-backed Engram tables for HY4 Flash on CUDA, preserving the existing pinned-RAM default.  ## Claims  - Keep the existing TP-local hash heads. Read only distinct needed rows from   lazy safetensors …

### #70323 — [[Rust Frontend] Classify output stream errors like submit errors](https://github.com/vllm-project/vllm/pull/70323)
- **作者**: BugenZhao  **时间**: 2026-11-27 16:29 CST
- **标签**: rust, agents
- **摘要**: ## Overview  Classify a request error that surfaces from the output stream in the Rust frontend like the same error at submission (400 / retryable 503 / 500), instead of always answering 500. Follow-up to the review of vllm-project/vllm#64729, kept out of it while vllm-project/vllm#68933 rewrote the…

### #70322 — [[Rust Frontend] Name the engine's device in the KV cache usage log lines](https://github.com/vllm-project/vllm/pull/70322)
- **作者**: BugenZhao  **时间**: 2026-11-27 16:29 CST
- **标签**: rust
- **摘要**: ## Overview  Name the engine's device in the Rust frontend's KV cache usage log lines, as Python's `LoggingStatLogger` does since vllm-project/vllm#55650, instead of always printing `GPU`. EngineCore reports its device type in the ready response.  ## Claims  - A CPU engine's periodic stats line read…

### #70321 — [[Rust Frontend] Log --enable-logging-iteration-details records like Python](https://github.com/vllm-project/vllm/pull/70321)
- **作者**: BugenZhao  **时间**: 2026-11-27 16:28 CST
- **标签**: rust
- **摘要**: ## Overview  Make `--enable-logging-iteration-details` work under the Rust frontend: decode the engine's per-iteration records and log one line per record in Python's format. Today the Rust frontend drops them, so the flag is silently a no-op.  ## Claims  - With `--enable-logging-iteration-details`,…

### #70320 — [[Rust Frontend] Report a malformed message content part's own error](https://github.com/vllm-project/vllm/pull/70320)
- **作者**: BugenZhao  **时间**: 2026-11-27 16:28 CST
- **标签**: rust
- **摘要**: ## Overview  Report a malformed chat or Responses message content part's own error in the Rust frontend's 400, instead of serde's `data did not match any variant of untagged enum MessageContent`.  ## Claims  - For a user message whose second part is `{"type": "image_url", "image_url": {}}`, the 400 …

### #70319 — [[Rust Frontend] Reject media the prompt has no placeholder for as a request error](https://github.com/vllm-project/vllm/pull/70319)
- **作者**: BugenZhao  **时间**: 2026-11-27 16:28 CST
- **标签**: rust
- **摘要**: ## Overview  Classify a request whose prompt has no placeholder for one of its media items as a request error in the Rust frontend (400 / `INVALID_ARGUMENT`), as Python does with `PromptUpdateMismatchError`, instead of an internal error.  ## Claims  | Request | Python | Rust before | Rust after | |-…
