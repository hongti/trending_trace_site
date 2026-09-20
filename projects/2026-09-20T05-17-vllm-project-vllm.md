# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-20 13:17 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
- **GLM-5.3-Flash-NVFP4 在 Hopper GPU 上的运行 Bug** (#57748)：用户报告在 Rocky Linux 9.8 环境下，于 Hopper GPU 上运行 GLM-5.3-Flash-NVFP4 模型时出现错误。

---

### 🚀 PR 动态

**1. Bug 修复**
- **Mamba 状态修复** (#57751)：修复了 `DecodeBenchConnector` 错误填充未分配的 Mamba blocks 的问题，避免不相关请求的状态被重置。
- **KV Offload 记录修复** (#57746)：修复了 GPU 本地前缀重用未被记录到 CPU offload 缓存过期策略中的问题。
- **CUDA 捕获修复** (#57745)：修复了在 PP=1（流水线并行大小为1）时，捕获 CUDA Graph 过程中 `intermediate_tensors` 未正确传递给模型的问题。
- **Outlines 后端错误修复** (#57743)：修复了 `outlines` 结构化输出后端在语法结束后拒绝 EOS 导致请求报 HTTP 500 错误的问题，并在验证阶段拒绝 `json_object`。

**2. 硬件适配与优化（ROCm / XPU）**
- **ROCm 量化调整** (#57750)：由于 RDNA3 架构不支持低精度（fp8/fp4）WMMA，禁用了 AITER attention 的 query 量化，以避免隐式类型转换带来的性能损耗。
- **ROCm 构建修复** (#57744)：在 vLLM 版本检测阶段过滤 crate tags，修复了 Docker 构建中的版本识别问题。
- **XPU Attention 支持** (#57747)：在 XPU 平台引入 `vllm-xpu-kernels` 的新特性，支持在 FlashAttention 2 中启用 per-sequence causal masking（逐序列因果掩码）。

**3. 功能增强与实验性特性**
- **投机解码 警告** (#57749)：当批次中的最大序列长度加上 draft query tokens 超出 draft context 限制时，新增警告提示（将跳过该批次的 draft 生成）。
- **多模态架构实验** (#57752)：[实验性 PR] 尝试将媒体（图像/视频/音频）解码从 chat 解析层移至多模态处理器（mm processor）内部进行，并将其置于处理器缓存之后。

**4. 文档完善**
- **EPD 文档补充** (#57742)：为 Disaggregated Encoder (EPD) 添加了 `ECMooncakeConnector` 的使用示例，包含可直接运行的 1E 级集成脚本指引。

---

### 📦 Release 动态
- 本次提供的动态中暂无新版本发布信息。

---

## 🐛 Issues

### #57748 — [[Bug]: Running GLM-5.3-Flash-NVFP4 on Hopper GPUs](https://github.com/vllm-project/vllm/issues/57748)
- **作者**: shahizat  **时间**: 2026-09-20 12:17 CST
- **标签**: bug, quantization, glm
- **摘要**: ### Your current environment  ``` `Collecting environment `information...` uv is set ==============================         System Info ============================== OS                           : Rocky Linux 9.8 (Blue Onyx) (x86_64) GCC version                  : (GCC) 11.5.0 20240719 (Red Hat 11.…

## 🔀 Pull Requests

### #57752 — [[Don't merge][Experiment] Move MediaIO decode to MM preprocessing](https://github.com/vllm-project/vllm/pull/57752)
- **作者**: Isotr0py  **时间**: 2026-09-20 13:00 CST
- **标签**: documentation, performance, frontend, needs-rebase, multi-modality
- **摘要**: ## Purpose  Move media **decoding** (image/video/audio) out of the chat parsing layer and into the mm processor, behind the processor cache. Downloads stay where they are (async, concurrent, in the renderer); what crosses the boundary is now a `LazyMedia` handle (encoded bytes + decode callable) ins…

### #57751 — [[Bugfix] Fill only allocated Mamba blocks in DecodeBenchConnector](https://github.com/vllm-project/vllm/pull/57751)
- **作者**: milesial  **时间**: 2026-09-20 12:55 CST
- **标签**: bug, kv-connector
- **摘要**: ## Purpose  DecodeBenchConnector fills every element of list/tuple state tensors for each fake-prefill request. Mamba state tensors have a leading block dimension, so this resets unrelated requests' state and writes more memory than needed.  Use the existing block-fill helper for each state tensor a…

### #57750 — [[ROCm] Disable AITER attention query quantization on RDNA3](https://github.com/vllm-project/vllm/pull/57750)
- **作者**: amd-xavierwang  **时间**: 2026-09-20 12:52 CST
- **标签**: rocm, quantization
- **摘要**: ## Purpose RDNA3 has no low precision(fp8, fp4) WMMA support, so matmul input needs to be implicitly casted back to fp16/bf16. Within attention, they are (QxK) and (PxV). However, `ROCM_AITER_UNIFIED_ATTN` always enable Query quantization:  https://github.com/vllm-project/vllm/blob/1c3ef2ad4cabfbc2e…

### #57749 — [[Spec Decode] Warn when the batch exceeds the draft context limit](https://github.com/vllm-project/vllm/pull/57749)
- **作者**: milesial  **时间**: 2026-09-20 12:51 CST
- **摘要**: ## Purpose  Warn once when the maximum sequence length in a batch, plus draft query tokens, exceeds the draft context limit. The warning explains that draft generation is skipped for the entire batch and identifies speculative_config.max_model_len as the relevant setting.  This changes logging only.…

### #57747 — [[XPU][Attention] Enable per-sequence causal masking with FA2](https://github.com/vllm-project/vllm/pull/57747)
- **作者**: gc-fu  **时间**: 2026-09-20 11:59 CST
- **标签**: intel-gpu
- **摘要**: ## Dependency and coordination  This PR depends on vllm-project/vllm-xpu-kernels#606 (or a release containing that change), which adds the optional `dynamic_causal` FA2 argument.  Open PR #45774 overlaps the DiffusionGemma XPU startup portion but targets an older tree and currently conflicts with `m…

### #57746 — [[Bugfix][KV Offload] Record GPU-local prefix reuse](https://github.com/vllm-project/vllm/pull/57746)
- **作者**: Alex-ai-future  **时间**: 2026-09-20 11:55 CST
- **标签**: bug, kv-connector
- **摘要**: # [Bugfix][KV Offload] Record GPU-local prefix reuse in offload recency  ## Summary  When a request reuses a GPU-resident prefix, the scheduler does not call `prepare_load()`. The CPU offload cache therefore misses this access and may evict a hot CPU copy.  This change records GPU-local prefix reuse…

### #57745 — [[Model] Pass intermediate_tensors to the model when capturing CUDA gr…](https://github.com/vllm-project/vllm/pull/57745)
- **作者**: Mi-Jiazhi  **时间**: 2026-09-20 11:49 CST
- **标签**: nvidia, mrv2
- **摘要**: The V2 execution path always puts "intermediate_tensors" into model_inputs (gpu/model_runner.py), but the capture path only adds it on non-first pipeline ranks. With PP=1 the key is therefore never present, and the capture calls model(**model_inputs) without it.  Models that declare intermediate_ten…

### #57744 — [[ROCm][Build] Filter crate tags from vLLM version detection](https://github.com/vllm-project/vllm/pull/57744)
- **作者**: reidliu41  **时间**: 2026-09-20 11:35 CST
- **标签**: rocm, ci/build
- **摘要**: ## Purpose    The ROCm, ROCk, and gfx1250 Dockerfiles calculate the source version in   standalone `vllm-version` stages. These stages used an unfiltered   `setuptools_scm.get_version()` call, allowing a closer `proto-v*` tag to   take precedence over the latest vLLM release tag.    The resulting ve…

### #57743 — [[Bugfix] Accept EOS after grammar finish in outlines backend; reject json_object at validation](https://github.com/vllm-project/vllm/pull/57743)
- **作者**: SIDDARTHAREDDY8  **时间**: 2026-09-20 11:33 CST
- **标签**: bug, structured-output
- **摘要**: ## Purpose  Fixes #57723 — with the `outlines` structured-output backend, every `choice` / `regex` / `json` request that completes returns HTTP 500.  Root cause: once the `outlines_core` guide reaches an accept state, `Guide.accepts_tokens()` rejects *every* token — including the EOS that the guide'…

### #57742 — [[Docs] Add ECMooncakeConnector usage example for EPD](https://github.com/vllm-project/vllm/pull/57742)
- **作者**: jiangkuaixue123  **时间**: 2026-09-20 11:22 CST
- **标签**: documentation, ready
- **摘要**: ## Purpose  The disaggregated encoder usage section only describes ExampleConnector. Add an ECMooncakeConnector example that links to the existing full-pipeline integration script, shows a runnable 1E + 1PD command, and explains the default model, baseline comparison, and TCP/RDMA selection.  Duplic…
