# verl-project/verl — 动态追踪

> 生成时间: 2026-08-26 10:10 CST

## AI 总结

## verl-project/verl 近期动态摘要

---

### 🐛 Issue

- **#7550** — 训练 Qwen3.5-122B 时恢复 KV Cache 出错。作者报告在 Python 3.11.9 环境下遇到 resume kv cache 错误，具体细节待进一步排查。

---

### 🔀 Pull Request

#### ⚠️ 破坏性变更
- **#7557 / #7556** — **将 NeoProto 设为 V0 DataProto 的默认实现**。两份关联 PR 重构了同步 V0 `RayPPOTrainer` 的数据容器，使 NeoProto 成为唯一的底层路径，同时保留原有 `DataProto` API 兼容性。属于 **Breaking Change**。

#### 🚀 新特性
- **#7552** — 新增 **NCCL M2N Reshard 检查点后端**，接入 veRL 的 FSDP rank-local shard 导出，是 NCCL M2N 集成分阶段方案的第二步。
- **#7555** — 支持通过环境变量 `VERL_MLFLOW_EXPERIMENT_PREFIX` 将 MLflow 实验嵌套到指定根路径下，适配 Databricks 等托管后端需求。

#### 🔧 Bug 修复
- **#7554** — **性能优化**：重新计算旧 log prob 时跳过不必要的全词表熵计算，对 legacy 和 V1 trainer 均生效。
- **#7553** — 修复动态 micro-batch packing 的 token 限制问题，原先仅用理论下界估算 micro-batch 数量，现强制执行严格限制。
- **#7551** — 修复 `DataProto` 在 Ray 跨进程传输时序列化方法读取不一致的问题，改为从 payload 本身读取。
- **#7548** — 防止 SFT trainer 在未配置验证集时进入 validation 流程，修复了周期性验证分支的绕过问题。
- **#7547** — 在 fully async 模式下，为进入流式生成的每条轨迹分配唯一优先级，修复了优先级缺失导致的排序不稳定问题。

#### 🧹 杂项
- **#7549** — 修正 Ascend CI 工作流的步骤命名，并禁用夜间 CI 的检查点保存。

---

### 📦 Release

本期无新版本发布。

---

## 🐛 Issues

### #7550 — [Resume kv cache error occurred during the training of Qwen3.5‑122B](https://github.com/verl-project/verl/issues/7550)
- **作者**: mikequan0425  **时间**: 2026-08-25 17:04 CST
- **标签**: bug
- **摘要**: ### System Info  ----------Python Info---------- Version      : 3.11.9 Compiler     : GCC 12.2.0 Build        : ('main', 'May 11 2026 12:00:44') Arch         : ('64bit', '') ------------Pip Info----------- Version      : 26.1.2 Directory    : /usr/local/lib/python3.11/site-packages/pip vllm         …

## 🔀 Pull Requests

### #7557 — [[BREAKING][data, trainer] refactor: make NeoProto the default V0 DataProto](https://github.com/verl-project/verl/pull/7557)
- **作者**: alanSquirrelyz  **时间**: 2026-08-26 05:06 CST
- **摘要**: ### What does this PR do?    This PR makes the NeoProto-backed `DataProto` the public data representation used by the synchronous V0 `RayPPOTrainer`.    The existing `DataProto` API remains available through `from verl import DataProto`, while the previous TensorDict-backed implementation remains ex…

### #7556 — [[BREAKING][data, trainer] refactor: make NeoProto the V0 DataProto implementation](https://github.com/verl-project/verl/pull/7556)
- **作者**: alanSquirrelyz  **时间**: 2026-08-26 02:43 CST
- **摘要**: ### What does this PR do?  This PR makes the NeoProto-backed `DataProto` the single data-container path for the synchronous V0 `RayPPOTrainer`, while preserving the established `DataProto` API and keeping the previous concrete implementation available explicitly as `LegacyDataProto`.  It is a follow…

### #7555 — [[training_utils, rollout] feat: nest MLflow experiments under VERL_MLFLOW_EXPERIMENT_PREFIX](https://github.com/verl-project/verl/pull/7555)
- **作者**: jmaicher  **时间**: 2026-08-25 23:25 CST
- **摘要**: ### What does this PR do?  Adds an opt-in `VERL_MLFLOW_EXPERIMENT_PREFIX` environment variable that nests the MLflow experiment under a root path. Managed MLflow backends such as Databricks require the experiment to be an absolute workspace path, but `trainer.project_name` is typically a bare name t…

### #7554 — [[trainer, perf] fix: skip unused entropy computation for old log probs](https://github.com/verl-project/verl/pull/7554)
- **作者**: Yaegaki1Erika  **时间**: 2026-08-25 22:03 CST
- **摘要**: ### What does this PR do?  This PR avoids unnecessary full-vocabulary entropy computation when recomputing old log probabilities.  Both the legacy `RayPPOTrainer` and the V1 trainer currently force `calculate_entropy=True` in `_compute_old_log_prob()`, even when the existing configuration disables e…

### #7553 — [[trainer] fix: enforce strict dynamic micro-batch token limits](https://github.com/verl-project/verl/pull/7553)
- **作者**: Begunner  **时间**: 2026-08-25 21:58 CST
- **摘要**: ### What does this PR do?  Dynamic micro-batch packing previously estimated the number of micro-batches using:  `ceil(total_sequence_length / max_token_len)`  This value is only a theoretical lower bound. Individual samples cannot be split, and the Karmarkar-Karp heuristic primarily balances computa…

### #7552 — [[ckpt, fsdp] feat: add NCCL M2N Reshard backend](https://github.com/verl-project/verl/pull/7552)
- **作者**: ss16118  **时间**: 2026-08-25 17:47 CST
- **摘要**: ### What does this PR do?  Adds the NCCL M2N Reshard checkpoint backend and connects it to veRL's FSDP rank-local shard export.  This is PR 2 in the staged NCCL M2N integration and depends on #7432. Until #7432 merges, GitHub will also show PR1's layout primitives in this PR's diff. The matching vLL…

### #7551 — [[ray] fix: read the DataProto serialization method from the payload](https://github.com/verl-project/verl/pull/7551)
- **作者**: alanhuangyoo  **时间**: 2026-08-25 17:46 CST
- **摘要**: ### What does this PR do?  `DataProto.__getstate__` and `__setstate__` each read `VERL_DATAPROTO_SERIALIZATION_METHOD` from their own process, but they run on opposite sides of a Ray transfer. The variable is not part of the runtime-env whitelist in `verl/trainer/constants_ppo.py` and nothing else a…

### #7549 — [[ci] chore: correct step naming for Ascend ci](https://github.com/verl-project/verl/pull/7549)
- **作者**: lxb007981  **时间**: 2026-08-25 16:27 CST
- **摘要**: ### What does this PR do? 1. Use the proper name for the step in the nightly_ascend.yml workflow. 2. Do not save checkpoints for the nightly CI.  ### Checklist Before Starting * [x]  Search for similar PRs. Paste at least one query link here: ... * [x]  Format the PR title as `[{modules}] {type}: {d…

### #7548 — [[trainer] fix: guard SFT validation without val data](https://github.com/verl-project/verl/pull/7548)
- **作者**: ai-yang  **时间**: 2026-08-25 15:52 CST
- **摘要**: ### What does this PR do?  Prevents both SFT trainers from entering validation when no validation dataset is configured.  Previously, the periodic-validation branch bypassed the `val_dataloader is not None` guard because of boolean-operator precedence. A job with `data.val_files=null` and `trainer.t…

### #7547 — [[fully_async, rollout] fix: assign unique priorities before streaming generation](https://github.com/verl-project/verl/pull/7547)
- **作者**: ai-yang  **时间**: 2026-08-25 15:51 CST
- **摘要**: ### What does this PR do?  When a fully async input does not already provide priorities, assigns a stable, distinct priority to every trajectory before it enters concurrent agent-loop generation.  The base `AgentLoopManager` injects priorities for ordinary batch generation, but fully async dispatche…
