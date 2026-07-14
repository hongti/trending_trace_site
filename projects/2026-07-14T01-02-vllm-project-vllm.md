# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-14 09:02 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 📋 Issues (2 条)

*   **[Bug] #48541 FlashInfer CUTLASS MoE 在不支持 fp4 的构建中被错误选择**：在 CUDA toolkit < 12.8（缺少 fp4 支持）的构建环境中，引擎启动时错误选择了 FlashInfer CUTLASS MoE 路径，导致 `gpt-oss` 崩溃。
*   **[RFC] #48540 aarch64 二进制依赖的 manylinux 兼容性基线**：Nvidia 提议为 aarch64 架构的 Python 依赖（如 `pynvvideocodec`）建立 manylinux 兼容性基线，以使其 wheel 发布矩阵与 vLLM 支持的平台基线对齐。

---

### 🔀 Pull Requests (10 条)

**🚀 性能优化**
*   **#48544 离线 prompt tokenization 提速 2.4x**：通过将离线补全的预处理由逐条同步调用改为批量处理，大幅提升了 tokenizer 的处理速度。
*   **#48537 DeepSeek v4 端到端 TTFT 提升 6.6%~6.8%**：将 padding 操作融合进 `qnorm_rope_kv_insert` 内核，消除了先前单独开辟空间和拷贝 Q heads 的开销，显著降低了首字延迟（TTFT）。
*   **#48533 为 flex 显式标记 `dynamic=true`**：避免因 `auto` 设置触发重编译限制，提升运行稳定性。

**✨ 新特性与功能完善**
*   **#48538 新增在线 nvfp4 每词元量化**：支持在线逐词元的 nvfp4 量化。在 Qwen3-30B-A3B 模型上测试，GSM8K 精度几乎无损（0.9120 降至 0.9080）。
*   **#48543 增加 `diarized_json` 转录支持**：为 MOSS-Transcribe-Diarize 等支持说话人分离的模型，在 `/v1/audio/transcriptions` 接口增加了与 OpenAI 兼容的 `diarized_json` 响应格式。
*   **#48542 Rust 前端支持 prompt 截断参数**：Rust 前端此前会硬性拒绝长 prompt 的截断参数，现已支持 `truncate_prompt_tokens` 和 `truncation_side`，允许保留最后 N 个词元。
*   **#48535 前端传递前缀缓存词元数**：计算写入前缀缓存的 prompt 词元数量（`num_cache_creation_tokens`），并将其传递给 OpenAI Chat Completions 及后续分析模块。

**🛠️ Bug 修复**
*   **#48536 修复 SM12x 架构 NVFP4 权重加载扭曲问题**：针对 RTX Pro 6000 / DGX Spark 的 b12x MoE 路径，修复了因 `w1_alpha` 混用 FC1 activation 全局缩放和 post-GEMM MoE block 缩放，导致 NVFP4 权重在加载时被破坏的 Bug。
*   **#48539 修复 Machete act-order 重载丢失问题**：修复了权重重载时 Machete act-order permutation 存储未能保留的 Bug。
*   **#48534 [KV-transfer] 增加 MoRIIO 每层读取完成屏障**：修复了 MoRIIO READ 模式下异步 RDMA 读取的竞态问题，在 `wait_for_layer_load` 中为每层增加了 READ-completion barrier，确保解码工作器正确同步。

---

### 📦 Releases

本次抓取的动态中**无新版本发布**信息。

---

## 🐛 Issues

### #48541 — [[Bug]: FlashInfer CUTLASS MoE selected on fp4-less builds (CUDA toolkit < 12.8); gpt-oss dies at engine start](https://github.com/vllm-project/vllm/issues/48541)
- **作者**: RyanClark2k  **时间**: 2026-07-14 07:52 CST
- **标签**: bug
- **摘要**: ## Environment  - H100 SXM (SM90), driver 550.163.01 (host CUDA toolkit 12.4 at `/usr/local/cuda`) - torch 2.11.0+cu129, Python 3.12; vLLM built from source at 34ad15827 (the #48438 branch — its diff doesn't touch FlashInfer or MoE backend selection, and the code cited below is identical on current …

### #48540 — [[RFC]: manylinux compatibility baseline for aarch64 binary dependencies](https://github.com/vllm-project/vllm/issues/48540)
- **作者**: brandonpelfrey  **时间**: 2026-07-14 07:45 CST
- **标签**: RFC
- **摘要**: ### Motivation.  We (Nvidia) maintain a native Python dependency (`pynvvideocodec`) consumed by a vLLM Linux/aarch64 workflow and would like to align its wheel matrix with vLLM's supported platform baseline.  vLLM's published CUDA wheels currently target `manylinux_2_28` / glibc 2.28, matching the c…

## 🔀 Pull Requests

### #48544 — [[Frontend] Speed up offline prompt tokenization by 2.4x](https://github.com/vllm-project/vllm/pull/48544)
- **作者**: trevorprater  **时间**: 2026-07-14 08:39 CST
- **标签**: frontend
- **摘要**: ## Summary  This batches synchronous offline completion preprocessing instead of invoking the tokenizer once per prompt:  - `OfflineInferenceMixin._add_completion_requests` preprocesses prompts in   bounded groups of 32. - `HfRenderer.tokenize_prompts` sends homogeneous plain-text groups to the   Hu…

### #48543 — [[Frontend] Add diarized_json support for MOSS-Transcribe-Diarize](https://github.com/vllm-project/vllm/pull/48543)
- **作者**: wskr00  **时间**: 2026-07-14 08:21 CST
- **标签**: documentation, frontend, multi-modality
- **摘要**: ## Purpose  Implements #48443.  Add OpenAI-compatible `diarized_json` responses to `/v1/audio/transcriptions` for models that explicitly support diarized transcription. `OpenMOSS-Team/MOSS-Transcribe-Diarize` is the first supported model.  MOSS emits a compact transcript in the form `[start][Sxx]tex…

### #48542 — [[Rust Frontend] Add support for truncate_prompt_tokens and truncation_side](https://github.com/vllm-project/vllm/pull/48542)
- **作者**: pranavthakur0-0  **时间**: 2026-07-14 08:17 CST
- **标签**: rust
- **摘要**: Resolves #44280  truncate_prompt_tokens   Purpose  Basically, the Rust frontend was hard-rejecting these parameters before. If you tried to pass a really long prompt and told it to just keep the last 50 tokens, it would just throw an invalid request error.   This fixes that by bringing it up to pari…

### #48539 — [[Bugfix] Preserve Machete act-order permutation storage across weight reload](https://github.com/vllm-project/vllm/pull/48539)
- **作者**: RyanClark2k  **时间**: 2026-07-14 06:59 CST
- **标签**: bug
- **摘要**: # [Bugfix] Preserve Machete act-order permutation storage across weight reloads  ## Purpose  Fixes the Machete act-order row of RFC #48312 (category 1, storage identity), which I reported there after a registry-wide reload simulation caught it.  `MacheteLinearKernel.process_weights_after_loading` co…

### #48538 — [[Quant] Add online nvfp4 per-token quantization](https://github.com/vllm-project/vllm/pull/48538)
- **作者**: mgoin  **时间**: 2026-07-14 06:29 CST
- **标签**: ready, nvidia, quantization
- **摘要**: ## Purpose  ## Test Plan  ## Test Result  ``` vllm serve Qwen/Qwen3-30B-A3B-Instruct-2507 GSM8K: 0.9120  vllm serve Qwen/Qwen3-30B-A3B-Instruct-2507 --quantization nvfp4_per_token GSM8K: 0.9080 ```  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ]…

### #48537 — [[DSv4 Perf] Fused padding to `qnorm_rope_kv_insert` kernel, 6.6%~6.8% E2E TTFT improvement](https://github.com/vllm-project/vllm/pull/48537)
- **作者**: yewentao256  **时间**: 2026-07-14 06:21 CST
- **标签**: ready, deepseek, nvidia
- **摘要**: ## Purpose  Before:  ```bash Q heads = 16 Attn kernel requires heads = 64  Before kernel output [N,16,512] → new_zeros [N,64,512] (padding) → copy_  previous 16 heads  Now kernel output [N,64,512] ```  So we save additional memory allocation and a copy!  ## Test  `vllm serve deepseek-ai/DeepSeek-V4-…

### #48536 — [[Bugfix] Stop baking NVFP4 weight_scale_2 into b12x MoE block scales](https://github.com/vllm-project/vllm/pull/48536)
- **作者**: yichengj0  **时间**: 2026-07-14 06:20 CST
- **标签**: bug, nvidia
- **摘要**: ## Purpose  The SM12x b12x MoE path (RTX Pro 6000 / DGX Spark) distorts NVFP4 weights at load time:  - The FlashInfer kernel's `w1_alpha` does two jobs: FC1 activation-quant global scale and post-GEMM multiplier. The checkpoint's per-expert weight scale (`weight_scale_2`, ~2e-5) would wreck activati…

### #48535 — [[Front-end] [Messages] Populate `num_cache_creation_tokens`](https://github.com/vllm-project/vllm/pull/48535)
- **作者**: yzong-rh  **时间**: 2026-07-14 06:19 CST
- **标签**: frontend, v1, rust
- **摘要**: ## Purpose  Follow up to https://github.com/vllm-project/vllm/pull/40912.  Compute the number of prompt tokens written to prefix cache and pass it through to OpenAI Chat Completions and eventually to Anthropic Messages API.  Ensures `num_cache_creation_tokens` is populated.  ## Test Plan  ``` .venv/…

### #48534 — [[Bugfix][KV-transfer] MoRIIO: per-layer READ-completion barrier in wait_for_layer_load](https://github.com/vllm-project/vllm/pull/48534)
- **作者**: edwinlim0919  **时间**: 2026-07-14 06:03 CST
- **标签**: bug, kv-connector
- **摘要**: ## Summary  In MoRIIO **READ** mode the decode worker posts a request's per-layer RDMA reads asynchronously in `start_load_kv`. Each read completes later when a CQ-poll thread flips its status to `Succeeded`. `wait_for_layer_load()` was a **no-op (`pass`)**, so the attention kernel for a layer could…

### #48533 — [mark dynamic=true for flex](https://github.com/vllm-project/vllm/pull/48533)
- **作者**: liangel-02  **时间**: 2026-07-14 05:50 CST
- **标签**: v1
- **摘要**: as title, to avoid hitting recompile limits we should mark dynamic = true explicitly instead of leaving it as auto
