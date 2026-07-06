# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-06 01:33 UTC

## AI 总结

以下是 **vllm-project/vllm** 仓库最近动态的简洁摘要：

### 🚀 Release（版本发布）
近期无新版本发布。

### 🐛 Issue（问题反馈）
近期无重点 Issue 列出。

### ✨ Pull Request（代码合并）

本次 PR 动态主要集中在**关键 Bug 修复**、**硬件/模型适配扩展**以及**新特性引入**：

**1. 重要 Bug 修复**
- **修复 Triton 解码注意力 int32 溢出** (#47671)：在 MLA 架构（c_kv 576维，PAGE_SIZE 480）下，页步长约 276k，导致页编号超过约 7.7k 时溢出为负偏移，引发 CUDA 非法内存访问。此 PR 修复了该溢出问题。
- **修复 Logits 处理器 FQCN 解析崩溃** (#47670)：防止在 FQCN 字符串缺少冒号或包含多个冒号时，Python 解包引发的 `ValueError` 崩溃。
- **升级 FlashInfer 至 0.6.14 修复流挂起** (#47669：解决 multi-CTA radix top-k sampler 中内核尾声重置软件屏障导致同伴 CTA 死锁永久自旋的问题。
- **回滚 torch.Event 替换** (#47668：回滚了先前将 `torch.cuda.Event` 替换为 `torch.Event` 的 PR #47140，因引发相关问题。
- **修复 ROCm aiter 探测崩溃** (#47667：在 disaggregated-serving 启动期间，防止 aiter 包重入导入引发的 `KeyError` 崩溃，使 fused-qk-rmsnorm 能力探测具备容错性。
- **修复 Responses API 音频输入报错** (#47662：修复了 `/v1/responses` 接口处理音频输入时抛出 422 `ValidationError` 的问题，使其与 `/v1/chat/completions` 行为一致。

**2. 新特性与功能改进**
- **Whisper 原生词级时间戳** (#47664：为 Whisper 模型新增可选的词级时间戳功能（基于交叉注意力 + DTW算法），通过 `response_format=verbose_json` 和 `timestamp_granularities[]=word` 启用。
- **Minimax-M3 支持 Triton fp8 索引缓存** (#47665：在非 SM100 平台（如 SM120 Blackwell）的 Triton indexer 上启用 fp8 (e4m3) 紧缩侧缓存，此前该平台会抛出 `NotImplementedError`。
- **KV Offload 读写指标分离** (#47666：将原本合并的 CPU KV 缓存使用率指标拆分为写入使用率（`write_usage_perc`）和读取使用率（`read_usage_perc`），便于更精细地监控读写压力。

**3. 底层架构与编译探索**
- **新增 stock torch.compile 替代实现** (#47663：作为 PR #46423 的平行替代方案，提供了一种基于原生 `torch.compile` (aot_compile 驱动) 的实现，供社区对比评估不同编译后端的性能。

---

## 🔀 Pull Requests

### #47671 — [[Bugfix] Fix int32 overflow in triton_decode_attention page offsets](https://github.com/vllm-project/vllm/pull/47671)
- **作者**: ivanium  **时间**: 2026-07-06 01:23 UTC
- **标签**: bug, v1
- **摘要**: ## Purpose  `_fwd_kernel_stage1` / `_fwd_grouped_kernel_stage1` load `kv_page_number` as int32 and multiply it by the KV buffer page stride in int32. With MLA c_kv (576 dims) and PAGE_SIZE 480 the page stride is ~276k, so page numbers above ~7.7k overflow to negative offsets → CUDA illegal memory ac…

### #47670 — [fix(sample): validate logits processor FQCN to prevent unpacking crashes](https://github.com/vllm-project/vllm/pull/47670)
- **作者**: aoright  **时间**: 2026-07-06 01:17 UTC
- **标签**: v1
- **摘要**: ### Description Fixes #44156.  In `_load_logitsprocs_by_fqcns()`, the FQCN string is split by `:` and unpacked directly: ```python module_path, qualname = logitproc.split(":") ``` If the FQCN does not contain a `:` or contains multiple colons, Python raises a `ValueError` during unpacking (e.g. `not…

### #47669 — [Bump flashinfer version to 0.6.14](https://github.com/vllm-project/vllm/pull/47669)
- **作者**: AmeenP  **时间**: 2026-07-06 00:59 UTC
- **标签**: ci/build, nvidia
- **摘要**: ## Purpose  Pick up flashinfer-ai/flashinfer#3615, which fixes the multi-CTA radix top-k sampler stream hang (flashinfer-ai/flashinfer#3610): the kernel's epilogue resets the software barrier's arrival counter with no sync against peer CTAs still polling it, so a peer can spin forever and permanentl…

### #47668 — [Revert "[Platform] Replace `torch.cuda.Event` with `torch.Event` (#47140)"](https://github.com/vllm-project/vllm/pull/47668)
- **作者**: jikunshang  **时间**: 2026-07-06 00:50 UTC
- **标签**: performance, intel-gpu, speculative-decoding, v1, cpu, kv-connector, nvidia
- **摘要**: This reverts commit e840f0d3f5d26803e907d64a84be521d9568900a.     ## Purpose see comments here: https://github.com/vllm-project/vllm/pull/47081#issuecomment-4874967250   ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ …

### #47667 — [[ROCm] Make aiter fused-qk-rmsnorm capability probe crash-safe](https://github.com/vllm-project/vllm/pull/47667)
- **作者**: raviguptaamd  **时间**: 2026-07-06 00:33 UTC
- **标签**: rocm
- **摘要**: ## Purpose `check_aiter_fused_qk_rmsnorm()` is a capability probe that tries to import `aiter.ops.fused_qk_norm_rope_cache_quant`. During disaggregated-serving bringup the probe can re-enter the `aiter` package while it is still mid-import; importlib then raises `KeyError('aiter')` from `_get_parent…

### #47666 — [[KV Offload] Split cpu_cache_usage_perc into write/read usage gauges](https://github.com/vllm-project/vllm/pull/47666)
- **作者**: Srinivasoo7  **时间**: 2026-07-06 00:28 UTC
- **标签**: v1
- **摘要**: ## Summary  `vllm:kv_offload_cpu_cache_usage_perc` currently reports one combined gauge for CPU KV-cache space pinned by any in-flight transfer. This PR splits it into two additional gauges so write pressure and read pressure can be told apart: - `vllm:kv_offload_cpu_cache_write_usage_perc` : fracti…

### #47665 — [[Minimax-M3] Enable fp8 index cache on the Triton indexer (non-SM100)](https://github.com/vllm-project/vllm/pull/47665)
- **作者**: jarrelscy  **时间**: 2026-07-05 23:58 UTC
- **摘要**: ## Purpose  Follow-up to #45892, which added fp8 (e4m3) index-K side caches routed through the `fmha_sm100` MSA score path on SM100. On every other platform (e.g. SM120 consumer Blackwell), `select_indexer_impl_cls` falls back to the Triton indexer, which still raises `NotImplementedError` for fp8. …

### #47664 — [[Feature][Whisper] Native word-level timestamps (cross-attention + DTW)](https://github.com/vllm-project/vllm/pull/47664)
- **作者**: yusufani  **时间**: 2026-07-05 23:16 UTC
- **标签**: frontend, v1
- **摘要**: # [Feature][Whisper] Native word-level timestamps (cross-attention + DTW)  ## Purpose  Adds opt-in **word-level timestamps** for Whisper. With `response_format=verbose_json` and `timestamp_granularities[]=word`, the response now fills `words[]` with per-word `start`/`end`, computed inside vLLM from …

### #47663 — [[compile] stock aot_compile driver: alternate to eval-frame stock torch.compile (#46423 sibling)](https://github.com/vllm-project/vllm/pull/47663)
- **作者**: bobrenjc93  **时间**: 2026-07-05 21:36 UTC
- **标签**: new-model, v1, gpt-oss, nvidia
- **摘要**: ## What this does  This is an **alternate implementation** of #46423, for side-by-side comparison. #46423 migrates GPT-OSS off `VllmBackend` onto **stock `torch.compile`** driven by the lazy eval-frame `nn.Module.compile()` (Dynamo installs a frame-evaluation hook and compiles on the first forward).…

### #47662 — [[Frontend][Responses] Accept input_audio content parts in user messages (#47659)](https://github.com/vllm-project/vllm/pull/47662)
- **作者**: shadowmodder  **时间**: 2026-07-05 20:47 UTC
- **标签**: frontend
- **摘要**: ## Summary  Audio inputs that work on `/v1/chat/completions` fail on `/v1/responses` with an opaque 422 `ValidationError` (#47659), even though the payload is identical.  **Root cause**: The openai SDK's `ResponseInputMessageContentListParam` — the type used for `EasyInputMessageParam.content` — onl…
