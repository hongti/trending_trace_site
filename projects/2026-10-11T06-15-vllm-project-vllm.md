# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-11 14:15 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期的动态摘要：

### 🐛 Issue 动态
本期共有 4 个关键问题被提出，主要涉及显存溢出（OOM）、服务关闭异常及部分硬件环境的兼容问题：
1. **NVIDIA vGPU 环境下 CPU KV 卸载崩溃**（#61052）：在 Proxmox VE 上的 H200 vGPU 环境中，CPU KV offload 触发 `cuMemcpyBatchAsync` 错误并导致 Xid 31 崩溃。
2. **无状态 DP 组形成卡死**（#61049）：在端口池路径下，当 leader 遇到端口占用（EADDRINUSE）静默重绑定时，成员仍在旧端口重试直至超时。
3. **Gemma 4 编码器 OOM**（#61045）：CUDA graph 捕获时会按 `max_frames_per_batch` 分配整图行，导致在 `--max-num-batched-tokens 4096` 下占用高达 38.5 GiB 显存并引发 OOM。
4. **关闭期间结构化输出请求报错**（#61043）：服务关闭时若有结构化输出请求，会因后端 `XgrammarBackend` 缺失 `compiler` 属性而失败。

### 🔧 PR 动态
本期共有 10 个合并请求，涵盖多项重要 Bug 修复与性能优化：

**重要修复（OOM 与内存管理）：**
- **限制 CoW 块拷贝瞬态分配**（#61051）：修复了在接近内存上限时，KV 块原地拷贝一次性实例化过多行导致的 OOM 问题。
- **提前创建 cuBLAS 句柄**（#61044）：在小显存 GPU（如 H200 35GB MIG 切片）上进行内存分析前预先创建 cuBLAS 句柄，避免引擎启动失败。

**模型与后端支持优化：**
- **DeepSeek-V4.1 相关修复与优化**：
  - 修复了使用 FlashInfer MegaMoE 后端在长提示词下 decoder-replay CUDA graph 的 dtype 不匹配崩溃（#61041）。
  - 优化 decode indexer，采用基于长度的候选块选择，减少静态缓冲区宽度，提升性能（#61039）。
- **TP 复制 Mamba 状态共享 KV 缓存组**（#61048）：允许 Qwen4Exp 的 Mamba 状态层共享分片 KV 缓存组，避免独占整个内存池造成浪费。
- **KV 卸载保留前缀**（#61050）：在 CPU-offload 遭遇 LRU 压力时，跨存储批次和 KV 组保留正在运行的请求前缀，提升前缀复用率。
- **修复结构化输出关闭报错**（#61047）：在销毁后端前排空编译任务，修复了 Issue #61043 中的崩溃问题。
- **Triton 上的 WNA16 MoE 调度修复**（#61038）：在 batch invariance 下保留传统的 Triton 调度逻辑，避免错误调用 CUDA 内核。

**前端与功能改进：**
- **优化 Scale-out 端点 404 提示**（#61042）：当未启用 `--enable-scale-out` 时，对 `/inference/v1/generate` 的 404 响应增加明确的配置引导提示。
- **Sleep 模式保留 Drafter 权重**（#61046）：扩展 `sleep_preserve_parameter_names`，在 level-2 sleep 仅更新目标模型权重时，保留当前 drafter 的私有权重。

### 🚀 Release 动态
本期数据范围内无新的 Release 版本发布。

---

## 🐛 Issues

### #61052 — [[Bug]: CPU KV offload crashes with Xid 31 on NVIDIA vGPU — cuMemcpyBatchAsync faults even with a single valid descriptor](https://github.com/vllm-project/vllm/issues/61052)
- **作者**: jerry102039  **时间**: 2026-10-11 14:01 CST
- **标签**: bug, nvidia
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information... uv is set ==============================         System Info ============================== OS                           : Ubuntu 26.04.1 LTS (x8…

### #61049 — [Stateless DP group formation (port-pool path): leader silently rebinds the pooled rendezvous port after EADDRINUSE while members retry the stale port for the full timeout](https://github.com/vllm-project/vllm/issues/61049)
- **作者**: amchen2310  **时间**: 2026-10-11 13:46 CST
- **摘要**: # [Bug]: Stateless DP group formation (port-pool path): leader silently rebinds the pooled rendezvous port after EADDRINUSE while members retry the stale port for the full timeout  ## Current behavior  When forming a stateless DP group in external load balancer mode (each instance runs its own API s…

### #61045 — [[Bug]: Gemma 4 encoder CUDA graph capture allocates `max_frames_per_batch` full-image rows per budget (38.5 GiB with `--max-num-batched-tokens 4096`) and OOMs](https://github.com/vllm-project/vllm/issues/61045)
- **作者**: toan-tq  **时间**: 2026-10-11 11:02 CST
- **摘要**: ### Your current environment  - vLLM `0.31.1rc1.dev159+g4274ae956` (main @ 4274ae956c), precompiled wheel. The code involved is unchanged on current main (3c062d391d); the links below point to 276fbcff27. - torch 2.13.0+cu132, torchvision 0.28.0, transformers 5.19.0, CUDA 13.2, driver 595.91.07 - 1×…

### #61043 — [[Bug]: structured outputs: a request during shutdown fails with "'XgrammarBackend' object has no attribute 'compiler'"](https://github.com/vllm-project/vllm/issues/61043)
- **作者**: BRANTD-veta  **时间**: 2026-10-11 09:48 CST
- **标签**: structured-output
- **摘要**: ## Your current environment  ```text vLLM Version: 0.27.1 (observed), 0.28.0 and main @ 276fbcff2 affected by inspection Engine: V1, single engine process, one GPU (NVIDIA GB10, aarch64) Model: google/gemma-4-E2B-it Structured outputs backend: default `auto` → xgrammar Served with: vllm serve, OpenA…

## 🔀 Pull Requests

### #61051 — [[Bugfix] Bound CoW block-copy transient to avoid OOM near the memory limit](https://github.com/vllm-project/vllm/pull/61051)
- **作者**: michael-gpu  **时间**: 2026-10-11 13:56 CST
- **标签**: bug
- **摘要**: ## Overview  `copy_kv_cache_blocks_inplace` (`vllm/v1/worker/utils.py`) ends with `blocks[dst] = blocks[src]`, whose RHS gather materializes `len(src)` rows in one allocation. Rows can be tens of MiB (mamba state blocks), so a large copy-on-write wave — many concurrent requests sharing one prefix — …

### #61050 — [[Bugfix][KV Offload] Preserve running request prefixes across LRU store batches](https://github.com/vllm-project/vllm/pull/61050)
- **作者**: ztysth  **时间**: 2026-10-11 13:51 CST
- **标签**: bug
- **摘要**: ## Overview  Keep positioned prefixes useful under CPU-offload LRU pressure while their request is still running, across store batches and KV groups. Related to [this observation in #58804](https://github.com/vllm-project/vllm/issues/58804#issuecomment-6098736505); this does not claim to explain or …

### #61048 — [[Core] Let TP-replicated Mamba state share a sharded state KV cache group](https://github.com/vllm-project/vllm/pull/61048)
- **作者**: kimseunghyun-kr  **时间**: 2026-10-11 12:45 CST
- **标签**: kv-cache-manager
- **摘要**: ## Overview  In the packed (block-outermost) hybrid KV layout, a TP-replicated Mamba state layer (Qwen4Exp's PLE short-conv state) always gets its own KV cache group. That group then takes whole pool blocks for a few hundred KB of real state: in align mode, 2 + `num_speculative_tokens` blocks per ru…

### #61047 — [[Bugfix][Structured Output] Drain compilation tasks before releasing the backend](https://github.com/vllm-project/vllm/pull/61047)
- **作者**: DimensionSTP  **时间**: 2026-10-11 12:43 CST
- **标签**: bug, structured-output
- **摘要**: ## Overview  Fixes #61043. Stop structured-output thread pools before destroying the backend, so an in-flight grammar compilation cannot access a released xgrammar compiler.  ## Claims  - Cancel queued compilation tasks and wait for running tasks before releasing the backend. - Close the optional ma…

### #61046 — [[Sleep] Retain draft-private weights with existing parameter selector](https://github.com/vllm-project/vllm/pull/61046)
- **作者**: aoshen02  **时间**: 2026-10-11 11:21 CST
- **标签**: documentation, ready
- **摘要**: ## Overview  Extend `sleep_preserve_parameter_names` to draft-private weights, so level-2 sleep can retain the current drafter across target-only weight updates. Related: #52479 and the existing retention API in #57891.  ## Claims  - Select draft-private parameters with `--sleep-preserve-parameter-n…

### #61044 — [Create cuBLAS handles before memory profiling to avoid OOM](https://github.com/vllm-project/vllm/pull/61044)
- **作者**: laithsakka  **时间**: 2026-10-11 09:48 CST
- **摘要**: Fix https://github.com/pytorch/pytorch/issues/199944 Purpose On small GPUs (e.g. an H200 35GB MIG slice), engine startup can fail during the memory profiling run with: ``` RuntimeError: CUDA error: CUBLAS_STATUS_ALLOC_FAILED when calling `cublasCreate(handle)` ``` This was seen in tests/v1/e2e/gener…

### #61042 — [[Frontend] Explain disabled scale-out endpoints with opt-in hints](https://github.com/vllm-project/vllm/pull/61042)
- **作者**: aoshen02  **时间**: 2026-10-11 09:30 CST
- **标签**: documentation, frontend, ready, ci/build, rust
- **摘要**: ## Purpose  After #54579 and #55176, existing callers of `/inference/v1/generate` receive an unexplained 404 unless the server is started with `--enable-scale-out`. This makes a deliberate configuration gate look like a missing or removed API.  Keep the existing opt-in default and dedicated render/t…

### #61041 — [[Bugfix][DSv4.1] Match decoder-replay input_ids dtype for FlashInfer MegaMoE](https://github.com/vllm-project/vllm/pull/61041)
- **作者**: wangshangsam  **时间**: 2026-10-11 09:21 CST
- **标签**: bug, deepseek, nvidia, DSv4.1
- **摘要**: ## Overview  Fixes #61040. DeepSeek-V4.1 served with `--moe-backend flashinfer_moe_ep_cutedsl` or `flashinfer_moe_ep_mega_deep_gemm` crashes in the decoder-replay CUDA graph once it serves long prompts or batched prefills (e.g. GSM8K at concurrency 128) (`RuntimeError: Expected out tensor to have dt…

### #61039 — [[Perf][DSv4.1] Length-aware candidate block selection in the decode indexer](https://github.com/vllm-project/vllm/pull/61039)
- **作者**: Juntian777  **时间**: 2026-10-11 08:52 CST
- **标签**: DSv4.1
- **摘要**: ## Purpose  DeepSeek-V4.1's decode indexer writes its logits into a `[rows, max_model_len]` buffer, a width that must stay static for CUDA graphs (1M with the default `max_model_len`). Each row only holds its own context, but `select_candidate_blocks` scored every block of the full width and then ra…

### #61038 — [[Bugfix] Keep legacy WNA16 MoE dispatch on Triton under batch invariance](https://github.com/vllm-project/vllm/pull/61038)
- **作者**: yifanFengg  **时间**: 2026-10-11 08:51 CST
- **标签**: bug
- **摘要**: ## Purpose  `should_moe_wna16_use_cuda` selects the CUDA WNA16 MoE kernel (`moe_wna16_gemm`) when there are at most six tokens per expert and the Triton `fused_moe_kernel_gptq_awq` otherwise. For the legacy `fused_experts` path with `use_int4_w4a16` under `VLLM_BATCH_INVARIANT=1` this means a reques…
