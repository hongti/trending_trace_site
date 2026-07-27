# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-27 09:07 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue
1. **无关模型启动崩溃 (#49920)**：`kernel_warmup()` 无条件导入 DiffusionGemma 的 `minimax_m3` 模块，导致 Triton JIT 解析 `index_topk` 内核失败，使得运行其他不相关模型时引擎直接崩溃。
2. **投机解码产生垃圾输出 (#49918)**：当 prompt 长度恰好为 `1 + num_speculative_tokens` 时，prefill 会被错误分类为 uniform decode。这导致 FULL spec-verify cudagraph 跳过 GDN/混合循环状态的写入，从而产生确定性的垃圾输出（影响 v0.25.1 和 v0.26.0 的所有投机解码方法）。

### 🔀 Pull Request
**核心与前端优化**
- **#49919 [Core]** 显式管理 worker 中的 torch CPU 线程：启动时限制线程数，服务时使用单线程。修复了 PyTorch 默认无视进程亲和性和 cgroup CPU 配额，导致多 worker 资源争抢的问题。
- **#49914 [Frontend]** 延迟初始化聊天媒体连接器：仅在请求包含多模态输入时才构建媒体连接器，避免了纯文本解析时重复的共享缓存初始化开销。

**Bug 修复**
- **#49917 [Bugfix]** 限制 FlashInfer 512 head size 仅在 Blackwell (SM100) trtllm-gen 架构上启用，修复了在其他硬件（如 SM86）上强制使用导致的崩溃。

**ROCm 硬件适配**
- **#49913** 修复 ROCm vllm_c RMSNorm 输出：当输入具有转置步幅时，确保分配连续的输出内存。
- **#49909** 修复 ReplaySSM 精度问题：ROCm 上改用 IEEE dot 精度（因 Triton 不支持 tf32x3，且原 TF32 默认值超出 fp32 容差）。

**CI 与测试优化**（本次集中优化了大量测试流程）
- **#49916 / #49915** 减少 ROCm 测试运行时间：通过拆分 MI300/MI355 测试分片、过滤不支持的融合 RMSNorm 参数、跳过不支持的 FP8 模块等方式提升效率。
- **#49910** 显式拆卸投机解码运行器：确保每个参数化引擎在下一个启动前关闭，避免资源冲突。
- **#49912** 初始化 DeepEP FP8 测试权重：用有界随机值替换依赖分配器的源权重，保留缓冲区复用。
- **#49911** ROCm 全局 GPU 内存清理改为按需启用：需设置 `VLLM_TEST_CLEAN_GPU_MEMORY=1`，避免误释放生命周期更长的 fixtures。

### 🚀 Release
- 近期无新版发布记录。

---

## 🐛 Issues

### #49920 — [[Bug]: DiffusionGemma - Unconditional minimax_m3 warmup import in kernel_warmup() crashes engine startup for unrelated models (Triton JIT fails to parse index_topk kernel)](https://github.com/vllm-project/vllm/issues/49920)
- **作者**: steveh250  **时间**: 2026-07-27 08:51 CST
- **标签**: bug
- **摘要**: ### Your current environment  # Current Environment:  ## Collection Collecting environment information... ==============================         System Info ============================== OS                           : Ubuntu 20.04.6 LTS (x86_64) GCC version                  : (Ubuntu 9.4.0-1ubuntu1…

### #49918 — [[Bug]: Prefill with prompt length == 1 + num_speculative_tokens misclassified as uniform decode → FULL spec-verify cudagraph skips GDN/hybrid recurrent-state write → deterministic garbage (any spec method incl. ngram; v0.25.1 & v0.26.0)](https://github.com/vllm-project/vllm/issues/49918)
- **作者**: avesed  **时间**: 2026-07-27 07:22 CST
- **标签**: quantization
- **摘要**: ### Summary  With speculative decoding enabled, a **prefill whose prompt tokenizes to exactly `1 + num_speculative_tokens` (K+1) tokens** is misclassified as a uniform spec-decode batch by `_is_uniform_decode` (`vllm/v1/worker/gpu_model_runner.py`, shape-only check) and dispatched through the captur…

## 🔀 Pull Requests

### #49919 — [[Core] Explicitly manage torch CPU threads in workers](https://github.com/vllm-project/vllm/pull/49919)
- **作者**: njhill  **时间**: 2026-07-27 08:13 CST
- **标签**: intel-gpu, v1
- **摘要**: Clamp for startup, single thread for serving.  torch defaults its intra-op thread pool to the host core count, ignoring process affinity, cgroup CPU quotas, and co-located worker processes. Any torch CPU op above the parallel_for grain size (32k elements) fans out across that pool, and the OpenMP wo…

### #49917 — [[Bugfix] Gate FlashInfer 512 head size on trtllm-gen (SM100)](https://github.com/vllm-project/vllm/pull/49917)
- **作者**: YM2132  **时间**: 2026-07-27 07:06 CST
- **标签**: bug, v1, nvidia
- **摘要**: ## Purpose  #38822 added head size 512 to FlashInfer's `get_supported_head_sizes()` arch-blind, but the 512 kernels are Blackwell trtllm-gen only. On other hardware (e.g. SM86), explicitly forcing `FLASHINFER` for Gemma4's 512-head-dim layers passes backend selection and then crashes at kernel dispa…

### #49916 — [[CI][ROCm] Reduce V1 attention test runtime](https://github.com/vllm-project/vllm/pull/49916)
- **作者**: AndreasKaratzas  **时间**: 2026-07-27 06:20 CST
- **标签**: rocm, ci/build, v1
- **摘要**: - Remove V1 attention from MI250 and split its MI300 and MI355 coverage into two pytest shards. - Classify unsupported MLA prefill combinations during collection and include the ROCm AITER FA prefill adapter.  https://buildkite.com/vllm/amd-ci/builds/11266/list?sid=019f9c06-99b2-4356-8a8f-9cf9e905a1…

### #49915 — [[CI][ROCm] Reduce kernel test runtime](https://github.com/vllm-project/vllm/pull/49915)
- **作者**: AndreasKaratzas  **时间**: 2026-07-27 06:20 CST
- **标签**: rocm, ci/build
- **摘要**: - Filter unsupported fused RMSNorm parameters during collection and skip the unsupported FP8 scaling module before GPU setup. - Shard MI300 kernel-core coverage three ways and increase Transformers Processing parallelism from four to eight.  https://buildkite.com/vllm/amd-ci/builds/11266/list?sid=01…

### #49914 — [[Frontend] Lazily initialize chat media connectors](https://github.com/vllm-project/vllm/pull/49914)
- **作者**: AndreasKaratzas  **时间**: 2026-07-27 06:20 CST
- **标签**: frontend
- **摘要**: - Defer media connector construction until a request contains multimodal input. - Avoid repeated shared-cache initialization for text-only chat parsing while covering synchronous and asynchronous paths.  https://buildkite.com/vllm/amd-ci/builds/11266/list?sid=019f9c06-996d-4bf8-8282-5a716ec70a7d&tab…

### #49913 — [[ROCm] Make vllm_c RMSNorm output contiguous](https://github.com/vllm-project/vllm/pull/49913)
- **作者**: AndreasKaratzas  **时间**: 2026-07-27 06:19 CST
- **标签**: rocm
- **摘要**: - Allocate contiguous ROCm vllm_c RMSNorm output when the input has transposed strides. - Cover the singleton-batch VibeVoice layout with an IR regression case.  https://buildkite.com/vllm/amd-ci/builds/11266/list?tab=output&jid=019f9c06-9a43-42d8-a972-5eac4371b598#L3072  No open PR covers this ROCm…

### #49912 — [[CI] Initialize DeepEP FP8 test weights](https://github.com/vllm-project/vllm/pull/49912)
- **作者**: AndreasKaratzas  **时间**: 2026-07-27 06:19 CST
- **摘要**: - Replace allocator-dependent FP8 source weights with small bounded random values. - Allocate through current_platform.device_type while preserving the buffer-reuse and tolerance fixes from #46758.  https://buildkite.com/vllm/amd-ci/builds/11266/list?sid=019f9c06-9989-45bd-9353-9a349d807bdc&tab=outp…

### #49911 — [[CI][ROCm] Keep global GPU memory cleanup opt-in](https://github.com/vllm-project/vllm/pull/49911)
- **作者**: AndreasKaratzas  **时间**: 2026-07-27 06:19 CST
- **标签**: rocm
- **摘要**: - Require VLLM_TEST_CLEAN_GPU_MEMORY=1 before enabling the global cleanup fixture; function-scoped cleanup cannot release longer-lived fixtures. - Keep the targeted Qwen-VL and Nixl teardown from #49242.  https://buildkite.com/vllm/amd-ci/builds/11263/list?tab=output&jid=019f9aea-ada5-4312-af96-9b60…

### #49910 — [[CI] Explicitly tear down speculative decode runners](https://github.com/vllm-project/vllm/pull/49910)
- **作者**: AndreasKaratzas  **时间**: 2026-07-27 06:19 CST
- **标签**: speculative-decoding, v1
- **摘要**: - Run the speculative max-length cases through VllmRunner contexts so each parametrized engine shuts down before the next starts. - Preserve the direct LLM settings explicitly.  https://buildkite.com/vllm/amd-ci/builds/11254/list?tab=output&jid=019f966b-1fa2-46d1-93b3-a3ae8c49ce21#L1704  No open PR …

### #49909 — [[ROCm] Use IEEE dot precision for ReplaySSM](https://github.com/vllm-project/vllm/pull/49909)
- **作者**: AndreasKaratzas  **时间**: 2026-07-27 06:19 CST
- **标签**: rocm, ci/build
- **摘要**: - Use IEEE dot precision on ROCm, where Triton does not support tf32x3 and its TF32 default exceeds the fp32 tolerance. - Keep tf32x3 on CUDA and move the ROCm Mamba kernel group from MI250 to MI300.  https://buildkite.com/vllm/amd-ci/builds/11254/list?sid=019f966b-1e70-4263-a40f-36dcf55d9601&tab=ou…
