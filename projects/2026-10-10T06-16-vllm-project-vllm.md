# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-10 14:16 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
* **#60964 安装问题**：用户报告了在使用 `pip install` 安装 vLLM 时遇到的环境问题，目前正处于收集环境信息与排查阶段。

### 🔧 PR 动态

**1. 新特性与功能**
* **视觉 Token 剪枝 (#60963)**：为图像输入引入了基于 RATE（冗余感知）方法的视觉 token 剪枝功能。用户可通过启动参数 `--image-pruning-rate` 开启，以降低多模态推理的计算开销。
* **CPU 可变大小通信 (#60958)**：为 CPU 通信器新增了 `all_gatherv()` 支持，利用 PyTorch 分布式集合通信实现了 CPU 端可变大小的 all-gather 操作。

**2. 性能优化**
* **MoE 内核提速 (#60959)**：针对 Hopper 架构上的 BF16/FP16 批次不变推理，调优了固定的 Triton MoE 配置。实测可将内核中位延迟降低约 23%。

**3. Bug 修复与改进**
* **前端与缓存逻辑**：
  - 修复流式解析器路径上缺失 logprobs 的问题 (#60962)。
  - 修复当结合 `prompt_logprob_token_ids` 时，限制前缀缓存重用范围以避免缓存命中导致跳过请求的 prompt 分数计算 (#60960)。
  - 修复请求对象重复入队后，在移除时仅删除首个匹配项导致其仍残留在运行队列中的 bug (#60956)。
* **内核与模型架构**：
  - 为 decode 阶段的 top-k 添加运行时宽度校验，拒绝超出 `[1, 8192]` 范围的请求以防 GPU 启动或整数截断错误 (#60961)。
  - 修复 MLA 注意力机制中缓存格式规范化写回共享配置的问题，将其限制在局部层中 (#60957)。
  - 恢复 AXK1 模型配置中被意外丢弃的默认参数（如 `scoring_func`, `topk_method` 等）(#60954)。
* **测试修复**：修复 FlashInfer prefill 测试未按参考实现应用因果掩码的问题 (#60955)。

### 🚀 Release 动态
* 近期无新版本发布动态。

---

## 🐛 Issues

### #60964 — [[Installation]:](https://github.com/vllm-project/vllm/issues/60964)
- **作者**: shahineyoung21  **时间**: 2026-10-10 14:08 CST
- **标签**: installation
- **摘要**: ### Your current environment  ```text The output of `python collect_env.py` ```   ### How you are installing vllm  ```sh pip install -vvv vllm ```   ### Before submitting a new issue...  - [x] Make sure you already searched for relevant issues, and asked the chatbot living at the bottom right corner…

## 🔀 Pull Requests

### #60963 — [[Feature] Add visual token pruning for image inputs](https://github.com/vllm-project/vllm/pull/60963)
- **作者**: garrygale  **时间**: 2026-10-10 13:33 CST
- **标签**: documentation, multi-modality, qwen
- **摘要**: ## Overview  Adds vision token pruning for image inputs with RATE (redundency-aware token pruning) method. Toggled with --image-pruning-rate in launch cli arg.  ## Claims  * Adds image token pruning * Implement the image token pruning with a method that can be applied to both image and video token p…

### #60962 — [[Bugfix][Frontend] Resolve and attach logprobs on streaming parser derender path](https://github.com/vllm-project/vllm/pull/60962)
- **作者**: nicholaskh-ai  **时间**: 2026-10-10 13:29 CST
- **标签**: bug, documentation
- **摘要**: ## Purpose Fixes the missing logprobs on the streaming parser derender path noted in RFC #47161 and PR #55029 ("That parsed path does not resolve logprobs yet; when it does, mirror the generate chat streaming path...").  Previously, `_derender_chat_stream_parsed()` in `OnlineDerenderer` dropped inco…

### #60961 — [[Bugfix][Kernel] Validate decode top-k runtime width](https://github.com/vllm-project/vllm/pull/60961)
- **作者**: arcusbuilds  **时间**: 2026-10-10 13:25 CST
- **标签**: bug
- **摘要**: ## Overview Fixes #60829: reject decode `topK` outside `[1, 8192]` before GPU launch or integer narrowing.  ## Claims Adds a native guard, five regression cases, and documents the limit; accepted-width computation is unchanged.  ## Validation - Rebuilt sampler on RTX 2000 Ada: `run_repository_tests.…

### #60960 — [[Bugfix] Bound prefix cache reuse by the first requested prompt score](https://github.com/vllm-project/vllm/pull/60960)
- **作者**: Sunt-ing  **时间**: 2026-10-10 13:18 CST
- **标签**: bug, scheduler, kv-cache-manager
- **摘要**: ## Overview  When `prompt_logprob_token_ids` is combined with explicit `skip_reading_prefix_cache=False`, limit local prefix reuse to rows before `prompt_logprob_start`. Otherwise a cache hit can skip rows whose scores were requested.  ## Claims  Preserve all requested scores while allowing cached, …

### #60959 — [[Perf][MoE] Tune fixed unquantized MoE tiles for batch-invariant Hopper inference](https://github.com/vllm-project/vllm/pull/60959)
- **作者**: Sunt-ing  **时间**: 2026-10-10 12:56 CST
- **标签**: quantization
- **摘要**: ## Overview  Tune the fixed Triton MoE configuration for BF16/FP16 batch-invariant inference on Hopper, using N=128/K=64 with four warps and three stages.  ## Claims  - Reduce median kernel latency by 23.6–38.5% across the 24 measured BF16/FP16 shape/token-count cases on H200. - Reduce warm-run elap…

### #60958 — [[CPU] Add variable-sized all-gather support to CPU communicator](https://github.com/vllm-project/vllm/pull/60958)
- **作者**: justinsortland  **时间**: 2026-10-10 12:35 CST
- **标签**: cpu
- **摘要**: ## Overview  Adds `all_gatherv()` support to `CpuCommunicator` using PyTorch distributed collectives.  Partially addresses #57837. This PR implements CPU variable-sized all-gather; `reduce_scatterv()` and end-to-end Wide Expert Parallelism support remain out of scope.  ## Claims  - Supports variable…

### #60957 — [[Bugfix][MLA] Keep ds_mla cache-format canonicalization layer-local](https://github.com/vllm-project/vllm/pull/60957)
- **作者**: ianmage  **时间**: 2026-10-10 12:32 CST
- **标签**: bug
- **摘要**: ## Purpose  Fixes #48406.  `MLAAttention.__init__` canonicalizes the sparse-MLA KV cache format for FLASHMLA_SPARSE / SM120 (`fp8` -> `fp8_ds_mla`, `nvfp4` kept) and wrote the result back into the **shared global** `cache_config.cache_dtype`. Spec-decode drafters (DSpark / DFlash family) reuse that …

### #60956 — [[Bugfix] drop every copy when removing one scheduled request](https://github.com/vllm-project/vllm/pull/60956)
- **作者**: Chessing234  **时间**: 2026-10-10 12:22 CST
- **标签**: bug, scheduler
- **摘要**: ## Summary - `remove_all` used `list.remove` for a one-item set, so only the first match was deleted. - A request object queued twice could stay in `running` after it was supposed to be removed. - The single-item path now drops every equal element in place. The multi-item path still returns a new li…

### #60955 — [[Bugfix][Test] Run the FlashInfer prefill tests causally, like their reference](https://github.com/vllm-project/vllm/pull/60955)
- **作者**: Shakhtar-Sankur  **时间**: 2026-10-10 12:20 CST
- **标签**: bug, nvidia
- **摘要**: ## Overview  `test_flashinfer_prefill_with_paged_kv` and `test_flashinfer_prefill_with_paged_fp8_kv` compare FlashInfer against `ref_paged_attn`, which applies a causal mask, but they call `BatchPrefillWithPagedKVCacheWrapper.plan()` without `causal=True`, and `plan()` defaults to `causal=False`. Th…

### #60954 — [[Bugfix][Model] Restore AXK1 config defaults dropped with the vendored config](https://github.com/vllm-project/vllm/pull/60954)
- **作者**: AndreasKaratzas  **时间**: 2026-10-10 12:19 CST
- **标签**: bug, ready
- **摘要**: - Default `scoring_func` to `sigmoid` and `topk_method` to `noaux_tc` in `AXK1MoE` - Default `moe_layer_freq` to 1 in `AXK1DecoderLayer._is_layer_sparse` - Drop the asserts that these fields exist on the config  https://github.com/vllm-project/vllm/pull/60623 replaced vLLM's vendored `AXK1Config` wi…
