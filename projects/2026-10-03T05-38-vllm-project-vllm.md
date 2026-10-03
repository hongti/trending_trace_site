# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-03 13:38 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要。本期提供的数据仅包含 Pull Request (PR) 活动，暂无 Issue 和 Release 的相关动态。

### 🚀 Pull Request (PR) 动态

本期 PR 主要围绕模型优化、核心功能增强及多项重要 Bug 修复展开：

**1. 模型与性能优化**
* **DeepSeek V4.1 系列优化**：
  - **ROCm MoE 支持**：在 ROCm 平台上为 DeepSeek V4.1 a4w4 启用了 MoRI FP4 分发机制 (#59852)。
  - **SM120 架构性能提升**：在 SM120/SM121 架构上，改用 FlashInfer 的 V4 sparse MLA 内核来解码 DSv4.1 的 KV 记录，并拒绝了 `--kv-cache-dtype nvfp4_ds_mla` 参数 (#59846)。
  - **内存占用优化**：按确切大小注册 CPU 卸载的 Engram 表，避免 `pin_memory=True` 将内存向上取整到 2 的幂次，从而减少不必要的内存浪费 (#59849)。
* **Qwen3.5**：去除了 4 倍的 M-RoPE cos/sin 缓存余量。该余量原本是为 Qwen2.5-VL 视频位置准备的，但 Qwen3.5 并不产生此类位置，此举有效缩减了缓存体积 (#59853)。
* **Qwen4Exp 内存修复**：以确切大小固定卸载的 PLE 表，显著降低每表内存占用（从 65.2 GiB 降至 47.8 GiB）(#59848)。

**2. 核心功能增强**
* **启动快照捕获**：为普通的 `vllm serve` 增加了启动快照捕获功能。允许外部捕获已初始化的引擎，随后通过兼容 OpenAI 的服务器激活原始进程或恢复的副本 (#59851)。

**3. Bug 修复**
* **KV 缓存注册修复**：修复了在 cuMem `runtime` 池内注册 KV 缓存连接器导致 GPU 缓冲区分配异常的问题，将其移至池外执行 (#59847)。
* **NVFP4 量化修复**：修复了 NVFP4 线性层在处理融合分片时，仅取最大全局权重缩放（`weight_scale_2`）且只发出警告的问题，现正确保留各分片自身的全局权重缩放 (#59845)。
* **DeepEP 竞争条件修复**：临时内置 DeepEP 补丁，通过引入“每上下文信号 + NVLink 栅栏”修复 GIN barrier 数据可见性竞争问题（注：目前仍为 Draft 状态，集群验证中）(#59844)。

**4. CI/测试维护**
* 移除了 Kimi K3 B200 任务中重复执行的 bf16 skinny GEMM 测试，避免无谓的重复构建 (#59850)。

---
*注：本期数据中未包含 Issue 和 Release 的更新信息。*

---

## 🔀 Pull Requests

### #59853 — [[Model] Drop the 4x M-RoPE cache headroom for Qwen3.5](https://github.com/vllm-project/vllm/pull/59853)
- **作者**: lesj0610  **时间**: 2026-10-03 13:32 CST
- **标签**: qwen
- **摘要**: ## Overview  Qwen3.5 builds its M-RoPE cos/sin cache at 4x `max_position_embeddings`. That headroom exists for Qwen2.5-VL video positions, which Qwen3.5 does not produce, so this sizes the cache like plain RoPE unless video pruning is on.  ## Claims  - 4x smaller cos/sin cache for Qwen3.5: 128 → 32 …

### #59852 — [[ROCm][MoE] Enable MoRI FP4 dispatch for DeepSeek V4.1 a4w4](https://github.com/vllm-project/vllm/pull/59852)
- **作者**: Fangzhou-Ai  **时间**: 2026-10-03 13:27 CST
- **标签**: rocm, deepseek, DSv4.1
- **摘要**: ## Overview  Enable MoRI FP4 dispatch for DeepSeek V4.1 a4w4 (`VLLM_ROCM_USE_AITER_MOE_A4W4_DSV4=1`). **Stacked on #59596**, which adds MoRI FP4 dispatch for MXFP4 W4A4 checkpoints. Only the last commit is new here; this PR will be rebased once #59596 lands.  ## Claims  * DeepSeek V4.1 a4w4 under `-…

### #59851 — [[Core] Add startup snapshot capture to ordinary serving](https://github.com/vllm-project/vllm/pull/59851)
- **作者**: matteso1  **时间**: 2026-10-03 12:55 CST
- **标签**: documentation, frontend
- **摘要**: ## Overview  Allow ordinary `vllm serve` to prepare an initialized engine for external capture, then activate the original process or a restored copy through the normal OpenAI-compatible server. This gives capture systems a path through the standard serving entrypoint, including its authentication a…

### #59850 — [[CI] Drop duplicate bf16 skinny GEMM test from Kimi K3 B200 job](https://github.com/vllm-project/vllm/pull/59850)
- **作者**: khluu  **时间**: 2026-10-03 12:39 CST
- **标签**: ci/build, kimi, k3
- **摘要**: ## Purpose  `tests/kernels/test_bf16_skinny_gemm.py` runs twice on B200 for every build that triggers the Kimi K3 job:  - `:nvidia: (B200) Kimi K3` (`models_basic.yaml`) — added in #50089, before B200 had a root-level kernels job. - `:nvidia: (B200) Miscellaneous Kernels` (`kernels.yaml`) — the `ker…

### #59849 — [[Perf][DSv4.1] Register CPU-offloaded Engram tables at their exact size](https://github.com/vllm-project/vllm/pull/59849)
- **作者**: gitbisector  **时间**: 2026-10-03 12:10 CST
- **标签**: deepseek, DSv4.1
- **摘要**: ## Overview  Same fix as #59848, for DSv4.1. Default `cpu_offload` Engram tables use `pin_memory=True`, which rounds each table up to a power of two. Register them at exact size through the existing THP helper, without huge pages. Draft: GB10 only. cc @Juntian777: could you run your #59327 lookup be…

### #59848 — [[Bugfix][Qwen4Exp] Pin the offloaded PLE table at its exact size](https://github.com/vllm-project/vllm/pull/59848)
- **作者**: gitbisector  **时间**: 2026-10-03 12:09 CST
- **标签**: bug, ci/build, qwen
- **摘要**: ## Overview  `pin_memory=True` rounds the offloaded PLE table up to a power of two. Register it at its exact size instead.  ## Claims  - 47.7 GiB per-rank table: 65.2 → 47.8 GiB host memory (GB10). - TP2 Qwen4Exp on 2× DGX Spark now boots at `--gpu-memory-utilization 0.78` (OOM before). - ROCm/XPU u…

### #59847 — [[Bugfix] Register KV caches with the connector outside the runtime pool](https://github.com/vllm-project/vllm/pull/59847)
- **作者**: aoshen02  **时间**: 2026-10-03 10:57 CST
- **标签**: bug, cpu, mrv2
- **摘要**: ## Purpose  #59158 runs `model_runner.initialize_kv_cache` inside the cuMem `runtime` pool. That call also covers the KV connector's `register_kv_caches`.  A connector that allocates a GPU buffer there and registers it for RDMA now gets runtime-tagged memory. LMCache PD (`LMCacheConnectorV1`, `pd_bu…

### #59846 — [[Perf][DSv4.1] SM120: decode DeepSeek-V4.1's own KV records with FlashInfer's DSv4.1 sparse MLA](https://github.com/vllm-project/vllm/pull/59846)
- **作者**: lucifer1004  **时间**: 2026-10-03 10:41 CST
- **标签**: deepseek, nvidia, DSv4.1
- **摘要**: ## Overview  On SM120/SM121, DeepSeek-V4.1 writes the V4 KV record and decodes it with FlashInfer's V4 sparse MLA kernels, and `--kv-cache-dtype nvfp4_ds_mla` is rejected. FlashInfer's SM120 sparse MLA also decodes V4.1's own records (flashinfer-ai/flashinfer#5197). This PR writes the V4.1 record on…

### #59845 — [[Bugfix][Quantization] NVFP4 linear: keep each fused shard's global weight scale](https://github.com/vllm-project/vllm/pull/59845)
- **作者**: avifenesh  **时间**: 2026-10-03 10:07 CST
- **标签**: bug, quantization
- **摘要**: ## Purpose  `KNvfp4Static.process` runs a fused NVFP4 linear (`q/k/v`, `gate/up`) under the max of its shards' global weight scales (`weight_scale_2`), and only warns when they differ. Every smaller shard is then dequantized `gs_max / gs_shard` times too large. ModelOpt quantizes each projection wit…

### #59844 — [[Bugfix][DeepEP] Close GIN barrier data-visibility race in vendored DeepEP (per-context signals + NVLink fence)](https://github.com/vllm-project/vllm/pull/59844)
- **作者**: tlrmchlsmth  **时间**: 2026-10-03 10:04 CST
- **标签**: bug, ci/build
- **摘要**: ## Summary  Workaround PR (draft, dirty): vendor a small DeepEP patch in the vLLM image build until upstream ships a fix. **Not ready for review** — cluster validation is still in progress; opening as a draft to share the state.  DeepEP `d4f41e4e` (the commit we pin in `docker/Dockerfile` / `tools/e…
