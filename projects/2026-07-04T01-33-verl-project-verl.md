# verl-project/verl — 动态追踪

> 生成时间: 2026-07-04 01:33 UTC

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 🚀 Release (发布)
本期无新的 Release 版本发布。

### 🐛 Issue (议题)
本期无新的 Issue 动态。

### 🔧 Pull Request (合并请求)
本期 PR 主要聚焦于**核心框架兼容性修复**与**文档更新**，具体如下：

**1. 核心框架与模型兼容性修复**
*   **Megatron-Core >= 0.18 兼容性修复** (#6927)：Megatron-Core 0.18 版本移除了 `ModelType.encoder_and_decoder` 枚举（仅保留 `encoder_or_decoder`），导致构建模型报错。此 PR 对相关代码进行了适配与防护。
*   **修复 Qwen3 MoE 在 Transformers 5 下的 FSDP 权重同步问题** (#6923)：Transformers 5 将 Qwen MoE 专家权重存储为打包的 3D 张量，导致原有 FSDP 与 vLLM rollout 的权重同步逻辑失效。此 PR 修复了该问题，并反向移植（Backport）到了 `release/v0.8.0` 分支。
*   **提升 Qwen3.5 Router Replay (R3) 的 vLLM 依赖版本** (#6922)：由于 vLLM 的 `enable_return_routed_experts` 特性仅在 vLLM >= 0.22.0 中为混合注意力 MoE 模型提供正确尺寸的 host buffer，此 PR 强制要求 R3 模式下的 vLLM 版本不低于 0.22.0，以避免运行报错。

**2. 文档更新**
*   **DeepSeek-V3 支持文档更新** (#6926)：补充和完善了 DeepSeek-V3 模型的相关使用文档。
*   **Ascend NPU 最佳实践版本兼容性更新** (#6924, #6925)：更新了 Ascend 环境下 Retool、DAPO 多模型、GSPO 等最佳实践文档的软件版本要求，统一对齐至 CANN 9.0.0.B160、Python 3.11、PyTorch/torch_npu 2.9.0、vLLM-Ascend 0.18.0 及 triton_ascend 3.2.1。其中 #6925 针对的是 `release/v0.8.0` 分支，#6924 针对主分支。

---

## 🔀 Pull Requests

### #6927 — [[megatron] fix: guard ModelType.encoder_and_decoder for Megatron-Core >= 0.18 compatibility](https://github.com/verl-project/verl/pull/6927)
- **作者**: chengcuiping  **时间**: 2026-07-03 14:32 UTC
- **摘要**: ### What `verl/utils/megatron_utils.py::get_model()` references `ModelType.encoder_and_decoder` in three places. That enum member was **removed in Megatron-Core 0.18** — `megatron.core.transformer.enums.ModelType` now only has `encoder_or_decoder` — so building any model on Megatron-Core ≥ 0.18 rais…

### #6926 — [[doc] fix: update docs for deepseekv3 support](https://github.com/verl-project/verl/pull/6926)
- **作者**: xiazhahe  **时间**: 2026-07-03 08:37 UTC
- **标签**: Ascend
- **摘要**: ### What does this PR do? update docs for deepseekv3 support   ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megat…

### #6925 — [[doc] chore: update ascend Retool/DAPO/GSPO best practice version compatibility for release/v0.8.0](https://github.com/verl-project/verl/pull/6925)
- **作者**: chengminhua  **时间**: 2026-07-03 06:58 UTC
- **摘要**: ### What does this PR do?  - Update software version requirements in three Ascend best-practice docs (Retool, DAPO multi-model, GSPO) to align with CANN 9.0.0.B160, Python 3.11, PyTorch/torch_npu 2.9.0, vLLM/vLLM-Ascend 0.18.0, and triton_ascend 3.2.1  ### Checklist Before Starting  - [ ] Search for…

### #6924 — [[doc] chore: update ascend Retool/DAPO/GSPO best practice version compatibility](https://github.com/verl-project/verl/pull/6924)
- **作者**: chengminhua  **时间**: 2026-07-03 06:57 UTC
- **摘要**: ### What does this PR do?  - Update software version requirements in three Ascend best-practice docs (Retool, DAPO multi-model, GSPO) to align with CANN 9.0.0.B160, Python 3.11, PyTorch/torch_npu 2.9.0, vLLM/vLLM-Ascend 0.18.0, and triton_ascend 3.2.1  ### Checklist Before Starting  - [ ] Search for…

### #6923 — [[fsdp] fix: Fix Qwen3 MoE FSDP weight sync for vLLM rollout in Transformers 5](https://github.com/verl-project/verl/pull/6923)
- **作者**: lxb007981  **时间**: 2026-07-03 01:48 UTC
- **摘要**: ### What does this PR do?  Backport https://github.com/verl-project/verl/pull/6896 to [release/v0.8.0](https://github.com/verl-project/verl/tree/release/v0.8.0)  Transformers 5 stores Qwen-style MoE expert weights as packed 3D `mlp.experts.gate_up_proj` and `mlp.experts.down_proj` tensors. During li…

### #6922 — [[vllm, rollout] fix: require vLLM>=0.22.0 for Qwen3.5 router replay (R3)](https://github.com/verl-project/verl/pull/6922)
- **作者**: dafu-wu  **时间**: 2026-07-03 00:41 UTC
- **摘要**: ### What does this PR do?  Rollout Router Replay in **R3** mode (`actor_rollout_ref.rollout.enable_rollout_routing_replay=True`) depends on vLLM's routed-experts capture path (`enable_return_routed_experts`). That path only produces a correctly-sized host buffer for **hybrid-attention MoE** models —…
