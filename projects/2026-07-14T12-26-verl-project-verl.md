# verl-project/verl — 动态追踪

> 生成时间: 2026-07-14 20:26 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 🚀 Release（版本发布）
近期无新的 Release 版本发布。

### 🐛 Issue（问题反馈）
近期无新的 Issue 动态。

### ✨ Pull Request（代码合并）
近期共有 7 个 PR，主要涵盖新特性增强、关键 Bug 修复及 CI 维护。重点变更如下：

**🔥 重要新特性与增强**
*   **#7037 [trainer, ckpt] 支持流式 Dataloader 与异步 Checkpoint 恢复**：这是一项重要增强，打破了 `train_batch_size` 的限制，允许在生成阶段添加任意数量的 prompt；同时改进了 v1 异步 trainer 的 checkpoint 恢复机制。
*   **#7032 [tool, rollout] Skip Manager 支持参数同步步数**：为 Skip Manager 增加了参数同步步骤的功能支持。
*   **#7033 [misc] 适配 FlashAttention 3.0 及最新 Transformers**：处理了对 `transformers >= 4.56.0` 版本的兼容性，当未安装 `flash_attn` 时回退到 `transformers` 自带的工具，从而支持 FlashAttention-3 运行。

**🛠️ 关键修复**
*   **#7038 [rollout, vllm] 阻止策略采样视觉占位符 Token**：修复了多模态 rollout 中的一个问题，防止策略在 vLLM 推理阶段错误地采样 `<|image_pad|>` 或 `<|video_pad|>` 等视觉占位符（这些 token 仅在存在真实图像/视频时才有意义）。
*   **#7031 [megatron] 修复 Qwen3VL Megatron 后端错误**：修复了 Qwen3VL 模型在 Megatron 后端运行时的报错。
*   **#7034 修正训练循环注释步骤编号**：修复了 `PPOTrainer.fit()` 训练循环注释中步骤编号从 1 跳到 4（缺失 step 3）的问题，提升了代码可读性。

**⚙️ CI 与杂项**
*   **#7036** 修复 veomni CI 流程。
*   **#7035** 更新 Ascend CI 的 docker tag 版本。

---

## 🔀 Pull Requests

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

### #7033 — [[misc] handle flash attention 3.0 with recent transformers version (>= 4.56.0)](https://github.com/verl-project/verl/pull/7033)
- **作者**: ArdalanM  **时间**: 2026-07-14 06:19 CST
- **摘要**: ## Summary `verl/utils/attention_utils.py` falls back to `transformers.modeling_flash_attention_utils` when `flash_attn` isn't installed, which is how we support running with FlashAttention-3 (`flash_attn_interface`) instead of FA2. That fallback needs `transformers>=4.56.0` (`_pad_input`/`_unpad_in…

### #7032 — [[tool, rollout] feat: support parameter sync steps in Skip Manager](https://github.com/verl-project/verl/pull/7032)
- **作者**: mikequan0425  **时间**: 2026-07-13 22:04 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  Related issue：https://github.com/verl-project/verl/issues/7007 Save the skipped rollout data in the format of global step + param s…

### #7031 — [[megatron] fix: Fix the error in the Qwen3VL Megatron backend](https://github.com/verl-project/verl/pull/7031)
- **作者**: Seren-hao  **时间**: 2026-07-13 20:27 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Fix the error in the Qwen3VL Megatron backend  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fs…
