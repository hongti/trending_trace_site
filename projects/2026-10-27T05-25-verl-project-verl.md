# verl-project/verl — 动态追踪

> 生成时间: 2026-10-27 13:25 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要。根据您提供的活动记录，本次更新主要集中在 Pull Request 上，无 Issue 和 Release 动态。

### 🚀 Pull Request (PR) 动态

近期 PR 主要涉及新特性引入、vLLM 与底层硬件的修复适配，以及 CI/CD 的优化。

#### 1. 新特性
*   **新增 `sharded_nixl` 检查点引擎** (#8395)：支持 disaggregated rollout，且无需训练端进行 all-gather 操作。每个训练 rank（支持 FSDP 或 Megatron）仅导出其持有的分片。
*   **实现 DAS 论文机制** (#8396)：在 rollout 中引入了 DAS 论文描述的机制，用于管理 per-problem suffixTree。

#### 2. 重要修复
*   **vLLM 模型与引擎逻辑修复**：
    *   修复权重同步前 `abort_all_requests` 的阻塞问题 (#8397)：将 submission-gate barrier 的释放时机从“生成首个 token 时”提前到“请求进入引擎时”，提升效率。
    *   恢复 Hopper 架构上 vLLM 的 MXFP4 MoE 默认选择 (#8398)：修复了之前强制使用 `deep_gemm` 的问题，原生 DeepSeek-V4 等模型将不再被强制覆盖。
*   **FSDP 与 NPU 适配修复**：
    *   修复 NPU 上 Qwen3.5-35B FSDPTurbo 训练报错 (#8400)：将不支持的 `eager/fused` 替换为 `all2all`，并为 actor 和 reference model 分别配置 `use_fused_ops`。
*   **MLflow 并发冲突修复** (#8393)：针对多 `AgentLoopWorker` 并发调用导致 MLflow SQLite 初始化失败的问题，引入了跨进程文件锁进行序列化处理。

#### 3. CI/CD 与文档维护
*   **Ascend 生态适配**：将剩余的 Ascend nightly CI 任务迁移至 800i runners (#8394)，更新相关模型路径；为 `npu_unit_tests` 添加诊断日志 (#8392)；修正 Ascend 官方文档中的细微错误 (#8391)。
*   **依赖更新**：为 fsdp-turbo backend 更新了 `transformers` 依赖库 (#8399)。

---
*注：本期输入数据中未包含 Issue 和 Release 相关动态。*

---

## 🔀 Pull Requests

### #8400 — [[fsdp] fix: update actor and ref FSDPTurbo settings for Qwen3.5-35B training on NPU](https://github.com/verl-project/verl/pull/8400)
- **作者**: renjunying2  **时间**: 2026-10-27 11:50 CST
- **摘要**: ### What does this PR do?  Fix Unsupported dispatcher fused in the Qwen3.5-35B FSDPTurbo example by replacing eager/fused with all2all and configuring use_fused_ops separately for actor and reference models.  Related: [FSDPTurbo PR #197](https://gitcode.com/Ascend/FSDPTurbo/pull/197), which changed …

### #8399 — [[ci] fix: update transformers for fsdp-turbo backend](https://github.com/verl-project/verl/pull/8399)
- **作者**: pengnuoheng  **时间**: 2026-10-27 11:16 CST
- **摘要**: ### What does this PR do?  update transformers for fsdp-turbo backend  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`…

### #8398 — [[rollout, vllm] fix: keep vLLM's MXFP4 MoE selection on Hopper instead of forcing deep_gemm](https://github.com/verl-project/verl/pull/8398)
- **作者**: linmuchuiyang  **时间**: 2026-10-27 10:28 CST
- **摘要**: ### What does this PR do?  Since #8308, `set_default_vllm_quantization_backends` defaults `moe_backend` to `deep_gemm` for every quantization config that is not NVFP4. A native DeepSeek-V4 checkpoint (Flash and Pro) carries only the FP8 block config under `quantization_config` and declares its route…

### #8397 — [[rollout, vllm] fix: release the submission-gate barrier when a request lands in the engine, not on its first token](https://github.com/verl-project/verl/pull/8397)
- **作者**: xiaoxlu  **时间**: 2026-10-27 04:55 CST
- **摘要**: ### What does this PR do?  Before a weight sync, `abort_all_requests()` waits for `_admitting == 0` (no request is between the submission gate and the engine) and only then calls `pause_generation()`. `generate()` decremented `_admitting` only on the request's **first output**, i.e. once it had been…

### #8396 — [[rollout, vllm] feat: Implementation of the DAS paper](https://github.com/verl-project/verl/pull/8396)
- **作者**: kfswain  **时间**: 2026-10-27 02:17 CST
- **摘要**: ### What does this PR do?  This PR implements the mechanisms to manage per-problem suffixTree as described in [DAS](https://arxiv.org/pdf/2511.13841)  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: https://github.com/verl-project/verl/pulls?q=is%3Apr…

### #8395 — [[ckpt, megatron, rollout, vllm, ci, doc] feat: sharded_nixl checkpoint engine with no trainer-side all-gather](https://github.com/verl-project/verl/pull/8395)
- **作者**: rafiw  **时间**: 2026-10-27 00:21 CST
- **摘要**: ### What does this PR do?  Adds `sharded_nixl`, a checkpoint engine for disaggregated rollout with no trainer-side all-gather. Every trainer rank (FSDP or Megatron) exports only the shards it holds and NIXL-WRITEs them into the rollout workers' staging buffers. Each rollout worker reports what it lo…

### #8394 — [[ci] fix: migrate remaining Ascend nightly jobs to 800i runners](https://github.com/verl-project/verl/pull/8394)
- **作者**: lxb007981  **时间**: 2026-10-26 21:47 CST
- **摘要**: ### What does this PR do?  Replace the remaining a3-8 and a3-16 runner labels with their 800i equivalents. Update explicit model paths and the VeOmni model symlink to use the ModelScope cache mount.  Retain the existing dataset paths and writable cache and lock directories.  ### Checklist Before Sta…

### #8393 — [fix: serialize mlflow sqlite init with a cross-process file lock](https://github.com/verl-project/verl/pull/8393)
- **作者**: analytistic  **时间**: 2026-10-26 20:42 CST
- **摘要**: ## Problem  With `actor_rollout_ref.rollout.trace.backend=mlflow` and the default `sqlite:///tmp/mlruns.db`, every `AgentLoopWorker` calls `RolloutTraceConfig.init` → `mlflow.set_experiment` concurrently. Alembic migrations on the same sqlite file race — worker startup crashes, and a half-migrated d…

### #8392 — [add diag log for ascend npu_unit_tests](https://github.com/verl-project/verl/pull/8392)
- **作者**: lxb007981  **时间**: 2026-10-26 20:24 CST
- **摘要**: ### What does this PR do?  test npu_unit_tests  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`,…

### #8391 — [[doc] chore: correct minor errors in Ascend docs](https://github.com/verl-project/verl/pull/8391)
- **作者**: lxb007981  **时间**: 2026-10-26 19:11 CST
- **摘要**: ### What does this PR do?  Correct minor errors in Ascend docs  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `mega…
