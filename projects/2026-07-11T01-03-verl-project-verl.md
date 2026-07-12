# verl-project/verl — 动态追踪

> 生成时间: 2026-07-11 09:03 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📋 Issue 动态
1. **FSDP2 权重导出性能优化建议 (#7015)**：指出当前 FSDP2 导出权重时逐参数进行 all-gather（7B模型约340次），建议通过 `FSDPModule.unshard()` 批处理来进一步降低导出开销。
2. **同步训练器 CI 适配 (#7007)**：提议在同步训练器 CI 中启用 SkipManager，并相应调整 `param_sync_step`。
3. **本地环境安装诉求 (#7006)**：用户请求提供包含所有依赖版本信息的 Dockerfile，以便在本地顺利安装 verl-0.8.0 并运行 GRPO 算法。

---

### 🔧 PR 动态
**核心新特性：**
* **新增 HY v3 Megatron GRPO 示例 (#7003)**：添加了基于 mBridge 的 DAPO-style GRPO Megatron 训练脚本，支持 `tencent/Hy3` 模型。

**重要修复与性能优化：**
* **FSDP 权重同步与导出优化 (#7014, #7005)**：
  * #7014 修复了 FSDP LoRA rollout 权重同步路径中的**过时权重（stale-weight）bug**，确保合并 LoRA 权重后在上下文退出前正确同步。
  * #7005 针对上述 Issue #7015，**跳过了 FSDP2 权重导出中整个本地分片的 GPU staging 往返开销**，显著降低了导出成本。
* **Checkpoint 状态持久化修复 (#7013)**：修复了 PPO 训练中 Adaptive KL Controller 的值在 checkpoint 恢复时被重置为初始系数的问题，现在会正确保存并恢复其演化状态。
* **Megatron CP 分割对齐修复 (#7012)**：修复了 `context_parallel_size > 1` 且批次样本长度可变时，`compute_forward_kl_topk` 的 AssertionError，对齐了 teacher 与 student 的 tensor seqlen。
* **异步训练锁与日志修复 (#7010, #7011)**：
  * #7010 修复了 Fully Async Rollouter 在等待 rollout 容量时持有状态锁导致阻塞 `reset_staleness()` 的问题（等待时释放锁）。
  * #7011 修复了异步训练完成序列可能导致 wandb logger 崩溃的关闭时序问题。

**文档与 CI 维护：**
* **Ascend / 昇腾生态更新 (#7002, #7008, #7009)**：更新了 Atlas 950DT A5 的 torch_npu/MindSpeed/Megatron 安装文档，并更新了 Ascend A2 Docker image tag 及相关文档中的 Docker 名称。

---

### 🚀 Release 动态
* 近期**无新版本发布**记录。（注：Issue #7006 中提及的 `verl-0.8.0` 为用户侧讨论的已有版本，本期无相关新 Release 动作）。

---

## 🐛 Issues

### #7015 — [[fsdp] FSDP2 weight export gathers per-parameter; batching via FSDPModule.unshard() could cut the export cost further](https://github.com/verl-project/verl/issues/7015)
- **作者**: ChangyiYang  **时间**: 2026-07-11 05:23 CST
- **摘要**: ### Context  `get_per_tensor_param` exports FSDP2 weights with a per-parameter `.to(device).full_tensor()` — one independent all-gather per parameter (~340 for a 7B model), plus one pageable H2D per parameter when the model is CPU-offloaded.  With the staging round trip in place this cost is invisib…

### #7007 — [Enable SkipManager in the sync trainer CI and adapt the param_sync_step accordingly.](https://github.com/verl-project/verl/issues/7007)
- **作者**: tardis-key  **时间**: 2026-07-10 16:30 CST
- **摘要**: TODO: Enable SkipManager in the sync trainer CI and adapt the param_sync_step accordingly.  _Originally posted by @tardis-key in https://github.com/verl-project/verl/issues/6897#issuecomment-4933501034_

### #7006 — [How to install verl-0.8.0 in a local environment? Could you provide a Dockerfile containing all packages and version dependencies?](https://github.com/verl-project/verl/issues/7006)
- **作者**: zyoohv  **时间**: 2026-07-10 15:54 CST
- **摘要**: ### Feature request  How to install verl-0.8.0 in a local environment? Could you provide a Dockerfile containing all packages and version dependencies?  I hope to successfully run the GRPO algorithm for Qwen3.6 based on vLLM + Megatron.  ### Motivation  I've noticed the configuration files under the…

## 🔀 Pull Requests

### #7014 — [[fsdp] fix: sync merged LoRA weights before context exit](https://github.com/verl-project/verl/pull/7014)
- **作者**: rongkunxue  **时间**: 2026-07-10 23:20 CST
- **摘要**: This PR fixes a stale-weight bug in the FSDP LoRA rollout weight sync path when `actor_rollout_ref.model.lora.merge=True`.  Before this change, `FSDPEngine.get_per_tensor_param()` built a `state_dict()` inside `merged_lora_context()`, but returned a lazy iterator that was consumed only after the con…

### #7013 — [[trainer, ckpt] fix: persist adaptive KL controller state across checkpoint resume](https://github.com/verl-project/verl/pull/7013)
- **作者**: Epochex  **时间**: 2026-07-10 22:19 CST
- **摘要**: Fixes #2653  ## Summary  `AdaptiveKLController.value` evolves during PPO training but was not included in trainer checkpoints. Resuming therefore restored the configured initial coefficient instead of the learned value, changing subsequent KL-penalized rewards.  This change adds a small versioned, t…

### #7012 — [[megatron] fix: align teacher tensor seqlen with student for forward_kl_topk CP split](https://github.com/verl-project/verl/pull/7012)
- **作者**: gaohongkui  **时间**: 2026-07-10 21:51 CST
- **摘要**: ### What does this PR do?  Fixes an `AssertionError` in `compute_forward_kl_topk` when `context_parallel_size > 1` and batch samples have variable lengths.  **Root cause**: `preprocess_bshd_engine` independently computes `aligned_max_seqlen` for teacher and student nested tensors from their respecti…

### #7011 — [[wandb] fix: fixes async training finish sequence](https://github.com/verl-project/verl/pull/7011)
- **作者**: theely  **时间**: 2026-07-10 21:00 CST
- **摘要**: ### What does this PR do?  Fixes the finish training sequence that might cause wandb logger to crash.   ### Test  Log output before fix is applied indicates wandb is not correctly shut down.  ``` (TaskRunner pid=47918) 'Final validation metrics: None' (TaskRunner pid=47918)  (TaskRunner pid=47918) E…

### #7010 — [[fully_async] fix: release state lock while waiting for rollout capacity](https://github.com/verl-project/verl/pull/7010)
- **作者**: kdubovikov  **时间**: 2026-07-10 19:54 CST
- **摘要**: ### What does this PR do?  `FullyAsyncRollouter._processor_worker()` currently waits for rollout capacity while holding `self.lock`. A long-running rollout can therefore block `reset_staleness()`, which needs the same lock after a parameter update, and serialize training behind the rollout tail.  Th…

### #7009 — [ci: update Ascend A2 Docker image tag](https://github.com/verl-project/verl/pull/7009)
- **作者**: 20000912gyf-dotcom  **时间**: 2026-07-10 17:35 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #7008 — [[doc] refactor: update ascend docker name ](https://github.com/verl-project/verl/pull/7008)
- **作者**: yyyy2000  **时间**: 2026-07-10 17:20 CST
- **摘要**: ### What does this PR do?  as title  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, `sglang`, …

### #7005 — [[fsdp] fix: skip the whole-shard staging round trip in FSDP2 weight export](https://github.com/verl-project/verl/pull/7005)
- **作者**: ChangyiYang  **时间**: 2026-07-10 15:00 CST
- **摘要**: ### What does this PR do?  `get_per_tensor_param` stages the entire local shard onto the GPU (`load_fsdp_model_to_gpu`) before `state_dict()` and offloads it back afterwards. That round trip is an **FSDP1 requirement** — FSDP1's state_dict export runs through the unshard machinery, which asserts fla…

### #7003 — [[megatron, trainer] feat: add HY v3 Megatron GRPO example](https://github.com/verl-project/verl/pull/7003)
- **作者**: xhx1022  **时间**: 2026-07-10 10:44 CST
- **摘要**: ### What does this PR do?  Adds HY v3 Megatron GRPO support pieces:  - Add `examples/grpo_trainer/run_hy_v3_megatron.sh`, a DAPO-style GRPO Megatron training script for `tencent/Hy3` with mBridge enabled. - Add HY v3 FLOPs estimation support in `verl/utils/flops_counter.py`.  Related Megatron-Bridge…

### #7002 — [[doc] feat: update installation instructions for Atlas 950DT A5](https://github.com/verl-project/verl/pull/7002)
- **作者**: fh188  **时间**: 2026-07-10 10:20 CST
- **摘要**: Updated the installation guidance for torch_npu and clarified the installation instructions for MindSpeed and Megatron.  ### What does this PR do?  update installation instructions for Atlas 950DT A5  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ..…
