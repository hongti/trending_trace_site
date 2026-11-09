# verl-project/verl — 动态追踪

> 生成时间: 2026-11-09 15:02 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📌 Issue 摘要
- **#8604 Megatron 检查点恢复导致权重静默重置**
  作者报告了一个严重问题：在使用 Megatron 后端恢复检查点时，如果未加载优化器状态（`load_contents` 不包含 "optimizer"），模型权重会在训练的第一步被静默重置为初始值。

### 🔧 Pull Request 摘要

**1. 重要 Bug 修复**
- **#8605 [megatron, ckpt] 修复仅恢复模型时的优化器参数重载问题**
  针对 Issue #8604 的修复。当仅恢复模型权重而不含优化器状态时，防止混合精度优化器保留基于初始模型权重的 FP32 主参数，避免权重被意外重置。

**2. 新特性**
- **#8602 [rollout, vllm, trainer] 同址训练时卸载部分 KV 到 CPU**
  引入了一项实验性功能：在 V1 `colocate_async` 训练中，当训练 Batch 准备就绪时，将未完成的 vLLM 请求的完整原生 KV/Runtime 状态备份保留至 CPU，以优化显存占用。
- **#8600 [megatron] 增加 Router Replay 诊断与标志保护**
  使 Router Replay 活动变得可观测，并提供环境变量 `VERL_ROUTER_REPLAY_DEBUG=1` 打印对齐日志；同时将一处静默回退改为硬报错，提升稳定性。

**3. 性能优化与重构**
- **`training_utils` 性能系列优化：**
  - **#8601**：预分配 Response Padding 的输出内存，优化 packed 模型输出到 padded 张量的转换。
  - **#8603**：复用 masked whitening 计算过程中的中间变量，大幅减少重复的张量遍历、类型转换和求和操作。
  - **#8607**：Batch 分发时使用原生的 2D Jagged chunk views，提升数据处理效率。
- **`megatron` Router Replay 热路径优化：**
  - **#8599**：缓存每个模型 chunk 的 router replay 列表，避免在每次查询/切换时重复遍历 Megatron Decoder 树。
  - **#8597**：跳过未使用的 replay position ids 并预归一化 replay mask，消除热路径上的冗余计算。

**4. 文档与测试**
- **#8606 [doc]**：统一了 Ascend 文档中关于 Atlas A2 和 A3 系列硬件产品的命名。
- **#8598 [test]**：新增 CPU 测试，覆盖了在缺失路由行时 `align_r3_router_replay_data` 拒绝对齐的边界情况。

### 🚀 Release 摘要
- 近期无新的 Release 版本发布。

---

## 🐛 Issues

### #8604 — [[megatron, ckpt] Resuming without the optimizer (load_contents without "optimizer") silently resets weights to the initial model on the first step](https://github.com/verl-project/verl/issues/8604)
- **作者**: zucanlv  **时间**: 2026-11-09 06:36 CST
- **摘要**: ### System Info  - verl: `main` @ `31cfccb4` (read from source); first observed on a downstream fork with the same `megatron_checkpoint_manager.py` load path - Megatron backend, distributed optimizer, BF16 params with FP32 main params - Megatron-Core: `megatron/core/optimizer/optimizer.py` on curren…

## 🔀 Pull Requests

### #8607 — [[training_utils, perf] refactor: use native 2D jagged chunk views](https://github.com/verl-project/verl/pull/8607)
- **作者**: Hang-get  **时间**: 2026-11-09 13:58 CST
- **摘要**: ### What does this PR do?  `BatchData.chunk` calls `chunk_tensordict` whenever a TensorDict batch is dispatched to workers. Text no-padding batches carry 2D jagged `input_ids`, `position_ids`, and `loss_mask`. The current helper unbinds every row and concatenates those rows again for each chunk, all…

### #8606 — [[doc] fix: unify Atlas A2 and A3 series product names](https://github.com/verl-project/verl/pull/8606)
- **作者**: likaizheng666-sketch  **时间**: 2026-11-09 11:47 CST
- **摘要**: ### What does this PR do?  Unify Atlas hardware names across the Chinese and English Ascend documentation, the image description, and one example script comment:  - Atlas 800T A3 and Atlas 900 A3 SuperPoD become Atlas A3 系列产品 / Atlas A3 series products. - Atlas 200T A2 Box16 and Atlas 900 A2 PoD (PO…

### #8605 — [[megatron, ckpt] fix: reload optimizer params after model-only resume](https://github.com/verl-project/verl/pull/8605)
- **作者**: Hang-get  **时间**: 2026-11-09 10:35 CST
- **摘要**: ### What does this PR do?  Fixes #8604.  When a Megatron checkpoint restores model weights without optimizer state, the mixed-precision optimizer can retain FP32 main parameters initialized from the original model. The first optimizer step can copy those stale parameters back and overwrite the resto…

### #8603 — [[training_utils, perf] refactor: reuse masked whitening reductions](https://github.com/verl-project/verl/pull/8603)
- **作者**: Hang-get  **时间**: 2026-11-09 03:03 CST
- **摘要**: ### What does this PR do?  Reduce repeated full-tensor work in masked advantage whitening.  `masked_whiten` previously computed the masked mean twice, converted the mask three times, summed the mask four times, and centered the values twice through its call to `masked_var`. It also performed two sca…

### #8602 — [[rollout, vllm, trainer] feat: offload partial KV during colocated training](https://github.com/verl-project/verl/pull/8602)
- **作者**: HaitaoWuTJU  **时间**: 2026-11-09 01:47 CST
- **摘要**: ### What does this PR do?  Adds experimental, opt-in CPU preservation of unfinished vLLM requests for V1 `colocate_async`. When a training batch is ready, pause with `keep`, back up full native KV/runtime pools, train on the shared GPUs, refit weights, restore KV to its original addresses, and conti…

### #8601 — [[training_utils, perf] refactor: preallocate response padding output](https://github.com/verl-project/verl/pull/8601)
- **作者**: Hang-get  **时间**: 2026-11-08 19:47 CST
- **摘要**: ### What does this PR do?  `no_padding_2_padding` converts packed model outputs back to padded response tensors for actor log-probabilities and entropy, critic values, score correction, and distillation. It runs repeatedly during inference and PPO microbatch processing.  The current implementation c…

### #8600 — [[megatron] feat: add router replay coverage diagnostics and flag guard](https://github.com/verl-project/verl/pull/8600)
- **作者**: ys2025-AI  **时间**: 2026-11-08 19:17 CST
- **摘要**: ### What does this PR do?  Makes router replay activity observable and turns one silent fallback into a hard failure:  - `VERL_ROUTER_REPLAY_DEBUG=1` emits a warning-level line per rank with the replay layer count, `vp_rank`, and the R3 mask's replay/native row split. Logging on every rank (rank inc…

### #8599 — [[megatron] perf: cache router replay router lists per model chunk](https://github.com/verl-project/verl/pull/8599)
- **作者**: ys2025-AI  **时间**: 2026-11-08 15:10 CST
- **摘要**: <!-- Branch: ys2025-AI/verl:router-replay-router-cache Commit: 2ccd80e2749c94d5f4a6e4192b76dc8e838bf9bb Base:   verl-project/verl:main @ b17218ba Title:  [megatron] perf: cache router replay router lists per model chunk -->  ### What does this PR do?  Avoids repeatedly walking the Megatron decoder t…

### #8598 — [[test] test: cover R3 alignment rejection for missing route rows](https://github.com/verl-project/verl/pull/8598)
- **作者**: ys2025-AI  **时间**: 2026-11-08 14:51 CST
- **摘要**: ### What does this PR do?  Adds a CPU test for the boundary of `align_r3_router_replay_data`: rollout routes may be missing at most the single final route row per sequence (the last generated token has no routing record). A sequence missing more than one row must fail loudly instead of being silentl…

### #8597 — [[megatron] perf: skip unused replay position ids and pre-normalize replay mask](https://github.com/verl-project/verl/pull/8597)
- **作者**: ys2025-AI  **时间**: 2026-11-08 14:49 CST
- **摘要**: ### What does this PR do?  Removes two pieces of redundant work on the router-replay hot path:  - Router replay only needs the packed rows from `preprocess_thd_engine`; it discards the returned position ids. Add an opt-in `need_position_ids=False` so replay packing does not allocate and fill a full …
