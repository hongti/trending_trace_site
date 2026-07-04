# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-04 01:35 UTC

## AI 总结

# vLLM 仓库近期动态摘要

---

## 📋 Issue

- **#47582 — 多轮对话增量提示词编码（RFC）**
  提出在多轮聊天中，每次请求都会对**完整历史**重新分词，而第 N 轮提示词几乎总是第 N+1 轮的严格前缀，造成大量浪费。建议引入 opt-in 的增量编码机制，仅对新增部分做分词，降低随对话长度线性增长的开销。

---

## 🔧 PR — Bug 修复

| PR | 修复内容 |
|---|---|
| **#47587** | TRT-LLM 8×4 FP4 scale padding 未正确清零，导致 FlashInfer NVFP4 布局下量化结果异常 |
| **#47586** | `find_loaded_library()` 以纯子串匹配 `/proc/self/maps`，可能误劫持同名子串的其它库（如 TileLang），改为匹配映射后的文件名 |
| **#47579** | WSL2 上 UVA 不可用（pin_memory 默认关闭）时 V2 ModelRunner 崩溃，改为自动回退到 V1 |
| **#47578** | XPU Docker 镜像启动时未加载 Intel OneAPI 环境，导致 torch 无法发现 XPU 设备而退出；修复 Dockerfile 确保初始化 |

---

## ⚡ PR — 性能优化

| PR | 优化内容 |
|---|---|
| **#47583** | 实现 #47582 RFC：为多轮对话引入 opt-in **增量提示词编码缓存**，避免每轮重分词全部历史，大幅降低前端分词耗时 |
| **#7584** | DSpark 投机解码共享目标模型 LM Head，每步草案需做全量 bf16 GEMM；改为 opt-in **rowwise-fp8** 草案 LM Head，显著减少计算量 |
| **#47580** | ModelOpt Llama-4 FP8 checkpoint 加载耗时 68s–495s，根因是非连续 CPU 张量的 H2D 拷贝；改为保持拷贝密集，大幅缩短加载时间 |
| **#47581** | Rust 前端处理多模态张量（如 pixel_values）时有两次多余拷贝；去除 `lower_text_request` 和序列化阶段的冗余 clone，降低延迟与内存占用 |

---

## 🆕 PR — 新特性 / 架构演进

| PR | 主要变更 |
|---|---|
| **#47585** | **ModelRunner V2 支持自定义 logits processor**——这是导致 V2 静默回退 V1 的最后一类功能缺口，本 PR 在 V2 GPU 上完整实现，消除回退 |
| **#47577** | **SM120（Blackwell RTX 50 系列 / RTX PRO 6000）自动选择 FLASHINFER_B12X NVFP4 MoE 后端**，并拒绝 EP 部署（NVFP4 W4A16 场景下 EP 不受支持），覆盖 `nvidia/Qwen3.6-35B-A3B-NVFP4` 等模型 |

---

## 📦 Release

本次周期内**无新版本发布**。

---

> **总览**：本期核心方向有三——① **多轮对话前端效率**（增量编码 RFC + 实现）；② **ModelRunner V2 功能补全**（logits processor 移植，消除 V1 回退）；③ **Blackwell NVFP4 生态**（MoE 后端自动选择、FP4 scale padding 修复）。多项性能修复（MoE 加载密集化、多模态零拷贝、DSpark fp8 草案头）和平台兼容性修复（WSL2 / XPU Docker）同步推进。

---

## 🐛 Issues

### #47582 — [[RFC]: Opt-in incremental prompt encoding for multi-turn chat](https://github.com/vllm-project/vllm/issues/47582)
- **作者**: bird  **时间**: 2026-07-03 23:07 UTC
- **摘要**: ## Motivation  vLLM's frontend re-tokenizes the **entire** prompt on every request. In multi-turn chat this is almost pure waste: each request re-sends the whole conversation, so the rendered prompt of turn `N` is (in the overwhelmingly common case) a strict string prefix of the prompt of turn `N+1`…

## 🔀 Pull Requests

### #47587 — [[Bugfix][Kernel] Zero TRT-LLM 8x4 FP4 scale padding](https://github.com/vllm-project/vllm/pull/47587)
- **作者**: jesco-absolut  **时间**: 2026-07-04 00:58 UTC
- **标签**: bug, nvidia
- **摘要**: ## Purpose  Fixes #37563.  When `scaled_fp4_quant(..., backend="trtllm")` uses FlashInfer's 8x4 NVFP4 scale layout (`m <= 32`), vLLM bypasses `create_fp4_scale_tensor()` and receives the scale buffer directly from `flashinfer_quant_nvfp4_8x4_sf_layout(...)`. That buffer is physically padded to 8-row…

### #47586 — [[Bugfix] Match the mapped filename in find_loaded_library](https://github.com/vllm-project/vllm/pull/47586)
- **作者**: lucifer1004  **时间**: 2026-07-04 00:43 UTC
- **标签**: bug
- **摘要**: ## Purpose  Fix a nondeterministic library hijack in `find_loaded_library()`.  The function picks the **first** `/proc/self/maps` line containing `lib_name` as a plain substring. Two problems compound:  1. Any co-loaded library whose *filename* merely contains the target matches — e.g. TileLang dlop…

### #47585 — [[ModelRunner V2] Support custom logits processors](https://github.com/vllm-project/vllm/pull/47585)
- **作者**: KasraBashirioskooei  **时间**: 2026-07-04 00:24 UTC
- **标签**: v1
- **摘要**: ## Purpose  Custom logits processors (`--logits-processors` / entry-point plugins in the `vllm.logits_processors` group) are one of the remaining features that force a silent fallback from the V2 model runner to V1 (`_get_v2_model_runner_unsupported_features`). This PR implements them in the V2 GPU …

### #47584 — [[Spec Decode][Perf] Rowwise-fp8 draft lm_head for DSpark (opt-in)](https://github.com/vllm-project/vllm/pull/47584)
- **作者**: bird  **时间**: 2026-07-03 23:07 UTC
- **标签**: speculative-decoding, v1
- **摘要**: ## Motivation  DSpark (#46995) shares the target model's LM head (`has_own_lm_head=False`: the `ParallelLMHead` is aliased onto the draft model at load). Every draft step, `DSparkSpeculator._sample_sequential` computes base proposal logits through that head — a full `[hidden, vocab]` bf16 GEMM (4096…

### #47583 — [[Perf][Frontend] Add opt-in incremental prompt-encoding cache for multi-turn chat](https://github.com/vllm-project/vllm/pull/47583)
- **作者**: bird  **时间**: 2026-07-03 23:07 UTC
- **标签**: performance
- **摘要**: RFC: #47582   ## Purpose  In multi-turn chat, every request re-sends the whole conversation, so the rendered prompt of turn `N` is almost always a strict string prefix of turn `N+1` — yet `BaseRenderer._tokenize_prompt` re-tokenizes the full prompt every turn. That cost is linear in history length (…

### #47581 — [[Rust Frontend] Avoid extra copies for multimodal tensors](https://github.com/vllm-project/vllm/pull/47581)
- **作者**: reidliu41  **时间**: 2026-07-03 23:03 UTC
- **标签**: rust
- **摘要**: ## Purpose   Multimodal requests with inline tensor payloads can carry large byte buffers,   such as image `pixel_values`. The Rust frontend previously copied those bytes   in two avoidable places before sending the request to engine:    1. `lower_text_request` cloned `request.mm_features` into `Gen…

### #47580 — [[Perf] Keep H2D transfers dense in fused-MoE weight loading](https://github.com/vllm-project/vllm/pull/47580)
- **作者**: ishrith-gowda  **时间**: 2026-07-03 22:59 UTC
- **摘要**: ## Purpose  Fix the root cause of #31624: ModelOpt Llama-4 (Scout/Maverick FP8) checkpoints take 68s-495s to load depending on hardware, dominated by H2D copies from non-contiguous CPU tensors.  These checkpoints store expert weights fused as `[num_experts, hidden_in, hidden_out]`. `Llama4Model.load…

### #47579 — [[Bugfix] Fall back to V1 model runner when UVA is unavailable (WSL2)](https://github.com/vllm-project/vllm/pull/47579)
- **作者**: SmartAI  **时间**: 2026-07-03 22:27 UTC
- **标签**: bug
- **摘要**: ## Purpose  FIX #47387 FIX #47292  On WSL2, `CudaPlatformBase.is_pin_memory_available()` returns `VLLM_WSL2_ENABLE_PIN_MEMORY`, which defaults to `0` — pinned memory was made opt-in for WSL2 in #41496 because it has a fixed per-tensor overhead there. That makes `is_uva_available()` `False`, and the …

### #47578 — [fix: Load OneAPI enviroment before running vllm on xpu Docker image](https://github.com/vllm-project/vllm/pull/47578)
- **作者**: telnetdoogie  **时间**: 2026-07-03 22:14 UTC
- **标签**: intel-gpu, ci/build
- **摘要**: Fix the Dockerfile.xpu to ensure OneAPI environment is loaded. Without this, no xpu devices are found by torch, and the container abends.  ## Purpose Ensure the XPU Docker image initializes the Intel oneAPI runtime environment when the container starts, so vllm serve can detect XPU devices without u…

### #47577 — [[MoE/b12x] Auto-select FLASHINFER_B12X NVFP4 MoE backend on SM120, reject EP deployments](https://github.com/vllm-project/vllm/pull/47577)
- **作者**: waynehacking8  **时间**: 2026-07-03 22:12 UTC
- **标签**: nvidia
- **摘要**: ## Purpose  Covers the SM120 NVFP4 side of #31085 (does not close it; the MXFP4/gpt-oss path is separate): on RTX PRO 6000 / RTX 50-series Blackwell (compute capability 12.0), NVFP4 MoE checkpoints that resolve as W4A16 (`weight_key=kNvfp4Static, activation_key=None`, e.g. `nvidia/Qwen3.6-35B-A3B-NV…
