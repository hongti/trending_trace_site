# verl-project/verl — 动态追踪

> 生成时间: 2026-08-20 10:02 CST

## AI 总结

## verl-project/verl 近期动态摘要

---

### 🐛 Issue

- **#7476** — `ulysses_sequence_parallel_size` 在 Qwen3.5 上静默失效：设置该参数后不会产生任何警告，但序列实际上并未被分片，每个 rank 的激活张量仍保持全长度。该问题影响训练效率与正确性，需关注。

---

### 🔀 Pull Request

**功能新增**
- **#7484** [megatron] 将 **LoRA+**（`lr_B = lora_plus_ratio * lr_A`）和 **shared-outer MoE LoRA** 导出功能接入 Megatron 后端的优化器/Rollout 路径。
- **#7483** [vllm] 新增 **DeepSeek V4 LoRA** 支持（依赖多个 vllm 上游 PR）。

**Bug 修复**
- **#7485** [megatron] 修复 DDP 梯度缓冲区 dtype 与优化器主梯度精度不一致的问题，统一为 FP32。
- **#7475** [fsdp] 修复 FSDP2 在梯度累积阶段因 PyTorch 版本误读 `_unsharded_param` 而崩溃的问题，提升兼容性。

**文档**
- **#7477** 为 Ascend Docker 镜像补充支持的 tag 文档。

**依赖升级**（Dependabot 自动化）
- **#7481** vllm `0.24.0` → `0.27.1`（大版本跳跃）
- **#7479** transformers `5.3.0` → `5.15.0`
- **#7478** trl `0.27.0` → `1.10.0`（大版本跳跃）
- **#7482** nvidia-modelopt `0.44.0` → `0.45.0`
- **#7480** tensordict 版本上限放宽至 `0.14.0`

---

### 📦 Release

> 本期无新版本发布。

---

**总结**：本期重点在于 **Megatron 后端 LoRA 生态扩展**（LoRA+ / MoE LoRA / DeepSeek V4 LoRA）和 **稳定性修复**（FSDP2 崩溃、DDP dtype 对齐），同时有多项核心依赖（vllm、transformers、trl）进行了较大幅度版本升级，Issue 中暴露的 Ulysses 序列并行静默失效问题值得关注。

---

## 🐛 Issues

### #7476 — [ulysses_sequence_parallel_size is a silent no-op on Qwen3.5: per-rank activations stay full length](https://github.com/verl-project/verl/issues/7476)
- **作者**: khazic  **时间**: 2026-08-19 19:34 CST
- **摘要**: ### What happens  `engine.ulysses_sequence_parallel_size=8` is accepted, appears in the resolved config, produces no warning, and does not shard the sequence. Per-rank activation tensors stay at the full sequence length, so sequence parallelism buys no memory at all.  Found while trying to fit a 27.…

## 🔀 Pull Requests

### #7485 — [[megatron] fix: align DDP gradient dtype with optimizer precision](https://github.com/verl-project/verl/pull/7485)
- **作者**: Mecoli1219  **时间**: 2026-08-20 05:56 CST
- **摘要**: ### What does this PR do?   Fix the Megatron-Bridge DDP configuration so its gradient-buffer dtype matches the optimizer’s effective main-gradient precision.  The conventional optimizer expects FP32 gradients, but the Bridge path omitted `grad_reduce_in_fp32` and inherited Megatron-Core’s `False` de…

### #7484 — [[megatron] feat: wire LoRA+ and shared-outer MoE LoRA export to the optimizer/rollout path](https://github.com/verl-project/verl/pull/7484)
- **作者**: HollowMan6  **时间**: 2026-08-20 03:21 CST
- **摘要**: ### What does this PR do?  Wire two LoRA enhancements into the Megatron backend (logic lives in Megatron-Bridge; this PR is the verl-side plumbing):  - **LoRA+**: `lr_B = lora_plus_ratio * lr_A`. New `model.lora.lora_plus_ratio` config field (default `1.0` = disabled). `get_megatron_optimizer` forwa…

### #7483 — [[vllm] feat: Support DeepSeek V4 LoRA](https://github.com/verl-project/verl/pull/7483)
- **作者**: HollowMan6  **时间**: 2026-08-20 01:51 CST
- **摘要**: ### What does this PR do?  Depends on: - https://github.com/vllm-project/vllm/pull/51368 - https://github.com/vllm-project/vllm/pull/52552 - https://github.com/vllm-project/vllm/pull/52626 - https://github.com/vllm-project/vllm/pull/52986  Adds DeepSeek V4 (DSV4) LoRA support to verl's vLLM weight-s…

### #7482 — [build(deps-dev): bump nvidia-modelopt from 0.44.0 to 0.45.0](https://github.com/verl-project/verl/pull/7482)
- **作者**: dependabot[bot]  **时间**: 2026-08-20 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [nvidia-modelopt](https://github.com/NVIDIA/Model-Optimizer) from 0.44.0 to 0.45.0. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/NVIDIA/Model-Optimizer/releases">nvidia-modelopt's releases</a>.</em></p> <blockquote> <h2>ModelOpt 0.45.0 Release</h2>…

### #7481 — [build(deps-dev): bump vllm from 0.24.0 to 0.27.1](https://github.com/verl-project/verl/pull/7481)
- **作者**: dependabot[bot]  **时间**: 2026-08-20 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [vllm](https://github.com/vllm-project/vllm) from 0.24.0 to 0.27.1. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/vllm-project/vllm/releases">vllm's releases</a>.</em></p> <blockquote> <h2>v0.27.1</h2> <p>This is a patch release on top of v0.27.0.</…

### #7480 — [build(deps): update tensordict requirement from !=0.9.0,<=0.10.0,>=0.8.0 to >=0.8.0,!=0.9.0,<=0.14.0](https://github.com/verl-project/verl/pull/7480)
- **作者**: dependabot[bot]  **时间**: 2026-08-20 01:34 CST
- **标签**: dependencies, python
- **摘要**: Updates the requirements on [tensordict](https://github.com/pytorch/tensordict) to permit the latest version. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/pytorch/tensordict/releases">tensordict's releases</a>.</em></p> <blockquote> <h2>TensorDict v0.14.…

### #7479 — [build(deps): bump transformers from 5.3.0 to 5.15.0](https://github.com/verl-project/verl/pull/7479)
- **作者**: dependabot[bot]  **时间**: 2026-08-20 01:33 CST
- **标签**: dependencies, python
- **摘要**: Bumps [transformers](https://github.com/huggingface/transformers) from 5.3.0 to 5.15.0. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/transformers/releases">transformers's releases</a>.</em></p> <blockquote> <h2>Release: v5.15.0</h2> <h1>Relea…

### #7478 — [build(deps-dev): bump trl from 0.27.0 to 1.10.0](https://github.com/verl-project/verl/pull/7478)
- **作者**: dependabot[bot]  **时间**: 2026-08-20 01:33 CST
- **标签**: dependencies, python
- **摘要**: Bumps [trl](https://github.com/huggingface/trl) from 0.27.0 to 1.10.0. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/trl/releases">trl's releases</a>.</em></p> <blockquote> <h2>v1.10.0</h2> <h2>Features</h2> <h3>🎓 <code>DistillationTrainer</co…

### #7477 — [[doc] fix: add supported tag doc for ascend docker](https://github.com/verl-project/verl/pull/7477)
- **作者**: yyyy2000  **时间**: 2026-08-19 19:39 CST
- **摘要**: ### What does this PR do?   add supported tag doc for ascend docker  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, …

### #7475 — [[fsdp] fix: keep FSDP2 training alive on torch releases that mis-read _unsharded_param](https://github.com/verl-project/verl/pull/7475)
- **作者**: khazic  **时间**: 2026-08-19 18:03 CST
- **摘要**: ### What breaks  FSDP2 training dies during gradient accumulation:  ``` File "torch/distributed/fsdp/_fully_shard/_fsdp_param.py", line 733,      in to_accumulated_grad_if_needed     or self._unsharded_param.grad is None AttributeError: 'FSDPParam' object has no attribute '_unsharded_param' ```  `_u…
