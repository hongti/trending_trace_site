# verl-project/verl — 动态追踪

> 生成时间: 2026-08-14 11:08 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 📌 Issue 动态
1. **SFT 训练严重 Bug**：从 epoch 边界的检查点恢复 SFT 训练时，会静默跳过下一个 epoch 并可能以状态码 0 退出，导致模型训练不足且难以察觉。（#7401）
2. **RFC：无学习信号快速失败检测器**：提议新增一个默认关闭的检测器，当训练全程无学习信号（如持续零优势方差/零梯度范数）时，自动快速终止运行，避免算力浪费。（#7405）
3. **小集群 CPU 资源耗尽问题**：`TransferQueue` 的存储单元默认值为 8，未感知集群规模，在小型单节点运行时可能导致 CPU 耗尽。（#7398）

### 📌 PR 动态
**🚀 新特性**
- **支持 WSD 学习率调度**：在统一的 FSDP 引擎中接入 Warmup-Stable-Decay (WSD) 学习率调度策略。（#7390）
- **支持 veOmni EP 训练**：允许 gpt-oss 系列模型结合 veOmni EP 进行 RL 训练。（#7397）

**🛠️ 重要修复**
- **修复 SFT 恢复跳过 Epoch 问题**：解决 Issue #7401，修复了恢复训练时 `StatefulDataLoader` 状态耗尽导致的静默失败问题。（#7402）
- **修复 SFT FSDP 检查点轮换 Bug**：修复了进程重启后 `max_ckpt_to_keep=1` 失效的问题，强制重启后最多保留一个检查点。（#7396）
- **修复多节点 vLLM 请求中止问题**：处理多节点副本上的请求中止逻辑，使非 rank 0 节点上的 abort 操作为空操作，并正确传播中止信号。（#7393）
- **修复图像数据内存泄漏**：避免 `RLHFDataset` 在解码图像后仍保留压缩的图像字节，大幅降低内存占用。（#7400）
- **修复配置类初始化错误**：移除 `RewardManagerConfig` 中无效的 `super().__post_init__()` 调用。（#7399）

**🔧 CI / 构建 / 文档**
- 新增 Ascend NPU 的 WheelNext 变体 wheel 构建与发布工作流。（#7392）
- 增加 NPU CI 环境变量以减少 `transformers` 庞大日志输出，并更新 NPU 文档。（#7404）
- 调整依赖安装顺序，防止 `requirements-npu.txt` 被 `setup.py` 意外覆盖。（#7403）

### 📌 Release 动态
- 近期无新版本发布。

---

## 🐛 Issues

### #7405 — [[RFC] Optional fail-fast detector for degenerate "no learning signal" runs (persistent zero advantage variance / zero grad_norm)](https://github.com/verl-project/verl/issues/7405)
- **作者**: nataliekung  **时间**: 2026-08-14 10:50 CST
- **摘要**: ### Summary  Propose an **opt-in, off-by-default** detector that fails a run fast when it produces **no learning signal for the whole run** (persistent zero in-group reward variance → zero advantage → zero `grad_norm`). Today such a degenerate run reports **success** and only shows the problem on th…

### #7401 — [[Bug][SFT] Resuming from an epoch-boundary checkpoint silently skips the next epoch](https://github.com/verl-project/verl/issues/7401)
- **作者**: ai-yang  **时间**: 2026-08-14 09:07 CST
- **摘要**: ## Severity  High. A resumed multi-epoch SFT job can exit with status 0 without performing the requested remaining updates. The failure is silent, so an operator or scheduler can treat an under-trained model as a completed run.  ## System Info  - Reproduced on latest verl main commit: `09ac37258ea66…

### #7398 — [[transfer_queue] num_data_storage_units default (8) is not cluster-size aware and can exhaust CPU on small single-node runs](https://github.com/verl-project/verl/issues/7398)
- **作者**: nataliekung  **时间**: 2026-08-14 06:13 CST
- **摘要**: ### Summary  When TransferQueue uses the `SimpleStorage` backend, the number of storage units defaults to `num_data_storage_units: 8` (`verl/trainer/config/transfer_queue/transfer_queue.yaml`). Each `SimpleStorageUnit` and the controller are created with `num_cpus=1`, so **9 CPUs are reserved regard…

## 🔀 Pull Requests

### #7404 — [[ci] chore: Add npu ci env ](https://github.com/verl-project/verl/pull/7404)
- **作者**: LeoYao123  **时间**: 2026-08-14 10:46 CST
- **摘要**: ### What does this PR do? 1. add env to avoid transformers huge log 2. update npu doc  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at l…

### #7403 — [[doc] chore: Change requirements install order](https://github.com/verl-project/verl/pull/7403)
- **作者**: MrJVium  **时间**: 2026-08-14 10:10 CST
- **摘要**: ### What does this PR do?  - Avoid requirements-npu.txt overwrited by setup.py  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least on…

### #7402 — [[ckpt, trainer] fix: resume SFT after epoch checkpoints](https://github.com/verl-project/verl/pull/7402)
- **作者**: ai-yang  **时间**: 2026-08-14 09:08 CST
- **摘要**: ### What does this PR do?  Fixes a silent SFT resume failure at epoch-boundary checkpoints.  SFT previously restored an exhausted `StatefulDataLoader` state at the beginning of the next epoch. The process could then exit successfully without running any remaining updates. This PR records the saved e…

### #7400 — [[data] fix: avoid retaining image bytes after decoding](https://github.com/verl-project/verl/pull/7400)
- **作者**: leooop-al  **时间**: 2026-08-14 08:23 CST
- **摘要**: ```markdown ### What does this PR do?  Addresses #7391.  `RLHFDataset._build_messages()` previously retained both the compressed image bytes and the decoded PIL image in `raw_prompt`. This could substantially increase host-memory and Ray serialization overhead during multimodal RL training.  This PR…

### #7399 — [[reward, cfg] fix: remove invalid super().__post_init__() in RewardManagerConfig](https://github.com/verl-project/verl/pull/7399)
- **作者**: nataliekung  **时间**: 2026-08-14 06:47 CST
- **摘要**: ### What does this PR do?  `RewardManagerConfig` is a direct subclass of `BaseConfig`, but `BaseConfig` (and its full MRO, which only adds `collections.abc.Mapping`) defines no `__post_init__`. Its `__post_init__` calls `super().__post_init__()` (`verl/workers/config/reward.py:52`), so default const…

### #7397 — [[veomni, vllm] feat: Allow gptoss ep with veomni](https://github.com/verl-project/verl/pull/7397)
- **作者**: wyettzeng  **时间**: 2026-08-14 03:53 CST
- **摘要**: ### What does this PR do?  Enable gpt-oss RL training with veomni EP for gpt-oss family model. In conjuncture with this vllm PR: https://github.com/vllm-project/vllm/pull/52209  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: previously found bug: htt…

### #7396 — [[ckpt, fsdp] fix: enforce max-one SFT retention after restart](https://github.com/verl-project/verl/pull/7396)
- **作者**: ai-yang  **时间**: 2026-08-13 23:34 CST
- **摘要**: ### What does this PR do?  Fixes SFT FSDP checkpoint rotation after a process restart when `trainer.max_ckpt_to_keep=1`.  `BaseCheckpointManager` keeps retention history only in `previous_saved_paths`, which starts empty in each process. SFT could load `global_step_1`, save and publish `global_step_…

### #7393 — [[vllm] fix: handle request aborts on multi-node replicas](https://github.com/verl-project/verl/pull/7393)
- **作者**: lxb007981  **时间**: 2026-08-13 21:25 CST
- **摘要**: ### What does this PR do? 1. Multi-node vLLM replicas only create an AsyncLLM instance on node rank 0. Makes abort_all_requests() and abort_request() no-ops on nonzero node ranks. 2. Propagates abort failures instead of silently continuing into weight synchronization. 3. Removes a redundant wait_for…

### #7392 — [[ci] feat: add wheelnext variant wheel build and release workflow for ascend npu](https://github.com/verl-project/verl/pull/7392)
- **作者**: wjunLu  **时间**: 2026-08-13 21:10 CST
- **摘要**: ## Summary  Add a WheelNext (PEP 817 wheel variants) build & release workflow for the Ascend NPU wheels of verl:  - **`.github/workflows/release_ascend_wheel.yml`** — two build jobs, one per CANN chip (`910b` / `a3`, each running inside its matching `quay.io/ascend/cann:9.1.0-<chip>-ubuntu22.04-py3.…

### #7390 — [[fsdp, trainer] feat: support WSD LR scheduling](https://github.com/verl-project/verl/pull/7390)
- **作者**: Sunbeam23333  **时间**: 2026-08-13 18:44 CST
- **摘要**: ### What does this PR do?  Wire Warmup-Stable-Decay (WSD) learning-rate scheduling into the unified FSDP engine.  The new `optim.lr_scheduler_type=wsd` path exposes `lr_wsd_stable_steps_ratio`, reuses `min_lr_ratio` and `num_cycles` for the decay phase, validates WSD-specific boundaries, supports ze…
