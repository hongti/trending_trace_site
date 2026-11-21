# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-21 15:37 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
- **KV Connector 缓存失效逻辑错误 (#69100)**
  当设置 `kv_load_failure_policy="recompute"` 时，同步加载失败的 KV 块仍被错误地保留在前缀缓存中。这会导致其他请求直接从从未真正加载的 KV 数据进行解码，引发潜在推理错误。

### 🚀 PR 动态
**修复与安全性：**
- **限制单 Prompt 视频处理量 (#69098)**：为 Qwen3-VL 和 LLaVA-OneVision 引入安全限制，默认单次 Prompt 最多处理 32 个视频，并支持自定义配置，防范恶意 OOM 攻击。
- **多模态输入解析修复 (#69099)**：修复 Muse Spark Open 预处理无法正确识别 NumPy 图像和 PIL 视频帧列表的问题。
- **Mooncake KV 连接器网络修复 (#69092)**：向 bootstrap 的 `httpx.AsyncClient` 传入 `trust_env=False`，避免受系统 HTTP 代理环境变量影响导致连接异常。
- **HF3FS KV 加载重试修复 (#69095)**：在 recompute 策略下，正确上报加载失败的 block ID，并阻止对失败块的反复远程重载。
- **Inkling2 DSpark 兼容性修复 (#69096)**：避免在 SM120 预编译镜像中调用不可用的 `reshape_and_cache_flash`，修复 DSpark 预热失败问题。

**性能与新特性：**
- **ROCm / DeepSeek 性能深度优化 (#69090, #69097)**：
  - 针对 gfx942 架构（MI300X/MI325X）优化 DeepSeek-V4 稀疏 MLA 解码内核，将每个 workgroup 处理的头数从 16 提升至 32，显著降低开销。
  - 为 MI355X 上的 DeepSeek-V4.1-Flash 引入了支持融合 EP8 MoE 的 `amd_kernels` 路由框架。
- **多模态图像处理性能优化 (#69093)**：针对刚加载的单帧 JPEG/PNG 等图片，实现原地的 EXIF 方向转换，减少不必要的内存拷贝。
- **Intel XPU 工具链支持 (#69091)**：vLLM Recipes 工具新增对 Intel Arc Pro B70 显卡的硬件自动检测与并行布局扫描调优支持。
- **Rust 前端对齐 (WIP) (#69094)**：正在开发使 Rust 前端与 `max_tokenize_tokens` 参数对齐的功能。

### 📦 Release 动态
- 本周期内无新的版本发布记录。

---

## 🐛 Issues

### #69100 — [[Bug][KV Connector] `kv_load_failure_policy="recompute"` keeps failed-load blocks as prefix-cache hits, so other requests decode from never-loaded KV](https://github.com/vllm-project/vllm/issues/69100)
- **作者**: SohamDutta02  **时间**: 2026-11-21 15:25 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details>  ```text Main: bf151b7eb1403d3671200e0cd19ad9821fe4320f. x86_64 VM (2 vCPU Intel Xeon, 7.8 GB RAM), Ubuntu 24.04.5; CPU only, no GPU. Python 3.13.18, PyTorch 2.14.0, Transformers 5.23.0. Main Python sources installed editable without native kernels (VLLM_TARGE…

## 🔀 Pull Requests

### #69099 — [[Bugfix][Multimodal] Accept NumPy images and PIL frame videos in Muse Spark Open](https://github.com/vllm-project/vllm/pull/69099)
- **作者**: yarikama  **时间**: 2026-11-21 15:13 CST
- **标签**: bug, multi-modality
- **摘要**: ## Overview  The parser passes NumPy images through as arrays and keeps PIL frames as a list inside a `(frames, metadata)` video, but Muse Spark Open's own preprocessing reads `.mode` on every image and `.shape` on every frame, so both inputs fail with `AttributeError`. This PR converts images with …

### #69098 — [[Security] Bound video preprocessing per prompt](https://github.com/vllm-project/vllm/pull/69098)
- **作者**: jperezdealgaba  **时间**: 2026-11-21 14:54 CST
- **标签**: documentation, multi-modality, qwen
- **摘要**: ## Overview  Bound video preprocessing per prompt for Qwen3-VL and LLaVA-OneVision.  ## Claims  - Default to 32 videos per prompt for both processors and honor explicit `--limit-mm-per-prompt.video` settings. - Cap retained video pixel tensors with `VLLM_MAX_MEDIA_PREPROCESSING_BYTES`. Include Qwen3…

### #69097 — [[DNM][Stacked on #67850][ROCm][DeepSeek-V4.1-Flash] amd_kernels routes for DEP8 on MI355X (fused EP8 MoE)](https://github.com/vllm-project/vllm/pull/69097)
- **作者**: Fangzhou-Ai  **时间**: 2026-11-21 14:47 CST
- **标签**: rocm, speculative-decoding, needs-rebase, ci/build, deepseek, kv-connector, mrv2, dflash, minimax, nixl
- **摘要**: ## Overview  Stacked on #67850 (amd_kernels routes for MiniMax-M3.1): the same route framework (`AmdKernelsConfig`, one gate module, lazy `amd_kernels` imports, stock fallback everywhere) now also serves **DeepSeek-V4.1-Flash at DEP8** (TP1 x DP8, EP8, DSpark speculative decoding, fp8 KV cache, MXFP…

### #69096 — [[Bugfix][Model] Avoid reshape_and_cache_flash in Inkling2 DSpark KV write](https://github.com/vllm-project/vllm/pull/69096)
- **作者**: ItsRoy69  **时间**: 2026-11-21 14:46 CST
- **标签**: bug, dflash, inkling
- **摘要**: ## Overview  Fix Inkling2 DSpark draft KV-cache writes to avoid reshape_and_cache_flash, which is unavailable on some SM120 prebuilt images and causes DSpark warmup to fail. Fixes #69081.  ## Claims  - Fixes #69081: Inkling2 DSpark speculative decoding no longer depends on reshape_and_cache_flash fo…

### #69095 — [[Bugfix][KV Connector] Report HF3FS failed loads and prevent repeated reloads](https://github.com/vllm-project/vllm/pull/69095)
- **作者**: ZYZ666-RGB  **时间**: 2026-11-21 14:46 CST
- **标签**: bug, kv-connector
- **摘要**: ## Overview    ## Claims   ## Overview  Related to #69058.  Report failed HF3FS load block IDs, retain all load batches for each request, and prevent repeated remote loads after a failed load under the recompute policy.  ## Claims  - Failed block IDs are returned in the same transfer-results pass. -…

### #69094 — [feat: alignment on rust frontend with max_tokenize_tokens](https://github.com/vllm-project/vllm/pull/69094)
- **作者**: seansie0830  **时间**: 2026-11-21 14:45 CST
- **标签**: rust
- **摘要**: this is still **drafting**!! > [!NOTE] > **Draft / WIP:** Work in progress. Core logic is being wired up, and E2E tests / edge-case checks are still being finalized. Feedback on potential edge cases or subtle bugs is very welcome.  ## Summary  Aligns the Rust frontend (`vllm-rs`) behavior and config…

### #69093 — [[Perf][Multimodal] Transpose EXIF orientation in place for loader-owned images](https://github.com/vllm-project/vllm/pull/69093)
- **作者**: CZT0  **时间**: 2026-11-21 14:35 CST
- **标签**: multi-modality
- **摘要**: ## Overview  Add a keyword-only `in_place: bool = False` to `vllm.multimodal.image.normalize_image()` and use it in `ImageMediaIO.load_bytes()` for freshly opened single-frame JPEG/PNG/WebP/BMP images (detaching the loaded `ImagingCore` onto a plain `Image.Image` via `image._new(image.im)`), so Pill…

### #69092 — [[Bugfix][KV Connector][Mooncake] Pass trust_env=False to bootstrap httpx.AsyncClient](https://github.com/vllm-project/vllm/pull/69092)
- **作者**: CZT0  **时间**: 2026-11-21 14:35 CST
- **标签**: bug, documentation, kv-connector, rust, mooncake
- **摘要**: ## Overview  Pass `trust_env=False` to both `httpx.AsyncClient` call sites used by Mooncake KV Connector bootstrap HTTP requests (`/register` and `/query`), so outbound proxy environment variables (`HTTP_PROXY` / `ALL_PROXY`) do not hijack internal bootstrap traffic between prefill and decode worker…

### #69091 — [[Recipes][XPU] Enable Intel Arc Pro B70 hardware detection and sweep tuning](https://github.com/vllm-project/vllm/pull/69091)
- **作者**: louie-tsai  **时间**: 2026-11-21 14:16 CST
- **标签**: documentation, intel-gpu
- **摘要**: ## Overview  Extend the vLLM Recipes tool to support Intel Arc Pro B70 (XPU) deployments, including automatic hardware detection, GPU-aware parallel-layout sweeping, structured validation results, and reasoning-parser asset preparation.  ## Claims  - Add Intel XPU hardware detection using PyTorch to…

### #69090 — [[ROCm][Perf][DSv4] gfx942 sparse MLA decode: one workgroup per 32 heads](https://github.com/vllm-project/vllm/pull/69090)
- **作者**: ZhengGong-amd  **时间**: 2026-11-21 14:08 CST
- **标签**: rocm, deepseek
- **摘要**: ## Overview  On gfx942 (MI300X/MI325X), the DeepSeek-V4/V4.1 sparse MLA decode launches `_sparse_attn_decode_partial_kernel` with one workgroup per 16 heads, so a rank with 32 or more heads gathers and dequantizes every KV tile once per 16 heads. When the rank's head count is a multiple of 32, this …
