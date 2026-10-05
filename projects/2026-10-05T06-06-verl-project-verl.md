# verl-project/verl — 动态追踪

> 生成时间: 2026-10-05 14:06 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

本期内（2026年10月4日至5日）仓库活动主要集中在代码提交（PR）上，未记录到新增的 Issue 或 Release 版本发布。

### 一、 Pull Request (PR)
本期共合并/提交 10 项 PR，主要涉及新硬件支持、异步训练指标监控、Checkpoint 管理优化及权重导出等关键改进：

**1. 新硬件支持**
*   **#8119 [Intel XPU 支持]**：新增 SGLang rollout 对 Intel GPU（PyTorch XPU）的支持，实现了基于 FSDP2 actor 和 SGLang 的端到端 GRPO 训练。

**2. 异步训练与监控指标增强**
*   **#8116 [性能监控]**：新增可选的更新阶段计时和生命周期指标，分离前向/反向传播与优化器的时间消耗和 MFU（算力利用率）统计。
*   **#8118 [陈旧度指标]**：在全异步 rollout 和 v1 trainer 中引入了 token 权重的生成陈旧度指标和行为版本溯源。
*   **#8117 [生成过程监控]**：增加对部分生成尝试和后端停止行为的观察记录，保留原始尝试次数和预填充时间间隔。
*   **#8113 [验证采样]**：支持在常规验证后运行额外的命名验证采样配置，分离指标和生成转储数据。

**3. Checkpoint 管理与恢复优化**
*   **#8114 [检查点保留]**：引入基于验证分数的 Checkpoint 保留策略，优先保留最新可恢复、最佳分数及符合条件的收敛候选检查点。
*   **#8112 [生成前缀保留]**：在 v1 异步检查点中保留客户端可见的单轮生成前缀，支持进行中的会话从保存的 token 恢复。
*   **#8111 [队列恢复控制]**：允许 v1 异步 trainer 在恢复模型、优化器状态时，可选地跳过检查点中的 rollout 队列数据。

**4. 权重导出与数据准备**
*   **#8110 [权重导出]**：新增可选的 VeOmni 导出路径，在聚合和专家广播前将临时分片转换为实际的 vLLM 参数 dtype。
*   **#8115 [数据准备]**：提供独立的数据准备脚本，将 AIME 2024/2025/2026 I 和 II 场次的验证集数据分离，方便按考试场次进行独立评估。

### 二、 Issue
*   本期内无公开的重要 Issue 活动。

### 三、 Release
*   本期内无新版本发布。

---

## 🔀 Pull Requests

### #8119 — [[hardware, fsdp, sglang] feat: Add SGLang rollout support on Intel XPU](https://github.com/verl-project/verl/pull/8119)
- **作者**: Amrutha-M05  **时间**: 2026-10-05 13:20 CST
- **摘要**: ### What does this PR do?  Adds **SGLang rollout** support on **Intel GPU (PyTorch XPU)**, enabling end-to-end **GRPO** training with **FSDP2** actors and **SGLang** colocated rollout, including SGLang **sleep/wake** in hybrid mode. It builds on the Intel GPU platform from #7371, which covers vLLM r…

### #8118 — [[rollout, trainer] feat: report token-weighted generation staleness](https://github.com/verl-project/verl/pull/8118)
- **作者**: tongyx361  **时间**: 2026-10-04 16:44 CST
- **摘要**: ### What does this PR do?  Add optional behavior-version provenance and token-weighted age metrics to fully asynchronous rollout and the v1 trainer. When a response resumes after a weight update, record each nonempty attempt's generation version and token count. Report coverage explicitly: missing o…

### #8117 — [[rollout, trainer] feat: observe partial generation attempts and backend stops](https://github.com/verl-project/verl/pull/8117)
- **作者**: tongyx361  **时间**: 2026-10-04 16:44 CST
- **摘要**: ### What does this PR do?  Partial generation can stop and resume multiple times before a trajectory is consumed. Add optional attempt counts and available prefill wall intervals, while retaining raw vLLM finish/stop reasons separately from the client's cumulative response-budget decision. A backend…

### #8116 — [[perf, trainer] feat: report opt-in update phase timing and lifecycle metrics](https://github.com/verl-project/verl/pull/8116)
- **作者**: tongyx361  **时间**: 2026-10-04 16:44 CST
- **摘要**: ### What does this PR do?  Separate forward/backward and optimizer wall-time reporting and MFU denominators, while retaining the whole-update MFU. Add host-side colocated rollout lifecycle intervals, complete-group wait time, and VeOmni microbatch counts. AI assistance was used to prepare this chang…

### #8115 — [[data] feat: prepare pinned AIME validation by exam session](https://github.com/verl-project/verl/pull/8115)
- **作者**: tongyx361  **时间**: 2026-10-04 16:44 CST
- **摘要**: ### What does this PR do?  Provide a standalone data-preparation script that keeps AIME 2024/2025/2026 I and II sessions separate in validation files and metric identities. Verify all 15 questions per session against pinned public sources and write an auditable provenance manifest. AI assistance was…

### #8114 — [[trainer, ckpt] feat: retain checkpoints by validation score](https://github.com/verl-project/verl/pull/8114)
- **作者**: tongyx361  **时间**: 2026-10-04 16:44 CST
- **摘要**: ### What does this PR do?  Add an opt-in validation-score retention policy for v1 checkpoints. Preserve the latest resumable checkpoints, the best score, eligible convergence candidates, and configured record milestones instead of keeping checkpoints only by save order. AI assistance was used to pre…

### #8113 — [[trainer, rollout] feat: add named validation sampling profiles](https://github.com/verl-project/verl/pull/8113)
- **作者**: tongyx361  **时间**: 2026-10-04 16:43 CST
- **摘要**: ### What does this PR do?  Run additional named validation sampling profiles on the same prompts after the normal validation pass. Profile metrics and generation dumps are separated while reward dispatch retains the original data source. The empty default preserves the existing single-pass behavior.…

### #8112 — [[rollout, ckpt] feat: checkpoint single-turn generation prefixes](https://github.com/verl-project/verl/pull/8112)
- **作者**: tongyx361  **时间**: 2026-10-04 16:43 CST
- **摘要**: ### What does this PR do?  Preserve client-visible single-turn generation prefixes across v1 asynchronous checkpoints. In-flight sessions resume from saved tokens with their remaining response budget, while completed queue trajectories stay available for training.  ### Checklist Before Starting  - […

### #8111 — [[trainer, ckpt] feat: optionally discard restored rollout queue](https://github.com/verl-project/verl/pull/8111)
- **作者**: tongyx361  **时间**: 2026-10-04 16:43 CST
- **摘要**: ### What does this PR do?  Allow v1 asynchronous trainers to resume model, optimizer, and dataloader state while optionally skipping checkpointed rollout queue data. The default continues to restore completed trajectories and in-flight prompt groups.  ### Checklist Before Starting  - [x] Search for …

### #8110 — [[veomni, vllm, rollout] feat: export weights using verified receiver dtypes](https://github.com/verl-project/verl/pull/8110)
- **作者**: tongyx361  **时间**: 2026-10-04 16:43 CST
- **摘要**: ### What does this PR do?  Add an opt-in VeOmni export path that casts temporary shards to the actual vLLM parameter dtype before gathering and expert broadcast. FP32 training parameters delivered to BF16 receiver parameters use half the tensor payload, while training parameters, optimizer/checkpoin…
