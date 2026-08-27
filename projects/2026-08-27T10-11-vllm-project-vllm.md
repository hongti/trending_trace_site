# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-27 18:11 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库最近动态的中文简洁摘要：

### 🐛 Issue 动态
*   **前缀缓存复用失效问题** (#54027)：开启 DFlash2 (K=7) + YaRN 后，相同的 1.04M token 长提示无法复用前缀缓存（复用率为 0），而仅使用 target-only 静态 YaRN 时可复用约 1.039M token。

### 🔀 PR 动态
**1. 新特性与内核支持**
*   **新增 MXFP4 MoE 后端** (#54032)：为 SM90 架构引入第三个 MXFP4 MoE 后端 `flashinfer_cutlass_humming`，支持 DeepSeek-V4 系列专家权重路由。
*   **扩展 Triton MLA 支持** (#54031)：为 `TRITON_MLA_SPARSE` 增加 NoPE dim_qk=512 几何结构及 SM120 架构支持。
*   **Zen CPU 支持 W4A8 量化** (#54024)：为 Zen CPU 的 dense 和 MoE 层新增 DA8W4 (W4A8) int4 量化支持。

**2. 性能优化与重构**
*   **融合直接-KV 发布内存屏障** (#54030)：将 peer 发布与等待操作融合进一个 Triton 内核，并将 epoch 推进从主机端移至设备端，减少了内核启动开销。
*   **重构 EC Connector** (#54033)：为 `ECCPUWorker` 添加后端扩展点，使 vllm-ascend 等项目可直接复用上游的调度器、共享区域及保存/加载流程，避免代码重复。

**3. Bug 修复**
*   **构建错误** (#54034)：修复 GDN decode 声明守卫使用了错误宏定义的问题，该问题导致在 SM80 构建时启用了错误的 decode 逻辑。
*   **API 请求校验** (#54026)：修复格式错误的 OpenAI 兼容请求直接返回 HTTP 500 的问题，现改为正确返回校验错误信息。
*   **投机解码填充** (#54028)：修复当 `long_prefill_token_threshold` 小于统一投机宽度时，等待请求被错误填充的问题。
*   **CPU 注意力回退** (#54029)：修复 GLM-Moe-DSA 在 CPU 上无法回退到 dense attention 的问题（将 `index_topk=0` 视为禁用 sparse MLA 的标志）。

**4. 代码清理**
*   **MoE 工具归档** (#54025)：将 `flashinfer_utils.py` 从量化 utils 移至 `fused_moe` 目录下，因其主要被 MoE 专用代码（含 BF16 和未量化）使用。

---

### 🚀 Release 动态
**v0.28.0**
*   **版本亮点**：本次版本包含 584 个提交，来自 270 位贡献者（其中 76 位为新贡献者）。**核心更新是针对 Kimi-K3 模型进行了重大的性能优化推送**。

---

## 🐛 Issues

### #54027 — [[Bug]: DFlash2 + YaRN identical 1.04M prompt gets zero prefix-cache reuse while target-only reuses ~1.039M tokens](https://github.com/vllm-project/vllm/issues/54027)
- **作者**: rufftruffles  **时间**: 2026-08-27 17:13 CST
- **摘要**: ### Your current environment  <details> <summary>Relevant <code>python -m vllm.collect_env</code> output</summary>  ```text OS: AlmaLinux 10.2 (x86_64), glibc 2.39 Python: 3.12.13 GPU: NVIDIA RTX PRO 6000 Blackwell Max-Q Workstation Edition, 97,887 MiB Driver: 610.57.04 PyTorch: 2.13.0+cu132 CUDA us…

## 🔀 Pull Requests

### #54034 — [[Bugfix][Build] Fix GDN decode declaration guard](https://github.com/vllm-project/vllm/pull/54034)
- **作者**: jsntcheng  **时间**: 2026-08-27 18:06 CST
- **标签**: bug
- **摘要**: The fused_gdn_decode_post_conv_mtp declaration was guarded by VLLM_ENABLE_FUSED_KDA_DECODE instead of VLLM_ENABLE_FUSED_GDN_DECODE.  On SM80 builds, GDN decode is enabled while KDA decode is disabled, causing _C_stable_libtorch compilation to fail with an undeclared identifier. Use the correct guard…

### #54033 — [[Refactor][EC Connector] Add backend extension points to ECCPUWorker](https://github.com/vllm-project/vllm/pull/54033)
- **作者**: Akine-Ko  **时间**: 2026-08-27 18:02 CST
- **标签**: needs-rebase, cpu
- **摘要**: ## Purpose  Refactor `ECCPUWorker` so vllm-ascend can reuse the upstream ECCPU scheduler, shared region, and common save/load flow instead of maintaining duplicated copies.  Related vllm-ascend PR:  https://github.com/vllm-project/vllm-ascend/pull/13894  The current `ECCPUWorker` directly performs s…

### #54032 — [[Kernel] Add a FlashInfer SM90 MXFP4 x FP8 fused MoE backend](https://github.com/vllm-project/vllm/pull/54032)
- **作者**: zhengd-nv  **时间**: 2026-08-27 17:52 CST
- **标签**: documentation, nvidia
- **摘要**: ## Purpose  This adds a third MXFP4 MoE backend for SM90: `--moe-backend flashinfer_cutlass_humming`, which routes DeepSeek-V4-family MXFP4 expert weights through FlashInfer's CUTLASS "humming" kernel with kernel-internal FP8 activation quantization. It is strictly opt-in — it is not in any priority…

### #54031 — [[Attention][Kernel] TRITON_MLA_SPARSE: NoPE dim_qk=512 geometry and SM120 support](https://github.com/vllm-project/vllm/pull/54031)
- **作者**: ima-helikoptaaa  **时间**: 2026-08-27 17:48 CST
- **摘要**: ### Summary  Continues #47629 (a rebase/takeover of #38476), which added the pure-Triton sparse-MLA path for the 576-wide rope-MLA layout on SM80/SM121. Building on that, this extends the `TRITON_MLA_SPARSE` backend to the NoPE MLA layout (`dim_qk=512`, `qk_rope_head_dim=0`) used by glm5_next / GLM-…

### #54030 — [[Perf][PCP] Fuse direct-KV publication fence](https://github.com/vllm-project/vllm/pull/54030)
- **作者**: LopezCastroRoberto  **时间**: 2026-08-27 17:44 CST
- **标签**: deepseek, mrv2
- **摘要**: ## Summary  Follow-up to #52863.  - Fuse peer publication and waiting into one Triton kernel. - Move epoch advancement from the host onto the device. - Reduce each direct-KV fence from two kernel launches to one. - Make epoch advancement work correctly during CUDA graph replay.  ## Motivation  #5286…

### #54029 — [[Bugfix][CPU] Allow dense attention fallback for GLM DSA](https://github.com/vllm-project/vllm/pull/54029)
- **作者**: libaojiang  **时间**: 2026-08-27 17:39 CST
- **标签**: bug, deepseek, glm
- **摘要**: ## Purpose  Fixes #54018.  GLM-Moe-DSA currently treats the presence of `index_topk` as enabling sparse MLA, so its typed config provides no CPU-compatible opt-out. This PR:  - treats `index_topk=0` as an explicit dense-MLA opt-out on the generic GLM-Moe-DSA path; - leaves the default positive `inde…

### #54028 — [[Bugfix][Core] Skip spec padding below prefill threshold](https://github.com/vllm-project/vllm/pull/54028)
- **作者**: yydhYYDH  **时间**: 2026-08-27 17:31 CST
- **标签**: bug, scheduler
- **摘要**: ## Purpose  Fix incorrect speculative-decode padding when `long_prefill_token_threshold` is smaller than the uniform speculative width.  A waiting request that needs one token can be padded to `1 + num_speculative_tokens` so it shares the running decode batch shape. The threshold was applied after t…

### #54026 — [[Bugfix] Return validation errors for malformed OpenAI request fields](https://github.com/vllm-project/vllm/pull/54026)
- **作者**: LangQi99  **时间**: 2026-08-27 16:48 CST
- **标签**: bug, frontend, tool-calling
- **摘要**: ## Purpose  Prevent malformed OpenAI-compatible requests from returning HTTP 500 when `mode="before"` validators inspect raw JSON values before Pydantic validates their field types.  The affected validators previously called `len()`, `.get()`, `list()`, iterated, or indexed values that clients can s…

### #54025 — [[Cleanup][MoE] Move FlashInfer MoE helpers under fused_moe](https://github.com/vllm-project/vllm/pull/54025)
- **作者**: TANGBUDU  **时间**: 2026-08-27 16:39 CST
- **标签**: ci/build, nvidia, quantization
- **摘要**: ## Purpose  Fixes #31414.  `flashinfer_utils.py` currently lives under quantization utils, but the helpers are MoE-specific and are also used by BF16 and unquantized MoE code. This PR moves the module to `vllm.model_executor.layers.fused_moe.flashinfer` and updates the current in-tree callers and th…

### #54024 — [[CPU][Zen] Add DA8W4 (W4A8) int4 support for dense and MoE layers](https://github.com/vllm-project/vllm/pull/54024)
- **作者**: ganeshr10  **时间**: 2026-08-27 16:36 CST
- **摘要**: ## Purpose Symmetric int4 checkpoints can now run with dynamically int8-quantized activations on Zen CPUs: the dense path extends ZentorchWNA16LinearKernel, and a new ZEN_CPU WNA16 MoE backend repacks stacked experts to s4 for zentorch_fused_moe.  ## Test Plan  ## Test Result  --- <details> <summary…

## 🚀 Releases

### [v0.28.0](https://github.com/vllm-project/vllm/releases/tag/v0.28.0)
- **作者**: khluu  **时间**: 2026-08-26 17:46 CST
- **摘要**: # v0.28.0  ## Highlights  This release features 584 commits from 270 contributors (76 new)!  * **Kimi-K3 performance push**: a major optimization effort for Kimi-K3 across the stack — Decode Context Parallel (DCP) support (#50484), fused FlashKDA decode and prefill kernels (#50654, #51311, #52458), …
