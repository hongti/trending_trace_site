# verl-project/verl — 动态追踪

> 生成时间: 2026-10-13 15:01 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🐛 Issue 动态
* **#8216 [Bug] PrimeRewardManager 处理字典类型奖励时报错**
  当奖励函数返回字典格式时，`PrimeRewardManager` 的 `parallel_compute_score_async` 循环会因 `KeyError: 0` 崩溃，导致整个批次的分数被默默记为 0.0。

### 🔀 Pull Request 动态
**新特性与优化**
* **#8214 支持自定义验证指标**：新增配置驱动钩子 `data.custom_validation_metrics`，用户无需继承 trainer 类即可添加自定义验证指标。
* **#8212 熵诊断改为可选**：将 PPO/V1 训练器在重算旧对数概率时的熵计算设为可选，有助于减少不必要的性能开销。
* **#8215 / #8213 动态批处理支持 BSHD padding 预算**：在 `use_dynamic_bsz=True` 且走 Megatron BSHD 路径时，新增对 padding token 的预算管理，优化批次重组逻辑。

**Bug 修复**
* **#8217 修复 PrimeRewardManager 字典返回值问题**：直接修复 Issue #8216，为字典返回类型的奖励函数补充了处理分支，避免错误转换为 float 导致崩溃。
* **#8211 修复 V1 Replay Buffer 的 UID 解析错误**：修复了 UID 包含下划线时，Replay Buffer 拆分键值错误导致 prompt 分组混淆的问题。
* **#8210 修复蒸馏 PG 的重要性权重处理**：修复了当 rollout 重要性权重为 `NestedTensor` 时未进行 padding 导致的策略梯度蒸馏报错问题。
* **#8209 允许 BSHD 禁用 MoE 路由损失**：修复 Megatron BSHD 路径限制，允许在没有 MoE router loss 激活时正常使用 `calculate_per_token_loss=True`。

**CI 与文档维护**
* **#8219 CI 密钥管理优化**：将 `FORK_REPO` 从 Variable 改用 Secret 存储，并将 `FORK_TOKEN` 重命名为更直观的 `PAT_TOKEN`。

**已关闭的 PR**
* **#8218 一步工具预读性能优化**：该 PR 试图引入工具预读以减少延迟，但经审查发现其 16% 的性能提升仅在特定的合成实验负载下有效，缺乏通用性，最终被关闭。

### 📦 Release 动态
* 该时间段内无新版本发布。

---

## 🐛 Issues

### #8216 — [[Bug] PrimeRewardManager silently scores the whole batch 0.0 when the reward function returns a dict](https://github.com/verl-project/verl/issues/8216)
- **作者**: Virtuoso461  **时间**: 2026-10-12 22:50 CST
- **摘要**: ### Describe the bug  `PrimeRewardManager` (`verl/workers/reward_manager/prime.py`) crashes the result-processing loop of `parallel_compute_score_async` with `KeyError: 0` whenever the reward function returns a dict such as `{"score": 1.0, "acc": True, "pred": "42"}` — which is exactly what `math_da…

## 🔀 Pull Requests

### #8219 — [[doc, ci] refactor: use secrets for FORK_REPO and rename FORK_TOKEN to PAT_TOKEN](https://github.com/verl-project/verl/pull/8219)
- **作者**: Geofeng12138  **时间**: 2026-10-13 10:24 CST
- **摘要**: ## 总结 1. **`FORK_REPO` 从 Repository Variable 改为 Secret 存储** — 仓库名也属于敏感信息，统一用 Secret 管理 2. **`FORK_TOKEN` 重命名为 `PAT_TOKEN`** — 命名更直观，明确这是一个 PAT（Personal Access Token）

### #8218 — [[tool, rollout, perf] feat: add opt-in one-step tool read-ahead](https://github.com/verl-project/verl/pull/8218)
- **作者**: jiacao-amd  **时间**: 2026-10-13 03:12 CST
- **摘要**: **Closed after an applicability review.** The measured ~16% reduction applies to a constructed linked-record workload with explicit successor IDs and synthetic tool delays. It does not establish a general rollout or end-to-end RL training speedup. The current implementation has not demonstrated valu…

### #8217 — [[reward] fix: handle dict-returning score functions in PrimeRewardManager](https://github.com/verl-project/verl/pull/8217)
- **作者**: Virtuoso461  **时间**: 2026-10-12 22:51 CST
- **摘要**: Fixes #8216  ### What does this PR do?  `parallel_compute_score_async` in `PrimeRewardManager` had no branch for dict results, so dict-returning reward functions (math_dapo, rlla, ...) hit `float(result[0])`, raised `KeyError: 0`, and `verify()`'s except-all fallback silently zeroed the whole batch.…

### #8215 — [[megatron, training_utils] feat: budget BSHD padded tokens in dynamic batching](https://github.com/verl-project/verl/pull/8215)
- **作者**: yellowbee686  **时间**: 2026-10-12 21:58 CST
- **摘要**: ### What does this PR do?  With `use_dynamic_bsz=True`, `rearrange_micro_batches` budgets each micro-batch by the sum of its real tokens. On the Megatron BSHD path (`use_remove_padding=False`), however, every micro-batch is stored as `batch_size * aligned_max_seq_len` tokens: it is padded to its lon…

### #8214 — [[trainer, cfg, doc] feat: support custom validation metrics](https://github.com/verl-project/verl/pull/8214)
- **作者**: yellowbee686  **时间**: 2026-10-12 21:51 CST
- **摘要**: ### What does this PR do?  Add a config-driven hook, `data.custom_validation_metrics.{path,name}`, that lets users add validation metrics without subclassing the trainer.  Built-in validation metrics are grouped only by `data_source` and use fixed statistics (mean/std/best/worst/maj @N). Metrics spl…

### #8213 — [[megatron, training_utils] feat: budget BSHD padded tokens in dynamic batching](https://github.com/verl-project/verl/pull/8213)
- **作者**: yellowbee686  **时间**: 2026-10-12 21:49 CST
- **摘要**: ### What does this PR do?  With `use_dynamic_bsz=True`, `rearrange_micro_batches` budgets each micro-batch by the sum of its real tokens. On the Megatron BSHD path (`use_remove_padding=False`), however, every micro-batch is stored as `batch_size * aligned_max_seq_len` tokens: it is padded to its lon…

### #8212 — [[trainer, cfg] feat: make old-log-prob entropy diagnostics optional](https://github.com/verl-project/verl/pull/8212)
- **作者**: yellowbee686  **时间**: 2026-10-12 20:52 CST
- **摘要**: ### What does this PR do?  Make the entropy computed during old-log-prob recomputation optional.  The PPO (`RayPPOTrainer`) and V1 trainers always request `calculate_entropy=True` when recomputing old log-probs, only to log `actor/entropy`. The actor update does not use this entropy; it requests its…

### #8211 — [[trainer] fix: preserve underscored UIDs in V1 replay buffer](https://github.com/verl-project/verl/pull/8211)
- **作者**: yellowbee686  **时间**: 2026-10-12 20:52 CST
- **摘要**: ### What does this PR do?  Keep prompt groups isolated in the V1 `ReplayBuffer` when a UID contains underscores.  Trajectory keys are `{uid}_{session_id}_{index}`, but the replay buffer recovers the UID with `key.split("_")[0]`. A UID such as `math_00017` maps to `math`. Its trajectories are then mi…

### #8210 — [[algo] fix: pad nested importance weights for distillation PG](https://github.com/verl-project/verl/pull/8210)
- **作者**: yellowbee686  **时间**: 2026-10-12 20:52 CST
- **摘要**: ### What does this PR do?  Fix policy-gradient distillation when rollout importance weights arrive as a NestedTensor.  Before calling the PPO policy loss, `distillation_loss()` converts `log_prob`, `old_log_prob`, advantages and `response_mask` from NestedTensor to padded dense tensors, but passes `…

### #8209 — [[megatron] fix: allow BSHD with inactive MoE router losses](https://github.com/verl-project/verl/pull/8209)
- **作者**: yellowbee686  **时间**: 2026-10-12 20:52 CST
- **摘要**: ### What does this PR do?  Allow the Megatron BSHD path (`use_remove_padding=False`) with `calculate_per_token_loss=True` when no MoE router loss is active.  `calculate_per_token_loss=True` is auto-enabled by Megatron-Bridge at CP>1. On the BSHD path the engine currently raises for every model, beca…
