# verl-project/verl — 动态追踪

> 生成时间: 2026-10-03 13:37 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📋 Issue
*   **#8099 [RFC] V1 trainer 多输出 rollouts 的优化器步数稳定性**
    *   **特性请求**：作者提议在 V1 trainer 中引入一种显式的批处理模式。该模式旨在当存在多输出 rollouts 时，依然将 Actor 优化器的更新步数与配置的源提示词批次数量保持绑定，以保证训练步数的稳定性。

### 🔧 Pull Request (PR)
**重要新特性与架构升级：**
*   **#8098 feat: 基于 TorchTitan 引擎为 TPU 添加 SFT 支持**
    *   新增了在 Google Cloud TPU 上进行端到端监督微调（SFT）的能力，通过 `verl-hardware-plugin` 调用 TorchTitan 引擎实现。
*   **#8092 feat: V1 trainer 启用 FusedWorker 及异步子 worker**
    *   这是拓扑结构 RFC (#7269，声明式模型拓扑 API) 的首个 PR。在默认 v1 路径中启用了 FusedWorker 容器，并支持等待异步子 worker 执行完成，属于重要的底层架构演进。
*   **#8094 [BREAKING] refactor: 将 checkpoint 转换迁移至 Megatron-Bridge**
    *   **破坏性变更**：重构了 Megatron 的检查点转换逻辑，使用 Bridge providers 和权重映射进行 HF/MCore 转换与导出。支持新的训练检查点布局并保留 HF artifacts。

**关键 Bug 修复：**
*   **#8100 fix: 修复向量化 GRPO 的 session grouping 问题**
    *   修复了 V1 trainer 中 `grpo_vectorized` 跳过会话感知路径的 bug，该问题曾导致复制到中间输出的奖励被重复计算。
*   **#8095 fix: 修复取消 acquire 后 router 分配泄露的问题**
    *   修复了当 `LLMServerClient.generate()` 被取消时，路由器已分配服务器但未返回 RPC 结果导致的 in-flight count 内存泄漏问题。
*   **#8096 & #8097 fix: 修复 FSDP2 下 LM heads 的 fused kernels 兼容问题**
    *   针对 Issue #8091，提供了两种修复方案（#8096 通过 FSDP2 hooks 运行，#8097 在进入 fused autograd 前显式 gather DTensor 权重），解决 FSDP2 独立分片 `lm_head` 时，fused 前向传播读取权重导致绕过分片解析和混合精度的问题。

### 🚀 Release
*   近期无新的版本发布记录。

---

## 🐛 Issues

### #8099 — [[RFC] Stable optimizer-step counts for multi-output rollouts in the V1 trainer](https://github.com/verl-project/verl/issues/8099)
- **作者**: luyuzhe111  **时间**: 2026-10-03 04:57 CST
- **摘要**: ## Feature request  Could the V1 trainer offer an explicit batching mode that keeps the number of actor optimizer steps tied to the configured source-prompt batch, while allowing the number of training rows per rollout to vary?  In a MigrationBench comparison, fixed-count batching reached a best mea…

## 🔀 Pull Requests

### #8100 — [[trainer] fix: preserve session grouping for vectorized GRPO](https://github.com/verl-project/verl/pull/8100)
- **作者**: Zethan06  **时间**: 2026-10-03 10:26 CST
- **摘要**: ### What does this PR do?  Fix session grouping for `grpo_vectorized` in the V1 trainer. It currently skips the session-aware path, so rewards copied to intermediate outputs are counted repeatedly in the group mean and standard deviation.  Use each session's final output to compute advantages, then …

### #8098 — [[doc, hardware, worker, trainer] feat: add SFT support for TPU using TorchTitan engine](https://github.com/verl-project/verl/pull/8098)
- **作者**: askhat-g  **时间**: 2026-10-03 03:29 CST
- **摘要**: ### What does this PR do?  Adds end-to-end Supervised Fine-Tuning (SFT) support on Google Cloud TPUs using the TorchTitan engine (`TorchTitanTPUEngineWithLMHead` from `verl-hardware-plugin`).  IMPORTANT: The script will be usable only after official torch_tpu release  ### Checklist Before Starting  …

### #8097 — [[fsdp, model] fix: gather DTensor LM heads for fused kernels](https://github.com/verl-project/verl/pull/8097)
- **作者**: kolehma8  **时间**: 2026-10-03 02:54 CST
- **摘要**: ## Summary  Alternative approach to #8091: explicitly materialize the FSDP2 LM-head weight before entering the Torch, Triton, or Liger fused autograd function. Only the weight is gathered; activations, logits, and labels are not gathered by this change.  This is a **draft for design comparison**, no…

### #8096 — [[fsdp, model] fix: run fused LM heads through FSDP2 hooks](https://github.com/verl-project/verl/pull/8096)
- **作者**: cyanseek  **时间**: 2026-10-03 02:05 CST
- **摘要**: ### What does this PR do?  Fixes #8091. With untied embeddings, FSDP2 shards `lm_head` independently, but fused forwards read its weight from the parent. This bypasses the head's unsharding, mixed precision, CPU offload and backward hooks. A two-GPU Qwen3 reproducer fails with mixed Tensor/DTensor o…

### #8095 — [[rollout] fix: release router allocations after cancelled acquire](https://github.com/verl-project/verl/pull/8095)
- **作者**: cyanseek  **时间**: 2026-10-03 00:40 CST
- **摘要**: ### What does this PR do?  Cancelling `LLMServerClient.generate()` while the router has allocated a server but has not returned the RPC result leaks the in-flight count: acquisition happens before the generation `try/finally`. This leaves stale load information for routing and can prevent fully-asyn…

### #8094 — [[3/N][BREAKING][megatron] refactor: migrate checkpoint conversion to Megatron-Bridge](https://github.com/verl-project/verl/pull/8094)
- **作者**: ji-huazhong  **时间**: 2026-10-02 21:38 CST
- **摘要**: ### What does this PR do?  Use Bridge providers and weight mappings for HF/MCore conversion and export. Support training checkpoint layouts(v2), preserve HF artifacts, and remove the obsolete model initializer.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query li…

### #8093 — [U/cwyh/read](https://github.com/verl-project/verl/pull/8093)
- **作者**: CWYH  **时间**: 2026-10-02 20:37 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #8092 — [[single_controller] feat: enable FusedWorker in v1 trainer and await async sub-workers](https://github.com/verl-project/verl/pull/8092)
- **作者**: ETOgaosion  **时间**: 2026-10-02 12:46 CST
- **摘要**: > This is the **first PR** of the topology RFC [#7269](https://github.com/verl-project/verl/issues/7269) (Declarative Model-Topology API). It enables the FusedWorker container in the default v1 path as a prerequisite; follow-up PRs will introduce atomic RL workers and the declarative `topology:` pla…
