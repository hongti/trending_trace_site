# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-19 09:04 CST

## AI 总结

## vllm-project/vllm 近期动态摘要

---

### 🐛 Issue

- **#49061 — `all_gather_interleave()` 绕过 vLLM 集合 API 导致非主流平台崩溃**
  PaddleOCR-VL / Ernie4.5-VL / GLM-4.1V 三个模型的视觉注意力模块各自内嵌了一份相同的 `all_gather_interleave()` 辅助函数，直接调用 `torch.distributed.all_gather()`，绕过了 vLLM 的统一集合通信 API，在非标准设备平台上触发 "No backend type associated with device type" 错误而崩溃。

---

### 🔀 Pull Request

#### 🚀 新特性 / 功能扩展

- **#49060 — 新增在线 NVFP4 密集线性量化（W4A16 + W4A4）**
  扩展 vLLM 的在线量化框架，支持在加载时将 BF16/FP16 权重直接量化为 NVFP4 格式，覆盖 W4A16 与 W4A4 两种模式。

#### 🔧 Bug 修复（核心）

- **#49062 — 移除 ViT split_qkv 中冗余的 `all_gather_interleave`（修复 #49061）**
  删除三个模型中重复的辅助函数，改用 vLLM 统一集合 API，彻底解决非主流平台崩溃问题。

- **#49059 — DeepSpeed v4 / SM120：跳过空的 sparse-MLA prefill chunk**
  防止 `FLASHINFER_MLA_SPARSE_DSV4` 将全-padding 的 prefill chunk 发送至 FlashInfer，避免 CUDA graph 场景下出错。

- **#49058 — 修复 LoRA + NVFP4/EP padded buffer 导致 Triton kernel OOB 崩溃**
  针对使用 `--enable-lora` + `--enable-exper` 服务 NVFP4 量化 MoE 模型时的 CUDA 崩溃，提供三项修复。

- **#49052 — KV Offload：修复非对齐 SWA 加载导致 EngineCore 致命 abort**
  CPU KV offload 在合法的非对齐滑动窗口外部加载时，因用 chunk-aligned 值约束物理 GPU block 而异常中止，现已修正边界计算。

- **#49054 — MiniMax-M3：避免空 sparse decode 行产生 NaN**
  CUDA graph padding 可能导致某行无有效 chunk，原逻辑出现 `-inf - -inf`；现在空行直接返回零值。

#### 🔧 Bug 修复（配置 / 接口 / CI）

- **#49057 — 修复 YAML CLI 配置中 `false` 值被丢弃的问题**
  布尔选项值为 `false` 时曾被错误忽略，现已正确保留。

- **#49056 — 修复 `encode_{audio,image,video}_url` 输出无效 media type**
  改进 MIME 类型解析，确保 data URL 始终携带合法的 media type。

- **#49053 — 拒绝负数 `--device-ids` 索引而非静默选择错误设备**
  对负数索引增加范围检查并报错，防止在 `CUDA_VISIBLE_DEVICES` 环境下误选 GPU。

- **#49055 — ROCm CI：确保滑动窗口测试间释放 GPU 内存**
  使用 `vllm_runner` 显式关闭引擎并等待 GPU 内存回收，避免参数化测试间因内存未释放而失败。

---

### 📦 Release

本次监测周期内**无新版本发布**。

---

> **总结**：本期活动以密集的 Bug 修复为主，核心亮点是 **NVFP4在线量化新特性（#49060）** 和 **修复多模型绕过集合API的严重崩溃（#49061→#49062）**，同时针对 DeepSpeed v4、LoRA+NVFP4、KV Offload、MiniMax-M3 等关键路径的多项稳定性问题进行了修复。

---

## 🐛 Issues

### #49061 — [[Bug]: `all_gather_interleave()` in PaddleOCR-VL/Ernie4.5-VL/GLM-4.1V bypasses vLLM's collective API and crashes with "No backend type associated with device type" on out-of-tree platforms](https://github.com/vllm-project/vllm/issues/49061)
- **作者**: wagnerpatriota  **时间**: 2026-07-19 06:50 CST
- **标签**: bug, rocm
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (x86_64) GCC version                  : (Ubuntu 11.…

## 🔀 Pull Requests

### #49062 — [[Bugfix][Model] Remove redundant all_gather_interleave in ViT split_qkv](https://github.com/vllm-project/vllm/pull/49062)
- **作者**: shivasathishs-rp  **时间**: 2026-07-19 08:50 CST
- **标签**: bug
- **摘要**: ## Purpose  Fixes #49061.  PaddleOCR-VL, Ernie4.5-VL, and GLM-4.1V vision attention each carried a byte-identical helper `all_gather_interleave()` that calls `torch.distributed.all_gather()` directly on the tensor-parallel group's raw `device_group`, followed by a per-rank re-split, inside `split_qk…

### #49060 — [[Quant] Add online NVFP4 dense-linear quantization (W4A16 + W4A4)](https://github.com/vllm-project/vllm/pull/49060)
- **作者**: bhoomit  **时间**: 2026-07-19 06:48 CST
- **标签**: nvidia
- **摘要**: # [Quant] Online NVFP4 dense-linear quantization (W4A16 + W4A4)  ## Purpose  Extend vLLM's online quantization framework to quantize **dense linear** weights to NVFP4 at load time from a plain BF16/FP16 checkpoint — no pre-quantized checkpoint, no calibration. Two schemes:  - **`--quantization nvfp4…

### #49059 — [[Bugfix][DSv4][SM120] Skip empty sparse-MLA prefill chunks](https://github.com/vllm-project/vllm/pull/49059)
- **作者**: ormandj  **时间**: 2026-07-19 06:32 CST
- **标签**: bug, nvidia
- **摘要**: ## Purpose  Prevent `FLASHINFER_MLA_SPARSE_DSV4` from sending all-padding prefill chunks to FlashInfer on SM120.  With `FULL_AND_PIECEWISE` CUDA graphs, vLLM can pad one real prefill to a larger captured request/token size. `split_decodes_and_prefills()` includes the padded request slots in `num_pre…

### #49058 — [fix: LoRA Triton kernel OOB crash with NVFP4/EP padded buffers](https://github.com/vllm-project/vllm/pull/49058)
- **作者**: fattchris  **时间**: 2026-07-19 06:05 CST
- **摘要**: ## Summary  Three fixes for the LoRA + NVFP4 (modelopt_fp4) CUDA crash reported in #48862.  When serving NVFP4-quantized MoE models (e.g. `nvidia/GLM-5.2-NVFP4`) with `--enable-lora` and `--enable-expert-parallel`, vLLM pre-allocates LoRA buffers to `max_num_batched_tokens` (8192). The Triton fused …

### #49057 — [[Bugfix] Fix false boolean values in YAML CLI configuration](https://github.com/vllm-project/vllm/pull/49057)
- **作者**: hsusul  **时间**: 2026-07-19 05:48 CST
- **标签**: bug
- **摘要**: ## Summary  Fix YAML configuration handling for boolean CLI options whose value is `false`.  Previously, false-valued YAML entries could be discarded while configuration values were converted into command-line arguments. This prevented YAML configuration from disabling options whose effective defaul…

### #49056 — [[Bugfix] Emit a valid media type from encode_{audio,image,video}_url](https://github.com/vllm-project/vllm/pull/49056)
- **作者**: vineethsaivs  **时间**: 2026-07-19 04:30 CST
- **标签**: bug, multi-modality
- **摘要**: ## Purpose  `encode_audio_url`, `encode_image_url` and `encode_video_url` in `vllm/multimodal/utils.py` build a data URL and resolve its media type with:  ```python mimetype = mimetypes.types_map.get("." + format.lower(), "image")  # or "audio"/"video" return f"data:{mimetype};base64,{...}" ```  Whe…

### #49055 — [[ROCm][CI] Ensure sliding window tests release GPU memory](https://github.com/vllm-project/vllm/pull/49055)
- **作者**: AndreasKaratzas  **时间**: 2026-07-19 04:09 CST
- **标签**: rocm, ready, v1
- **摘要**: Use `vllm_runner` in the sliding-window correctness test to explicitly shut down the engine and wait for GPU memory to settle between parameterized cases. This prevents the next test from failing its startup memory check while the previous engine is still releasing VRAM. Motivation: - https://buildk…

### #49054 — [[Bugfix][MiniMax-M3] Avoid NaNs for empty sparse decode rows](https://github.com/vllm-project/vllm/pull/49054)
- **作者**: thepowerfuldeez  **时间**: 2026-07-19 04:00 CST
- **标签**: bug
- **摘要**: ## Summary  I avoid `-inf - -inf` in the MiniMax M3 split-K merge when CUDA-graph padding leaves a row without valid chunks. Active rows keep the existing normalization, and empty rows return zeros.  ## Duplicate check  I searched open issues and PRs for MiniMax M3 sparse attention, padded or empty …

### #49053 — [[Bugfix] Reject negative --device-ids indices instead of selecting the wrong device](https://github.com/vllm-project/vllm/pull/49053)
- **作者**: vineethsaivs  **时间**: 2026-07-19 03:58 CST
- **标签**: bug
- **摘要**: ## Purpose  `EngineArgs._resolve_device_ids` composes `--device-ids` with `CUDA_VISIBLE_DEVICES`: when CVD is set, each `--device-ids` value is treated as an index into the visible set. The range check only guarded the upper bound:  ```python for i in int_ids:     if i >= len(cvd_ids):         raise…

### #49052 — [[Bugfix][KV Offload] Bound unaligned SWA loads by physical GPU blocks](https://github.com/vllm-project/vllm/pull/49052)
- **作者**: coltonottley  **时间**: 2026-07-19 03:22 CST
- **标签**: bug, v1, kv-connector
- **摘要**: ## Summary  Fixes #48959.  CPU KV offload can fatally abort `EngineCore` on a valid unaligned sliding-window external load because the scheduler bounds pending physical GPU blocks using a chunk-aligned approximation.  For the production geometry:  - sliding window: 4,096 tokens; - GPU block: 32 toke…
