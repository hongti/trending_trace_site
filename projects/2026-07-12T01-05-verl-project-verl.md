# verl-project/verl — 动态追踪

> 生成时间: 2026-07-12 09:05 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 🛑 Issue
* **FSDP2 下 Qwen3-MoE 训练失败 (#7016)**：在使用 FSDP2 引擎训练 `Qwen3-30B-A3B` (Qwen3MoeForCausalLM) 时，在 `update_actor` 的反向传播阶段持续报错。开启梯度检查点会触发 `CheckpointError`，关闭则导致原生 worker 崩溃。值得注意的是，相同配置在 FSDP1 下可正常训练。这是一个影响 MoE 模型在 FSDP2 下兼容性的关键阻塞问题。

### 🔧 Pull Request (PR)
* **【功能优化】v1 trainer 启用 SkipManager 及参数同步步长适配 (#7020)**：在 v1 trainer 的同步训练 CI 中启用了 `SkipManager`，并相应适配了 `param_sync_step`。此 PR 解决了先前 PR (#6897) 中遗留的 TODO，进一步完善了同步训练流程。
* **【Bug 修复】Rollout Agent Loop 支持 dataset output 字段 (#7019)**：修复了 Agent 循环采样中非 tensor 数据的 `output` 字段传递逻辑冲突问题（Fixes #5388），确保 agent loop 能够正确接受并处理 dataset 的输出字段。
* **【文档修复】更新 kimi-checkpoint-engine 说明 (#7017, #7018)**：两个 PR 均针对 README 进行了修复，明确说明 `kimi-checkpoint-engine` 目前不支持 ascend A5 环境，避免了用户的误配置。

### 🚀 Release
* 近期无新版本发布。

---

## 🐛 Issues

### #7016 — [[fsdp2] Qwen3-MoE actor backward fails under FSDP2: CheckpointError with gradient checkpointing, native worker crash without it (FSDP1 works)](https://github.com/verl-project/verl/issues/7016)
- **作者**: ChangyiYang  **时间**: 2026-07-11 14:38 CST
- **摘要**: ### Summary  Training **Qwen3-30B-A3B** (`Qwen3MoeForCausalLM`) with the FSDP2 engine consistently fails in the **first `update_actor` backward**. The same config trains fine with FSDP1, and dense Qwen2.5 models (7B/32B/72B) train fine with FSDP2 — the failure is specific to FSDP2 × Qwen3-MoE.  Two …

## 🔀 Pull Requests

### #7020 — [[v1 trainer] feat: Enable SkipManager and adapt param_sync_step](https://github.com/verl-project/verl/pull/7020)
- **作者**: chethanuk  **时间**: 2026-07-12 00:31 CST
- **摘要**: This PR enables SkipManager in the sync trainer CI and adapts the param_sync_step accordingly. This resolves the inline TODO from a prior PR (#6897).  ## What does this PR do?  1. Modified `verl/trainer/ppo/v1/agent_loop_tq.py` to add `@SkipManager.annotate(role="rollout")` decorator to `generate_se…

### #7019 — [[rollout] fix: accept dataset output fields in agent loop](https://github.com/verl-project/verl/pull/7019)
- **作者**: Epochex  **时间**: 2026-07-11 23:08 CST
- **摘要**: ### What does this PR do?  Fixes #5388.  Agent-loop samples can contain an `output` field in their non-tensor data. `_run_agent_loop()` forwards those fields through `**kwargs` while also passing the runtime `AgentLoopOutput` positionally, so Python raises `TypeError: got multiple values for argumen…

### #7018 — [[doc] fix: update the readme about kimi-checkpoint-engine](https://github.com/verl-project/verl/pull/7018)
- **作者**: beirong8kmiles  **时间**: 2026-07-11 21:58 CST
- **摘要**: ### What does this PR do?  kimi-checkpoint-engine hasn't been supported in ascend A5，this PR just update the readme of it.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (This will…

### #7017 — [[doc] fix: update the readme about kimi-checkpoint-engine](https://github.com/verl-project/verl/pull/7017)
- **作者**: beirong8kmiles  **时间**: 2026-07-11 21:57 CST
- **摘要**: ### What does this PR do?  kimi-checkpoint-engine hasn't been supported in ascend A5，this PR just update the readme of it.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will…
