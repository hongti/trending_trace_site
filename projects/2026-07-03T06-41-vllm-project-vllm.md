# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-03 06:41 UTC

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 📋 Issue 动态
近期主要报告了两个核心架构/Bug 问题：
1. **DeepSeek-V4-Pro FP4 MoE 并行乱码** (#47528)：DeepSeek-V4-Pro 在使用张量并行（TP）时输出乱码/退化，但在数据并行+专家并行（DP+EP）下可正常工作。涉及 `scale_fmt=ue8m0` 的 FP4 MoE 机制。
2. **MLA 预填充在 sm80 崩溃** (#47522)：在 A100（sm80，无原生FP8支持）上运行 GLM-5.2-FP8 等模型时，MLA 分块上下文预填充与 Marlin FP8 交互时发生崩溃，提示 `kv_c_normed` 转换至 packed-int32 类型不支持。

### 🛠️ PR 动态
近期 PR 集中在**核心推理性能优化、新硬件/量化支持及稳定性修复**：

**1. 核心架构与性能优化（重要变更）**
* **Attention 降级修复** (#47520)：修复了长上下文 Decode 场景下，当序列数>8 时 Triton 统一注意力机制从 3D split-KV 退化为 2D 导致延迟翻倍的“断崖式降级”问题，现改为基于设备 SM 数量动态推导阈值。
* **采样精度提升** (#47524)：为 fp32 Gumbel sampling 引入 64-bit 均匀分布绘制，修复了 Triton `tl.rand` 因 int32 映射导致精度不足（无法解析低于 `2^-31` 的值）的问题，且不损失性能。

**2. 模型与硬件支持（新特性）**
* **SM120 MLA 解码修复** (#47527)：修复 SM120 架构下 FlashInfer sparse MLA 解码路径，使其支持 packed `fp8_ds_mla` KV 布局，影响 GLM 及 DeepSeek V3.2 等模型。
* **ROCm DSV4 融合内核** (#47518)：为 DeepSeek-V4 在 ROCm 上的 Decode 阶段启用 fused AITER mHC post+pre 内核，替代之前的非融合版本以提升性能。
* **INT2 量化支持** (#47521)：新增 INC（Intel Neural Compressor）INT2 XPU Linear 量化支持，拓展低比特量化生态。

**3. 稳定性防护与 CI 修复**
* **Triton Kernel 防护** (#47525)：增加断言确保 `group_size` 是 `BLOCK_K` 的倍数，防止 Kernel 迭代跨越两个缩放组导致读取错误的 scales/zero_points。
* **ROCm CI 修复** (#47519)：修复 MI355 上 Kernels/Attention 测试因 `prefill_backend` 为 None 导致的克隆失败，及 MoE OCP MX 测试失败问题。

**4. 周边与工程化**
* **NIXL SSE 修复** (#47526)：修正 NIXL toy proxy 流式响应的 Content-Type 为 `application/json`，使客户端正确处理 SSE `data:` 格式。
* **Rust 前端测试加速** (#47523)：减少重复渲染解析，将 Rust 前端聊天 roundtrip 测试的耗时从 >20 秒大幅缩短。
* **文档修正** (#47517)：修复 VLM2Vec benchmark 示例中的 chat template 路径指向错误。

### 🚀 Release 动态
* **近期无新版本发布**（提供的动态中未包含 Release 信息）。

---

## 🐛 Issues

### #47528 — [[Bug]: DeepSeek-V4-Pro (DeepseekV4ForCausalLM, scale_fmt=ue8m0 / FP4 MoE) produces garbled / degenerate output under tensor parallelism (TP), while data parallelism + expert parallelism (DP+EP) works correctly](https://github.com/vllm-project/vllm/issues/47528)
- **作者**: LiAnQing279  **时间**: 2026-07-03 06:40 UTC
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text python3 collect_env.py  Collecting environment information... ==============================         System Info ============================== OS                           : Ubuntu 2…

### #47522 — [[Bug]: MLA chunked-context prefill crashes on sm80 with Marlin FP8: kv_c_normed cast to packed-int32 weight dtype (`unsupported \`a\` scalar_type`)](https://github.com/vllm-project/vllm/issues/47522)
- **作者**: biondogs  **时间**: 2026-07-03 06:10 UTC
- **摘要**: ### Your current environment  - vLLM 0.23.0 (official image), PyTorch 2.x, Triton, NCCL 2.28.9 - 12× NVIDIA A100-SXM4-80GB (compute capability **8.0**, no native FP8) across 3 nodes - Model: GLM-5.2-FP8 (fp8 weights → `MarlinFP8ScaledMMLinearKernel` / `MARLIN Fp8 MoE`), PP=3 × TP=4, `--enable-expert…

## 🔀 Pull Requests

### #47527 — [[Bugfix][SM120][MLA] Support FlashInfer packed sparse MLA decode](https://github.com/vllm-project/vllm/pull/47527)
- **作者**: ChamHerry  **时间**: 2026-07-03 06:37 UTC
- **标签**: bug, ci/build, v1, nvidia
- **摘要**: ## Purpose  This PR fixes the released-wheel SM120 FlashInfer sparse MLA path used by GLM / Deepseek V3.2 style sparse MLA models with FP8 KV cache.  The current path can select the SM120 FlashInfer sparse MLA backend, but the decode API needs the packed `fp8_ds_mla` KV layout and `kv_scale_format` …

### #47526 — [fix: return SSE content type from NIXL toy proxy](https://github.com/vllm-project/vllm/pull/47526)
- **作者**: Spycsh  **时间**: 2026-07-03 06:30 UTC
- **标签**: v1, kv-connector
- **摘要**: ## Purpose Minor fix of the NIXL integration toy proxy streaming response content type.  The proxy forwards OpenAI-compatible streaming chunks in SSE format (`data: ...` / `data: [DONE]`). Returning them as `application/json` can make clients parse the raw `data:` lines as JSON, rather than treat th…

### #47525 — [Add assertion for group_size and BLOCK_K consistency](https://github.com/vllm-project/vllm/pull/47525)
- **作者**: hnhyzz  **时间**: 2026-07-03 06:17 UTC
- **摘要**: Add assertion to ensure group_size is a multiple of BLOCK_K.     ## Purpose Avoid Tuning BLOCK_K breaks the triton kernel's assumption that each K iteration shares the same group scales and zero points.  If group size is not divisible by BLOCK_K. The K iteration may span two groups, and wrong scales…

### #47524 — [[MRV2] Draw 64-bit uniforms for fp32 Gumbel sampling](https://github.com/vllm-project/vllm/pull/47524)
- **作者**: WoosukKwon  **时间**: 2026-07-03 06:15 UTC
- **标签**: ready, v1
- **摘要**: ## Purpose  Improve the precision of the fp32 Gumbel sampling path (the default since #41775) at no cost to performance. The fp64 path is unchanged.  Triton's `tl.rand` maps an int32 to `|x| * ~2**-31`, so the uniform it returns never resolves below `2**-31`. With the flipped transform from #45996 (…

### #47523 — [[Rust Frontend] Speed up chat roundtrip tests](https://github.com/vllm-project/vllm/pull/47523)
- **作者**: BugenZhao  **时间**: 2026-07-03 06:13 UTC
- **标签**: ready, rust
- **摘要**: Signed-off-by: Bugen Zhao <i@bugenzhao.com><!-- markdownlint-disable -->   ## Purpose  Rendering-parsing roundtrip tests now take a long time when running all unit tests in the Rust frontend, resulting in more than 20 seconds of wall time.  This PR speeds up roundtrip tests by... 1. reducing repeate…

### #47521 — [[Quantization][INC] Support INT2 XPU Linear](https://github.com/vllm-project/vllm/pull/47521)
- **作者**: Zhenzhong1  **时间**: 2026-07-03 06:04 UTC
- **标签**: intel-gpu
- **摘要**: python3 examples/basic/offline_inference/generate.py --model OPEA/Qwen2.5-72B-Instruct-int2-sym-inc --block-size 64 --enforce-eager --max-model-len 512 --tensor-parallel-size 2 --gpu-memory-utilization 0.15  ```python -------------------------------------------------- Prompt: 'Hello, my name is' Gen…

### #47520 — [[Attention] Derive Triton 3D flash-decoding threshold from SM count a…](https://github.com/vllm-project/vllm/pull/47520)
- **作者**: tuananhlfc  **时间**: 2026-07-03 05:55 UTC
- **标签**: v1
- **摘要**: ## Problem The Triton unified-attention 2D/3D split-KV selector uses a batch-only heuristic (`MIN_LAUNCH_GRID_SIZE_2D // num_kv_heads`) that ignores KV length and device SM count. Above ~8 sequences, long-context decode drops from 3D split-KV to 2D, roughly doubling inter-token latency (hard cliff a…

### #47519 — [[ROCm][CI] Fix Kernels and Kernels attention test failures](https://github.com/vllm-project/vllm/pull/47519)
- **作者**: cpersson-amd  **时间**: 2026-07-03 05:38 UTC
- **标签**: rocm
- **摘要**: ## Purpose  This PR fixes failures on ROCm tests MI355 Kernels and Kernels Attention. The tests were failing due to a clone operation being attempted on the `prefill_backend` which was set to None for testing.  This PR also fixes a failure on the Kernels MoE test (`test_ocp_mx_moe.py`), which was fa…

### #47518 — [[ROCm][DSV4] Enable fused AITER mHC post+pre kernel for decode](https://github.com/vllm-project/vllm/pull/47518)
- **作者**: Fangzhou-Ai  **时间**: 2026-07-03 05:31 UTC
- **标签**: rocm
- **摘要**: ## Summary  Follow-up to #43950, which made the **unfused** AITER mHC `pre`/`post` kernels the default ROCm path for DeepSeek‑V4 and explicitly left the **fused** post+pre kernel unused ("aiter provides unfused pre/post ops but no combined fused post+pre alternative"). AITER does ship a fused `mhc_f…

### #47517 — [[Doc] Fix VLM2Vec benchmark chat template path](https://github.com/vllm-project/vllm/pull/47517)
- **作者**: kalyanamdewri  **时间**: 2026-07-03 05:28 UTC
- **标签**: documentation
- **摘要**: Summary: - Fix the VLM2Vec benchmarking example to point at the checked-in `examples/pooling/embed/template/vlm2vec_phi3v.jinja` template. - Keep the benchmarking docs consistent with the pooling model docs and example script. - Checked open issues and PRs for this path and did not find an existing …
