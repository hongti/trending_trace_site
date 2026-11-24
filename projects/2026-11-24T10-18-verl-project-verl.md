# verl-project/verl — 动态追踪

> 生成时间: 2026-11-24 18:18 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📦 Release 动态
*近期无版本发布记录。*

### 🛠️ Pull Request 动态
近期共有 10 个 PR 活动，主要涵盖模型支持、训练机制增强以及多处重要 Bug 修复，归纳如下：

#### 1. 新特性与功能增强
*   **蒸馏与 KL 散度计算**：实现了在 TransferQueue (TQ) 中传递教师模型隐状态的功能，以支持 on-policy 蒸馏 (OPD) 的全词表前向 KL 计算，并补充了相关 Codex 测试逻辑（#8827, #8820）。
*   **KV-Cache 卸载优化**：简化了 vLLM 的 `MooncakeStoreConnector` 集成，为多轮 Agentic RL 提供了开箱即用的 YAML 预设（`+kv_offload=mooncake`）和调优指南，降低运行内存占用（#8828）。
*   **新模型与权重支持**：支持从 safetensors 流式加载冻结的 Engram/PLE n-gram 查找表，并新增对 `Qwen3.8-Flash-Next` 和 `DeepSeek-V4.1` 的跨引擎（VeOmni、FSDP、vLLM）支持（#8825）。
*   **Checkpoint 异步写入**：为 Megatron 引擎引入了可选的线程化 checkpoint 写入器，通过暂存 CPU 快照实现异步写入，避免训练过程阻塞（#8823）。

#### 2. 重要 Bug 修复
*   **VeOmni 混合架构修复**：修复了 VeOmni R3 中混合 dense/MoE 解码器的路由专家重放逻辑，确保按绝对解码层验证并选择正确的槽位（#8829）；同时启用了 score centering 配置（#8824）。
*   **Megatron 加载逻辑解耦**：修复了 `dist_checkpointing_path` 未设置时，Megatron 错误选择原生加载器的问题，将 HF 初始化与原生 checkpoint 保存解耦（#8821）。
*   **FSDP Logits 处理修复**：修复了非融合 dense FSDP 输出路径（`use_remove_padding=False`）下，错误打包 token 对数概率 logits 的问题（#8826）。
*   **DeepSeek-V4 提示词修正**：修复了 DeepSeek-V4 的提示词构建逻辑，确保推理强度能正确选择目标 prompt，并修正了连续 Token 追加逻辑（#8822）。

### ❗ Issue 动态
*近期无公开的 Issue 动态。*

---

## 🔀 Pull Requests

### #8829 — [[veomni] fix: replay routed experts by absolute decoder layer in R3](https://github.com/verl-project/verl/pull/8829)
- **作者**: yeyeyeping  **时间**: 2026-11-24 17:41 CST
- **摘要**: VeOmni R3 receives one rollout slot per absolute text decoder layer. For mixed dense/MoE decoders, validate the full layer count and select the slots of routed layers before replay. For example, routers in layers 1 and 3 consume slots 1 and 3. R2 retains its compact layout.

### #8828 — [[vllm, rollout, cfg, doc] feat: streamline MooncakeStoreConnector integration, YAML preset, and tuning guide](https://github.com/verl-project/verl/pull/8828)
- **作者**: FZYsheep  **时间**: 2026-11-24 17:27 CST
- **摘要**: ### What does this PR do?  Streamlines vLLM `MooncakeStoreConnector` KV-cache offload support in `verl` for multi-turn Agentic RL with a minimal runtime footprint, a ready-to-use `+kv_offload=mooncake` Hydra YAML preset, and an updated `docs/perf/rollout_kv_offload.md` tuning guide.  ### Checklist B…

### #8827 — [[trainer, rollout, vllm] feat: transfer teacher hidden states for KL ](https://github.com/verl-project/verl/pull/8827)
- **作者**: qinyi810  **时间**: 2026-11-24 17:22 CST
- **摘要**: ### What does this PR do?  Implement full-vocabulary forward KL for on-policy distillation (OPD) with teacher hidden states in TransferQueue (TQ). The teacher prefills the whole conversation and sends one final-layer row for each token that contributes to the loss. The actor applies the matching tea…

### #8826 — [[fsdp] fix: avoid packing dense logits for token log-probabilities](https://github.com/verl-project/verl/pull/8826)
- **作者**: hhpjjpy  **时间**: 2026-11-24 16:19 CST
- **摘要**: ### What does this PR do?  The non-fused dense FSDP output path (`use_remove_padding=False`) narrows logits to valid sequence lengths and concatenates them into a second `[total_valid_tokens, vocab]` tensor before computing token log-probabilities. Even a single unpadded sequence incurs this copy. I…

### #8825 — [[veomni, rollout] feat: stream frozen Engram/PLE weights from safetensors and support Qwen3.8](https://github.com/verl-project/verl/pull/8825)
- **作者**: wuxibin89  **时间**: 2026-11-24 16:14 CST
- **摘要**: ### What does this PR do?  Support frozen Engram / PLE n-gram lookup tables (`Qwen3.8-Flash-Next` / `qwen4_exp` and `DeepSeek-V4.1`) across the VeOmni and FSDP training engines and vLLM rollout weight sync without gathering or transferring multi-tens-of-GiB frozen lookup tables over IPC, enable per-…

### #8824 — [[veomni] fix: enable score centering config for veomni](https://github.com/verl-project/verl/pull/8824)
- **作者**: wuxibin89  **时间**: 2026-11-24 16:00 CST
- **摘要**: ### What does this PR do?  Enable score centering config for veomni

### #8823 — [[megatron, ckpt] feat: add trainer-owned threaded checkpoint writes](https://github.com/verl-project/verl/pull/8823)
- **作者**: tongyx361  **时间**: 2026-11-24 14:52 CST
- **摘要**: ### What does this PR do?  Add an opt-in thread writer for callers that coordinate their own native Megatron checkpoints. The manager stages independent CPU snapshots and writes the native checkpoint in the background. The caller finalizes the save across ranks and can publish the latest checkpoint …

### #8822 — [[model, rollout] fix: correct DeepSeek-V4 reasoning effort and prompt encoding](https://github.com/verl-project/verl/pull/8822)
- **作者**: yyDing1  **时间**: 2026-11-24 14:44 CST
- **摘要**: ### What does this PR do?  Correct DeepSeek-V4 prompt construction in `verl/utils/tokenizer/deepseek.py` so the requested reasoning effort selects the intended prompt, and Continuous Token appends preserve conversation-level instructions and turn boundaries.  Previously, `reasoning_effort="high"` em…

### #8821 — [[megatron, ckpt] fix: decouple HF initialization from native checkpoint saves](https://github.com/verl-project/verl/pull/8821)
- **作者**: tongyx361  **时间**: 2026-11-24 13:57 CST
- **摘要**: ### What does this PR do?  With `use_dist_checkpointing: true`, Megatron engine initialization currently selects the native loader even when `dist_checkpointing_path` is unset. This patch uses the existing Hugging Face loader when the native path is unset, while keeping native checkpoint saves enabl…

### #8820 — [Codex/answer teacher forward kl](https://github.com/verl-project/verl/pull/8820)
- **作者**: EricWuji  **时间**: 2026-11-24 13:33 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…
