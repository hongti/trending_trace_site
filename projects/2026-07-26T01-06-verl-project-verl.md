# verl-project/verl — 动态追踪

> 生成时间: 2026-07-26 09:06 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期的动态摘要：

### 🐛 Issue 动态
*   **FSDP 梯度检查点 Bug (#7146)**：使用 FSDP backend 训练 Nemotron-H 模型时，无法成功开启梯度检查点功能。
*   **长时间训练 OOM 问题 (#7145)**：在 verl 0.7.1 版本中，使用 DAPO 算法对 Qwen3-4B-instruct 进行长时间强化训练时，出现显存溢出（OOM）错误。

### 🛠 PR 动态 (重点变更与修复)

**✨ 新特性**
*   **分片 Delta 权重同步及 HF 导出 (#7144)**：为 Engine Core 和 FSDP backend 引入了 `delta_sharded` 分片权重同步机制，并支持由 Backend 控制的 HuggingFace 格式导出，扩展了 FSDP2 的 `Shard(0)` 功能，优化了 Checkpoint 与权重管理。
*   **新增 Ascend NPU Megatron Backend (#7142)**：为 Ascend NPU 添加了 `megatron_adaptor+te-npu` 计算后端（注册为 `strategy="megatron_adaptor"`），作为 MindSpeed 路径的替代方案，复用了通用 Megatron 逻辑。

**🔧 修复与优化**
*   **vLLM MoE 显存优化 (#7143)**：在 vLLM 逐层重载 MoE 权重时，通过提前过滤掉非本地的 Expert 权重，避免了不必要的 CUDA 缓冲区克隆，显著降低了 Rollout 阶段的 GPU 显存占用。
*   **vLLM Legacy Loader 兼容性修复 (#7147)**：修复了在使用模块化 vLLM 导入 `verl.utils.vllm` 时出现的 `AttributeError`（`weight_loader` 属性缺失），为旧版 FusedMoE loader 增加了保护补丁。
*   **Ascend CI 修复 (#7148)**：更新并修复了 Ascend nightly CI 的运行错误。

### 🚀 Release 动态
*   近期无新版本 Release 发布。

---

## 🐛 Issues

### #7146 — [[Bug][FSDP] Nemotron-H actors cannot enable gradient checkpointing](https://github.com/verl-project/verl/issues/7146)
- **作者**: CedricHwong  **时间**: 2026-07-25 13:13 CST
- **摘要**: ## System Info  - **verl:** `main@c791da0bfcd7d7b560b1e461d2c188145b39c353` - **Transformers:** `5.3.0` - **PyTorch:** `2.11.0+cu130` - **GPU:** NVIDIA H200 (141 GB) - **Training backend:** FSDP actor, BF16, LoRA - **Model:** `NVIDIA-Nemotron-3-Nano-30B-A3B-BF16` (`model_type=nemotron_h`) - **Rollou…

### #7145 — [OOM error is reported during long-time training in version 0.7.1 of verl.](https://github.com/verl-project/verl/issues/7145)
- **作者**: iiiiice-seabird  **时间**: 2026-07-25 10:48 CST
- **摘要**: When the dapo algorithm is used to perform reinforcement training on qwen3-4b-instruct-2057, a memory overflow error is reported.  The error information in the log is as follows: PID:xxx Memory_Allocation_Failure(EL0004): Failed to allocate memory requested by APP module. xxx Possible Cause: Availab…

## 🔀 Pull Requests

### #7148 — [[ci] fix: update Ascend nightly CI](https://github.com/verl-project/verl/pull/7148)
- **作者**: Mengyuyang  **时间**: 2026-07-25 15:35 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review. Fix ascend nightly CI error. <img width="941" height="73" alt="image" src="https://github.com/user-attachments/assets/d870da98-3ba7-…

### #7147 — [[vllm] fix: guard legacy FusedMoE loader patch](https://github.com/verl-project/verl/pull/7147)
- **作者**: Mengyuyang  **时间**: 2026-07-25 15:05 CST
- **摘要**: ### What does this PR do?  Fixes `AttributeError: 'function' object has no attribute 'weight_loader'` when importing `verl.utils.vllm` with modular vLLM.Use latest vllm-ascend&verl&vllm. <img width="834" height="66" alt="PixPin_2026-07-25_15-32-29" src="https://github.com/user-attachments/assets/771…

### #7144 — [[ckpt, fsdp] feat: sharded delta block placements + backend-owned HF export (engine core + FSDP)](https://github.com/verl-project/verl/pull/7144)
- **作者**: ChangyiYang  **时间**: 2026-07-25 10:39 CST
- **摘要**: ## What does this PR do?  First half of the #7085 split requested in review: the sharded delta weight sync (`delta_sharded`, #6974) engine core and the FSDP backend. Extends the flat FSDP2 `Shard(0)` fast path to **block placements** (`Shard(k)`, multi-Shard-dim meshes, manual splits) and moves the …

### #7143 — [[rollout, vllm] fix: skip non-local experts before reload cloning](https://github.com/verl-project/verl/pull/7143)
- **作者**: Mecoli1219  **时间**: 2026-07-25 09:36 CST
- **摘要**: ### What does this PR do?  This PR reduces GPU memory used by vLLM layerwise MoE weight reloads by filtering expert weights that are not local to the current EP worker before Verl clones the reused CUDA-IPC buffer view.  Both Megatron and FSDP reconstruct logical Hugging Face tensors before sending …

### #7142 — [[megatron] feat(npu): add megatron_adaptor+te-npu backend for Ascend NPU](https://github.com/verl-project/verl/pull/7142)
- **作者**: Bruce-rl-hw  **时间**: 2026-07-25 08:10 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Adds a megatron_adaptor engine backend for Ascend NPU, as an alternative to the MindSpeed path. It registers under strategy="megatron_adaptor" and reuses the generic MegatronEngineWithLMHead/ValueHead (a thin ~80-line shell), applying the Ascend NPU patches once at import …
