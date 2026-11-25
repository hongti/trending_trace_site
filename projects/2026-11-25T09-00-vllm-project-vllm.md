# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-25 17:00 CST

## AI 总结

# vLLM 仓库动态摘要

## 📌 Release
本周期内无新版本发布。

## 📌 Issue（1 项）
- **#69854 [RFC] Mamba2 混合模型低秩循环状态压缩（SPARE）**：作者提出针对 Mamba2 混合语言模型的循环状态压缩方案。尽管循环状态大小与上下文长度无关，但在多并发请求场景下仍可能成为瓶颈，因此建议引入低秩压缩以降低显存占用。属于探索性讨论，尚未落地。

## 📌 Pull Request（11 项）

### 🔒 安全相关（3 项）
- **#69852**：在 chat 请求预处理阶段校验用户传入的 HF tokenizer / chat template 选项，防止与服务端默认值合并时引入未受控参数。
- **#69847**：将 Dynamic NTK RoPE 缓存按模型配置上下文长度预分配，避免旋转位置编码缓存越界或过度增长。
- **#69846**：在 Pillow 服务端加载路径上强化解码图像尺寸校验，直接拒绝 BLP、ICO、AVIF 等潜在风险格式，防止异常大图造成的内存问题。

### ⚡ 性能优化（2 项）
- **#69849**：DeepSeek V4.1 多模态推理中跳过冗余的 `Image.convert("RGB")` 调用——对于已是 RGB 的输入，Pillow 会做整缓冲区拷贝，跳过后可减少不必要开销。
- **#69843**：新增可选 KV load/recompute 策略，基于双阈值窗口决定是从外部加载 KV cache 还是重新计算，与现有 `skip_reading_external_kv` 在同一决策点生效。

### 🧮 量化（1 项）
- **#69851**：在 `Fp8LinearMethod` 与 `Fp8PerTensorOnlineLinearMethod` 上暴露 `input_quant_key`，使外部传入的 `QuantizedActivation` 可直接复用，跳过 kernel 内的重复输入量化。

### 🦀 Rust 前端（1 项）
- **#69848**：在 Rust 前端的 `/metrics` 端点导出 `OffloadingConnector` 的 KV offload 相关指标（`vllm:kv_offload_*`、`tiering_*`、`config_info`），与 Python 前端对齐，便于监控 KV 卸载行为。

### 🖥️ 硬件 / XPU（1 项）
- **#69845**：实现基于 Level Zero Sysman（`pyzes`）的 `ZesXPUPlatform`，镜像 CUDA 平台的 NVML 实现，并启用自动 NUMA 绑定检测，提升 Intel XPU 部署的吞吐与稳定性。

### 📚 文档（2 项）
- **#69853**：将 MoRIIO worker 发现监听器在单机示例中限定到 loopback，并补充多节点部署应使用的私有接口说明。
- **#69850**：修复 pre-check 脚本中指向 Read The Docs 的失效链接。

### 🛠️ 其他
- **#69850 / #69853** 见上文文档分类。

---

### 总体观察
- 本周期无新 Release，但 PR 活跃，且**安全加固**占比显著（3 项），覆盖 tokenizer 选项校验、RoPE 缓存 sizing、图像解码限制——显示项目在持续收紧输入边界。
- 性能与硬件支持并行推进：DeepSeek V4.1 多模态路径、FP8 量化路径、XPU NUMA 绑定均有实质改进。
- Rust 前端持续向 Python 前端的功能对齐靠拢（本次为 metrics 导出）。
- Mamba2 相关的 RFC 表明社区在探索 SSM 混合架构下的显存优化新方向。

---

## 🐛 Issues

### #69854 — [[RFC]: Low-Rank Recurrent State Compression for Mamba2 Hybrid Models (SPARE)](https://github.com/vllm-project/vllm/issues/69854)
- **作者**: tbn5pj  **时间**: 2026-11-25 16:58 CST
- **标签**: RFC
- **摘要**: ### Motivation.  Mamba2-based hybrid language models maintain a persistent recurrent state for each active request. Although its size is independent of context length, the recurrent state can become a significant memory-capacity and memory-bandwidth bottleneck during high-concurrency inference.  As …

## 🔀 Pull Requests

### #69853 — [[Docs] Scope MoRIIO worker discovery to the worker network](https://github.com/vllm-project/vllm/pull/69853)
- **作者**: jperezdealgaba  **时间**: 2026-11-25 16:54 CST
- **标签**: documentation
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Scope the MoRIIO worker discovery listener to loopback in the single-host examples and document the private interface to use for multi-node deployments.  ## Claims  - The Docker and native Router commands use `127.0.0.1:36367`, matching the existing worker…

### #69852 — [[Security] Reserve HF tokenizer options in chat requests](https://github.com/vllm-project/vllm/pull/69852)
- **作者**: jperezdealgaba  **时间**: 2026-11-25 16:53 CST
- **标签**: documentation, frontend, pooling
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Validate request-provided chat template options before the Hugging Face renderer merges them with server defaults. The validation runs in shared chat preprocessing and in chat-shaped pooling preprocessing.  ## Claims  - Reserve tokenizer controls for the s…

### #69851 — [[Quantization][FP8] Expose input_quant_key on Fp8LinearMethod and Fp8PerTensorOnlineLinearMethod](https://github.com/vllm-project/vllm/pull/69851)
- **作者**: lnkerr  **时间**: 2026-11-25 16:44 CST
- **标签**: quantization
- **摘要**: ## Purpose  `FP8ScaledMMLinearKernel.apply_weights` already accepts a matching `QuantizedActivation` and skips in-kernel input quantization, and `CutlassFP8ScaledMMLinearKernel.input_quant_key()` exposes both `kFp8StaticTensorSym` and `kFp8DynamicTokenSym` (#67393).  However, unlike `CompressedTenso…

### #69850 — [[Docs] Fix Read The Docs Link](https://github.com/vllm-project/vllm/pull/69850)
- **作者**: alex-jw-brooks  **时间**: 2026-11-25 16:44 CST
- **标签**: documentation
- **摘要**: Fixes a broken link in the pre check script for read the docs.

### #69849 — [[Perf][Multimodal] Skip redundant RGB copies in DeepSeek V4.1](https://github.com/vllm-project/vllm/pull/69849)
- **作者**: frankie-ys  **时间**: 2026-11-25 16:41 CST
- **标签**: deepseek
- **摘要**: ## Overview  DeepSeek V4.1 calls `Image.convert("RGB")` on every PIL image before resizing. Pillow copies the full pixel buffer when the image is already RGB. Skip the conversion for RGB inputs and keep the existing conversion for other modes.  ## Claims  - Avoids a full-resolution CPU image copy fo…

### #69848 — [[Rust Frontend] Export OffloadingConnector KV offload metrics](https://github.com/vllm-project/vllm/pull/69848)
- **作者**: BugenZhao  **时间**: 2026-11-25 16:35 CST
- **标签**: kv-connector, rust
- **摘要**: ## Overview  Export the `OffloadingConnector` metrics (`vllm:kv_offload_*`, `vllm:kv_offload_tiering_*` and `vllm:kv_offload_config_info`) from the Rust frontend's `/metrics`, as the Python frontend does. Today the Rust frontend has typed decoders only for NIXL, Mooncake (transfer and store), UMBP a…

### #69847 — [[Security] Size Dynamic NTK RoPE cache for the served context](https://github.com/vllm-project/vllm/pull/69847)
- **作者**: jperezdealgaba  **时间**: 2026-11-25 16:26 CST
- **标签**: documentation
- **摘要**: ## Overview  Size the factor-based Dynamic NTK RoPE cache to cover the configured model context. PR #55910 proposed a similar allocation change and closed unmerged.  ## Claims  - The initialized rotary cache covers the served context, and existing cache rows retain their values. - A model that alrea…

### #69846 — [[Security] Harden decoded image size checks](https://github.com/vllm-project/vllm/pull/69846)
- **作者**: jperezdealgaba  **时间**: 2026-11-25 16:22 CST
- **标签**: documentation, multi-modality
- **摘要**: ## Overview  Enforce decoded-image size limits across the Pillow-backed serving loader. This revision incorporates the format audit requested in review.  ## Claims  - Reject BLP, ICO, and AVIF at the shared image gate. This gate runs before selecting Pillow or TorchCodec. - Apply the pixel and reque…

### #69845 — [[XPU] Implement pyzes-based ZesXPUPlatform and enable auto NUMA binding](https://github.com/vllm-project/vllm/pull/69845)
- **作者**: xinyu-intel  **时间**: 2026-11-25 16:21 CST
- **标签**: intel-gpu, ci/build
- **摘要**: ## Overview  Implement a Level Zero Sysman (`pyzes`) backed `ZesXPUPlatform` (mirroring `NvmlCudaPlatform` / `NonNvmlCudaPlatform` in `vllm/platforms/cuda.py`) and enable automatic NUMA binding detection on Intel XPU.  ## Claims  - Adds `ZesXPUPlatform` and `NonZesXPUPlatform` (splitting `XPUPlatfor…

### #69843 — [feat: opt-in KV load/recompute policy with dual-threshold window](https://github.com/vllm-project/vllm/pull/69843)
- **作者**: zhaoly-shelley  **时间**: 2026-11-25 16:05 CST
- **标签**: scheduler
- **摘要**: ## Overview  Adds an opt-in scheduler-side KV load/recompute policy (`kv_load_policy` in `--kv-transfer-config`), applied at the same decision point as the existing `skip_reading_external_kv` override. Default `None`: no behavior change.  ## Claims  - Dual-threshold window (tau1=1024, tau2=23296) on…
