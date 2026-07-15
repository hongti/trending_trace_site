# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-14 20:28 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 🚀 Release (版本发布)
**v0.25.1**
- **版本亮点**：这是一个补丁版本，主要包含两个针对性的 Bug 修复，由 2 位贡献者（含 1 位新贡献者）完成。

### 🐛 Issues (问题反馈)
1. **MiniMax M3 Triton 路径缓冲区解析错误 (#48603)**：在 8x A100 上运行 MiniMax-M3-MXFP8 时，Triton 路径误解析了 token-major 的 top-k 缓冲区维度。
2. **DeepSeek-V4 Flash 服务崩溃 (#48602)**：在 RTXpro6000 上运行 vLLM benchmark（输入8192/输出1024）时，DeepSeek-V4-Flash 模型导致服务端直接崩溃报错。

### 🔀 Pull Requests (代码合并)
**🛠️ Bug 修复**
- **修复 MiniMax M3 top-k 缓冲区视图 (#48604)**：直接回应 Issue #48603，保持缓冲区为 token-major `[T, H, K]`，并在 Triton indexer 写入和 sparse-attention 读取时传递 head-major 转置视图 `[H, T, K]`。
- **修复视频加载帧采样逻辑 (#48608)**：解决带有 edit-list trims 的 MP4 文件（如 ffmpeg 无损裁剪、iPhone 录制）加载时静默帧覆盖崩溃的问题，改为基于可呈现帧数而非头部采样数进行采样。
- **修复 MRV2 Warmup OOM (#48601)**：对 flashinfer 的 top-k/top-p sampler 进行 profiling，解决在 H100 上使用 V2 Model Runner 服务 Qwen3.6-27B-FP8 时遇到的预热阶段 OOM 问题。
- **修复 ROCm CI Gemma4MTP 初始化失败 (#48607)**：保留 Gemma4MTP 初始化测试的原始层数，修复 AMD ROCm CI 中的报错。

**✨ 新特性与功能增强**
- **支持 Quark AWQ INT4 量化加载 (#48606)**：新增原生支持加载 AMD Quark AWQ INT4 导出（W4A16 linear 和 MoE），不再将其作为 AutoAWQ 检查点处理。
- **TritonMLA 支持滑动窗口注意力 (SWA) (#48605)**：为 TritonMLA 启用滑动窗口注意力，支持窗口解码及超长提示词的局部窗口预填充。
- **DeepSeek-V4 Attention 支持 3-D Packed Hidden States (#48598)**：在 DeepSeek-V4 的 attention forward 路径中增加对 3-D packed hidden states 的处理，为后续 NVFP4 仿真内核支持铺路。

**🏗️ 基础架构与重构**
- **回退稀疏 MLA 短序列密集 MHA 路径 (#48609)**：回退了 PR #47327（为稀疏 MLA 短序列添加密集 MHA 路径），因其引发了行为变更。
- **平台环境变量检查接口重构 (#48599)**：将环境变量检查移至平台接口层，允许不同平台（如 vllm-ascend）自定义环境变量，避免触发“未知 vLLM 环境变量”警告。
- **CI 发布产物注释分类 (#48600)**：将 Buildkite 中共享的 `release-artifacts` 注释按类型拆分为 wheels、images 和 manifests，优化内联报告展示。

---

## 🐛 Issues

### #48603 — [[Bug]: MiniMax M3 Triton path misinterprets token-major top-k buffer](https://github.com/vllm-project/vllm/issues/48603)
- **作者**: RedHeartSecretMan  **时间**: 2026-07-14 18:14 CST
- **摘要**: ### Your current environment  - vLLM: v0.25.1 and current `main` - Model: MiniMax-M3-MXFP8 - Hardware: 8x NVIDIA A100 (SM80) - Tensor parallel size: 8  ### Model Input Dumps  No model input dump is required to reproduce the layout mismatch. It occurs when the MiniMax M3 Triton indexer receives the m…

### #48602 — [[Bug]: DeepSeekv4 flash on RTXpro6000 vllm benchmark input 8192 output 1024 encounter a err](https://github.com/vllm-project/vllm/issues/48602)
- **作者**: iceCreeam  **时间**: 2026-07-14 18:13 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Your output of `python collect_env.py` here ```  </details>   ### 🐛 Describe the bug  <img width="1379" height="140" alt="Image" src="https://github.com/user-attachments/assets/2c1f94…

## 🔀 Pull Requests

### #48609 — [Revert "[1/N] Add dense MHA path for sparse MLA short sequences" (#47327)](https://github.com/vllm-project/vllm/pull/48609)
- **作者**: vllm-agent  **时间**: 2026-07-14 20:07 CST
- **标签**: documentation, performance, rocm, intel-gpu, v1, nvidia
- **摘要**: ## Revert of #47327  This reverts commit c4f5cd60dae386d106c9b8a12dbab24e2e9dda0b (merge commit for PR #47327).  **Original PR:** https://github.com/vllm-project/vllm/pull/47327  ### Reason  PR #47327 changed the MLA prefill backend selector (`mla_attention.py`, `prefill/selector.py`), which caused …

### #48608 — [[Bugfix] Video loading: sample over presentable frames, not header sample count (MP4 edit-list trims)](https://github.com/vllm-project/vllm/pull/48608)
- **作者**: AmitMY  **时间**: 2026-07-14 19:31 CST
- **标签**: bug, multi-modality
- **摘要**: ## Purpose  Fix silent frame-coverage collapse when loading MP4s trimmed losslessly (e.g. `ffmpeg -ss ... -c copy`, PyAV stream-copy remuxes, iPhone recordings with edit lists).  Such files keep the decode lead-in packets (needed to decode from the preceding keyframe) and hide them with an MP4 **edi…

### #48607 — [[ROCm][CI] Keep original layer count for Gemma4MTP init test](https://github.com/vllm-project/vllm/pull/48607)
- **作者**: stefankoncarevic  **时间**: 2026-07-14 19:15 CST
- **标签**: rocm
- **摘要**: ## Purpose  `test_can_initialize_large_subset[Gemma4MTPModel]` fails in the AMD ROCm CI (basic models init tests) with:  ``` File "vllm/v1/worker/utils.py", line 475, in add_kv_sharing_layers_to_kv_cache_groups     tgt_kv_cache_group = layer_to_kv_cache_group[target_layer_name] KeyError: 'language_m…

### #48606 — [[Quantization] Support Quark AWQ INT4 exports in vLLM](https://github.com/vllm-project/vllm/pull/48606)
- **作者**: limitmhw  **时间**: 2026-07-14 19:13 CST
- **标签**: qwen
- **摘要**: ## Summary  This PR enables vLLM to load AMD Quark AWQ INT4 exports through Quark's native quantization path instead of treating them as AutoAWQ checkpoints.  - Add native Quark W4A16 INT4 linear and MoE methods for packed signed INT4 weights. - Extend the shared AWQ Triton and fused MoE kernels wit…

### #48605 — [[Kernel] TRITON_MLA SWA](https://github.com/vllm-project/vllm/pull/48605)
- **作者**: NickLucche  **时间**: 2026-07-14 19:03 CST
- **标签**: ready, needs-rebase, v1, nvidia
- **摘要**: Enables TritonMLA to run sliding-window attention.  What this PR enables:   - Windowed decode for TritonMLA   - Windowed new-token prefill — prompts longer than the window now attend only within it, instead of full-causal.   - Windowed chunked-prefill with context — cached context beyond the window …

### #48604 — [[Model] Fix MiniMax M3 top-k buffer views](https://github.com/vllm-project/vllm/pull/48604)
- **作者**: RedHeartSecretMan  **时间**: 2026-07-14 18:15 CST
- **摘要**: ## Summary  - keep the MiniMax M3 persistent top-k buffer token-major (`[T, H, K]`) on NVIDIA and AMD - pass head-major transposed views (`[H, T, K]`) at Triton indexer write and sparse-attention read boundaries - prevent incorrect shape/stride interpretation that causes CUDA illegal memory access o…

### #48601 — [[MRV2] Profile flashinfer top-k/top-p sampler to fix warmup OOM](https://github.com/vllm-project/vllm/pull/48601)
- **作者**: JaredforReal  **时间**: 2026-07-14 18:09 CST
- **标签**: v1
- **摘要**: ## Purpose Using H100 to serve Qwen3.6-27B-FP8 ``` VLLM_USE_V2_MODEL_RUNNER=1 vllm serve Qwen/Qwen3.6-27B-FP8 --served-model-name qwen -tp 4 ``` We will encounter an OOM at kernel warmup: ``` (Worker_TP2 pid=3264437) INFO 07-14 09:50:23 [gpu_worker.py:855] Free memory on device (77.57/79.11 GiB) on …

### #48600 — [[CI/Build] Split release artifact annotations by type](https://github.com/vllm-project/vllm/pull/48600)
- **作者**: khluu  **时间**: 2026-07-14 18:03 CST
- **标签**: ci/build
- **摘要**: ## Purpose  Split the shared `release-artifacts` Buildkite annotation into dedicated `release-wheels`, `release-images`, and `release-manifests` annotations. This preserves inline reporting after each artifact is produced while making release outputs easier to scan.  The annotation helper now requir…

### #48599 — [[Platform] Move env check function to platform interface](https://github.com/vllm-project/vllm/pull/48599)
- **作者**: wangxiyuan  **时间**: 2026-07-14 17:18 CST
- **摘要**: ## Purpose Different platform may add its own env vars. Take vllm-ascend as an example. It contains some envs like `VLLM_ASCEND_XXX`. Once it's set, vllm will print warning "Unknown vLLM environment variable detected: VLLM_ASCEND_XXX", this log may mislead users.   This PR move the check function to…

### #48598 — [[DeepSeek-V4] Handle 3-D packed hidden states in attention forward](https://github.com/vllm-project/vllm/pull/48598)
- **作者**: jimmy-adams  **时间**: 2026-07-14 17:13 CST
- **标签**: deepseek
- **摘要**: ## Summary This PR adds handling for 3-D packed hidden states in the DeepSeek-V4 attention forward path. It is split out from #47972 ("Support DeepSeek-V4 AMD Quark NVFP4 with emulation kernel", https://github.com/vllm-project/vllm/pull/47972). During review of that PR it was pointed out that this a…

## 🚀 Releases

### [v0.25.1](https://github.com/vllm-project/vllm/releases/tag/v0.25.1)
- **作者**: khluu  **时间**: 2026-07-14 16:51 CST
- **摘要**: # vLLM v0.25.1  ## Highlights  This release features 2 commits from 2 contributors (1 new)!  v0.25.1 is a patch release containing two targeted bug fixes on top of v0.25.0.  ### Bug Fixes * **Avoid blocking model launching when no system FFmpeg is available for TorchCodec** (#47888). Previously `imp…
