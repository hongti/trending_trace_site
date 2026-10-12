# verl-project/verl — 动态追踪

> 生成时间: 2026-10-12 14:22 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📌 Pull Requests (代码合并请求)
本期 PR 主要围绕 PPO 训练机制优化、Megatron 支持以及数据处理与权重精度修复：

**1. 重要新特性与优化**
*   **PPO 负载均衡优化 (#8201)**：改进了 PPO 训练逻辑，现在会对 **每一个 PPO mini-batch** 在 DP（数据并行）ranks 之间进行负载均衡（此前仅对整个 train batch 均衡）。同时，将 `actor.shuffle` / `critic.shuffle` 与 DP 均衡逻辑解耦，使其互不影响。
*   **Megatron Actor 分数居中 (#8199)**：在 Megatron actor 上引入了分数居中（score centering）机制。通过对词表并行分片（vocab-parallel shards）的采样头对数概率进行归一化（行最大值与归一化因子跨 TP 进行 all-reduced），提升训练稳定性。

**2. 关键修复**
*   **多轮 SFT 推理内容丢失修复 (#8197)**：修复了 `MultiTurnSFTDataset` 在分词时单独渲染每条消息导致的问题。此修复确保了类似 Qwen3 的聊天模板能正确保留 `<reasoning>` 块的内容，避免多轮对话中推理上下文丢失。
*   **FSDP 合并 LoRA 导出精度修复 (#8198)**：修复了 FSDP2 合并 LoRA 导出至 HF 格式时的精度损失问题。此前复用 rollout 生成器会将 FP32 主权重提前转为 BF16，修复后保留了合并 LoRA 的导出精度与分阶段权重。

**3. CI 与基础设施**
*   **Ascend 镜像仓库描述 (#8200)**：为 `quay.io/ascend/verl` 添加了版本控制的仓库描述，完善 CI 配置。

### 📌 Issues (问题反馈) 与 Releases (版本发布)
*本次提供的近期动态中未包含相关 Issue 讨论与 Release 发布信息。*

---

## 🔀 Pull Requests

### #8201 — [[trainer] feat: balance every PPO mini-batch across DP ranks, shuffle on the trainer](https://github.com/verl-project/verl/pull/8201)
- **作者**: FZYsheep  **时间**: 2026-10-12 14:15 CST
- **摘要**: ### What does this PR do?  Balance **every PPO mini-batch** across DP ranks instead of only the whole train batch, and make `actor.shuffle` / `critic.shuffle` orthogonal to DP balancing: if shuffle is enabled, the trainer first shuffles samples into mini-batches, then balances each mini-batch.  Work…

### #8200 — [[ci] feat: Add Ascend Quay repository description](https://github.com/verl-project/verl/pull/8200)
- **作者**: lxb007981  **时间**: 2026-10-12 09:32 CST
- **摘要**: ### What does this PR do?  Add a version-controlled description for quay.io/ascend/verl.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modul…

### #8199 — [[algo, megatron, cfg, doc, tests] feat: score centering on the Megatron actor](https://github.com/verl-project/verl/pull/8199)
- **作者**: cr-gao  **时间**: 2026-10-12 06:22 CST
- **摘要**: ### What does this PR do?  Score centering (#8010, #8196) on the Megatron actor. The sampler head log-probs are normalized over the vocab-parallel shards (row max and normalizer all-reduced over the TP group, masked local gather summed, backward local), the heads are split by context parallel like t…

### #8198 — [[fsdp, ckpt] fix: preserve merged LoRA export precision and staged weights](https://github.com/verl-project/verl/pull/8198)
- **作者**: cyanseek  **时间**: 2026-10-12 00:29 CST
- **摘要**: ### What does this PR do?  Follow up on #8180: FSDP2 merged-LoRA HF export reuses the rollout generator, which casts FP32 master weights to BF16 before the HF writer applies its export dtype policy. FP16 and FP32 exports therefore lose precision and contain BF16 tensors. Preserve stored precision fo…

### #8197 — [[data] fix: preserve reasoning content in multi-turn SFT tokenization](https://github.com/verl-project/verl/pull/8197)
- **作者**: akushonkamen  **时间**: 2026-10-11 16:32 CST
- **摘要**: ### What does this PR do?  Fixes #7807. `MultiTurnSFTDataset` rendered each message in isolation with `apply_chat_template([message])`, but chat templates such as Qwen3 build the `<think>` block from `reasoning_content` only when the assistant message is rendered together with its preceding conversa…
