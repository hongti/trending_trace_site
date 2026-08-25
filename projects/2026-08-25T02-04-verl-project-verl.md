# verl-project/verl — 动态追踪

> 生成时间: 2026-08-25 10:04 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 🐛 Issue 动态

*   **#7540 [distillation][bug] OPD 多教师蒸馏损失在 v0.9.0 与 HEAD 版本间出现发散**
    作者报告在代码数值逻辑未变的情况下，当前 HEAD 版本的 OPD 多教师蒸馏 loss 相比 v0.9.0 出现异常发散，该问题已在 NVIDIA B300 与寒武纪 MLU 硬件上复现。

---

### 🔀 Pull Request 动态

近期的 PR 主要集中在**硬件适配（NPU/Ascend）**、**序列化与内存修复**以及**训练逻辑修复**三大方向：

**1. 硬件适配与支持（NPU / Ascend）**
*   **#7535 [feat]** 新增 Ascend (Atlas 800T A2/A3) 对 **ReMax Qwen3-8B FSDP** 训练的支持，采用原生 vllm_ascend 推理后端。
*   **#7538 [fix]** 修复 NPU 上使用模块化 vLLM 时的启动报错，适配了新版 vLLM 移除旧版 `FusedMoE` 而改用 `FusedMoEFactory` 的变更。
*   **#7541 [ci]** 优化 NPU 夜间 CI，改为从 `verl-ascend-recipe` 仓库获取检查脚本与 A2/A3 性能基线，不再依赖 CI Runner 本地文件。

**2. 数据序列化与内存优化**
*   **#7539 [fix]** 修复了在启用 NumPy 序列化时，`DataProto.__getstate__()` 会触发不必要的全批次 TensorDict 内存分配问题。
*   **#7534 [fix]** 修复了当 TensorDict 包含零元素字段时，NumPy 序列化器在反序列化阶段引发的崩溃问题。

**3. 训练与配置修复**
*   **#7532 [fix]** 修复 `rearrange_micro_batches` 的误判逻辑：原先会错误拒绝填充宽度超限但去除 padding 后实际有效长度合格的批次。
*   **#7533 [fix]** 修复旧版 Megatron merger 因导入已被移除的 `get_hf_config_and_tokenizer_checkpoint_path` 而报错的问题。
*   **#7536 [fix]** 移除未使用的 ref router replay 配置，修复由 #7466 引入的配置不兼容报错。

**4. 文档重构**
*   **#7537 [doc]** 将 "Megatron Lite" 文档重构为 **Megatron Agent Compose** 预览版，以对齐 Megatron-LM 上游的实验性路径规划。

---

### 🚀 Release 动态

*   近期无新的版本发布。

---

## 🐛 Issues

### #7540 — [[distillation][bug] OPD multi-teacher distillation loss diverges between v0.9.0 and HEAD despite unchanged numerical code path (reproduced on NVIDIA B300 and Cambricon MLU)](https://github.com/verl-project/verl/issues/7540)
- **作者**: yuki1ssad  **时间**: 2026-08-24 19:01 CST
- **标签**: bug
- **摘要**: ### System Info  ## System Info - **verl version**:    - baseline: `v0.9.0`    - buggy: `HEAD` (commit `1dda039b9c6311889a9c9a26040677e1254814f9`) - **Python**: 3.12.3 - **OS**: Ubuntu 24.04.3 LTS (Noble Numbat), kernel 6.6.98-40.4.tl4.x86_64 - **GPU**: NVIDIA B300  × 4 - **NVIDIA Driver**: 580.159.…

## 🔀 Pull Requests

### #7541 — [[ci] chore: use Ascend recipe baselines for NPU nightly CI](https://github.com/verl-project/verl/pull/7541)
- **作者**: lxb007981  **时间**: 2026-08-24 19:38 CST
- **摘要**: ### What does this PR do? Fetch check_npu.py and the matching A2/A3 performance baselines from verl-ascend-recipe instead of relying on files stored on CI runners.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{mo…

### #7539 — [[ray] fix: skip unused TensorDict consolidation in NumPy DataProto serialization](https://github.com/verl-project/verl/pull/7539)
- **作者**: Sky-Trigger  **时间**: 2026-08-24 16:38 CST
- **摘要**: <!-- PR title: [ray] fix: skip unused TensorDict consolidation in NumPy DataProto serialization Base: verl-project/verl:main Head: Sky-Trigger:numpy-dataproto-serialization Commit: 4d7b20eb -->  ### What does this PR do?  Fixes an unnecessary full-batch TensorDict allocation in `DataProto.__getstate…

### #7538 — [[vllm, hardware] fix: support modular FusedMoE on NPU](https://github.com/verl-project/verl/pull/7538)
- **作者**: ZihaoW123  **时间**: 2026-08-24 15:50 CST
- **摘要**: ### What does this PR do?  Fix NPU startup with modular vLLM releases that expose `FusedMoEFactory` but no longer export the legacy `FusedMoE` class.  `apply_npu_vllm_patches()` previously imported `FusedMoE` unconditionally, so importing `verl.utils.vllm` failed before the existing legacy-class gua…

### #7537 — [[doc] refactor: reframe Megatron Lite doc as Megatron Agent Compose preview](https://github.com/verl-project/verl/pull/7537)
- **作者**: ISEEKYAN  **时间**: 2026-08-24 15:00 CST
- **摘要**: ### What does this PR do?  Reframes `docs/advance/megatron_lite_backend.rst` around **Megatron Agent Compose**, the intended upstream experimental path in Megatron-LM. "Megatron Lite" was the name this component carried during hot development; the doc previously described it exclusively under that n…

### #7536 — [[cfg] fix: drop unused ref router replay config](https://github.com/verl-project/verl/pull/7536)
- **作者**: ji-huazhong  **时间**: 2026-08-24 13:28 CST
- **摘要**: ### What does this PR do?  Fixes a reference configuration incompatibility introduced by #7466.  Original error report from: https://github.com/verl-project/verl/actions/runs/32689421121/job/97320321754?pr=7530  #7466 removed the unused top-level ActorConfig.router_replay field, while ref/ref.yaml c…

### #7535 — [[trainer, doc] feat: enable ReMax Qwen3-8B FSDP on Ascend](https://github.com/verl-project/verl/pull/7535)
- **作者**: yukinotech  **时间**: 2026-08-24 12:33 CST
- **摘要**: ### What does this PR do?  Add Ascend (Atlas 800T A2/A3) support for **ReMax** training of **Qwen3-8B** with the **FSDP training backend**, using verl's native **vllm_ascend inference backend**. This is the Ascend counterpart of `examples/remax_trainer/run_qwen3_8b_fsdp.sh`.  ReMax requires one samp…

### #7534 — [[misc] fix: handle zero-element numpy serialization](https://github.com/verl-project/verl/pull/7534)
- **作者**: YZJF  **时间**: 2026-08-24 12:14 CST
- **摘要**: ### What does this PR do?  Fixes a crash in the opt-in NumPy `DataProto` serializer when a TensorDict contains a field with zero elements. Serialization produces an empty `uint8` array, but deserialization passed that empty buffer to `torch.frombuffer`, which raises `ValueError`.  The fix reconstruc…

### #7533 — [[megatron, ckpt] fix: restore legacy merger config discovery](https://github.com/verl-project/verl/pull/7533)
- **作者**: liuhao-labs  **时间**: 2026-08-24 12:05 CST
- **摘要**: ### What does this PR do?  Fixes #6065.  The legacy Megatron merger imports `get_hf_config_and_tokenizer_checkpoint_path`, which has been removed from `verl.utils.megatron_utils`. As a result, it raises `ImportError` before loading a checkpoint or reaching any Transformers serialization code.  This …

### #7532 — [[training_utils] fix: validate remove-padding batches by effective length](https://github.com/verl-project/verl/pull/7532)
- **作者**: liuhao-labs  **时间**: 2026-08-24 09:09 CST
- **摘要**: ### What does this PR do?  Fixes #3088.  `rearrange_micro_batches` currently rejects a dense padded batch when its padded width exceeds `max_token_len`, even when padding is removed and every sample's effective token count fits the limit. The same function then uses effective lengths for micro-batch…
