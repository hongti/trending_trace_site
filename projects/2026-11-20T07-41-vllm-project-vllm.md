# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-20 15:41 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 📋 Issue 摘要
- **架构与容错改进探讨**：
  - **#68893**：提出分片 RDT 权重同步的容错改进方案（RFC），旨在当某个推理引擎发生故障时，其他存活的引擎能继续完成权重同步，避免单点故障导致全局失败。
  - **#68891**：探讨编码器解耦的 Token 级交接方案（RFC），目的是让 P/PD 节点免于重复执行聊天模板渲染、提示词分词及多模态占位符重建等繁杂工作。
- **新功能请求与 Bug 反馈**：
  - **#68890**：请求在 MRV2 prefill 上下文并行（PCP）模式下支持仅注意力 LoRA，目前该模式会在启动时拒绝 LoRA。
  - **#68885**：反馈 Muse-Spark-Open 多模态模型的 OOM Bug：编码器性能分析时硬编码使用 4 帧的虚拟视频，而模型实际最高可采样 600 帧，导致处理最大长度视频时所有 TP 进程 OOM。

### 🔨 PR 摘要
- **新模型与性能优化**：
  - **#68882**：新增对 Glide 27b 模型的支持。
  - **#68889**：为 DeepSeek-V4.1 在 SM90 PCP prefill 阶段启用 candidate-only scoring，提升处理性能。
  - **#68892**：修复 ROCm 平台 KV Connector 在 MI355X 上的内存分配报错（HIP error 209），引入 `AMD_GEM` 自定义内存池以优化显存管理。
- **重构与重要 Bug 修复**：
  - **#68888**：将 GLM-5.3 的 MegaMoE checkpoint 加载逻辑重构提取为共享的 block-FP8 专家层，便于其他模型复用。
  - **#68886**：修复 Issue #68885 中的 OOM Bug，使 Muse-Spark-Open 编码器在性能分析时使用真实帧数。
  - **#68880**：修复 `validate_config.py` 中的三个 AST 验证 Bug（涉及多装饰器、PEP 604 Literal union 及 ClassVar 检查）。
- **CI 与测试改进**：
  - **#68887**：修复 Python 3.12 冒烟测试中 Triton 构建失败的问题，改用 uv 管理 Python 环境。
  - **#68884 / #68881**：修复 XPU 平台测试中的内存余量不足和 Engram peer group 测试兼容性问题。
  - **#68883**：在低于 Hopper 架构的设备上跳过 HYV4Flash 的 PP 等价性测试，修复 Nightly 构建失败。

### 🚀 Release 摘要
- 本周期内无新的 Release 发布。

---

## 🐛 Issues

### #68893 — [[RFC]: Fault-tolerant sharded RDT weight sync: continue on surviving inference engines](https://github.com/vllm-project/vllm/issues/68893)
- **作者**: hao-aaron  **时间**: 2026-11-20 15:37 CST
- **标签**: RFC
- **摘要**: ### Motivation.  With the sharded RDT weight-transfer backend (as redesigned on the load plan in #68038, part of #64418), one failed inference engine fails the weight sync for every engine. An engine crash fails the round within seconds. A lost RPC fails it only after the 300 s HTTP timeout or stall…

### #68891 — [[RFC]: Token-Level Handoff for Encoder Disaggregation](https://github.com/vllm-project/vllm/issues/68891)
- **作者**: gty111  **时间**: 2026-11-20 15:34 CST
- **标签**: RFC
- **摘要**: ### Motivation.  The current Chat Completions EPD path can keep raw media off P/PD, but P/PD still renders the chat template, tokenizes the prompt, and reconstructs multimodal placeholders from model-specific metadata. E's multimodal processor has already computed that layout.  We propose forwarding…

### #68890 — [[Feature]: Support attention-only LoRA under MRV2 prefill context parallelism (PCP)](https://github.com/vllm-project/vllm/issues/68890)
- **作者**: zyulai  **时间**: 2026-11-20 15:32 CST
- **摘要**: ### 🚀 The feature, motivation and pitch  MRV2 prefill context parallelism (PCP, #46570) currently rejects LoRA at startup. In `vllm/v1/worker/gpu/pcp_manager.py:175` (upstream main `898a7c68c2`):  ```python if vllm_config.lora_config is not None:     raise NotImplementedError("MRV2 PCP does not supp…

### #68885 — [[Bug][Multimodal] Muse-Spark-Open: encoder profiling uses a 4-frame dummy while the model samples up to 600 — a single max-length video OOMs all TP ranks](https://github.com/vllm-project/vllm/issues/68885)
- **作者**: praveingk  **时间**: 2026-11-20 15:02 CST
- **标签**: multi-modality
- **摘要**: ### Describe the bug  Muse-Spark-Open's profiling dummy builds its video with a hardcoded `num_frames=4` (`vllm/models/muse_spark/common/processor.py`, `MuseSparkOmniDummyInputsBuilder.get_dummy_mm_data`), while the model's video path samples up to `num_frames_per_video` = **600** frames per video (…

## 🔀 Pull Requests

### #68892 — [[ROCm][KV Connector] MoRIIO: custom_mem_pool=AMD_GEM keeps the KV cache in amdgpu GEM BOs](https://github.com/vllm-project/vllm/pull/68892)
- **作者**: isytwu  **时间**: 2026-11-20 15:34 CST
- **标签**: documentation, rocm, kv-connector
- **摘要**: <!-- markdownlint-disable -->  ## Overview  On MI355X, MoRIIO PD serving can fail at the first requests with `HIP error 209` while VRAM is free: amdgpu counts `hipMalloc` VRAM that a NIC pins twice in the KFD allocation limit ([ROCm/mori#721](https://github.com/ROCm/mori/issues/721)). This PR adds a…

### #68889 — [[Perf][DSV4.1] Enable candidate-only scoring for SM90 PCP prefill](https://github.com/vllm-project/vllm/pull/68889)
- **作者**: lengrongfu  **时间**: 2026-11-20 15:24 CST
- **标签**: documentation, ci/build, deepseek
- **摘要**: ## Overview  Extend DeepSeek-V4.1's existing SM90 CuTe candidate-only indexer to eligible prefill with replicated-cache Prefill Context Parallelism (PCP). Users opt in explicitly. The implementation targets standalone eager, synchronous serving.  ## Claims  - Score only the existing candidate pool o…

### #68888 — [[Refactor][MoE] Extract reusable block-FP8 MegaMoE experts](https://github.com/vllm-project/vllm/pull/68888)
- **作者**: chaunceyjiang  **时间**: 2026-11-20 15:19 CST
- **标签**: ci/build
- **摘要**: ## Overview  Extract the GLM-5.3 MegaMoE checkpoint loader into the shared fused MoE layer as BlockFp8MegaMoEExperts, allowing other models to reuse its BF16 and block-FP8 loading logic.  ## Claims  - Preserve existing weight loading, scale finalization, shared-expert handling, and reload behavior. …

### #68887 — [[CI] Use a uv-managed Python in the Python 3.12 smoke test](https://github.com/vllm-project/vllm/pull/68887)
- **作者**: csahithi  **时间**: 2026-11-20 15:08 CST
- **摘要**: ## Overview  The nightly `(H200 MIG 18GB) [Nightly] Python 3.12 Smoke` job has failed in every build since #68457 switched it from Python 3.14 to 3.12. Triton can't build its driver stub: `fatal error: Python.h: No such file or directory`. This makes `python_only_compile.sh` ask uv for a managed int…

### #68886 — [[Bugfix][Multimodal] Profile the Muse-Spark-Open encoder at real frame counts](https://github.com/vllm-project/vllm/pull/68886)
- **作者**: praveingk  **时间**: 2026-11-20 15:03 CST
- **标签**: bug, multi-modality
- **摘要**: ## Overview  Fixes #68885. The Muse-Spark-Open profiling dummy built its video with a hardcoded `num_frames=4` while the model samples up to `num_frames_per_video` (600) frames per video, so the encoder was profiled at a ~150x smaller activation than a real maximum-length request — and a single 600-…

### #68884 — [[CI][XPU] Use test runner for sampling and ChatGLM LoRA tests](https://github.com/vllm-project/vllm/pull/68884)
- **作者**: hlin99  **时间**: 2026-11-20 15:00 CST
- **标签**: intel-gpu, glm
- **摘要**: This test directly constructs  LLM , bypassing the device-specific settings in  VllmRunner , so it uses the default gpu_memory_utilization=0.92 , which leaves insufficient memory headroom on XPU. Reuse  vllm_runner so each platform can apply its own settings (currently  0.9  for XPU).  <img width="2…

### #68883 — [[CI] Skip HYV4Flash in PP equivalence below Hopper](https://github.com/vllm-project/vllm/pull/68883)
- **作者**: JaredforReal  **时间**: 2026-11-20 14:51 CST
- **标签**: ready
- **摘要**: ## Overview  `(L4) [Nightly] PP Equivalence Shard 2/3` fails on `HYV4FlashForCausalLM` since the model landed in #68296 (nightly [104240](https://buildkite.com/vllm/ci/builds/104240), daily [104395](https://buildkite.com/vllm/ci/builds/104395)). It uses FlashMLA sparse attention like `HYV4ForCausalL…

### #68882 — [Feat/glide 27b support](https://github.com/vllm-project/vllm/pull/68882)
- **作者**: Newboyleo  **时间**: 2026-11-20 14:49 CST
- **标签**: documentation, new-model, frontend, qwen
- **摘要**: ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this PR.  - [ ] **Design Fit:** Minimizes…

### #68881 — [[CI][XPU] Fix Engram peer group test capability](https://github.com/vllm-project/vllm/pull/68881)
- **作者**: hlin99  **时间**: 2026-11-20 14:46 CST
- **标签**: intel-gpu
- **摘要**: to make test align with production code https://github.com/vllm-project/vllm/blob/main/vllm/config/engram.py#L219, otherwise the test fails on XPU and CUDA GPUs below SM89, since mocking  is_cuda()  alone is insufficient.

### #68880 — [[Pre-commit][Bugfix] Fix multi-decorator, PEP 604 Literal union, and qualified ClassVar checks in validate_config.py](https://github.com/vllm-project/vllm/pull/68880)
- **作者**: vibhanshu-s  **时间**: 2026-11-20 14:43 CST
- **标签**: bug
- **摘要**: ## Overview  Fixes three AST validation bugs in `tools/pre_commit/validate_config.py`: an unreachable multi-decorator error check, missed PEP 604 `Literal[...] | Literal[...]` unions, and missed module-qualified annotations/decorators (`typing.ClassVar`, `dataclasses.InitVar`, `typing.Union`, `typin…
