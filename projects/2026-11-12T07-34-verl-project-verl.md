# verl-project/verl — 动态追踪

> 生成时间: 2026-11-12 15:34 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🚀 Pull Requests (PR)
近期 PR 主要集中在**性能优化、架构重构、新特性支持以及 CI 调试**方面：

**1. 重要重构与破坏性变更**
*   **PR #8666 [BREAKING]**: 重构了 rollout 元数据传递机制。将之前盲目的 `tool_extra_fields` 序列化替换为统一的声明式契约（`rollout_fields`），仅合并已声明的字段，提升了 Reward 计算环节的清晰度与性能。

**2. 新特性与功能增强**
*   **PR #8668**: 支持通过 CLI 直接路由预设奖励函数（如 `math_dapo`, `gsm8k` 等），无需再修改底层的 Parquet 数据文件。
*   **PR #8661**: 引入异步配置项 `trainer.v1.separate_async.refill_after_sample`，允许在采样后提前提交下一批 prompt，实现 rollout 与训练的时间重叠，提升训练效率。
*   **PR #8660**: 启用 VeOmni 策略并加入针对 score centering 的快速失败配置检查，完善了跨 FSDP/Megatron/VeOmni 的多标签融合交叉熵支持。
*   **PR #8669**: 添加了 AIME 数学竞赛变体及 2024-2026 考试场次划分的数据准备指南与脚本。

**3. 性能修复与优化**
*   **PR #8663**: 优化 PPO Loss 计算，使用 masked sum 替代布尔索引来统计 `old_logprobs` 不匹配的数量，降低开销。
*   **PR #8662**: 优化 PPO Trainer v1，仅在真正需要时才计算全词表的 entropy，大幅减少 TorchTitan TPU 引擎上的计算成本。

**4. CI 调试与文档**
*   **PR #8667 & #8664**: 针对 Ascend 和 GLM5.2 多节点训练时出现的 HCCL 通信初始化失败（错误码 7）及参数同步问题进行隔离调试与诊断。
*   **PR #8665**: 文档修复，补充了 Ascend 配方仓库链接。

### 📝 Issues
*   本次动态中未提供 Issue 相关信息。

### 📦 Release
*   本次动态中未发布新版本（Release）。

---

## 🔀 Pull Requests

### #8669 — [[data, doc, tests] feat: catalog AIME variants and prepare pinned 2024-2026 exam splits](https://github.com/verl-project/verl/pull/8669)
- **作者**: tongyx361  **时间**: 2026-11-12 15:24 CST
- **摘要**: ### What does this PR do?  Add an **AIME Variants and Exam-Session Splits** guide (`docs/data/aime_exam_splits.md`, indexed under `Data Preparation` in `docs/index.rst`) and standalone preparation script (`examples/data_preprocess/aime_exam_splits.py`):  1. **Catalogs existing public Hugging Face AI…

### #8668 — [[reward, cfg, doc, tests] feat: support CLI preset reward routing without modifying Parquet files](https://github.com/verl-project/verl/pull/8668)
- **作者**: tongyx361  **时间**: 2026-11-12 14:54 CST
- **摘要**: ### What does this PR do?  Allow selecting built-in preset reward functions (`math_dapo`, `math_reward`, `math_verify`, `gsm8k`, `prime_math`, `prime_code`, `geo3k`, `search_r1_like_qa_em`, `search_r1_like_qa_subem`) directly via CLI/Hydra config (`reward.preset` and `reward.data_source_presets`) wi…

### #8667 — [[ci] debug: isolate GLM5.2 multinode HCCL initialization failure](https://github.com/verl-project/verl/pull/8667)
- **作者**: fangweii1  **时间**: 2026-11-12 14:50 CST
- **摘要**: The GLM5.2 Top16 multinode GRPO job fails while Megatron-Bridge broadcasts reference-model weights: `hcclCommInitRootInfoConfig(...), error code is 7 (resource unavailable)`, followed by `ERR02200 DIST call hccl api failed`.  This draft is based on PR #8620 at `c9b47e75e2fc7182ab65b355596722898600e5…

### #8666 — [[BREAKING][rollout, reward, perf, doc, tests] refactor: filter rollout metadata for reward extra_info by declaration](https://github.com/verl-project/verl/pull/8666)
- **作者**: tongyx361  **时间**: 2026-11-12 13:08 CST
- **摘要**: ### What does this PR do?  Replaces blind `tool_extra_fields` serialization in `AgentLoopWorker._compute_score` with a single uniform declaration contract (`rollout_fields`, default `()`), merging only explicitly declared rollout fields into a fresh per-sample `non_tensor_batch["extra_info"]` dictio…

### #8665 — [[doc] fix: add Ascend recipe repository links](https://github.com/verl-project/verl/pull/8665)
- **作者**: renjunying2  **时间**: 2026-11-12 11:52 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #8664 — [[ci] debug: run Ascend CI with console HCCL diagnostics](https://github.com/verl-project/verl/pull/8664)
- **作者**: fangweii1  **时间**: 2026-11-12 11:16 CST
- **摘要**: The Ascend `e2e_ppo_trainer_fsdp_vllm_ascend` job intermittently fails during critic FSDP parameter synchronization: `hcclCommInitRootInfoConfig(...), error code is 7 (resource unavailable)`. Original failure: https://github.com/verl-project/verl/actions/runs/42825993909/job/129865148684.  Run Ascen…

### #8663 — [[trainer, perf] fix: count old_logprobs mismatches with a masked sum instead of boolean indexing](https://github.com/verl-project/verl/pull/8663)
- **作者**: syangx39  **时间**: 2026-11-12 08:07 CST
- **摘要**: ### What does this PR do?  `actor/old_logprobs_mismatch_count`, added in #8026, is computed in `ppo_loss` with boolean-mask indexing:  ```python old_logprobs_mismatch = (log_prob.float() != old_log_prob.float())[response_mask].sum() ```  The indexed tensor has one element per valid response token of…

### #8662 — [[trainer, perf] fix: only compute entropy in v1 old_log_prob when it is used](https://github.com/verl-project/verl/pull/8662)
- **作者**: jialei777  **时间**: 2026-11-12 03:50 CST
- **摘要**: ### What does this PR do?  `PPOTrainer._compute_old_log_prob` (v1) always sends `calculate_entropy=True` to `compute_log_prob`. Entropy over the full vocabulary is expensive: on the TorchTitan TPU engine it is a chunked pass over 248k logits per token. The trainer then pulls the `entropy` field back…

### #8661 — [[trainer, cfg] feat: separate_async refill_after_sample to overlap rollout with training](https://github.com/verl-project/verl/pull/8661)
- **作者**: jialei777  **时间**: 2026-11-12 03:49 CST
- **摘要**: ### What does this PR do?  Adds an opt-in `trainer.v1.separate_async.refill_after_sample` (default `false`).  In separate-async mode, `prepare_step()` submits the next prompt batch at the start of step *k+1*. That happens after step *k*'s `old_log_prob`, `update_actor` and weight sync have all finis…

### #8660 — [[algo, veomni, cfg, doc, tests] feat: enable VeOmni strategy and fail-fast config checks for score centering](https://github.com/verl-project/verl/pull/8660)
- **作者**: tongyx361  **时间**: 2026-11-12 03:43 CST
- **摘要**: ### What does this PR do?  Completes the configuration validation and documentation layer on top of #8654 (unified TileLang multi-label fused linear cross-entropy across FSDP, Megatron, and VeOmni) so `model_engine=veomni` (`actor.strategy=veomni`) passes `validate_config` for Score Centering ([Mare…
