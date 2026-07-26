# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-26 09:07 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期的动态摘要：

### 🚀 Release (版本发布)
**v0.26.0**
- **版本亮点**：本版本包含 411 个提交，来自 212 位贡献者（其中 61 位为新加入的贡献者）。
- **核心新特性**：正式引入了全新的 **Inkling 模型家族** 并提供完整支持。

---

### 🐛 Issue (问题追踪)
**#49844 [Bug] PP=2 + GlmMoeDsa 模型输出乱码**
- 报告了在流水线并行（PP=2）配置下运行 GLM-5.2 (744B MoE) 模型时，若同时开启 TorchInductor 编译与 CUDA Graph 捕获，会产生垃圾输出；而单独使用其中一项则输出正常。该问题在 v0.24 和 v0.26 版本中均复现。

---

### 🔧 PR (拉取请求/代码变更)

**1. 新模型支持**
- **#49842 [Model] 支持 ERNIE 4.5 VL**：引入原生 Transformers 架构的 ERNIE 4.5 VL 模型。锁定不可变的原生快照以避免无限字体下载，并桥接了文本、视觉、MoE、图像与视频的复合架构。

**2. 核心架构与逻辑修复**
- **#49845 [Bugfix] 统一 KV block size 选择逻辑**：修复了混合使用不同注意力后端时，系统仅选取第一个后端的 block size 导致不兼容的问题，现在会选取所有后端均支持的 KV cache block size。
- **#49841 [Bugfix] Worker 初始化失败后清理分布式状态**：修复了 Worker 初始化失败时未能正确拆除模型并行和分布式状态的问题，避免状态残留。
- **#49846 [Bugfix] 修复测试中环境变量突变**：修复了测试代码设置 `VLLM_ENFORCE_STRICT_TOOL_CALLING=0` 导致依赖该变量默认值为 1 的测试失败的潜在 Bug。

**3. ROCm 生态优化与修复**
- **#49843 / #49836 [Bugfix][ROCm] CPU KV cache 批量 DMA 加载**：为 ROCm 加入了现有的 XPU batch-DMA 回退机制（用于 CPU 到 GPU 的 KV 传输），替代了原先直接加载原始共享 mmap 文件的 Triton 路径，CUDA 与 XPU 路径保持不变。
- **#49839 [Test][ROCm] 适配 gfx950 FP8 RMSNorm 舍入**：针对 gfx950 架构融合 RMSNorm 的 FP8 舍入差异，调整了测试的 `allclose` 约定，将可接受差异限制在 1 个 FP8 ULP 内。
- **#49837 [CI][ROCm] NFS 上的 hf-xet 安全重建**：在 AMD 硬件测试 runner 中检测 NFS 挂载的 `HF_HOME`，将临时的 Xet 状态移至本地存储，避免共享 NFS 上的并发冲突。

**4. 基础设施与工具链**
- **#49840 [Bugfix] 关闭私有 Tensorizer 引擎**：显式关闭序列化后由 `tensorize_vllm_model` 持有的渲染器和 EngineCore，并在清理异常时保留主要序列化错误信息。
- **#49838 [Test] 新增 HF 缓存工件验证器**：添加了可选的测试会话 fixture，用于验证 Hugging Face Hub 快照或文件的 Git/LFS 内容地址及大小，确保缓存完整性。

---

## 🐛 Issues

### #49844 — [[Bug]: PP=2 + GlmMoeDsa: inductor compile combined with CUDA-graph capture produces garbage output; either alone is clean (v0.24 & v0.26)](https://github.com/vllm-project/vllm/issues/49844)
- **作者**: divyvasal  **时间**: 2026-07-26 07:25 CST
- **摘要**: ### Your current environment  - vLLM: reproduced on both **v0.24.0** and **v0.26.0** (official `vllm/vllm-openai` images) - Model: GLM-5.2 (`GlmMoeDsaForCausalLM`, 744B MoE, 78 layers, 256 routed experts), NVFP4 compressed-tensors checkpoint (FP4 routed experts; attention / shared experts / dense la…

## 🔀 Pull Requests

### #49846 — [[Bugfix] Fix VLLM_ENFORCE_STRICT_TOOL_CALLING mutation in tests](https://github.com/vllm-project/vllm/pull/49846)
- **作者**: yzong-rh  **时间**: 2026-07-26 07:49 CST
- **标签**: bug, tool-calling
- **摘要**: ## Purpose  Fix a code smell / latent bug where tests sets env variable VLLM_ENFORCE_STRICT_TOOL_CALLING=0, potentially leading to tests assuming  VLLM_ENFORCE_STRICT_TOOL_CALLING=1 to fail.  Occurred for https://github.com/vllm-project/vllm/pull/45560.  ## Test Plan  ``` pytest  tests/parser ```  #…

### #49845 — [[Bugfix] Pick a KV block size supported by every attention backend](https://github.com/vllm-project/vllm/pull/49845)
- **作者**: divyvasal  **时间**: 2026-07-26 07:47 CST
- **标签**: bug, v1
- **摘要**: ## Purpose  Fixes #48286.  `Platform.update_block_size_for_backend()` picks the KV cache block size from the **first** non-SSM attention backend only. Models that mix attention backends with disjoint block-size support break: GLM-DSA / DeepSeek-V3.2-style models pair a sparse-MLA main backend (`FLAS…

### #49843 — [[Bugfix][ROCm] Use batch DMA for CPU KV cache loads](https://github.com/vllm-project/vllm/pull/49843)
- **作者**: AndreasKaratzas  **时间**: 2026-07-26 05:49 CST
- **标签**: bug, rocm, v1
- **摘要**: - Add ROCm to the existing XPU batch-DMA fallback for CPU-to-GPU KV transfers. - Keep CUDA, XPU, and every other prior dispatch path unchanged.  The ROCm Triton path directly loaded a raw shared-mmap host pointer. The reproduced fault address matched the mmap base plus the selected block offset in t…

### #49842 — [[Model] Support native Transformers ERNIE 4.5 VL](https://github.com/vllm-project/vllm/pull/49842)
- **作者**: AndreasKaratzas  **时间**: 2026-07-26 05:49 CST
- **标签**: multi-modality
- **摘要**: - Pin the immutable native Transformers snapshot instead of mutable remote code that performs an unbounded font download during import. - Bridge the native composite text, vision, MoE, image, and video contracts while retaining legacy flat-config behavior. - Preserve native image/video token identit…

### #49841 — [[Bugfix] Clean distributed state after worker initialization failure](https://github.com/vllm-project/vllm/pull/49841)
- **作者**: AndreasKaratzas  **时间**: 2026-07-26 05:49 CST
- **标签**: bug, v1
- **摘要**: - Tear down model-parallel and distributed state when a spawned worker fails before `WorkerProc` construction completes. - Add a focused regression for cleanup order on the partial-initialization path.  With `spawn`, constructor failure can occur after global groups and their shared-memory queue are…

### #49840 — [[Bugfix] Shut down private Tensorizer engines](https://github.com/vllm-project/vllm/pull/49840)
- **作者**: AndreasKaratzas  **时间**: 2026-07-26 05:49 CST
- **标签**: bug
- **摘要**: - Explicitly close the renderer and EngineCore owned by `tensorize_vllm_model` after serialization. - Preserve the primary serialization error while attempting both cleanup paths and give nested workers a bounded reap grace. - Use the existing runner context in the direct serialization test.  The pr…

### #49839 — [[Test][ROCm] Account for gfx950 FP8 RMSNorm rounding](https://github.com/vllm-project/vllm/pull/49839)
- **作者**: AndreasKaratzas  **时间**: 2026-07-26 05:49 CST
- **标签**: rocm
- **摘要**: - Use the shared FP8 `allclose` contract only for the measured gfx950 fused RMSNorm cases. - Bound every accepted difference to one FP8 ULP while leaving scales, residuals, other dtypes, other architectures, and other ROCm checks unchanged.  E4M3 adjacent codes can differ by 12.5%, and its minimum s…

### #49838 — [[Test] Add a Hugging Face cache artifact verifier](https://github.com/vllm-project/vllm/pull/49838)
- **作者**: AndreasKaratzas  **时间**: 2026-07-26 05:49 CST
- **摘要**: - Add the requested opt-in session fixture for verifying a materialized Hub snapshot or one file beneath it. - Check Git and LFS content addresses and recorded sizes, and memoize a digest only while the file identity is unchanged.  The fixture is explicit: it does not intercept, repair, or claim to …

### #49837 — [[CI][ROCm] Make hf-xet reconstruction safe on shared NFS](https://github.com/vllm-project/vllm/pull/49837)
- **作者**: AndreasKaratzas  **时间**: 2026-07-26 05:48 CST
- **标签**: rocm, ci/build
- **摘要**: - Detect an NFS-backed `HF_HOME` in the AMD hardware test runner. - Keep Xet and the persistent Hub cache enabled while moving ephemeral Xet state to local storage and disabling high-performance vectored reconstruction only on NFS.  hf-xet stalled on the large vectored write after the first tokenize…

### #49836 — [[Bugfix][ROCm] Use batch DMA for CPU KV cache loads](https://github.com/vllm-project/vllm/pull/49836)
- **作者**: AndreasKaratzas  **时间**: 2026-07-26 05:14 CST
- **标签**: bug, rocm, v1
- **摘要**: ## Summary - Add ROCm to the existing XPU batch-DMA fallback for CPU-to-GPU KV transfers. - Keep CUDA, XPU, and every other prior dispatch path unchanged.  The ROCm Triton path directly loaded a raw shared-mmap host pointer. The reproduced fault address matched the mmap base plus the selected block …

## 🚀 Releases

### [v0.26.0](https://github.com/vllm-project/vllm/releases/tag/v0.26.0)
- **作者**: khluu  **时间**: 2026-07-25 18:38 CST
- **摘要**: # vLLM v0.26.0 Release Notes  ## Highlights  This release features 411 commits from 212 contributors (61 new)!  * **New Inkling model family** with a full support stack: base modeling (#48799), piecewise CUDA graph support (#48822), Hopper FA4 relative attention (#48858), MTP=1 speculative decoding …
