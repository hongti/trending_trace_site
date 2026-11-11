# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-11 14:59 CST

## AI 总结

以下是 `vllm-project/vllm` 仓库近期动态的中文摘要：

### 📋 Issue
- **#66974 [RFC] `vllm bench serve` 与 `vllm-bench` 的流一致性测试**
提议为 Python 版的 `vllm bench serve` 和 Rust 版的 `vllm-bench` 建立共享的 SSE 流一致性测试用例，以确保两者在判定 TTFT（首 Token 时间）和 ITL（Token 间延迟）等核心性能指标时逻辑一致。

### 🔨 Pull Requests (PR)

**🚀 性能优化**
- **#66972 [FlashInfer] 缓存 KV 写入视图**：为 FlashInfer 的 KV 缓存写入操作增加张量级别的视图缓存，避免每次写入都重复执行 `transpose`/`split` 操作，提升效率。
- **#66970 [KV Connector] 优化 NIXL 区域注册**：在 block-outermost KV 缓存布局下，为 GQA 缓存启用整行区域注册，避免 NixlConnector 此前为每层单独注册的性能开销。
- **#66967 [ROCm] RDNA 平台性能优化**：针对 RDNA (gfx11/gfx12) 架构优化 GDN prefill 相关内核（kkt/solve_tril/fwd_h），引入 head-first grid、向量化存储及无溢出配置。

**🐛 缺陷修复**
- **#66968 [Core] 修复前缀缓存重置事件延迟**：修复了调度器重置前缀缓存后，`AllBlocksCleared` 事件未立即发布，而是被延迟到下一次更新才处理的问题。
- **#66966 [Bugfix] 修复 0-size views 导致的内存泄漏**：修复了 `weak_ref_tensor` 在处理大小为 0 的视图时（如 GLM-5.3-Flash 的 MLA 场景），错误保留基础张量导致可破坏 CUDA graph 的问题。

**🖥️ 硬件兼容性 (XPU / Intel GPU)**
- **#66975 [XPU] 启用 Ray 通信测试**：修复 Ray 多进程 worker 中硬编码 "cuda" 导致在 Intel XPU 上测试失败的问题。
- **#66969 [XPU] HF3FS 测试设备无关化**：移除 HF3FS mock client 和测试脚本中硬编码 CUDA 的逻辑，使其能在 XPU 上正常执行。
- **#66973 [XPU] 更新 CI 工作目录**：将 Intel GPU CI 环境的工作目录从 `/workspace/vllm` 更新为 `/vllm-workspace/`。

**🧪 测试与 CI**
- **#66976 [CI] 切换 GSM8K 测试集**：将 lm-eval-harness 中的 GSM8K 准确率测试从标准 `gsm8k` 切换为质量更高的 `gsm8k_platinum` 数据集。
- **#66971 [Test] 恢复 CUDA 上的 Qwen3-8B EAGLE3 测试**：移除平台限制，在 CUDA 上恢复运行原本仅在 ROCm 上执行的 `qwen3_eagle3-transformers` 正确性测试。

### 🚀 Release
- 本期动态中暂无新版本发布信息。

---

## 🐛 Issues

### #66974 — [[RFC]: Shared stream conformance cases for `vllm bench serve` and `vllm-bench`](https://github.com/vllm-project/vllm/issues/66974)
- **作者**: dilberx  **时间**: 2026-11-11 14:23 CST
- **摘要**: ### Motivation.  `vllm bench serve` (Python) and `vllm-bench` (Rust) each parse the same SSE streams and decide the same things: which chunk sets TTFT, which chunks add an ITL sample, and whether the request succeeded. The two parsers are written and tested separately, so each rule gets fixed twice,…

## 🔀 Pull Requests

### #66976 — [[CI] Switch lm-eval GSM8K accuracy tests to gsm8k_platinum](https://github.com/vllm-project/vllm/pull/66976)
- **作者**: Pranjal101Shrivastava  **时间**: 2026-11-11 14:38 CST
- **标签**: ci/build
- **摘要**: Overview Switch the lm-eval-harness GSM8K accuracy tests in .buildkite/lm-eval-harness/ from lm-eval's gsm8k task to gsm8k_platinum. That task uses [GSM8K Platinum](https://huggingface.co/datasets/madrylab/gsm8k-platinum), a manually cleaned version of the GSM8K test set. vLLM already uses gsm8k_pla…

### #66975 — [[XPU][Test] Enable Ray-based test_comm_ops tests on XPU](https://github.com/vllm-project/vllm/pull/66975)
- **作者**: yisustc  **时间**: 2026-11-11 14:38 CST
- **标签**: intel-gpu, ray
- **摘要**: The Ray-based multi-process workers hard-coded "cuda" devices, which fails on XPU with "cuda doesn't match the current accelerator xpu". Additionally, Ray only reports XPUs as the GPU resource when dpctl is installed, so @ray.remote(num_gpus=1) workers otherwise wait forever.  Use current_platform.d…

### #66973 — [[XPU][CI]Update working_dir from /workspace/vllm to /vllm-workspace/](https://github.com/vllm-project/vllm/pull/66973)
- **作者**: zxd1997066  **时间**: 2026-11-11 14:18 CST
- **标签**: documentation, intel-gpu, ci/build, rust
- **摘要**: ## Overview  Per discussion in https://github.com/vllm-project/ci-infra/pull/857#pullrequestreview-5836620393, update working_dir from /workspace/vllm to /vllm-workspace/ in Intel GPU CI  ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I …

### #66972 — [[Perf][Attention] Cache FlashInfer KV write views per tensor](https://github.com/vllm-project/vllm/pull/66972)
- **作者**: cjwystz  **时间**: 2026-11-11 14:18 CST
- **标签**: nvidia
- **摘要**: ## Overview  Cache the K/V views used by `FlashInferImpl.do_kv_cache_update` for the current KV tensor instead of repeating `transpose`/`split` on every write. This covers the FlashInfer slice of #64572; it does not close the other backend tasks.  ## Claims  - Reuse the same zero-copy K/V views whil…

### #66971 — [[Test] Restore Qwen3-8B EAGLE3 Transformers coverage on CUDA](https://github.com/vllm-project/vllm/pull/66971)
- **作者**: tgghzyh  **时间**: 2026-11-11 14:17 CST
- **标签**: speculative-decoding, qwen, nvidia
- **摘要**: ## Overview  Enable the existing `qwen3_eagle3-transformers` correctness case on CUDA. The case currently runs only on ROCm; this change removes that platform skip and covers the corresponding item in [the Transformers backend tracker](https://github.com/vllm-project/vllm/issues/38379).  ## Claims  …

### #66970 — [[KV Connector][NIXL] Opt-in whole-row regions for GQA caches under block-outermost layouts](https://github.com/vllm-project/vllm/pull/66970)
- **作者**: TanishGudise  **时间**: 2026-11-11 14:08 CST
- **标签**: documentation, kv-connector, nixl
- **摘要**: ## Overview  With a block-outermost KV cache layout (`VLLM_KV_CACHE_LAYOUT=BLHNC`), every layer's page of a block sits in one contiguous row, but NixlConnector still registers one region per layer for full-attention (GQA) caches. A layer's consecutive blocks are then a whole row apart, so NIXL canno…

### #66969 — [[XPU][CI] Make HF3FS mock client and tests device-agnostic](https://github.com/vllm-project/vllm/pull/66969)
- **作者**: faaany  **时间**: 2026-11-11 14:07 CST
- **标签**: intel-gpu, kv-connector
- **摘要**: ## Overview `tests/v1/kv_connector/unit/test_hf3fs_connector.py::TestHf3fsMockClient::test_multiple_tensors_rw` fails on Intel XPU, because both the test helper and the HF3FS mock client hard-code CUDA APIs. This PR makes it device-agnostic.   ## Validation ```bash pytest -sv tests/v1/kv_connector/u…

### #66968 — [[Bugfix][Core] Publish KV cache events immediately after prefix cache reset](https://github.com/vllm-project/vllm/pull/66968)
- **作者**: apex-mochen  **时间**: 2026-11-11 14:07 CST
- **标签**: bug, scheduler
- **摘要**: ## Purpose  `Scheduler.reset_prefix_cache()` clears the block pool immediately, but the resulting `AllBlocksCleared` event was only drained by `_publish_kv_cache_events()` at the end of the next `update_from_output()`. On an engine that goes idle after the reset, no scheduler step runs, so the event…

### #66967 — [[ROCm][Perf] RDNA: head-first grid, vectorized stores and spill-free configs for GDN prefill kkt/solve_tril/fwd_h](https://github.com/vllm-project/vllm/pull/66967)
- **作者**: Shan2L  **时间**: 2026-11-11 14:04 CST
- **标签**: rocm
- **摘要**: > **Stacked on #66958.** Only the last commit is new here; the first two belong to #66958 and will drop out once it merges.  ## Overview  Applies the same RDNA (gfx11/gfx12) fixes from #66958 to the three remaining GDN prefill kernels: - `chunk_scaled_dot_kkt_fwd`: head-first grid. - `solve_tril`, B…

### #66966 — [[Bugfix] Stop weak_ref_tensor from keeping the base of 0-size views alive](https://github.com/vllm-project/vllm/pull/66966)
- **作者**: ihbang  **时间**: 2026-11-11 13:58 CST
- **标签**: bug
- **摘要**: ## Overview  With `qk_rope_head_dim=0` (GLM-5.3-Flash), MLA passes a 0-size view of `qkv_lora` to every eager break of a breakable CUDA graph. `weak_ref_tensor` returned 0-size tensors unchanged, so the replay closures kept one `qkv_lora` per capture size and MLA layer alive in the CUDA graph pool. …
