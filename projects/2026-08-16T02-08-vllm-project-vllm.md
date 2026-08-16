# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-16 10:08 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
近期问题高度集中于 **MTP 推测解码** 和 **睡眠模式** 相关的 Bug：
1. **MTP 推测解码兼容性问题**：
   - **Qwen3.5 多模态误导警告**：对 Qwen3.5 系列模型使用 MTP 推测解码时，会输出关于草稿模型的误导性多模态警告日志 (#52481)。
   - **多卡加载失败**：`qwen3_5_mtp` 在张量并行（tensor-parallel-size >= 2）下因草稿模型权重形状不匹配无法加载 (#52480)。
   - **计算架构崩溃**：在 RTX 5090 (sm_120) 上，Qwen3.8-27B 混合 GDN 模型配合 `turboquant_*` KV 缓存使用 MTP 推测解码时，会出现重复生成崩溃 (#52475)。
2. **Sleep-mode Level 2 唤醒缺陷**：服务器从 Level 2 睡眠模式唤醒时，未能重新加载推测解码的草稿权重，导致静默性能下降（解码速度约慢 2 倍，接受率降至 0） (#52479)。

---

### 🔀 PR 动态
修复与性能优化并行，重点关注推理后端、量化及分布式稳定性：
1. **核心特性与性能优化**：
   - **GLM-5.2 TurboQuant 稀疏后端**：为 GLM-5.2 新增稀疏 MLA 路径，支持 packed 4-bit latent KV 存储、融合稀疏解码/预填充及 GLM-4 MoE MTP (#52472)。
   - **Triton 内核性能提升**：优化 `reshape-and-cache` 内核，根据 tile 大小动态缩放 `num_warps`，避免小 tile 下多余的 warp 空转 (#52468)。
2. **关键 Bug 修复**：
   - **V1 多模态**：修复了同一步骤内过期驱逐通知导致的编码器缓存未命中问题 (#52482)。
   - **推测解码+结构化输出**：修复 DSpark 自适应验证在零预算时的崩溃，改为依赖 GPU 最终的 per-request logit 计数来驱动 grammar mask (#52477)。
   - **KV Transfer**：修复 Mooncake connector 在请求被抢占时可能存储损坏 KV 数据的问题 (#52470)。
   - **Mamba & Quark**：避免 causal conv1d 元数据指针的对齐特化导致的错误 (#52476)；修复 Quark 配置映射中结构化量化列表丢失的问题 (#52474)。
   - **ROCm**：修复 MI300 睡眠唤醒重映射大量显存时偶发的 GPU 内存错误 (#52478)。
3. **工程与规范**：
   - 修复 dp supervisor 未继承 vLLM uvicorn 访问日志配置的问题 (#52473)。
   - 规范错误类型：Cohere 请求验证改用 `VLLMValidationError` 替代原生 `ValueError` (#52467)。

---

### 🚀 Release 动态
近期无新增版本发布信息。

---

## 🐛 Issues

### #52481 — [[Bug]: MTP speculative decoding on Qwen3.5-family models logs misleading multimodal warnings ("treated as multimodal but has no registered multimodal processor")](https://github.com/vllm-project/vllm/issues/52481)
- **作者**: jianliao  **时间**: 2026-08-16 10:00 CST
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information... uv is set ==============================         System Info ============================== OS                           : Ubuntu 24.04.4 LTS (x8…

### #52480 — [[Bug]: qwen3_5_mtp fails to load at tensor-parallel-size >= 2 (drafter weight shape mismatch)](https://github.com/vllm-project/vllm/issues/52480)
- **作者**: sahaq  **时间**: 2026-08-16 09:59 CST
- **标签**: quantization
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information... uv is set ==============================         System Info ============================== OS                           : Ubuntu 26.04 LTS (x86_…

### #52479 — [[Bug]: Sleep-mode Level 2 wake does not reload speculative-decoding draft weights — silent ~2x decode slowdown, acceptance drops to 0](https://github.com/vllm-project/vllm/issues/52479)
- **作者**: Suppressor72  **时间**: 2026-08-16 09:53 CST
- **标签**: quantization
- **摘要**: # [Bug]: Sleep-mode Level 2 wake does not reload speculative-decoding draft weights — silent ~2x decode slowdown, acceptance drops to 0  ## Summary  After waking a vLLM server from Level 2 sleep using the documented staged sequence (`wake_up(tags=["weights"])` → `collective_rpc("reload_weights")` → …

### #52475 — [[Bug]: MTP speculative decoding produces repetition collapse with turboquant_* KV cache on sm120 (Qwen3.8-27B hybrid GDN)](https://github.com/vllm-project/vllm/issues/52475)
- **作者**: mechramc  **时间**: 2026-08-16 06:36 CST
- **标签**: quantization
- **摘要**: ### Summary  On an RTX 5090 (sm_120), `Qwen3.8-27B-NVFP4` (`model_type: qwen3_5`, hybrid Gated DeltaNet) serves cleanly with MTP speculative decoding when the KV cache is `fp8`, and degenerates into repetition collapse when the KV cache is `turboquant_4bit_nc` or `turboquant_3bit_nc`.  The failure i…

## 🔀 Pull Requests

### #52482 — [[Bugfix][V1][Multimodal] Ignore stale same-step encoder cache evictions](https://github.com/vllm-project/vllm/pull/52482)
- **作者**: gty111  **时间**: 2026-08-16 10:02 CST
- **标签**: bug
- **摘要**: ## Purpose  Fix an encoder cache miss caused by a stale same-step eviction notification.  An entry can be evicted early in `schedule()` and added to `self.freed`, then allocated again later in the same scheduling pass. Previously, the stale eviction was still sent to the model runner, which removed …

### #52478 — [[ROCm][CI] Fix intermittent GPU memory fault during sleep-mode wake-up](https://github.com/vllm-project/vllm/pull/52478)
- **作者**: AndreasKaratzas  **时间**: 2026-08-16 09:19 CST
- **标签**: rocm, mrv2
- **摘要**: ROCm sleep-mode wake-up can intermittently abort the EngineCore on MI300 while remapping roughly 172 GiB, as seen in [build 12090](https://buildkite.com/vllm/amd-ci/builds/12090/list?sid=01a004a6-9a94-48ba-8be1-20f23544b8e0&tab=output) and the earlier signature-matched [build 11862](https://buildkit…

### #52477 — [[Bugfix][Spec Decode][Structured Output] Drive grammar masks from GPU logit counts](https://github.com/vllm-project/vllm/pull/52477)
- **作者**: LucasWilkinson  **时间**: 2026-08-16 07:34 CST
- **标签**: bug, structured-output, speculative-decoding, mrv2
- **摘要**: ## Purpose  Fix the DSpark adaptive-verification zero-budget crash with structured outputs by making the grammar-mask application depend on the GPU's finalized per-request logit counts.  The scheduler generates masks from scheduled drafts, but adaptive verification can compact the active logits to b…

### #52476 — [[Bugfix][Mamba] Avoid causal conv1d metadata alignment specialization](https://github.com/vllm-project/vllm/pull/52476)
- **作者**: sylvesterkaczmarek  **时间**: 2026-08-16 06:37 CST
- **标签**: bug
- **摘要**: ## Purpose  Fixes #52413.  The causal-conv1d Triton kernels currently opt only `num_cache_lines` out of alignment specialization. Several other arguments are scalar metadata pointers whose addresses can vary with runtime slicing and batching even though the kernels only use scalar loads from them. T…

### #52474 — [[Bugfix][Quark] Preserve structured quantization config lists](https://github.com/vllm-project/vllm/pull/52474)
- **作者**: sylvesterkaczmarek  **时间**: 2026-08-16 06:25 CST
- **标签**: bug, quantization
- **摘要**: ## Purpose  Fixes #52454.  `QuarkConfig.apply_vllm_mapper()` currently sends every list-valued quantization config entry through `WeightsMapper.apply_list()`, which assumes every element is a string module name. Newer Quark configs can contain structured lists (for example dictionaries describing qu…

### #52473 — [using existing uvicorn configuration for dp supervisor](https://github.com/vllm-project/vllm/pull/52473)
- **作者**: Gregory-Pereira  **时间**: 2026-08-16 06:05 CST
- **标签**: frontend
- **摘要**: ## Purpose  I was puzzled to see logs showing up with `--disable-access-log-for-endpoints=/health,/metrics,/v1/models,/readyz` on. Turns out dp supervisor was not inheriting vLLMs Uvicorn access log  example:  ```log (APIServer_DP3 pid=286) INFO:     Started server process [286] (APIServer_DP3 pid=2…

### #52472 — [[Attention][MLA] Add GLM-5.2 TurboQuant sparse backend with DCP/MTP](https://github.com/vllm-project/vllm/pull/52472)
- **作者**: ketor  **时间**: 2026-08-16 04:56 CST
- **标签**: performance, speculative-decoding, deepseek, nvidia, quantization
- **摘要**: ## Summary  - extends the TurboQuant MLA work in #41803 with the GLM-5.2 sparse MLA path on current main - adds packed 4-bit latent KV storage, fused sparse decode, sparse prefill, and GLM-4 MoE MTP plumbing - adds DCP/MTP/PP correctness fixes, canonical DCP interleave handling, CUDA-graph-safe work…

### #52470 — [fix(kv_transfer): abort async store jobs on preemption in Mooncake connector](https://github.com/vllm-project/vllm/pull/52470)
- **作者**: patrickswedish  **时间**: 2026-08-16 04:17 CST
- **标签**: kv-connector
- **摘要**: ### Problem  When a request is preempted under GPU memory pressure, `MooncakeStoreConnector` could store corrupted KV data that does not belong to the request's key hashes.  1. During a step, the worker enqueues store jobs onto `KVCacheStoreSendingThread`'s queue. 2. In a subsequent step (or during …

### #52468 — [[Perf] Scale num_warps with tile size in Triton reshape-and-cache kernels](https://github.com/vllm-project/vllm/pull/52468)
- **作者**: samuelkim7  **时间**: 2026-08-16 03:54 CST
- **摘要**: ## Purpose  Both CUDA launch sites in `triton_reshape_and_cache_flash.py` hardcode `num_warps = 16`, i.e. 512 threads, regardless of tile size. When the tile is smaller than that, the surplus warps hold thread slots without doing work, which limits how many blocks stay resident on an SM.  `_reshape_…

### #52467 — [[Misc] Use VLLMValidationError in Cohere request validation](https://github.com/vllm-project/vllm/pull/52467)
- **作者**: frank-suwen  **时间**: 2026-08-16 03:36 CST
- **标签**: frontend, cohere
- **摘要**: ## Purpose  Part of #48227.  Like #51753 and #51931, this is an independent file-level Step 5 migration.  Migrate the three client-facing validators in `CohereChatV2Request` from raw `ValueError` to `VLLMValidationError`:  - empty `model` - negative `max_tokens` - empty `messages`  The existing vali…
