# verl-project/verl — 动态追踪

> 生成时间: 2026-09-20 13:16 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📌 Issue 动态
近期共收到 3 个 Bug 报告，主要集中在全异步策略和监控可视化模块：
* **#7941** [rl-insight] 在 verl 0.8.0 中，OPD 的 teacher replicas 无法在 Grafana timeline 中正常显示。
* **#7938** [fully_async] 全异步策略的 staleness reset 机制存在缺陷，会重复计算已完成的任务并可能导致训练进度阻塞。
* **#7937** [fully_async] 启用动态资源调度后，在 producer EOF（数据生产结束）后，全异步调度会陷入无限等待。

### 🔧 PR 动态
本期共合并 9 个 PR，主要涵盖异步训练缺陷修复、VeOmni 兼容性修复及性能优化：

**1. 全异步策略修复**
集中修复了上述相关 Issue：
* **#7940** 修复 #7938：将未完成样本的计算与任务回收解耦，移除任务回收顺序对 staleness budget 的影响。
* **#7939** 修复 #7937：在 producer 结束后主动释放 `wait_for_enough_samples()`，防止队列死锁。
* **#7936** 修复 #7934：在 `FullyAsyncTrainer.fit()` 结束前正确排空 final dump futures。
* **#7935** 修复 #7933：修复断点续训时已完成步数重复自增的问题。

**2. 监控与可视化修复**
* **#7946** 修复 #7941：调整 RL-Insight 的 `state_lane_id` 逻辑，将 student 和 teacher 副本的 spans 分离，避免 Grafana 时间轴冲突。

**3. VeOmni 与 FSDP 兼容性修复**
* **#7943** 修复使用 `use_fused_kernels=True` 时，root FSDP unit 被错误分片导致的 `update_actor` 崩溃问题。
* **#7942** 使 FSDPModelMerger 能够正确接受 VeOmni 引擎的 `dp_shard` mesh 作为 1-D FSDP 分片，修复 checkpoint 合并报错。
* **#7932** 修复 TRL Value-head critics 模型中，因 `**kwargs` 转发导致 FSDP 引擎无法检测 packed boundaries 的问题。

**4. 性能优化与新特性**
* **#7944** [性能优化] 针对 FSDP/Automodel，将 logits 除以温度的操作改为原地计算，避免实例化大体积张量时的内存浪费。
* **#7945** [新特性] 为 Megatron 的 fused output head 新增可选的 `liger_tp` backend，同时保留原有 Triton 实现以便进行 A/B 测试。

### 🚀 Release 动态
近期无新版本发布。

---

## 🐛 Issues

### #7941 — [[bug][rl-insight] opd's teacher replicas cannot display in the grafana timeline with rl-insight](https://github.com/verl-project/verl/issues/7941)
- **作者**: alwaysyiyu  **时间**: 2026-09-19 22:02 CST
- **标签**: bug
- **摘要**: ### System Info  verl  0.8.0 + rl-insight   ### Information  - [ ] The official example scripts - [ ] My own modified scripts  ### Tasks  - [ ] An officially supported task in the `examples` folder (such as GLUE/SQuAD, ...) - [ ] My own task or dataset (give details below)  ### Reproduction  in the …

### #7938 — [[Bug] Fully async staleness reset double-counts completed tasks and can block progress](https://github.com/verl-project/verl/issues/7938)
- **作者**: liruiluo  **时间**: 2026-09-19 19:52 CST
- **摘要**: ### System Info  - Official `verl-project/verl` main at `3efe38c759c14622fd1b2c9e3679f2d02f86bdac`. - Affected path: `verl/experimental/fully_async_policy` staleness reset/admission accounting. Dynamic resource scheduling is not required. - Python 3.12 / pytest / macOS, deterministic asyncio reprodu…

### #7937 — [[Bug] Fully async dynamic scheduling can wait forever after producer EOF](https://github.com/verl-project/verl/issues/7937)
- **作者**: liruiluo  **时间**: 2026-09-19 19:52 CST
- **摘要**: ### System Info  - Official `verl-project/verl` main at `3efe38c759c14622fd1b2c9e3679f2d02f86bdac`. - Affected path: `verl/experimental/fully_async_policy`, with dynamic resource scheduling enabled and the deactivation wait selected. - Reproduced with Python 3.12 and pytest on macOS using the checke…

## 🔀 Pull Requests

### #7946 — [[rollout, tool] fix: keep RL-Insight timeline lanes distinct](https://github.com/verl-project/verl/pull/7946)
- **作者**: axelray-dev  **时间**: 2026-09-20 07:16 CST
- **摘要**: ### What does this PR do?  RL-Insight uses `state_lane_id` to group spans into Grafana timeline lanes. Student and teacher replicas both started at `replica_0`, so teacher spans collided with rollout spans. This change carries the rollout role and replica key into the server processes and gives vLLM…

### #7945 — [[megatron] feat: add Liger TP fused cross entropy backend](https://github.com/verl-project/verl/pull/7945)
- **作者**: kolehma8  **时间**: 2026-09-20 04:23 CST
- **摘要**: ### What does this PR do?  Adds an opt-in `liger_tp` backend for Megatron's fused output head while preserving the existing Triton implementation for controlled A/B testing.  The new path calls Liger Kernel 0.8.3's native CUTLASS + NVSHMEM tensor-parallel fused linear scaled cross entropy on BF16 Ho…

### #7944 — [[fsdp, automodel] perf: scale logits by temperature in place](https://github.com/verl-project/verl/pull/7944)
- **作者**: dafu-wu  **时间**: 2026-09-20 02:14 CST
- **摘要**: ### What does this PR do?  Eager policy engines materialize a `(..., vocab_size)` logits tensor before computing token log-probabilities, then divide it by the per-sample temperature. The out-of-place division transiently holds two full-vocabulary tensors. For Qwen3.5 (`vocab_size=248320`) a 44k-tok…

### #7943 — [[veomni] fix: keep root FSDP unit unsharded when use_fused_kernels](https://github.com/verl-project/verl/pull/7943)
- **作者**: dafu-wu  **时间**: 2026-09-20 02:14 CST
- **摘要**: ### What does this PR do?  With `model_engine=veomni`, FSDP2 full shard and `use_fused_kernels=True`, the first `update_actor` fails deterministically in the fused linear-CE backward:      RuntimeError: setStorage: sizes [2048, 248320], strides [248320, 1], ...                   storage of size 0   …

### #7942 — [[veomni] fix: accept VeOmni's `dp_shard` mesh as a 1-D FSDP shard](https://github.com/verl-project/verl/pull/7942)
- **作者**: dafu-wu  **时间**: 2026-09-20 02:13 CST
- **摘要**: ### What does this PR do?  `FSDPModelMerger._calculate_shard_configuration` asserts `mesh_dim_names in (("fsdp",), ("ddp", "fsdp"))`. Checkpoints written by the VeOmni engine (`model_engine=veomni`, FSDP2 full shard) carry DTensors whose mesh is `DeviceMesh((dp_shard=N))` with placements `(Shard(0),…

### #7940 — [[fully_async] fix: count outstanding samples independently of task reaping](https://github.com/verl-project/verl/pull/7940)
- **作者**: liruiluo  **时间**: 2026-09-19 19:57 CST
- **摘要**: ### What does this PR do?  Fixes #7938.  Remove task-reaping order from the staleness budget. Done tasks can remain in `active_tasks` after their samples have been published or consumed; adding that set's size to queue size can create phantom reservations and block subsequent production.  ### Checkl…

### #7939 — [[fully_async] fix: release queue wait after producer completion](https://github.com/verl-project/verl/pull/7939)
- **作者**: liruiluo  **时间**: 2026-09-19 19:57 CST
- **摘要**: ### What does this PR do?  Fixes #7937.  Release `wait_for_enough_samples()` when the producer has finished, even if the remaining queue cannot reach the dynamic-scheduling threshold. This lets the existing consumer observe queued data and EOS instead of waiting forever before the consumer runs.  ##…

### #7936 — [[fully_async] fix: drain final dump futures before fit returns](https://github.com/verl-project/verl/pull/7936)
- **作者**: liruiluo  **时间**: 2026-09-19 18:07 CST
- **摘要**: ### What does this PR do?  Fixes #7934.  Await the inherited dump-executor drain at normal `FullyAsyncTrainer.fit()` completion, after the final validation check and checkpointing. Use `asyncio.to_thread` to keep the actor event loop responsive and propagate tail I/O errors.  ### Checklist Before St…

### #7935 — [[fully_async] fix: restore completed step count on resume](https://github.com/verl-project/verl/pull/7935)
- **作者**: liruiluo  **时间**: 2026-09-19 18:07 CST
- **摘要**: ### What does this PR do?  Fixes #7933.  Restore the completed learner-step count in `load_checkpoint()` and let the existing `fit()` increment select the first resumed update. This fixes the double increment without changing publication cadence or checkpoint loading.  ### Checklist Before Starting …

### #7932 — [[fsdp, model] fix: preserve packed boundaries for TRL value-head critics](https://github.com/verl-project/verl/pull/7932)
- **作者**: liruiluo  **时间**: 2026-09-19 14:32 CST
- **摘要**: ### What does this PR do?  Fixes #7931.  The FSDP engine detects packed-boundary support from `module.forward`'s explicit signature. TRL's `AutoModelForCausalLMWithValueHead.forward` forwards `**kwargs` to its pretrained model without exposing `cu_seqlens` in that signature. This leaves the capabili…
