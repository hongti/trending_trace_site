# verl-project/verl — 动态追踪

> 生成时间: 2026-07-03 06:41 UTC

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文简洁摘要：

### 🛠️ Pull Request (拉取请求)

**✨ 新特性**
*   **#6920 添加基于 FSDP Teacher 的 Reverse-KL On-Policy 蒸馏**：引入 student-top-K 支持，遵循《Rethinking On-Policy Distillation》论文的损失公式 (`KL(Q_student_topk || P_teacher_topk)`)，实现 Teacher log-probs 的按需计算。

**🐛 关键修复与优化**
*   **#6923 修复 Qwen3 MoE 在 vLLM rollout 的 FSDP 权重同步问题**：针对 Transformers 5 将 Qwen 式 MoE 专家权重存储为 packed 3D tensor 的变更，修复了权重同步逻辑，并回补至 `release/v0.8.0` 分支。
*   **#6922 Qwen3.5 Router Replay 要求 vLLM>=0.22.0**：修复 R3 模式下的 Router Replay 问题，确保 hybrid-attention MoE 模型能生成正确大小的 host buffer（依赖 vLLM 0.22.0+ 的特性）。
*   **#6919 修复 vLLM rollout 分桶传输中的非连续权重问题**：解决 Actor 导出有效但非连续 tensor 时，vLLM 权重更新因 `view` 操作导致的崩溃问题。
*   **#6921 修复多轮工具 Agent 循环边界丢失 turn separator 的问题**：在 `ToolAgentLoop` 中，修复模型生成 assistant turn 后未补全聊天模板尾部 turn separator（如 `\n` / id 198）的遗漏。
*   **#6917 优雅终结 V1 PPO Trainer 的 Tracking 和 Dataloader Workers**：在 `fit()` 退出时通过 `try/finally` 显式清理资源，不再依赖可能延迟或失效的 `__del__` / 垃圾回收机制。
*   **#6916 更新 TorchtitanEngine 以支持 SPMD 类型**：新增 Titan 的 `spmd_types` 支持（需 PyTorch nightly >= 0624），默认使用 `spmd_type` 运行而非 `DTensor`，运行速度与 DTensor 相当且快于 FSDP engine；同时修复了 API 兼容性问题。

**📝 文档**
*   **#6918 更新 Ascend quick_start 文档**：修正快速入门指南的相关内容。

---

### 🐛 Issue (问题)
*   **近期无重点新增 Issue 动态。**

---

### 🚀 Release (发布版本)
*   **近期无新版本发布动态。**（注：PR #6923 涉及向 `release/v0.8.0` 分支的回补，预示该版本正在积极修复与打磨中，但尚未正式发布。）

---

## 🔀 Pull Requests

### #6923 — [[fsdp] fix: Fix Qwen3 MoE FSDP weight sync for vLLM rollout in Transformers 5](https://github.com/verl-project/verl/pull/6923)
- **作者**: lxb007981  **时间**: 2026-07-03 01:48 UTC
- **摘要**: ### What does this PR do?  Backport https://github.com/verl-project/verl/pull/6896 to [release/v0.8.0](https://github.com/verl-project/verl/tree/release/v0.8.0)  Transformers 5 stores Qwen-style MoE expert weights as packed 3D `mlp.experts.gate_up_proj` and `mlp.experts.down_proj` tensors. During li…

### #6922 — [[vllm, rollout] fix: require vLLM>=0.22.0 for Qwen3.5 router replay (R3)](https://github.com/verl-project/verl/pull/6922)
- **作者**: dafu-wu  **时间**: 2026-07-03 00:41 UTC
- **摘要**: ### What does this PR do?  Rollout Router Replay in **R3** mode (`actor_rollout_ref.rollout.enable_rollout_routing_replay=True`) depends on vLLM's routed-experts capture path (`enable_return_routed_experts`). That path only produces a correctly-sized host buffer for **hybrid-attention MoE** models —…

### #6921 — [[rollout] fix: restore turn separator dropped at multi-turn tool agent loop boundaries](https://github.com/verl-project/verl/pull/6921)
- **作者**: abtonmoy  **时间**: 2026-07-02 23:26 UTC
- **摘要**: ### What does this PR do?  In the multi-turn tool agent loop (`ToolAgentLoop`), the token sequence is built incrementally. After the model generates an assistant turn it stops at the assistant close token (e.g. `<|im_end|>`) and never emits the chat template's trailing turn separator (`\n`, id 198 f…

### #6920 — [[trainer, fsdp, worker, algo, cfg, recipe] feat: add reverse-KL on-policy distillation with FSDP teacher](https://github.com/verl-project/verl/pull/6920)
- **作者**: YangxSong  **时间**: 2026-07-02 16:46 UTC
- **摘要**: ### What does this PR do?  Adds reverse-KL on-policy distillation with student-top-K support (verl issue #6676), following the formulation in [Rethinking On-Policy Distillation](https://arxiv.org/abs/2604.13016v1): the loss is `KL(Q_student_topk || P_teacher_topk)`, which the teacher log-probs are e…

### #6919 — [[vllm, rollout] fix: handle non-contiguous weights in bucketed transfer](https://github.com/verl-project/verl/pull/6919)
- **作者**: Mecoli1219  **时间**: 2026-07-02 16:42 UTC
- **摘要**: ### What does this PR do?  Fixes vLLM rollout weight updates when the actor exports a valid but non-contiguous tensor. The bucketed weight sender previously packed weights with `weight.view(-1).view(torch.uint8)`, which crashes for strided tensors with:  ```text RuntimeError: view size is not compat…

### #6918 — [[doc] fix: Update ascend quick_start.rst](https://github.com/verl-project/verl/pull/6918)
- **作者**: yyyy2000  **时间**: 2026-07-02 13:55 UTC
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Update ascend quick_start.rst  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`,…

### #6917 — [[trainer] fix: gracefully finalize tracking and dataloader workers at fit() exits](https://github.com/verl-project/verl/pull/6917)
- **作者**: seokhyunan  **时间**: 2026-07-02 07:34 UTC
- **摘要**: ### What does this PR do?  The V1 PPO trainer finalizes two end-of-run resources only via `__del__` / garbage collection, which under Ray / interpreter teardown runs too late. This PR finalizes both explicitly, in a single `try/finally` around the training loop, so cleanup runs on normal completion,…

### #6916 — [[trainer] fix: Update latest TorchtitanEngine](https://github.com/verl-project/verl/pull/6916)
- **作者**: acisseJZhong  **时间**: 2026-07-02 07:18 UTC
- **摘要**: ### What does this PR do?  1. Support Titan with `spmd_types`(requires pytorch nightly >= 0624), and defaulted to `spmd_type` run instead of using `DTensor`. Note: `spmd_types` and `DTensor` speed is roughly on par, both faster than fsdp engine. See below.  2. API-compatibility fixes so TorchTitan e…
