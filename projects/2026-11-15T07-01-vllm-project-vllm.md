# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-15 15:01 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
- **LLaVA-OneVision 视频处理崩溃 (#67743)**：用户反馈在 vLLM 最新开发版中，当单个请求包含帧数不同的多个视频时，LLaVA-OneVision 模型会发生崩溃。

### 🔧 Pull Request 动态
**1. 多模态与核心 Bug 修复**
- **修复 LLaVA-OneVision 视频帧数不一致问题 (#67744)**：针对上述 Issue，调整为每个视频返回二维嵌入序列，并添加了真实模型回归测试，确保不同帧数视频能在同一请求中正常处理。
- **优化文件下载机制 (#67739)**：将同步 HTTP 文件下载方式改为直接流式写入磁盘，避免将整个文件缓存在内存中导致潜在问题。
- **修复扩展流式生成参数失效 (#67736)**：在 scale-out 流式生成端点 (`/inference/v1/generate`) 中恢复了对 `--enable-force-include-usage` 参数的支持。

**2. 性能优化**
- **Qwen4Exp (Qwen3.8-Flash-Next) 推理加速 (#67740, #67741)**：针对 SM100 架构 GPU（如 Blackwell），优化了 QSA（Query Sparse Attention）机制。Prefill 阶段路由至 DeepGEMM 的高效 MQA logits kernel，top-k 操作默认使用 FlashInfer 的 `gvr_2`，显著提升 FP8 处理性能。

**3. 架构重构与量化改进**
- **MoE 接口统一重构 (#67746)**：将 Humming MoE 迁移至通用的 oracle 接口，这是 MoE oracle 统一化工作的重要一步，移除了原有的独立辅助函数。
- **SM86 GPU 量化后端优化 (#67745)**：对于使用 block-scaled FP8 的模型（如 `Qwen3-8B-FP8`），在 SM86 GPU（如 A10, A40, RTX 30 系列）上，`--linear-backend auto` 将优先选择 Humming 后端而非 Marlin，以获取更好的权重计算表现。

**4. 分布式与 KV Cache 调度**
- **KVConnector 异步 RL 更新支持 (#67737, #67738)**：引入了 RFC #66231 的两个实现变体（A 和 B），支持在 Prefill 和 Decode 引擎上以任意顺序执行 `pause(mode="keep")`。这使得在完全异步的强化学习 (RL) 权重更新过程中，能够保持 KV Cache 不被清除，且变体 B 无需修改底层调度器代码。

**5. CI 与测试**
- **强化批量不变性测试 (#67742)**：在 derender parity 测试中强制启用 `VLLM_BATCH_INVARIANT=1`，以确保精确的 Token 对比和生成一致性。

### 🚀 Release 动态
- **本期暂无新版本发布**。

---

## 🐛 Issues

### #67743 — [[Bug]: LLaVA-OneVision crashes when videos in one request have different frame counts](https://github.com/vllm-project/vllm/issues/67743)
- **作者**: chuncaohenli  **时间**: 2026-11-15 14:13 CST
- **标签**: bug
- **摘要**: ### Your current environment  - vLLM: `0.34.0rc1.dev366+gc07d58dc8`, commit `c07d58dc82d75f7d1a947b3264b460384ec3e18d`. - Model: `llava-hf/llava-onevision-qwen2-0.5b-ov-hf`, revision `74dd0bf867a4cda7950c17663794267c60cf4b40`. - Python 3.12.3; PyTorch `2.14.0+cu132`; Transformers `5.22.2`. - Kaggle,…

## 🔀 Pull Requests

### #67746 — [[MoE][Refactor] Migrate Humming MoE to common oracle interface (#54959)](https://github.com/vllm-project/vllm/pull/67746)
- **作者**: sayaklearner-prog  **时间**: 2026-11-15 14:57 CST
- **标签**: quantization
- **摘要**: ## Motivation Part of the MoE oracle unification effort (#54959, #37753).  Previously, Humming MoE relied on separate, standalone helper functions (`select_humming_moe_experts` and `make_humming_moe_kernel` in `vllm/model_executor/layers/quantization/utils/humming/moe.py`) and duplicate expert class…

### #67745 — [[Quantization] Prefer Humming over Marlin for block-scaled FP8 on SM86](https://github.com/vllm-project/vllm/pull/67745)
- **作者**: jhss  **时间**: 2026-11-15 14:53 CST
- **标签**: documentation, quantization
- **摘要**: ## Overview  On SM86 GPUs (A10, A40, RTX 30 series), block-scaled FP8 checkpoints such as `Qwen/Qwen3-8B-FP8` run weight-only, and `--linear-backend auto` picks Marlin for them. This PR makes `auto` pick Humming there. It also closes two gaps in Humming's FP8 linear kernel that the new default would…

### #67744 — [[Bugfix][Multimodal] Fix LLaVA-OneVision embeddings for unequal video frame counts](https://github.com/vllm-project/vllm/pull/67744)
- **作者**: chuncaohenli  **时间**: 2026-11-15 14:41 CST
- **标签**: bug, multi-modality
- **摘要**: ## Overview  Fix LLaVA-OneVision generation when one request contains videos with different frame counts. Return a two-dimensional embedding sequence for each video and add a real-model regression.  Fixes #67743.  Related historical report: #15078, closed by #15082.  ## Claims  - A request containin…

### #67742 — [[CI] Require batch invariance for derender parity tests](https://github.com/vllm-project/vllm/pull/67742)
- **作者**: gty111  **时间**: 2026-11-15 14:05 CST
- **标签**: batch-invariance
- **摘要**: ## Overview  Enable `VLLM_BATCH_INVARIANT=1` in the derender parity server fixture. Independent coupled and inline generations need batch invariance for exact token comparisons.  Observed on [main CI #102812](https://buildkite.com/vllm/ci/builds/102812) and [PR #65534 CI #102838](https://buildkite.c…

### #67741 — [[Perf][Qwen4Exp] QSA top-k: default to FlashInfer gvr_2 on SM100](https://github.com/vllm-project/vllm/pull/67741)
- **作者**: gau-nernst  **时间**: 2026-11-15 13:45 CST
- **标签**: ci/build, qwen
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Default Qwen3.8-Flash-Next (Qwen4Exp) QSA indexer top-k to FlashInfer’s `gvr_2` on SM100-family devices for supported tensors, retaining the existing cooperative/persistent chain otherwise.  ## Claims  - Reduces measured top-k latency by **1.7–2.7× for 1,0…

### #67740 — [[Perf][Qwen4Exp] Use DeepGEMM for FP8 QSA prefill scoring](https://github.com/vllm-project/vllm/pull/67740)
- **作者**: gau-nernst  **时间**: 2026-11-15 13:23 CST
- **标签**: ci/build, qwen
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Route Qwen3.8-Flash-Next (Qwen4Exp) QSA prefill scoring through DeepGEMM’s contiguous MQA logits kernel on SM100-family GPUs with eligible FP8 indexer inputs.  ## Claims  - Accelerates QSA prefill score + top-k at long contexts: **1.04x at 32k, 1.14x at 64…

### #67739 — [[Bugfix] Stream synchronous asset downloads to disk](https://github.com/vllm-project/vllm/pull/67739)
- **作者**: raunakpandey1  **时间**: 2026-11-15 13:11 CST
- **标签**: bug, ci/build
- **摘要**: ## Overview  Stream synchronous HTTP file downloads to disk instead of buffering the entire body before opening the destination. Fixes #67731.  ## Claims  - `HTTPConnection.download_file()` writes received chunks before the response completes, bounding Python body buffering by chunk size rather than…

### #67738 — [[KVConnector] S1 variant B: P/D keep pause without scheduler changes (RFC #66231, alt. to #67737)](https://github.com/vllm-project/vllm/pull/67738)
- **作者**: BeichenHuang  **时间**: 2026-11-15 12:35 CST
- **标签**: documentation, needs-rebase, kv-connector, mooncake, nixl
- **摘要**: ## Overview  Variant B of RFC #66231 (collapsed section "Variant B (glue)"): the same goal as #67737, `pause(mode="keep", clear_cache=False)` on every P and D in any order, but with no changes under `vllm/v1/core/`. Opened as a draft so that the two designs can be reviewed side by side; the RFC reco…

### #67737 — [[Core][KVConnector] P/D continuous-rollout pauses (S1): transfer ledger + mode-keyed pause table (RFC #66231, variant A)](https://github.com/vllm-project/vllm/pull/67737)
- **作者**: BeichenHuang  **时间**: 2026-11-15 12:34 CST
- **标签**: documentation, needs-rebase, kv-connector, scheduler, mooncake, nixl
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Implements variant A of RFC #66231: `pause(mode="keep", clear_cache=False)` on every P and D engine, in any order, for fully-async RL weight updates. Opened as a draft for design review, together with variant B (#67738, no changes under `vllm/v1/core/`). F…

### #67736 — [[Bugfix] Support --enable-force-include-usage in scale-out generate streaming (#67247)](https://github.com/vllm-project/vllm/pull/67736)
- **作者**: Abdulrehman1978  **时间**: 2026-11-15 12:29 CST
- **标签**: bug, frontend
- **摘要**: ## Overview  Fixes #67247: Support `--enable-force-include-usage` on scale-out streaming endpoints (`/inference/v1/generate`).  ## Claims  - When `--enable-force-include-usage` is enabled on `vllm serve --enable-scale-out`, streaming `/inference/v1/generate` requests emit usage on data chunks and th…
