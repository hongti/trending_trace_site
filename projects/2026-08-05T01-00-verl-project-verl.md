# verl-project/verl — 动态追踪

> 生成时间: 2026-08-05 09:00 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 一、 Issue 动态
*   **[RFC] 新增 `nccl_parallel` 检查点引擎** (#7262)：
    提议新增一种累加式的检查点引擎后端 `nccl_parallel`，旨在消除现有 `nccl` 引擎在跨节点权重同步时单发送方（actor rank 0 广播）的性能瓶颈，通过聚合多个发送方 NIC 实现多节点并发传输。

### 二、 PR 动态
本期 PR 以 Bug 修复为主，同时落地了上述 RFC 的新特性：

*   **🚀 新特性**
    *   **实现 `nccl_parallel` 检查点引擎** (#7263)：落地了 Issue #7262 的提案，所有 S actor ranks 并发发送权重，彻底解决跨节点同步的单点瓶颈。

*   **🛠️ Bug 修复**
    *   **Megatron 维度不匹配** (#7261)：修复了处理多维 THD 张量（如 Router Replay 数据）时，上下文并行导致的维度不匹配报错。
    *   **验证阶段随机性污染** (#7260)：修复了 `bootstrap_metric()` 在验证阶段调用 `np.random.seed()` 导致进程全局 NumPy 随机流被重置的问题，隔离了验证阶段的随机性。
    *   **配置校验逻辑错误** (#7259)：修复了静态 actor 批处理下，`validate_config()` 对每 GPU 静态微批次整除性校验的计算逻辑。
    *   **检查点恢复步数解析错误** (#7257)：修复了 SFT `CheckpointHandler` 从路径解析恢复步数时，因正则匹配到早期目录而获取错误步数的问题。
    *   **SGLang 失败进程未回收** (#7256)：修复了 SGLang 服务器启动失败时，子进程未被正确回收（reap）导致的僵尸进程问题。
    *   **计算得分计时指标缺失** (#7255)：修复了 V1 trainer（包括 `separate_async` 模式）中未上报 `compute_score` 耗时指标的问题。
    *   **PRIME 代码进程泄漏** (#7254)：修复了 PRIME 代码奖励评估中创建的 `multiprocessing.Manager` 未被清理，导致进程/资源泄漏的问题。
    *   **变长对话数组转换报错** (#7253)：修复了 NumPy 2.x 环境下，包含不同轮次的变长对话数据使用 `np.array()` 转换时引发 `ValueError` 的问题。

*   **🔧 CI/杂项**
    *   **清理 NPU 测试环境变量** (#7258)：移除了 GSPO Qwen3-8B FSDP2 NPU 夜间测试中的 `RAY_DEDUP_LOGS=0`，恢复使用 Ray 的默认日志设置。

### 三、 Release 动态
*   近期无新版本发布。

---

## 🐛 Issues

### #7262 — [[RFC] `nccl_parallel` checkpoint engine: aggregate sender NICs for cross-node weight sync](https://github.com/verl-project/verl/issues/7262)
- **作者**: savaresejeremy  **时间**: 2026-08-05 03:23 CST
- **摘要**: ### Summary  We propose an additive checkpoint-engine backend, `nccl_parallel`, that removes the single-sender bottleneck in the `nccl` engine's cross-node weight sync. Instead of actor rank 0 broadcasting every bucket, all S actor ranks send concurrently, each owning a stripe of the global bucket s…

## 🔀 Pull Requests

### #7263 — [[ckpt] feat: add nccl_parallel checkpoint engine (all actor ranks send)](https://github.com/verl-project/verl/pull/7263)
- **作者**: savaresejeremy  **时间**: 2026-08-05 03:28 CST
- **摘要**: ### What does this PR do?  Adds an additive checkpoint-engine backend, `nccl_parallel`, that removes the single-sender bottleneck in cross-node weight sync. All S actor ranks send concurrently, each owning a stripe of the global bucket sequence over its own NCCL group; receivers join all S groups an…

### #7261 — [[megatron] fix: pad multidimensional THD tensors along the sequence dimension](https://github.com/verl-project/verl/pull/7261)
- **作者**: yyDing1  **时间**: 2026-08-04 19:51 CST
- **摘要**: ### What does this PR do?  Fix a context-parallel shape mismatch in `preprocess_thd_engine` when processing multidimensional nested tensors such as Router Replay data.  The observed error was:      RuntimeError: The expanded size of the tensor (1) must match     the existing size (0) at non-singleto…

### #7260 — [[trainer] fix: isolate validation bootstrap randomness](https://github.com/verl-project/verl/pull/7260)
- **作者**: RerankerGuo  **时间**: 2026-08-04 19:28 CST
- **摘要**: ### What does this PR do?  `bootstrap_metric()` currently calls `np.random.seed(seed)` before drawing bootstrap indices. Validation therefore resets the process-wide NumPy random stream every time these metrics are computed.  A minimal reproduction shows unrelated random values changing after a metr…

### #7259 — [[cfg] fix: validate per-GPU static micro batches](https://github.com/verl-project/verl/pull/7259)
- **作者**: RerankerGuo  **时间**: 2026-08-04 19:23 CST
- **摘要**: ### What does this PR do?  For static actor batching, `validate_config()` checks that:  ```text train_batch_size * rollout.n ```  is divisible by the smallest batch the workers can consume. The Megatron path already uses `DP size * ppo_micro_batch_size_per_gpu`, but other backends only use the GPU c…

### #7258 — [[ci] chore: remove RAY_DEDUP_LOGS=0 from GSPO Qwen3-8B FSDP2 NPU nightly test](https://github.com/verl-project/verl/pull/7258)
- **作者**: chengminhua  **时间**: 2026-08-04 19:04 CST
- **摘要**: ### What does this PR do?  Remove `export RAY_DEDUP_LOGS=0` from `tests/special_npu/nightly_ci_ascend/run_gspo_qwen3_8b_fsdp2_npu.sh` so the GSPO Qwen3-8B FSDP2 NPU nightly test uses Ray's default log dedup behavior (less noisy logs / avoid duplicate log flooding).  ### Checklist Before Starting  - …

### #7257 — [[ckpt] fix: parse resume step from checkpoint directory](https://github.com/verl-project/verl/pull/7257)
- **作者**: RerankerGuo  **时间**: 2026-08-04 18:37 CST
- **摘要**: ### What does this PR do?  The SFT `CheckpointHandler` derives the resumed training counter with:  ```python re.search(r"global_step_(\d+)", path) ```  Because the search scans the complete path, an earlier marker in a parent directory wins. For example:  ```text /tmp/global_step_900/archive/global_…

### #7256 — [[sglang] fix: reap failed server processes](https://github.com/verl-project/verl/pull/7256)
- **作者**: RerankerGuo  **时间**: 2026-08-04 18:35 CST
- **摘要**: ### What does this PR do?  `launch_server_process()` reports two classes of SGLang startup failure:  - the child exits before health or cache initialization finishes; - health or cache initialization exceeds the overall timeout.  The current paths raise immediately, or call `terminate()` and then ra…

### #7255 — [[fully_async, trainer] fix: report compute score timing metrics](https://github.com/verl-project/verl/pull/7255)
- **作者**: YolandaLyj  **时间**: 2026-08-04 18:24 CST
- **摘要**: ### What does this PR do?  Report agent-loop `compute_score` timing metrics in the V1 trainer, including `trainer.v1.trainer_mode=separate_async`.  `AgentLoopWorkerTQ` already stores `AgentLoopOutput.metrics` with each trajectory in TransferQueue, but `PPOTrainer._compute_metrics()` does not read th…

### #7254 — [[reward] fix: clean up PRIME code worker processes](https://github.com/verl-project/verl/pull/7254)
- **作者**: RerankerGuo  **时间**: 2026-08-04 17:51 CST
- **摘要**: ### What does this PR do?  `prime_code.check_correctness()` creates a `multiprocessing.Manager` for every code-reward evaluation and returns its `ListProxy` metadata directly. Keeping that return value alive also keeps the manager server process alive:  ```text metadata_type=ListProxy new_children_w…

### #7253 — [[misc] fix: preserve variable-length generation chats](https://github.com/verl-project/verl/pull/7253)
- **作者**: RerankerGuo  **时间**: 2026-08-04 17:48 CST
- **摘要**: ### What does this PR do?  `main_generation_server` converts parquet chat rows with `np.array(chat_lst)`. With NumPy 2.x, valid chats containing different numbers of turns raise:  ```text ValueError: setting an array element with a sequence ```  Even when every chat has the same turn count, NumPy ca…
