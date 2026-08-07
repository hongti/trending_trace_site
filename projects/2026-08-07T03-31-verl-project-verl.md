# verl-project/verl — 动态追踪

> 生成时间: 2026-08-07 11:31 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📌 Issue 概览
*   **配置缺失问题** (#7295)：用户反馈在 PPO 训练中无法覆盖设置 `trainer.max_ckpt_to_keep`，原因是配置结构中缺少该 Key。

---

### 🔧 Pull Request 概览
本次 PR 以**缺陷修复**为主，同时包含部分新特性支持，主要集中在训练算法、引擎后端及配置管理方面：

**1. 新特性与功能增强**
*   **支持 Assistant Role** (#7296)：在 Continuous Token 的上下文合并与编码中新增了对 Assistant Role 的支持，以适配带有回滚能力的 Agent 行为。
*   **DeepSeek-V4 上下文并行修复** (#7297)：修复了 Megatron 引擎中 DeepSeek-V4 的上下文并行（Context Parallelism）实际无法运行的问题，使其真正可用。

**2. 算法与训练修复**
*   **REINFORCE++ 奖励丢失** (#7300)：修复了 REINFORCE++ 在多轮观察期间丢失结果奖励的问题，确保 `running_return` 正确传递。
*   **HFRollout 参数遗漏** (#7298)：修复了 `HFRollout` 未将 `repetition_penalty` 参数从配置传递给 `GenerationConfig` 的问题。

**3. 引擎与后端修复**
*   **FSDP1 崩溃** (#7303)：修复了 `DetachActorWorker` 在 FSDP1 下重算 `old_log_prob` 时崩溃的问题，将 FSDP1 路由至专用的保存/加载处理器。
*   **LoRA 恢复报错** (#7304)：修复了 LoRA adapter 路径在首次恢复前未设置 `sleep_level`，导致 SGLang 报 `KeyError: 'weights'` 的问题。
*   **Peft 配置不一致** (#7302)：修复了不同引擎返回的 `peft_config` 形状不一致的问题，将其值统一标准化为字符串。

**4. 配置与检查点修复**
*   **修复 PPO 配置缺失** (#7301)：直接解决了 Issue #7295，将 `max_ckpt_to_keep` 添加到 PPO YAML 配置文件中。
*   **本地检查点清理失效** (#7299)：修复了 `del_local_ckpt_after_load=True` 实际上未能删除本地检查点目录的问题。

**5. CI/CD 杂项**
*   **更新 CI 镜像** (#7293)：更新了 cicd Docker 镜像。

---

### 🚀 Release 概览
*   近期**无**新版发布。

---

## 🐛 Issues

### #7295 — [trainer.max_ckpt_to_keep can not be set now?](https://github.com/verl-project/verl/issues/7295)
- **作者**: cqray1990  **时间**: 2026-08-06 23:21 CST
- **标签**: bug
- **摘要**: ### System Info  Could not override 'trainer.max_ckpt_to_keep'. To append to your config use +trainer.max_ckpt_to_keep=2 Key 'max_ckpt_to_keep' is not in struct     full_key: trainer.max_ckpt_to_keep     object_type=dict  Set the environment variable HYDRA_FULL_ERROR=1 for a complete stack trace.  #…

## 🔀 Pull Requests

### #7304 — [fix: set sleep_level before first resume on LoRA adapter path (#7289)](https://github.com/verl-project/verl/pull/7304)
- **作者**: shotsan  **时间**: 2026-08-07 07:27 CST
- **摘要**: ## Summary  Fixes #7289 — LoRA adapter path resumes `weights` tag that was never released, causing `KeyError: 'weights'` in SGLang's `resume_memory_occupation`.  ## Root Cause  In `engine_workers.py`, the `update_weights` method:  1. **Line 766**: Unconditionally calls `resume(tags=["weights"])` 2. …

### #7303 — [fix: route FSDP1 to dedicated save/load handlers in DetachActorWorker (#7249)](https://github.com/verl-project/verl/pull/7303)
- **作者**: shotsan  **时间**: 2026-08-07 07:21 CST
- **摘要**: ## Summary  Fixes #7249 — `DetachActorWorker` crashes on FSDP1 when recomputing `old_log_prob` with `bypass_mode=False`.  ## Root Cause  `DetachActorWorker._get_strategy_handlers()` grouped `"fsdp"` (FSDP1), `"fsdp2"`, and `"veomni"` together, routing all three to `fsdp2_sharded_save_to_cpu`. That f…

### #7302 — [fix: normalize peft_config to string values across engines (#7290)](https://github.com/verl-project/verl/pull/7302)
- **作者**: shotsan  **时间**: 2026-08-07 07:18 CST
- **摘要**: ## Summary  Fixes #7290 — the two engines return differently shaped `peft_config` from `get_per_tensor_param`.  ## Root Cause  | Engine | `task_type` | `peft_type` | |--------|-------------|-------------| | FSDP (`LoraConfig.to_dict()`) | `TaskType` enum | `PeftType` enum | | Megatron (`build_peft_c…

### #7301 — [fix: add trainer.max_ckpt_to_keep to PPO config (#7295)](https://github.com/verl-project/verl/pull/7301)
- **作者**: shotsan  **时间**: 2026-08-07 07:14 CST
- **摘要**: ## Summary  Fixes #7295 — `trainer.max_ckpt_to_keep` cannot be set on PPO runs because the key is missing from the PPO YAML config.  ## Root Cause  The PPO trainer YAML (`ppo_trainer.yaml`) defines `max_actor_ckpt_to_keep` and `max_critic_ckpt_to_keep` but not `max_ckpt_to_keep`. When Hydra struct m…

### #7300 — [fix: carry running_return through observation spans in REINFORCE++ (#7278)](https://github.com/verl-project/verl/pull/7300)
- **作者**: shotsan  **时间**: 2026-08-07 06:02 CST
- **摘要**: ## Summary  Fixes #7278 — REINFORCE++ drops outcome rewards assigned before multi-turn observation spans.  ## Root Cause  In `compute_reinforce_plus_plus_outcome_advantage` (`verl/trainer/ppo/core_algos.py`), the reverse return scan multiplies `running_return` by `response_mask[:, t]` at every step.…

### #7299 — [fix: del_local_ckpt_after_load now removes local checkpoint directories](https://github.com/verl-project/verl/pull/7299)
- **作者**: shotsan  **时间**: 2026-08-07 05:59 CST
- **摘要**: ## Summary  Closes #7213  ## Problem  `del_local_ckpt_after_load=True` is documented to remove local checkpoints after loading, but the cleanup code never actually deletes anything for local paths. Three sites are affected:  1. **`FSDPCheckpointManager.load_checkpoint`** 2. **`MegatronCheckpointMana…

### #7298 — [fix: pass repetition_penalty from config to GenerationConfig in HFRollout](https://github.com/verl-project/verl/pull/7298)
- **作者**: shotsan  **时间**: 2026-08-07 05:56 CST
- **摘要**: ## Summary  Closes #7270  ## Problem  `HFRollout._generate_minibatch` reads `top_p`, `top_k`, and `temperature` from the rollout config and passes them to `GenerationConfig`, but omits `repetition_penalty`. Because `GenerationConfig` back-fills unset fields from the model's `generation_config.json`,…

### #7297 — [[megatron] fix: make DeepSeek-V4 context parallelism actually runnable](https://github.com/verl-project/verl/pull/7297)
- **作者**: HaochenYuan  **时间**: 2026-08-07 01:42 CST
- **摘要**: ### What does this PR do?  #7221 added the `contiguous` context-parallel (CP) row layout that DeepSeek-V4 requires and wired it into `MegatronEngine`. Turning on `context_parallel_size > 1` for DeepSeek-V4 still fails, because the chosen layout is never communicated to the attention kernels or to th…

### #7296 — [[rollout][continuous_token] Support Assistant Role in Merge and Encode Context](https://github.com/verl-project/verl/pull/7296)
- **作者**: gxlvera  **时间**: 2026-08-07 00:10 CST
- **摘要**: ## Summary  This PR is stacked on unmerged pr #6804. It updates Continuous Token context merging to support rollback-style agent harness behavior.  Previously, CT exposed `merge_non_assistant_*` APIs for harness-returned messages, assuming those incremental messages only contained non-assistant role…

### #7293 — [[ci] chore: Update ci image](https://github.com/verl-project/verl/pull/7293)
- **作者**: LeoYao123  **时间**: 2026-08-06 22:18 CST
- **摘要**: ### What does this PR do? Update cicd docker image  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ]…
