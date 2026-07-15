# verl-project/verl — 动态追踪

> 生成时间: 2026-07-15 08:59 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文简洁摘要：

### 🚧 Issue 动态
本期暂无提供 Issue 相关动态。

### 📦 Release 动态
本期暂无新版本发布。

### 🔀 PR 动态 (Pull Requests)
近期共处理 10 个 PR，主要集中在训练机制增强、多模态与显存 Bug 修复、以及 CI/环境维护：

**🚀 重要新特性**
*   **#7037 [trainer, ckpt] 支持流式数据加载器与异步检查点恢复**：这是本次核心变更之一。该 PR 解除了 `train_batch_size` 的限制，允许从数据加载器向生成阶段添加任意数量的提示词；同时优化了 v1 异步训练器的检查点恢复机制。
*   **#7043 [data] TinyLLaVA-Video-R1 数据集自动下载**：当未指定 `--data_dir` 时，数据预处理脚本将自动下载并解压所需数据集，免去手动配置的繁琐。

**🔧 关键 Bug 修复**
*   **#7041 [fully_async] 修复 qwen2.5-0.5b 全异步 OOM 问题**：解决了权重同步期间 `recv_buf` 额外内存分配导致的显存溢出（OOM）Bug。
*   **#7038 [rollout, vllm] 阻止采样视觉占位符**：修复了策略在 vLLM rollout 时错误采样 `<|image_pad|>` / `<|video_pad|>` 等无意义视觉占位符 token 的问题，确保多模态生成逻辑正确。
*   **#7040 [ci, trainer] 修复 Ascend 环境变量断言错误**：修正了 `CUDA_DEVICE_MAX_CONNECTIONS` 环境变量引发的 CI AssertionError，保障昇腾 NPU 上的 LLM RL 训练 E2E 测试顺利运行。
*   **#7034 修正训练循环步骤注释**：修复了 `PPOTrainer.fit()` 注释中步骤编号跳跃的问题（1→2→4 修正为连续编号），提升代码可读性。

**🛠️ CI 与环境维护**
*   **#7042 移除 vllm 补丁**：清理历史遗留的 vllm patch 代码。
*   **#7039 补充 megatron-bridge 依赖**：在 sglang docker 环境中新增 `megatron-bridge` 安装，以适配先前 PR (#6951) 的默认值变更。
*   **#7036 & #7035 CI 流程修复与更新**：修复了 veomni CI 流程，并更新了 Ascend CI 的 Docker 镜像标签。

---

## 🔀 Pull Requests

### #7043 — [[data] feat: auto-download TinyLLaVA-Video-R1 dataset when --data_dir is not given](https://github.com/verl-project/verl/pull/7043)
- **作者**: ArdalanM  **时间**: 2026-07-15 07:55 CST
- **摘要**: ### What does this PR do?  `examples/data_preprocess/tinyllava_video_r1.py` currently requires `--data_dir` to already point at a manually downloaded + extracted copy of `Zhang199/TinyLLaVA-Video-R1-training-data` (jsonl + `NextQA.zip`), otherwise it exits with `--data_dir is required`.  This PR mak…

### #7042 — [[ci] chore: remove vllm patch](https://github.com/verl-project/verl/pull/7042)
- **作者**: wucong25  **时间**: 2026-07-14 22:15 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  remove vllm patch  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, `…

### #7041 — [[fully_async] fix: qwen2.5-0.5b fully async OOM bug fix](https://github.com/verl-project/verl/pull/7041)
- **作者**: zhouhengan1211  **时间**: 2026-07-14 21:16 CST
- **摘要**: ### What does this PR do?  recv_buf is the additional memory that the fully asynchronous algorithm needs to allocate during weight synchronization, used to store model weights. An OOM (Out of Memory) error occurs when allocating recv_buf because the buffer requests too much memory, or the inference …

### #7040 — [[ci, trainer] fix: Fix ci AssertionError for the environment variable CUDA_DEV…](https://github.com/verl-project/verl/pull/7040)
- **作者**: pengnuoheng  **时间**: 2026-07-14 20:52 CST
- **摘要**: …ICE_MAX_CONNECTIONS  ### What does this PR do?  fix: Fix ci AssertionError for the environment variable CUDA_DEVICE_MAX_CONNECTIONS related ci case: E2E Ascend testing for RL training scenarios of LLM models using MindSpeed_LLM engine traceback: AssertionError: Using tensor model parallelism or con…

### #7039 — [[env] chore: add megatron-bridge in sglang docker env](https://github.com/verl-project/verl/pull/7039)
- **作者**: zhouhengan1211  **时间**: 2026-07-14 20:43 CST
- **摘要**: ### What does this PR do?  Seeing https://github.com/verl-project/verl/pull/6951, it changed the default value used by the bridge to megatron-bridge. However, if the megatron-bridge is not installed in the sglang-related images, it will cause the script to fail.  > Add **concise** overview of what t…

### #7038 — [[rollout, vllm] fix: stop the policy from sampling vision placeholder tokens](https://github.com/verl-project/verl/pull/7038)
- **作者**: zhshj0110  **时间**: 2026-07-14 11:27 CST
- **摘要**: ### What does this PR do?  Stops the policy from sampling `<|image_pad|>` / `<|video_pad|>` during vLLM rollout.  A vision placeholder token only means something when a real image or video sits behind it: the k-th run of placeholders pairs with the k-th row of `image_grid_thw` / `video_grid_thw`. No…

### #7037 — [[trainer, ckpt] feat: support streaming dataloader and async trainer checkpoint recovery](https://github.com/verl-project/verl/pull/7037)
- **作者**: Begunner  **时间**: 2026-07-14 11:25 CST
- **摘要**: ### What does this PR do?  This PR  1. supports adding arbitrary number of prompts from dataloader to generation without the `train_batch_size` limitation.  2. improves v1 async trainers by modifying`drop` strategy: dropping stale prompt groups and refilling new prompts. 3. allows restarting pending…

### #7036 — [[ci] chrore: fix veomni ci](https://github.com/verl-project/verl/pull/7036)
- **作者**: wuxibin89  **时间**: 2026-07-14 11:08 CST
- **摘要**: ### What does this PR do?  As title.

### #7035 — [[ci] test: update ascend ci docker tag](https://github.com/verl-project/verl/pull/7035)
- **作者**: yyyy2000  **时间**: 2026-07-14 10:53 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  update ascend ci docker tag  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `…

### #7034 — [fix: renumber step comments in fit() training loop](https://github.com/verl-project/verl/pull/7034)
- **作者**: cyyueyang  **时间**: 2026-07-14 09:39 CST
- **摘要**: ## Summary   Fix the step numbering in `PPOTrainer.fit()` training loop comments. The comments currently go from 1 → 2 → 4 → 5 → 6 → 7, skipping step 3, which is confusing for readers tracing the training flow.  ## Change  Renumbered the step comments in `verl/trainer/ppo/v1/trainer_base.py` inside …
