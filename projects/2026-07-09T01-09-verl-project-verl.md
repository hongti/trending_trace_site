# verl-project/verl — 动态追踪

> 生成时间: 2026-07-09 09:09 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📋 Issue 动态
1. **功能请求：为多轮 Rollout 中的 BaseTool 故障添加重试策略** (#6978)
   - 作者指出，当前多轮工具调用（`ToolAgentLoop._call_tool`）会将工具执行异常直接转换为文本响应，这会导致 worker 中断。建议增加可选的（opt-in）重试机制来处理可恢复的故障。
2. **疑问：`separate_async` 模式下样本全量丢弃的后果** (#6975)
   - 作者在学习 PPOTrainerSeparateAsync 代码时提出疑问：当配置为 `max_off_policy_strategy=drop` 时，如果某一步的所有样本都被 `ReplayBuffer._drop_max_off_policy_samples` 丢弃，系统会发生什么情况？

---

### 🔀 PR 动态
**核心特性与架构改进：**
1. **[重大性能优化] 解耦式 Rollout 通过 NCCL 同步增量权重** (#6974)
   - **亮点**：针对解耦式架构新增了**增量权重同步**功能。每次训练步后，Trainer 仅广播发生变化的参数，而 RL 训练通常有 >99% 的参数未变动，此特性极大减少了通信开销。同时包含了分片快照变体。

**重要修复：**
2. **[BREAKING] 修复 `separate_async` trainer 的步长粒度对齐问题** (#6977)
   - 修复了异步训练与同步训练在全局步长语义上的不一致，确保 `separate_async` 在一个 trainer step 内执行完整的 `parameter_sync_step` 循环，并聚合内部更新的 metrics。
3. **修复 vLLM ZMQ handles 的 rank 推导问题** (#6980, #6979)
   - 两个 PR 均针对同一问题（#6615），从 vLLM 数据并行 rank 推导 colocated 权重同步接收器的 socket rank，修复了 ZMQ 通信问题。(#6980 为从 main 分支的 cherry-pick)。
4. **精准限制 Megatron 的 CUDA 环境变量** (#6976)
   - 修复了 `CUDA_DEVICE_MAX_CONNECTIONS=1` 的设置范围，仅针对 Hopper/Ampere 架构的 Megatron 训练生效，避免对其他计算场景产生负面影响。

**生态与工程化：**
5. **更新 Atlas 950DT A5 安装文档** (#6981)
   - 更新了 CANN、MindSpeed 和 Megatron 的依赖版本及安装指引，适配昇腾硬件生态。
6. **新增昇腾平台 Nightly CI 测试** (#6973)
   - 为 Ascend 平台新增了 GRPO Qwen3.5-35B Megatron 训练 + vLLM rollout 的夜间测试工作流，提升对异构硬件的持续集成保障。

---

### 🚀 Release 动态
- **近期无新版本发布记录**。

---

## 🐛 Issues

### #6978 — [[Feature Request] Add opt-in retry policy for recoverable BaseTool failures in multi-turn rollout](https://github.com/verl-project/verl/issues/6978)
- **作者**: yue-zeng-yue  **时间**: 2026-07-08 17:49 CST
- **摘要**: ### Feature request  ## Motivation  In multi-turn tool rollout, `ToolAgentLoop._call_tool` currently catches tool execution exceptions and converts them into tool response text. This prevents the worker from crashing, but transient infrastructure failures such as sandbox timeout, endpoint connection…

### #6975 — [[separate_async]when trainer_mod=separate_async, max_off_policy_strategy=drop, what will happen if all the samples are dropped in one step ?](https://github.com/verl-project/verl/issues/6975)
- **作者**: Gym-Gary  **时间**: 2026-07-08 16:25 CST
- **摘要**: I am reading and learning the code for PPOTrainerSeparateAsync, and I have a question when I came across the `ReplayBuffer._drop_max_off_policy_samples` function. @wuxibin89  This function checks the samples obtained from the tq, and if a sample spans too many model versions, it discards it. If all …

## 🔀 Pull Requests

### #6981 — [[doc] feat: update installation instructions for Atlas 950DT A5](https://github.com/verl-project/verl/pull/6981)
- **作者**: fh188  **时间**: 2026-07-08 23:38 CST
- **摘要**: Updated dependency versions and installation instructions for CANN, MindSpeed, and Megatron in the installation guidance.  ### What does this PR do?  This PR updates installation instructions for Atlas 950DT A5 to the documentation, including the required software versions, dependencies, and environ…

### #6980 — [[vllm] fix: use data-parallel rank in vLLM ZMQ handles](https://github.com/verl-project/verl/pull/6980)
- **作者**: zjchenn  **时间**: 2026-07-08 18:37 CST
- **摘要**: ### What does this PR do?  cherry-pick from main to fix #6615.   relate PR https://github.com/verl-project/verl/pull/6620  - derive the colocated vLLM weight-sync receiver socket rank from the vLLM data-parallel local rank and tensor-parallel rank - keep the existing single-DP socket layout unchange…

### #6979 — [[vllm]fix: use data-parallel rank in vLLM ZMQ handles](https://github.com/verl-project/verl/pull/6979)
- **作者**: autbuster  **时间**: 2026-07-08 18:24 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #6977 — [[BREAKING][trainer] fix: separate_async should use the same step granularity with other trainers](https://github.com/verl-project/verl/pull/6977)
- **作者**: Begunner  **时间**: 2026-07-08 17:11 CST
- **摘要**: ### What does this PR do?  - Align `separate_async` global step semantics with sync training by running a full `parameter_sync_step` cycle inside one trainer step.  - Aggregate per-inner-update metrics across the cycle.  ### Test  E2E separate async training. Metrics aggregator and replay buffer tes…

### #6976 — [[trainer] fix: only set CUDA_DEVICE_MAX_CONNECTIONS=1 for megatron in Hopper/Ampere](https://github.com/verl-project/verl/pull/6976)
- **作者**: wuxibin89  **时间**: 2026-07-08 16:30 CST
- **摘要**: ### What does this PR do?  `CUDA_DEVICE_MAX_CONNECTIONS` ontrols the number of concurrent compute and copy engine connections (work queues). - megatron: set `CUDA_DEVICE_MAX_CONNECTIONS=1` to make sure collective(all-gather/all-reduce) scheduled before compute kernels. - fsdp/veomni/torchtitan/autom…

### #6974 — [[checkpoint_engine][rollout] Delta weight sync over NCCL for disaggregated rollout (+ sharded-snapshot variant)](https://github.com/verl-project/verl/pull/6974)
- **作者**: ChangyiYang  **时间**: 2026-07-08 14:31 CST
- **摘要**: Adds **delta weight sync** for the disaggregated (one-step-off) path: after each training step the trainer broadcasts only the parameters that changed since the previous sync — RL updates leave >99% of BF16 weight bytes unchanged step-over-step — cutting weight-sync traffic to the sparsity ratio whi…

### #6973 — [[ci] feat: add GRPO Qwen3.5-35B Megatron vLLM nightly test for Ascend](https://github.com/verl-project/verl/pull/6973)
- **作者**: chengminhua  **时间**: 2026-07-08 11:13 CST
- **摘要**: ### What does this PR do?  Add a new nightly CI workflow for GRPO Qwen3.5-35B Megatron training on Ascend using the vLLM rollout backend.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {descripti…
