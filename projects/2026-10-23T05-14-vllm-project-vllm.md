# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-23 13:14 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
- **Jetson Thor (sm_110) CUDA 构建无法启动 (#63267)**
  - **环境与现象**：在无 Flash Attention (FA) 扩展的 Jetson Thor (aarch64, sm_110) 环境下，vLLM 无法正常启动。
  - **核心问题**：`vllm_flash_attn` 在导入时即抛出异常，但系统在后续路由中仍自动选择 FA 作为注意力后端，导致启动失败。

### 🚀 PR 动态
本期合并的 PR 主要集中在 **ROCm 修复与性能优化**、**量化与内核调度优化** 以及 **底层 Bug 修复**：

1. **ROCm 生态相关**
   - **Bugfix (#63268)**：修复了 gfx942 架构上 Triton paged decode 未正确写入 output heads 的问题（通过偏移 64 位输出指针并保持 32 位 per-head 存储偏移解决）。
   - **Perf (#63263)**：在 TP > 1 且 prefill 批次 $\ge$ 1024 tokens 时，对 ROCm 上的 Muse Spark MXFP4 MoE down projection 进行列切分，提升计算性能。
   - **CI / 维护**：将 MI355 MiniMax M3 MonoKernel 的 CI 超时时间从 20 分钟提升至 45 分钟 (#63269)；将 sparse PA KV block 计数修复 backport 至 MiniMax-M3 分支 (#63270)。

2. **量化与架构优化**
   - **量化调度优化 (#63265)**：在 Hopper (SM90) 架构上，针对 4-bit 权重，自动内核选择策略现在优先使用 Humming 而非 Machete，提升了 GPTQ/AWQ 等 W4A16 模型的执行效率。

3. **Bug 修复与核心机制改进**
   - **KV 卸载修复 (#63266)**：修复了当 GPU 前缀结束于 attention chunk 内部时 `SimpleCPUOffloadConnector` 输出错误的问题，改为安全拒绝未对齐的 CPU 后缀命中并重算后缀。
   - **DeepSeek-V4.1 修复 (#63264)**：修复了 DSv4.1 稀疏 prefill 索引在复用工作区中陈旧的问题，在 combine kernel 中写入 prefill index sentinels，去除了之前粗暴的 full-workspace memset。
   - **指标修复 (#63261)**：修复了 `vllm:lora_requests_info` 中 LoRA 适配器标签错误的问题，确保 waiting 和 running 队列的标签不再串位。
   - **关闭机制优化 (#63260)**：允许通过 `VLLM_WORKER_SHUTDOWN_TIMEOUT_SECONDS` 环境变量设置 EngineCore 在立即 abort 后的清理宽限期。
   - **自动化流程 (#63262)**：无代码变更，仅处理了由于缺少 `ready` 标签导致的 CI 预检失败问题。

### 📦 Release 动态
- 本期提供的数据中 **无 Release 相关内容**。

---

## 🐛 Issues

### #63267 — [[Bug]: CUDA builds without FA extensions (Jetson Thor sm_110) cannot start: vllm_flash_attn raises at import time and FA is still auto-selected](https://github.com/vllm-project/vllm/issues/63267)
- **作者**: potoior  **时间**: 2026-10-23 12:32 CST
- **摘要**: ### Your current environment  Collected from a Jetson Thor (aarch64, sm_110) dev box; vLLM is a source checkout of `main` @ `b3b0bc3059` running against the box's Jetson-built extension set (torch 2.11.0+cu130, CUDA 13.0, driver 595.78). The failing import chain below is pure Python and does not dep…

## 🔀 Pull Requests

### #63270 — [[ROCm][MiniMax-M3] Backport #61889: fix sparse PA KV block count](https://github.com/vllm-project/vllm/pull/63270)
- **作者**: benenzhu  **时间**: 2026-10-23 12:59 CST
- **标签**: rocm, minimax
- **摘要**: ## Purpose  Backport of #61889 (by @ykamiset, still open against `main`) to `minimax-m31-flash` (#63232). The commit is cherry-picked unchanged, author kept.  With an fp8 KV cache and `VLLM_ROCM_SHUFFLE_KV_CACHE_LAYOUT=1` (the AITER sparse PA path, which the ROCm mono decode `VLLM_ROCM_MINIMAX_M3_MO…

### #63269 — [[CI][ROCm] Raise the MI355 MiniMax M3 MonoKernel timeout to 45 minutes](https://github.com/vllm-project/vllm/pull/63269)
- **作者**: AndreasKaratzas  **时间**: 2026-10-23 12:46 CST
- **标签**: rocm, ci/build, minimax
- **摘要**: - Raise the `:amd: (MI355) MiniMax M3 MonoKernel` step timeout in `models_basic.yaml` from 20 to 45 minutes.  #62994 added `test_eagle3_verify_steps_match_native` (~8 minutes) to `models/minimax_m3/amd_specialized_decode`, so the step's first pytest command alone now takes about 19 minutes. With ima…

### #63268 — [[ROCm][Bugfix] Fix Triton paged decode leaving output heads unwritten on gfx942](https://github.com/vllm-project/vllm/pull/63268)
- **作者**: AndreasKaratzas  **时间**: 2026-10-23 12:45 CST
- **标签**: bug, rocm
- **摘要**: - `chunked_prefill_paged_decode.py`: offset the output pointer to the token's row as a 64-bit scalar and keep the per-head store offsets 32-bit. - `test_prefix_prefill.py`: fill the output with NaN, so a row the kernel skips fails every time instead of passing on leftover memory.  `test_qwen3_nonsta…

### #63266 — [[Bugfix][KV Offload] Reject misaligned chunked-local CPU suffix hits](https://github.com/vllm-project/vllm/pull/63266)
- **作者**: charxwu  **时间**: 2026-10-23 12:29 CST
- **标签**: bug, documentation
- **摘要**: ## Overview  Fix incorrect output from `SimpleCPUOffloadConnector` when a GPU prefix ends inside an attention chunk. Keep the GPU prefix and recompute the suffix when CPU lookup cannot safely preserve absolute chunk boundaries.  ## Claims  - Reject CPU suffix lookup when the local prefix is not alig…

### #63265 — [[Quantization] Prefer Humming before Machete for 4-bit weights on SM90](https://github.com/vllm-project/vllm/pull/63265)
- **作者**: Rockyeast  **时间**: 2026-10-23 12:27 CST
- **标签**: documentation, nvidia, quantization
- **摘要**: ## Overview  On Hopper (SM90), automatic kernel selection for weight-only linear layers with 4-bit integer weights and FP16/BF16 activations (GPTQ, AWQ and compressed-tensors W4A16 checkpoints) now tries Humming before Machete. #56997 already moved Humming ahead of Marlin on SM90, but Machete ranks …

### #63264 — [[Bugfix][Perf][DSv4.1] Write prefill index sentinels in combine kernel](https://github.com/vllm-project/vllm/pull/63264)
- **作者**: Alex-ai-future  **时间**: 2026-10-23 12:26 CST
- **标签**: bug, performance, deepseek, DSv4.1
- **摘要**: <!-- markdownlint-disable -->  ## Overview  This PR fixes stale DeepSeek-V4.1 sparse-prefill indices in reused workspaces (#63250) and removes the full-workspace memset previously used for that fix. It includes #60613 and moves sentinel initialization into the existing combine kernel.  ## Claims  - …

### #63263 — [[ROCm][Perf] Muse Spark: column-shard the MXFP4 MoE down projection at TP prefill](https://github.com/vllm-project/vllm/pull/63263)
- **作者**: fanxingran  **时间**: 2026-10-23 12:22 CST
- **标签**: rocm
- **摘要**: ## Overview  Muse Spark MXFP4 on ROCm at TP > 1: for eager prefill batches of >= 1024 tokens, compute the MoE down projection column-sharded from a second, row-resharded copy of `w2`, so the per-route `[T, top_k, H]` reduce-scatter before the post-expert norm is replaced by much smaller gathers.  ##…

### #63262 — [No code change: pre-run-check passed after readiness label](https://github.com/vllm-project/vllm/pull/63262)
- **作者**: Copilot  **时间**: 2026-10-23 12:17 CST
- **摘要**: The check failed because the PR lacked a readiness label and its author had fewer than four merged PRs. After the `ready` label was applied, the rerun passed `pre-run-check`.  - **Changes:** None; the failure was expected gate behavior, not a workflow defect.

### #63261 — [[Bugfix][Metrics] Fix waiting and running adapter labels in vllm:lora_requests_info](https://github.com/vllm-project/vllm/pull/63261)
- **作者**: Eliaaazzz  **时间**: 2026-10-23 12:11 CST
- **标签**: bug
- **摘要**: ## Overview  Fixes the adapter labels of `vllm:lora_requests_info`, which have been wrong since #26801. Every adapter with a tracked request is listed in both `running_lora_adapters` and `waiting_lora_adapters`, and a request queued at the engine never shows up in `waiting_lora_adapters`.  ## Claims…

### #63260 — [[Core] Let VLLM_WORKER_SHUTDOWN_TIMEOUT_SECONDS set the EngineCore cleanup grace after an immediate abort](https://github.com/vllm-project/vllm/pull/63260)
- **作者**: AndreasKaratzas  **时间**: 2026-10-23 12:07 CST
- **摘要**: With the default `--shutdown-timeout 0`, the process manager sends EngineCore a SIGTERM and then kills it with SIGKILL straight away on every platform except ROCm, which gets 15 s since #52281. This PR lets an explicitly set `VLLM_WORKER_SHUTDOWN_TIMEOUT_SECONDS` set that cleanup window on every pla…
