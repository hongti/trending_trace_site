# verl-project/verl — 动态追踪

> 生成时间: 2026-11-21 15:36 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📋 Issue
* **#8777 [RFC] SkipManager 重构**
  * **摘要：** 提出对 `SkipManager.rollout_tq` 进行重构的请求。当前该组件在重放缓存的 V1 TransferQueue 批次时存在硬编码行为，强制每个 prompt 对应 `rollout.n` 行，并将所有轨迹键重写为 `{new_uid}_{session_id}_0`，计划对此进行优化重构。

### 🔨 Pull Requests (PR)
* **#8782 降低单卡 RL 实验门槛及配置校验**
  * **新特性：** 引入了快速失败配置预检功能；同时为单 GPU（1xH20）环境下运行 Qwen2.5-0.5B GRPO 提供了性能调优层级，大幅降低了小规模强化学习实验的入门成本。
* **#8781 修复 NCCL 检查点广播竞态问题**
  * **Bug 修复：** 修复了 NCCL checkpoint 后端在分桶广播权重至 rollout workers 时的两个竞态条件，提升了 checkpoint 传输的稳定性和安全性。
* **#8780 vLLM 权重更新机制重构与版本升级**
  * **重要重构：** 摒弃了 verl 自定义的各方案权重更新补丁，改为通过 vLLM 原生的 reload 生命周期（`start_reload` / `finish_reload`）在直接（原地）模式下适配权重重载。同时将 vLLM 依赖版本升级至 0.34.0。
* **#8778 / #8779 Ascend 硬件 CI 性能对比测试**
  * **CI 测试：** 针对 Ascend Qwen E2E 测试用例，添加了 V2 runner 与 R3 runner 的性能基准对比。测试发现 R3 运行在 vLLM 配置校验阶段会中断，因为路由专家捕获仍需要依赖 V2 模型运行。

### 🚀 Release
* **近期无新版本发布。**

---

## 🐛 Issues

### #8777 — [[RFC] Skipmanager Refactoring](https://github.com/verl-project/verl/issues/8777)
- **作者**: mikequan0425  **时间**: 2026-11-20 15:29 CST
- **摘要**: # Summary  `SkipManager.rollout_tq` currently replays a cached V1 TransferQueue batch by forcing each prompt to exactly `rollout.n` rows and rewriting every trajectory key as `{new_uid}_{session_id}_0`. This is incorrect for agentic RL, where one session may emit multiple `AgentLoopOutput` rows with…

## 🔀 Pull Requests

### #8782 — [[cfg, perf, doc] feat: add fail-fast config validation and 1xH20 tuning tiers for Qwen2.5-0.5B GRPO](https://github.com/verl-project/verl/pull/8782)
- **作者**: StephenAvA  **时间**: 2026-11-21 15:21 CST
- **摘要**: ## What does this PR do?  Two related changes that lower the entry cost of small-scale RL experiments on a single GPU:  1. **Fail-fast config preflight** (`verl/utils/config.py`): a new `_validate_parallel_and_memory_config()` called at the top of `validate_config()` catches 6 classes of common misc…

### #8781 — [[ckpt] fix: prevent two races in NCCL checkpoint bucket broadcasts](https://github.com/verl-project/verl/pull/8781)
- **作者**: ewan0x79  **时间**: 2026-11-20 22:45 CST
- **摘要**: ### What does this PR do?  Training rank 0 pulls weights from the export iterator and broadcasts them to rollout workers in buckets. This fixes two races in the NVIDIA `nccl` checkpoint backend.  - **Before the first bucket.** The NCCL barrier can return while a rollout worker's ZeroMQ subscription …

### #8780 — [[vllm, rollout] refactor: refit through vLLM reload lifecycle, drop weight-update patches, bump vllm to 0.34.0](https://github.com/verl-project/verl/pull/8780)
- **作者**: aoshen02  **时间**: 2026-11-20 19:49 CST
- **摘要**: ### What does this PR do?  Refit the vLLM rollout through vLLM's own reload lifecycle (`start_reload` / `finish_reload`, vLLM 0.34) in direct (in-place) mode, and delete verl's per-scheme weight-update staging patches (FP8 / MXFP4 / NVFP4 / unquantized MoE, ~2.6k lines). Bumps the default vLLM to 0.…

### #8779 — [[ci] test: compare Ascend Qwen R3 performance with V2 runner](https://github.com/verl-project/verl/pull/8779)
- **作者**: Rain6ow  **时间**: 2026-11-20 17:33 CST
- **摘要**: The two existing Ascend Qwen E2E cases need baseline-versus-R3 measurements. In #8764, both R3 runs stopped during vLLM configuration validation because routed-expert capture requires the V2 model runner.  This replacement keeps the six-step disabled/R3 comparisons and always-uploaded logs, enables …

### #8778 — [[ci] test: compare Ascend Qwen R3 performance with V2 runner](https://github.com/verl-project/verl/pull/8778)
- **作者**: Rain6ow  **时间**: 2026-11-20 17:28 CST
- **摘要**: The two existing Ascend Qwen E2E cases need baseline-versus-R3 measurements. In #8764, both R3 runs stopped during vLLM configuration validation because routed-expert capture requires the V2 model runner.  This replacement keeps the six-step disabled/R3 comparisons and always-uploaded logs, enables …
