# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-05 14:06 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期的动态摘要：

### 🐛 Issue 动态
1. **性能问题**：混合 Mamba2 模型在开启前缀缓存（`mamba_cache_mode="align"`）时，若无缓存命中会导致吞吐量下降 11-16%。原因包括每组 eager 块表操作、CUDA graphs 外部的对齐内核开销以及块边界预填充拆分（#60008）。
2. **批不变性 Bug**：TRITON_MLA 在分块预填充且开启 `VLLM_BATCH_INVARIANT=1` 时不具备批不变性，影响 DeepSeek-V2-Lite 和 DeepSeek-V3.1 等模型（#60007）。
3. **KV 卸载 Bug**：在 LHBNC 布局下，`OffloadingConnector` 在 CPU 命中时返回错误输出（#60004）。

### 🔀 PR 动态
**新特性与核心改进**
* **张量注册强化**：强制将所有持久化设备张量注册为 parameter 或 buffer，并在 CI 中强制检查，提升了 Sleep mode level 2 和 CUDA 图地址检查的稳定性（#60009）。
* **Rust 前端**：Rust 前端的渲染接口新增可选的源字符跨度返回，方便客户端对齐提示词文本（#60002）。

**重要修复**
* **量化修复**：修复 AWQ 模型在 `torch.compile` 下通过 Triton 路径时不具备批不变性的问题（#60013）。
* **分布式修复**：修复 `--disable-custom-all-reduce` 参数在 Hopper 和 Blackwell 架构上未能正确回退至 NCCL 的问题（#60011）。
* **内存泄漏修复**：修复首次 level-1 sleep 后，唤醒时未释放锁页 CPU 权重备份导致内存占用的问题（#60003）。
* **KV 卸载修复**：针对 LHBNC 布局导致 CPU 命中输出错误的问题，在启动时直接拒绝该布局以规避错误（#60005）。
* **指标修复**：修复多 API 服务（`--api-server-count > 1`）下，首次统计前调度器状态指标无数据的问题（#60001）。
* **基准测试修复**：清理服务退出后残留的 sweep workers 进程（#60010）。

**性能与文档**
* **ROCm 性能优化**：gfx950 平台感知设备长度的 top-k 拆分策略现支持 k=2048（#60012）。
* **文档完善**：重写预加载（`ipc_cache`指南，并补充 Docker 和 Kubernetes 环境下的使用方法（#60006）。

### 🚀 Release 动态
* 本次提供的数据中未包含 Release 相关信息。

---

## 🐛 Issues

### #60008 — [[Performance]: Hybrid Mamba prefix caching (`mamba_cache_mode="align"`) costs 11-16% throughput on Nemotron-3.5-Lightning with no cache hits: per-group eager block-table ops, per-step align kernels outside CUDA graphs, and block-boundary prefill splits](https://github.com/vllm-project/vllm/issues/60008)
- **作者**: aoshen02  **时间**: 2026-10-05 11:24 CST
- **标签**: performance
- **摘要**: ## Summary  Turning on prefix caching for a hybrid Mamba2 model forces `mamba_cache_mode="align"` (`vllm/model_executor/models/config.py:640-656`; since https://github.com/vllm-project/vllm/pull/58997 the only modes are `none` and `align`). On NVIDIA-Nemotron-3.5-Lightning-30B-A3B-NVFP4 (4x GB200, D…

### #60007 — [[Bug]: TRITON_MLA is not batch invariant under chunked prefill with `VLLM_BATCH_INVARIANT=1` (DeepSeek-V2-Lite, DeepSeek-V3.1)](https://github.com/vllm-project/vllm/issues/60007)
- **作者**: yifanFengg  **时间**: 2026-10-05 11:14 CST
- **标签**: bug, deepseek
- **摘要**: ### Your current environment  - vLLM `main` at `f590eb2448a51028f6044c6f60104ce0ef3189e9`, torch 2.13.0+cu130 - NVIDIA GB300 (SM 10.3), driver 580.167.08, CUDA toolkit 13.1 - Models: `deepseek-ai/DeepSeek-V2-Lite-Chat` (TP=1) and `deepseek-ai/DeepSeek-V3.1` (TP=4) - Attention backend: `TRITON_MLA`, …

### #60004 — [[Bug]: OffloadingConnector with LHBNC returns wrong output on CPU hits](https://github.com/vllm-project/vllm/issues/60004)
- **作者**: hyunnnchoi  **时间**: 2026-10-05 10:57 CST
- **摘要**: ### Your current environment  vLLM 0.30.0 (ced6857), torch 2.13.0+cu129, NVIDIA A100 80GB PCIe, driver 575.51.03. The code below is unchanged on main (0c16eee).  ### 🐛 Describe the bug  With `VLLM_KV_CACHE_LAYOUT=LHBNC` and the native CPU offload, a request whose prefix is loaded back from the CPU t…

## 🔀 Pull Requests

### #60013 — [[Bugfix][Quantization] Keep AWQ batch-invariant under torch.compile](https://github.com/vllm-project/vllm/pull/60013)
- **作者**: Manfredss  **时间**: 2026-10-05 13:09 CST
- **标签**: bug, torch.compile, ci/build, quantization
- **摘要**: ## Overview  Fix #59086: with `VLLM_BATCH_INVARIANT=1`, AWQ models served through the Triton AWQ path (`AutoAWQLinearMethod`) are not batch-invariant under torch.compile. `AutoAWQLinearMethod.apply` now calls `mm_batch_invariant` directly in batch-invariant mode, as `UnquantizedLinearMethod` already…

### #60012 — [[ROCm][Perf] gfx950 device length aware top-k split policy for k=2048](https://github.com/vllm-project/vllm/pull/60012)
- **作者**: LaelaZorana  **时间**: 2026-10-05 12:43 CST
- **标签**: rocm
- **摘要**: This fixes #55327.  The change generalizes `topKPerRowDecodeGfx950DeviceLengthAware` to work with different values of k, and adds `gfx950TopK2048ActiveBlocks`. The grid still only depends on `numRows`, so graph capture stays stable, and the number of splits is now determined on device per row, ensur…

### #60011 — [[Distributed] Make --disable-custom-all-reduce fall back to NCCL](https://github.com/vllm-project/vllm/pull/60011)
- **作者**: roy6n23  **时间**: 2026-10-05 12:22 CST
- **标签**: documentation, nvidia
- **摘要**: ## Overview  Fixes #59987. `--disable-custom-all-reduce` is documented as "Disable the custom all-reduce kernel and fall back to NCCL", but on Hopper and Blackwell it only removes the `CUSTOM` backend. The TP group keeps the FlashInfer and symmetric-memory all-reduce, and at the default `-O2` the Fl…

### #60010 — [[Bugfix][Benchmark] Clean up sweep workers after the server exits](https://github.com/vllm-project/vllm/pull/60010)
- **作者**: zupengwang  **时间**: 2026-10-05 12:16 CST
- **标签**: bug, performance
- **摘要**: ## Overview  Fix benchmark sweep cleanup when a server exits before its workers: terminate the surviving process group and reap the directly owned server process.  ## Claims  - `ServerProcess.stop()` terminates workers that remain in the original process group after its leader has exited and been re…

### #60009 — [[Model][Core] Register every persistent device tensor as a parameter or buffer](https://github.com/vllm-project/vllm/pull/60009)
- **作者**: aoshen02  **时间**: 2026-10-05 11:28 CST
- **标签**: rocm, intel-gpu, speculative-decoding, qwen, deepseek, cpu, nvidia, quantization, mistral, DSv4, dflash, kimi, k3, inkling, DSv4.1, pooling
- **摘要**: ## Overview  Every persistent device tensor becomes a parameter or a buffer of the module that owns it, and CI enforces it on every registered architecture. Sleep mode level 2, CUDA-graph address checks and weight reload only see parameters and buffers, so a tensor stored any other way comes back st…

### #60006 — [[Docs] Rewrite the preload (`ipc_cache`) guide and add Docker/Kubernetes usage](https://github.com/vllm-project/vllm/pull/60006)
- **作者**: gaby  **时间**: 2026-10-05 11:10 CST
- **标签**: documentation, frontend
- **摘要**: ## Overview  Rewrites `docs/features/preload.md` (the `vllm preload` / `--load-format ipc_cache` weight cache daemon) around the operator workflow and adds the missing Docker and Kubernetes guidance, plus pointers from the Docker deployment guide, the faster-startup list and the CLI reference.  ## C…

### #60005 — [[Bugfix][KV Offload] Reject LHBNC at startup](https://github.com/vllm-project/vllm/pull/60005)
- **作者**: hyunnnchoi  **时间**: 2026-10-05 10:57 CST
- **标签**: bug, kv-connector
- **摘要**: ## Purpose  Fixes #60004. With `VLLM_KV_CACHE_LAYOUT=LHBNC`, `OffloadingConnector` offloads only head 0 of layer 0 per block, so CPU-tier hits return wrong output; reject that layout at registration instead.  ### User case  1. A user sets `VLLM_KV_CACHE_LAYOUT=LHBNC` and enables native CPU offload. …

### #60003 — [[Bugfix] Free the pinned CPU weight backup on wake_up](https://github.com/vllm-project/vllm/pull/60003)
- **作者**: aoshen02  **时间**: 2026-10-05 10:50 CST
- **标签**: bug, nvidia
- **摘要**: ## Purpose  After the first level-1 sleep, every worker keeps a host copy of its weights in pinned memory for the rest of the engine's life, including while it is awake.  **Cause.** - `CuMemAllocator.sleep()` backs each offloaded allocation up into `torch.empty(..., pin_memory=True)`. That memory co…

### #60002 — [[Feature][Rust Frontend] Return token offsets from render endpoints](https://github.com/vllm-project/vllm/pull/60002)
- **作者**: ryouol  **时间**: 2026-10-05 10:47 CST
- **标签**: documentation, rust
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Add opt-in source-character spans to Rust completion and chat render responses, covering `return_token_offsets` in #44280. Clients can align prompt text with the server's tokens without reproducing its tokenizer and chat template.  ## Claims  - Return Unic…

### #60001 — [[Metrics] Export scheduler-state gauges as 0 before the first stats with --api-server-count > 1](https://github.com/vllm-project/vllm/pull/60001)
- **作者**: roy6n23  **时间**: 2026-10-05 10:26 CST
- **摘要**: ## Overview  Fixes #59988. With `--api-server-count > 1`, `vllm:num_requests_running`, `vllm:num_requests_waiting`, `vllm:num_requests_waiting_by_reason` and `vllm:kv_cache_usage_perc` have no samples until the first scheduler stats arrive. This PR sets them to 0 when `PrometheusStatLogger` creates …
