# verl-project/verl — 动态追踪

> 生成时间: 2026-10-20 13:10 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📌 Issue 动态
本期未提供相关 Issue 动态。

### 📦 Release 动态
本期未提供相关 Release 信息。

### 🔧 Pull Request 动态
近期 PR 主要集中在**性能分析增强、硬件适配（特别是 TPU）、关键 Bug 修复以及架构重构**。重要变更如下：

**1. 架构重构与新特性**
*   **统一 QAT 与低精度 Rollout** (#8287) ⚠️ *Breaking Change*：重构了 Megatron QAT 和低精度 rollout 逻辑，围绕统一的训练端权重量化和推理端 refit 边界进行设计，增加了对 `compressed-tensors` 的兼容性。
*   **Nsight 性能分析增强** (#8289, #8290)：支持对 `RayWorkerGroup` 外部创建的 Reward 环路 worker 和 vLLM rollout server 进行独立的性能分析；修复了 Nsight 自动捕获窗口计数与 PPO Trainer 不匹配的问题。

**2. 性能优化**
*   **ReplayBuffer 唤醒机制优化** (#8291)：移除了硬编码的 `2.0s` 轮询间隔，改为在 prompt 完成时直接唤醒 ReplayBuffer，减少了 TransferQueue 元数据同步的等待时间。

**3. 硬件适配与兼容性**
*   **TPU 支持完善** (#8288, #8292)：新增基于公共包构建的 TPU 镜像、双芯片 GRPO 冒烟测试，并修复了 TPU GRPO 镜像因缺少 `orjson` 模块导致导入失败的问题。
*   **移除硬编码设备限制** (#8284)：将单元测试和 Triton 线性交叉熵内核主机代码中硬编码的 `"cuda"` 和 `"nccl"` 替换为统一的 `verl.utils.device` 接口，提升对非 CUDA 硬件的兼容性。

**4. 关键 Bug 修复**
*   **vLLM 推理修复** (#8293)：修复了 vLLM 自动选择的 DeepSeek-V4 MXFP4 MoE backend 因权重后处理未走 `replace_parameter` 而导致无法重拟合的问题。
*   **Megatron 优化器修复** (#8286)：修复了开启 `optimizer_cpu_offload` 时 `HybridDeviceOptimizer` 状态在 offload 过程中被破坏的问题。
*   **多模态模型修复** (#8285)：修复了 Qwen2-VL 和 GLM-4V 在不支持的设备上使用 flash-attention 时引发的裸 `NameError`。

---

## 🔀 Pull Requests

### #8293 — [[rollout, vllm] fix: keep vLLM's auto-selected DeepSeek-V4 MXFP4 MoE backend refittable](https://github.com/verl-project/verl/pull/8293)
- **作者**: linmuchuiyang  **时间**: 2026-10-20 11:34 CST
- **摘要**: ### What does this PR do?  Fixes #8236.  vLLM's `select_deepseek_v4_mxfp4_moe_backend` prefers FlashInfer TRT-LLM on SM100+. That kernel's weight post-processing does not go through `replace_parameter`, so verl's MXFP4 refit cannot replay it. With `rollout.engine_kwargs.vllm.moe_backend` left at `au…

### #8292 — [[hardware] fix: pin tensordict and install orjson in the TPU GRPO image](https://github.com/verl-project/verl/pull/8292)
- **作者**: jialei777  **时间**: 2026-10-20 11:26 CST
- **摘要**: ### What does this PR do?  Fixes the TPU GRPO image (`examples/tpu/grpo/Dockerfile`, from #8288): an image built today fails at `import verl` with  ``` ModuleNotFoundError: No module named 'orjson' ```  `verl/utils/tracking.py` imports `orjson`, which verl does not list as a dependency. The image us…

### #8291 — [[trainer, perf] feat: wake ReplayBuffer on prompt completion instead of 2 s polling](https://github.com/verl-project/verl/pull/8291)
- **作者**: lixiufei-leo  **时间**: 2026-10-20 11:11 CST
- **摘要**: ### What does this PR do?  `ReplayBuffer._wait_for_next_poll()` sleeps a hard-coded `poll_interval=2.0` between TransferQueue metadata syncs (used by both the sync `sample()` and the async `wait_for_sampleable()`). The trainer therefore notices the last finished prompt up to 2 s late, and `timing_s/…

### #8290 — [[2/3][perf, trainer] fix: count and forward worker profiling windows](https://github.com/verl-project/verl/pull/8290)
- **作者**: jiefangxuanyan  **时间**: 2026-10-20 10:56 CST
- **摘要**: ### What does this PR do?  Make Nsight's automatic capture count match the windows opened by PPO trainers. With `discrete: false` and steps `[1, 2, 5]`, per-step profiling opens three windows, while continuous profiling opens two. Counting three in the continuous case makes Nsight wait for a capture…

### #8289 — [[1/3][perf, reward, rollout] feat: support profiling independent Ray actors](https://github.com/verl-project/verl/pull/8289)
- **作者**: jiefangxuanyan  **时间**: 2026-10-20 10:56 CST
- **摘要**: ### What does this PR do?  Add Nsight profiling support for reward-loop workers and vLLM rollout servers created outside `RayWorkerGroup`, with role-local rank selection and a separate GPU metrics collection process. These processes need their own Ray launch options and profiler controls to appear i…

### #8288 — [[hardware, doc] feat: TPU examples: public-package image, 2-chip GRPO smoke test, DP shard size from TP, SFT torchrun fixes](https://github.com/verl-project/verl/pull/8288)
- **作者**: jialei777  **时间**: 2026-10-19 23:44 CST
- **摘要**: ### What does this PR do?  Combines three TPU example PRs into one so they can be reviewed and merged together, and builds the TPU image only from public packages. It supersedes #8266, #8271 and #8280. Only `examples/tpu/` changes; no library code.  ### Changes  **1. 2-chip GRPO smoke test (from #82…

### #8287 — [[BREAKING][megatron, vllm, sglang, rollout, doc] refactor: unify QAT and low-precision rollout](https://github.com/verl-project/verl/pull/8287)
- **作者**: wuxibin89  **时间**: 2026-10-19 20:46 CST
- **摘要**: ### What does this PR do?  This PR refactors Megatron QAT and low-precision rollout around a unified trainer-side weight quantization and rollout-side refit boundary.  - Add compressed-tensors-compatible trainer-side exporters for FP8_BLOCK, MXFP8, MXFP4, and NVFP4. - Add ModelOpt-backed Megatron QA…

### #8286 — [[megatron] fix: leave HybridDeviceOptimizer state untouched in optimizer offload (#8237)](https://github.com/verl-project/verl/pull/8286)
- **作者**: Jackie2049  **时间**: 2026-10-19 19:59 CST
- **摘要**: ### What does this PR do?  Megatron's `optimizer_cpu_offload=True` turns the optimizer into a **HybridDeviceOptimizer (HDO)**: it permanently splits optimizer state between CPU memory and GPU memory via `optimizer_offload_fraction`. The GPU-side state is owned by a fused CUDA optimizer that can only…

### #8285 — [[model] fix: avoid bare NameError in Qwen2-VL/GLM-4V flash-attn forward on unsupported devices](https://github.com/verl-project/verl/pull/8285)
- **作者**: kahlun  **时间**: 2026-10-19 18:05 CST
- **摘要**: ### What does this PR do?  `_custom_flash_attention_forward` in `glm4v.py`/`qwen2_vl.py` referenced `flash_attn_func`/`flash_attn_varlen_func` unconditionally, but they were only ever defined inside the `is_flash_attn_2_available()`/`is_npu_available` branches. On any other device (e.g. XPU, or CUDA…

### #8284 — [[hardware, test] fix: convert hardcoded cuda/nccl from unit tests and linear cross-entropy kernel host code](https://github.com/verl-project/verl/pull/8284)
- **作者**: kahlun  **时间**: 2026-10-19 17:58 CST
- **摘要**: ### What does this PR do?  Replace hardcoded `"cuda"`, `torch.cuda.*`, and the `"nccl"` backend in unit tests and the Triton linear cross-entropy kernel host code with the existing `verl.utils.device` helpers (`get_device_name`, `get_torch_device`, `get_device_id`, `get_nccl_backend`, `is_device_ava…
