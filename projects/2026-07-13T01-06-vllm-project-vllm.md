# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-13 09:06 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 📌 Issue 概览
1. **新特性请求**：呼吁支持 OpenMOSS Transcribe-Diarize 的 `response_format=diarized_json` 格式，以在兼容 OpenAI 的语音转写端点中实现说话人分离输出 (#48443)。
2. **性能/执行缺陷**：在非平衡 Pipeline stages 场景下，旧版 non-SPMD `RayGPUExecutorAsync` 的 Local PP Rank0 会出现严重停滞，执行耗时异常增加 10-30 倍 (#48441)。
3. **性能/缓存缺陷**：针对 hybrid-SWA 模型（如 Gemma-4-31B），在多会话轮询调度且池占用率约 25% 时，跨请求的 prefix-cache 复用率会完全崩溃降至零 (#48435)。

---

### 🔧 PR 概览

**🛠️ 缺陷修复**
- **WSL 循环导入**：修复了 WSL 环境下由 `pin_memory` 警告日志引发的循环导入问题 (#48444)。
- **MLA 模型启动崩溃**：修复了 MLA 模型在启用 `--kv-cache-dtype fp8` 时因形状不匹配导致的启动崩溃 (#48439)。
- **Marlin 权重重载**：在权重重载期间保留 Marlin 运行时张量存储，避免了图耦合翻转测试中的死锁问题 (#48438)。
- **LoRA 校验**：在 `PEFTHelper` 中增加校验，确保 LoRA rank (`r`) 为正数，防止除零或缩放因子计算错误 (#48437)。
- **KV Connector 异步加载**：修复了异步 KV 加载的 block 预留被普通本地请求过早驱逐的问题，确保异步任务完成前 block 不被回收 (#48433)。
- **异构注意力头支持**：修复了 Transformers backend 中默认所有层注意力头数一致的 bug，现可正确支持每层异构的注意力头数 (#48432)。
- **ROCm 性能回退**：重新禁用 ROCm 上的 CUDA graph 内存 profiling（恢复 `is_cuda()` 门控检查），修复了由此导致的稳态性能退化 (#48440)。

**⚡ 性能优化**
- **零拷贝序列化**：在 `shm_broadcast MessageQueue` 中实现了 `torch.Tensor` 的零拷贝 pickle 序列化，移除了原先无效的(out-of-band)死代码路径，减少数据拷贝开销 (#48442)。
- **KV Offload 内存节省**：绕过 PyTorch `CUDACachingHostAllocator` 将 pinned memory 分配向上取整到 2 的幂次的机制（例如原先 33GB 会分配 64GB），大幅减少了 KV offload 时的 CPU 内存浪费 (#48436)。
- **KV Offload 批处理优化**：在调用 `cuMemcpyBatchAsync` 前合并连续的 descriptor 运行，显著降低了小数据块场景下的 CPU 提交开销（每 descriptor 约 0.5µs） (#48434)。

---

### 🚀 Release 概览
本次提供的动态数据中**无新版本 Release 发布**记录。

---

## 🐛 Issues

### #48443 — [[Feature]: Support response_format=diarized_json for OpenMOSS Transcribe-Diarize](https://github.com/vllm-project/vllm/issues/48443)
- **作者**: wskr00  **时间**: 2026-07-13 08:51 CST
- **标签**: feature request
- **摘要**: ### 🚀 The feature, motivation and pitch  OpenMOSS Transcribe-Diarize already exposes speaker diarization through the OpenAI-compatible "/v1/audio/transcriptions" endpoint. However, diarization is currently returned using the Whisper-style "verbose_json" response format.  It would be useful for vLLM …

### #48441 — [[Bug]: Local PP Rank0 stalls in legacy non-SPMD Ray AsyncLLMEngine under imbalanced pipeline stages](https://github.com/vllm-project/vllm/issues/48441)
- **作者**: ApacheWang  **时间**: 2026-07-13 07:49 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>Environment (manually collected because this is a customized Jetson build)</summary>  ```text OS: Ubuntu 22.04 (aarch64, Jetson Linux) GPU: NVIDIA Jetson AGX Orin 32GB, SM 8.7, one GPU per node Nodes: 4 nodes connected by 1 GbE Ethernet Python: 3.10 P…

### #48435 — [[Bug/Perf]: hybrid-SWA prefix caching collapses to zero for ALL requests in multi-session round-robin at ~25% pool occupancy (Gemma-4-31B; eager-freed SWA tails recycled tail-first)](https://github.com/vllm-project/vllm/issues/48435)
- **作者**: claudematttest-dev  **时间**: 2026-07-13 05:09 CST
- **摘要**: ## Summary  For hybrid-SWA models (tested: Gemma-4-31B, 10 full-attention / 50 sliding-window(1024) layers), **cross-request prefix-cache reuse collapses to exactly zero for every request in a multi-session round-robin workload once the combined working set exceeds a sharp threshold far below pool c…

## 🔀 Pull Requests

### #48444 — [[Bugfix] Fix WSL circular import from pin_memory warning_once](https://github.com/vllm-project/vllm/pull/48444)
- **作者**: AlejandroParedesLT  **时间**: 2026-07-13 08:55 CST
- **标签**: bug, ci/build, nvidia
- **摘要**: ## Purpose  Fixes #48397.  PR #46511 changed `Platform.is_pin_memory_available()` (interface.py) and `CudaPlatformBase.is_pin_memory_available()` (cuda.py) to log the WSL pinned-memory-disabled message via `logger.warning_once()` instead of `logger.warning()`.  `warning_once()` dispatches through `_…

### #48442 — [[Perf] Zero-copy torch.Tensor pickling in shm_broadcast MessageQueue](https://github.com/vllm-project/vllm/pull/48442)
- **作者**: mrn3088  **时间**: 2026-07-13 08:01 CST
- **摘要**: ## Purpose  `torch.Tensor.__reduce_ex__` copies tensor bytes into the pickle stream and never emits a `PickleBuffer`, so the out-of-band buffer path added to `MessageQueue.enqueue` in #26961 is dead code for tensors. Large CPU tensors broadcast from the engine core to workers — e.g. `prompt_embeds` …

### #48440 — [Re-disable CUDA graph memory profiling on ROCm](https://github.com/vllm-project/vllm/pull/48440)
- **作者**: Rohan138  **时间**: 2026-07-13 07:41 CST
- **标签**: rocm, v1, nvidia
- **摘要**: Reverts the `vllm/v1/worker/gpu_worker.py` change from #47366, restoring the pre-#47366 `is_cuda()` gate so ROCm skips cudagraph memory profiling. On ROCm the profiling capture regresses steady-state decode throughput. #47366's test-side fix (`wait_for_rocm_memory_to_settle`) is retained.  AI assist…

### #48439 — [[Bugfix][MLA] Fix fp8 KV cache crash on MLA models at startup](https://github.com/vllm-project/vllm/pull/48439)
- **作者**: jahnavi-yelamanchi  **时间**: 2026-07-13 07:39 CST
- **标签**: bug, v1
- **摘要**: ## Purpose  Fixes #48405  Serving an MLA model with `--kv-cache-dtype fp8` crashes at startup:  ``` RuntimeError: shape '[512, 64, 576]' is invalid for input of size 21495808 ```  The KV cache is allocated in one layout but read back in another. The allocation uses 656 bytes per token (the packed fp…

### #48438 — [[Bugfix] Preserve Marlin runtime tensor storage across weight reload](https://github.com/vllm-project/vllm/pull/48438)
- **作者**: RyanClark2k  **时间**: 2026-07-13 06:00 CST
- **标签**: bug
- **摘要**: The fix is verified at three levels: a red/green CPU regression test (in this PR), live-engine GPU validation on an RTX 4090 (pointer identity across reload, a graph-coupling flip test, and a livelock demonstration on unfixed main), and a pointer-identity audit of the sibling Marlin paths. Full meth…

### #48437 — [fix(lora): validate LoRA rank is positive in PEFTHelper](https://github.com/vllm-project/vllm/pull/48437)
- **作者**: ErenAta16  **时间**: 2026-07-13 05:46 CST
- **摘要**: ## Problem  `PEFTHelper.__post_init__` computes `vllm_lora_scaling_factor` as `lora_alpha / r` (or `lora_alpha / sqrt(r)` for rsLoRA) without checking that `r` is positive. `r` comes straight from an adapter's `adapter_config.json`, parsed before `validate_legal()` -- the class's own intended valida…

### #48436 — [[KV Offload] Bypass power-of-2 rounding in KV offload CPU pinned allocation](https://github.com/vllm-project/vllm/pull/48436)
- **作者**: procr1337  **时间**: 2026-07-13 05:27 CST
- **标签**: v1
- **摘要**: ## Purpose  PyTorch's CUDACachingHostAllocator rounds every pin_memory=True allocation up to the next power of 2 (e.g. 33 GiB -> 64 GiB). In CPUOffloadingWorker this caused each TP worker to allocate far more host memory than requested when kv_offloading_size was not an exact power of 2 per rank.  F…

### #48434 — [[KV Offload] Coalesce contiguous descriptor runs before batch submission](https://github.com/vllm-project/vllm/pull/48434)
- **作者**: Etelis  **时间**: 2026-07-13 05:06 CST
- **标签**: performance, v1
- **摘要**: `cuMemcpyBatchAsync` costs **~0.5 µs of CPU per descriptor at submission** — more than the copy itself below 32 KiB (6.33 ms enqueue vs 1.94 ms execution at N=12,288, 8 KiB pages). The offloading connector submits one descriptor per (GPU block × layer), so per-layer-KV models (gpt-oss, Gemma-3, Llam…

### #48433 — [[Bugfix][KV Connector] Preserve async load reservations](https://github.com/vllm-project/vllm/pull/48433)
- **作者**: GirasoleY  **时间**: 2026-07-13 04:59 CST
- **标签**: bug, ready, v1, kv-connector
- **摘要**: ## Summary  Async KV loads reserve the blocks they still need to finish, but the scheduler only honored that reservation when admitting another async load. Ordinary local admissions and subsequent running chunks could consume the reserved headroom. The full-sequence admission check also ignored `res…

### #48432 — [[Bugfix] Support heterogeneous attention head counts per layer in Transformers backend](https://github.com/vllm-project/vllm/pull/48432)
- **作者**: melcheikh  **时间**: 2026-07-13 04:49 CST
- **标签**: bug
- **摘要**: ### Summary Fixes #44494.  This PR fixes a bug in the generic Transformers model executor backend (used via `--model-impl transformers`) where all attention instances are assumed to have a uniform number of heads and KV heads across all layers.   For models with heterogeneous layer configurations, s…
