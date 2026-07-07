# verl-project/verl — 动态追踪

> 生成时间: 2026-07-07 11:38 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 🐛 Issue 概况
1. **GPU 显存泄漏 (#6953)**：使用 Megatron 进行 CPU offload 时，Transformer-Engine 的 FP8 weight workspaces 未被释放，导致在 FP8/native-FP8 检查点下出现显存泄漏。
2. **Docker 兼容性故障 (#6949)**：Docker 镜像 `verlai/verl:vllm023.dev1` 在 B300/GB300 (sm_103) 架构上运行崩溃。
3. **MTP 权重异常 (#6947)**：开启 MTP 训练后，保存的 MTP 权重似乎保持不变，未发生预期更新。

### 🔧 PR 概况（重要变更与修复）

**🔑 核心修复（高优先级）**
- **修复 Megatron FP8 显存泄漏 (#6952)**：与 Issue #6953 对应，在模型 CPU offload 时补充了清理 Transformer-Engine FP8 weight-workspace 缓存的逻辑，彻底解决显存泄漏问题。
- **修复 SPMD 与 Activation Checkpointing 组合崩溃 (#6950)**：修复了在 `spmd_backend=spmd_types` + `selective activation checkpoint` + 未开启 torch compile 的组合下，Actor 反向传播阶段引发的 `SpmdTypeError` 崩溃问题。
- **修复 FSDP Logits 原地操作 Bug (#6945)**：避免对 squeezed logits view 进行原地 temperature scaling，防止潜在的数据污染或计算错误。

**🚀 架构与功能更新**
- **Megatron Bridge 设为默认 (#6951)**：弃用 vanilla mBridge 并添加弃用警告，正式将 Megatron Bridge 作为默认选项。
- **适配 Qwen3.5 (#6955, WIP)**：为即将发布的 Verl 0.8.0 版本 Docker 容器适配 Qwen3.5 模型。
- **新增 Titan Engine 支持 (#6954)**：为 Titan Engine 添加了说明文档和 CI 测试。
- **新增 NPU 每夜 CI (#6956)**：引入针对 NPU 硬件的自动化每夜构建与测试流水线。

**⚡ 性能与计算修正**
- **修正 AMD GPU FLOPS 值 (#6942)**：更新了 AMD CDNA 系列 GPU（如 MI350X 等）的 BF16/FP16 峰值算力表，确保 MFU 计算的准确性。
- **修正 Qwen-VL FLOPs 计算器 (#6944)**：修复了 `flops_counter.py` 中 Qwen3-VL 的 `head_dim` 不一致及 merger token 计数错误，并更新了相关测试基线。
- **移除 NPU 脚本中的 Eager 模式 (#6948)**：因在 vllm018 上引起性能劣化（融合算子被拆分为小算子），从 NPU 脚本中移除了 eager 模式配置。

### 📦 Release 概况
近期**无新版本发布**动态。（注：PR #6955 提到了为 Verl 0.8.0 版本做准备，但该版本尚未正式 Release）。

---

## 🐛 Issues

### #6953 — [[megatron] Transformer-Engine FP8 weight workspaces not freed by offload_megatron_model_to_cpu (GPU memory leak on FP8 / native-FP8 checkpoints)](https://github.com/verl-project/verl/issues/6953)
- **作者**: YQ-Wang  **时间**: 2026-07-07 07:30 CST
- **标签**: bug
- **摘要**: ### System Info  VeRL from the official docker image.  ### Information  - [ ] The official example scripts - [x] My own modified scripts  ### Tasks  - [ ] An officially supported task in the `examples` folder (such as GLUE/SQuAD, ...) - [x] My own task or dataset (give details below)  ### Reproducti…

### #6949 — [[docker] verlai/verl:vllm023.dev1 breaks on B300/GB300 (sm_103)](https://github.com/verl-project/verl/issues/6949)
- **作者**: borisfom  **时间**: 2026-07-07 02:17 CST
- **标签**: bug
- **摘要**: ### System Info  VeRL installed from source/trunk.  Environment: verlai/verl:vllm023.dev1 container on B300/GB300   ### Information  - [x] The official example scripts - [ ] My own modified scripts  ### Tasks  - [x] An officially supported task in the `examples` folder (such as GLUE/SQuAD, ...) - [ …

### #6947 — [Has anyone noticed that the saved MTP weights remain unchanged after enabling MTP training?](https://github.com/verl-project/verl/issues/6947)
- **作者**: zwc163  **时间**: 2026-07-06 20:31 CST
- **摘要**: Has anyone noticed that the saved MTP weights remain unchanged after enabling MTP training? I tested it on verl-0.7.1 actor_rollout_ref.model.mtp.detach_encoder=True

## 🔀 Pull Requests

### #6956 — [add npu nightly ci](https://github.com/verl-project/verl/pull/6956)
- **作者**: chengminhua  **时间**: 2026-07-07 10:18 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #6955 — [【wip】[env] chore: Adapt qwen3.5 to Docker containers for Verl release 0.8.0](https://github.com/verl-project/verl/pull/6955)
- **作者**: ruanhao566  **时间**: 2026-07-07 10:15 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?   Adapt qwen3.5 to Docker containers for Verl release 0.8.0  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}…

### #6954 — [[trainer] fix: Add readme and CI test for Titan Engine](https://github.com/verl-project/verl/pull/6954)
- **作者**: acisseJZhong  **时间**: 2026-07-07 08:21 CST
- **摘要**: As title

### #6952 — [[megatron] fix: free Transformer-Engine FP8 weight workspaces on CPU offload](https://github.com/verl-project/verl/pull/6952)
- **作者**: alexxu-roblox  **时间**: 2026-07-07 05:11 CST
- **摘要**: Co-authored with @YQ-Wang.  ### What does this PR do?  `offload_megatron_model_to_cpu` frees DDP buffers and walks `.parameters()`, but never clears Transformer-Engine's FP8 weight-workspace caches (`module._fp8_workspaces`). With an FP8 param dtype or a natively-FP8 checkpoint, TE `Linear` / `Group…

### #6951 — [[megatron] chore: deprecate vanilla mBridge and make Megatron Bridge default](https://github.com/verl-project/verl/pull/6951)
- **作者**: HollowMan6  **时间**: 2026-07-07 05:11 CST
- **摘要**: ### What does this PR do?  Add vanilla mBridge deprecation warning and make Megatron Bridge default.  ### Checklist Before Starting  - [X] Search for similar PRs. Paste at least one query link here: ... - [X] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)…

### #6950 — [[trainer] fix: spmd_types and activation checkpointing composability bug ](https://github.com/verl-project/verl/pull/6950)
- **作者**: acisseJZhong  **时间**: 2026-07-07 03:59 CST
- **摘要**: With `spmd_backend=spmd_types` + `activation_checkpoint=selective` + `use_torch_compile=False`, training crashes during the actor backward:  ``` spmd_types.types.SpmdTypeError: assert_type(...) requires an active mesh, but no current mesh is set. ```  ## Root cause spmd.assert_type needs the thread-…

### #6948 — [[doc] fix: no use eager in npu scrip](https://github.com/verl-project/verl/pull/6948)
- **作者**: yyyy2000  **时间**: 2026-07-06 22:17 CST
- **摘要**: ### What does this PR do?   eager模式在vllm018上存在性能劣化，因为vllm-ascend为了易用性，将融合算子split_qkv_rmsnorm_rope_kernel替换成了小算子  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked…

### #6945 — [[fsdp] fix: FSDP logits temperature scaling for view tensors](https://github.com/verl-project/verl/pull/6945)
- **作者**: zjchenn  **时间**: 2026-07-06 19:00 CST
- **摘要**: ### What does this PR do?  This PR updates the FSDP language-model eager logits path to avoid in-place temperature scaling on a squeezed logits view.  `output.logits.squeeze(0)` can return a view. For models whose logits are produced through custom autograd functions, applying `div_()` to that view …

### #6944 — [fix(flops_counter): correct Qwen-VL head_dim consistency and merger token count (#6903)](https://github.com/verl-project/verl/pull/6944)
- **作者**: chethanuk  **时间**: 2026-07-06 18:38 CST
- **摘要**: ### What does this PR do?  Corrects two objective inaccuracies in the Qwen3-VL FLOPs estimators in `verl/utils/flops_counter.py`, and refreshes the affected test goldens. Addresses verl-project/verl#6903 (claims 1 and 4 only).  1. **`head_dim` sourcing (`_estimate_qwen3_vl_flops`)** — read `head_dim…

### #6942 — [[hardware, perf] fix: update AMD GPU FLOPS values](https://github.com/verl-project/verl/pull/6942)
- **作者**: Vivicai1005  **时间**: 2026-07-06 17:35 CST
- **摘要**: Updates the _DEVICE_FLOPS table in verl/utils/flops_counter.py to reflect AMD's official BF16/FP16 dense matrix peak for CDNA GPUs, so MFU is computed correctly on these devices:    - Adds MI350X (2.3 PFLOPS dense) and MI355X (2.5 PFLOPS dense) — previously unlisted, so get_device_flops() returned i…
