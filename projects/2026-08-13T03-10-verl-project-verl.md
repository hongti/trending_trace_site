# verl-project/verl — 动态追踪

> 生成时间: 2026-08-13 11:10 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 🐛 Issue
*   **MultiTurnSFTDataset Tokenize 机制疑问 (#7378)**：作者对 `MultiTurnSFTDataset` 中逐条对 messages 进行 tokenize 的做法提出疑问，探讨了其背后的设计与影响。

### 🔧 Pull Request
PR 活动较为活跃，主要涵盖新硬件支持、性能优化、重要模型/框架的修复及依赖清理：

**✨ 新特性与性能优化**
*   **新增 Intel GPU (XPU) 支持 (#7371)**：引入 Intel GPU 作为新硬件后端，支持使用 FSDP/FSDP2 actor 与 vLLM 共置进行端到端的 GRPO、PPO 及 SFT 训练。
*   **减少 Trainer GPU 闲置 (#7373)**：在 `separate_async` 模式下，允许将空闲的 Trainer GPU 临时切换至 Rollout 模式用于生成任务，显著提升资源利用率。
*   **FSDP 计算优化 (#7370)**：在 Torch fused LM head 中，利用 response mask 跳过对不参与 policy loss 计算的行进行词表投影，仅投影活跃行以节省计算量。
*   **新增 OPD full-vocab 支持 (#7375)**：添加了 OPD 全词表功能（WIP 阶段）。

**🛠️ 关键修复**
*   **修复 Qwen3.5-VL LoRA 训练崩溃 (#7376)**：修复了 Megatron 到 vLLM 权重同步时 `resolve_weight_name` 对 `.base_layer` 处理不当的问题。
*   **修复 DeepSeek-V4 全量权重同步失败 (#7369)**：使 DeepSeek-V4 的全量（非 delta）权重同步在 SGLang + Megatron 组合下能够端到端正常运行。
*   **修复 NPU 上 VLM 注意力掩码错误 (#7372)**：针对 NPU 环境，修复了启用 `use_remove_padding` 后 TND 格式下 VLM 注意力掩码形状构建错误的问题。
*   **稳定 ROCm PPO Trainer CI (#7377)**：通过更新至 ROCm 7.14 镜像、依赖内置 Megatron 并禁用 vLLM custom all-reduce，修复了 CI 不稳定的问题。

**🧹 清理与依赖**
*   **移除 mindspeedllm 后端 (#7374)**：清理代码，正式移除对 mindspeedllm 后端引擎的支持。
*   **依赖更新 (#7379)**：将 `packaging` 库的版本要求从 `>=20.0` 升级至 `>=26.3`。

### 🚀 Release
*   近期无新版本发布。

---

## 🐛 Issues

### #7378 — [Why are messages tokenized one by one in MultiTurnSFTDataset?](https://github.com/verl-project/verl/issues/7378)
- **作者**: wangtiance  **时间**: 2026-08-12 21:18 CST
- **标签**: bug
- **摘要**: ### System Info  original code: https://github.com/verl-project/verl/blob/535c47799b537a0e3602b9839344ee53d2a47128/verl/utils/dataset/multiturn_sft_dataset.py#L301  The issue is that when doing SFT for Qwen 3.5 with a multi-turn conversation starting with a system prompt, I get the error `jinja2.exc…

## 🔀 Pull Requests

### #7379 — [build(deps): update packaging requirement from >=20.0 to >=26.3](https://github.com/verl-project/verl/pull/7379)
- **作者**: dependabot[bot]  **时间**: 2026-08-13 01:33 CST
- **标签**: dependencies, python
- **摘要**: Updates the requirements on [packaging](https://github.com/pypa/packaging) to permit the latest version. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/pypa/packaging/releases">packaging's releases</a>.</em></p> <blockquote> <h2>26.3</h2> <!-- raw HTML omi…

### #7377 — [[ci, hardware] fix: stabilize ROCm PPO trainer CI](https://github.com/verl-project/verl/pull/7377)
- **作者**: PeterYang12  **时间**: 2026-08-12 21:17 CST
- **摘要**: ### What does this PR do?  Updates the ROCm PPO trainer workflow to use the ROCm 7.14 image, rely on the Megatron dependencies bundled in that image, and disable vLLM custom all-reduce for the affected ROCm jobs.  This does not duplicate an existing PR: searches for [ROCm vLLM custom all-reduce](htt…

### #7376 — [[megatron] fix: per-name, mapper-aware .base_layer strip in resolve_weight_name](https://github.com/verl-project/verl/pull/7376)
- **作者**: HollowMan6  **时间**: 2026-08-12 18:29 CST
- **摘要**: ### What does this PR do?  Fixes the `.base_layer` weight-name reconciliation in `resolve_weight_name` (`verl/utils/vllm/utils.py`) that crashed Qwen3.5-VL LoRA training at Megatron→vLLM weight sync  On vLLM 0.26.0, `_HAS_LORA_LOAD_WEIGHTS = False` — LoRA linear layers (`ReplicatedLinearWithLoRA`, e…

### #7375 — [[wip][feat]: add opd full-vocab](https://github.com/verl-project/verl/pull/7375)
- **作者**: wucong25  **时间**: 2026-08-12 18:14 CST
- **摘要**: ### What does this PR do?  add opd full-vocab  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, …

### #7374 — [[ci, trainer] feat: remove mindspeedllm backend engine support.](https://github.com/verl-project/verl/pull/7374)
- **作者**: pengnuoheng  **时间**: 2026-08-12 17:22 CST
- **摘要**: ### What does this PR do?  remove mindspeedllm backend engine support.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp…

### #7373 — [[trainer, vllm] feat: lend idle trainer GPUs to generation in separate_async](https://github.com/verl-project/verl/pull/7373)
- **作者**: Begunner  **时间**: 2026-08-12 16:46 CST
- **摘要**: ### What does this PR do?  Allow the hybrid GPUs to switch to the rollout mode in separate_async trainer to reduce trainer idle.  Once the prompts for a step have been submitted, the trainer keeps its replicas in rollout mode and helps generate until the replay buffer holds enough sampleable groups,…

### #7372 — [[megatron] fix: Modify the VLM attention mask shape in TND format for NPU](https://github.com/verl-project/verl/pull/7372)
- **作者**: zhouhengan1211  **时间**: 2026-08-12 16:31 CST
- **摘要**: ### What does this PR do?  - After the use_remove_padding option is enabled for a model with vision_model, the construction mode of the VLM attention mask is modified. - This issue occurs because when the qwen3.5 vl model uses the mindspeed bridge backend, the GND module needs to identify the paddin…

### #7371 — [[hardware, fsdp, vllm] feat: add Intel GPU (XPU) support](https://github.com/verl-project/verl/pull/7371)
- **作者**: kahlun  **时间**: 2026-08-12 16:28 CST
- **标签**: Hardware
- **摘要**: ### What does this PR do?  Adds **Intel GPU (PyTorch XPU)** as a hardware backend for verl, enabling end-to-end **GRPO, PPO, and SFT** training with **FSDP/FSDP2** actors and **vLLM** colocated rollout. Intel GPU is integrated through the existing platform-abstraction layer (the same path used by CU…

### #7370 — [[fsdp] feat: project only active rows in the Torch fused LM head](https://github.com/verl-project/verl/pull/7370)
- **作者**: Zhaoyi-Tian  **时间**: 2026-08-12 14:30 CST
- **摘要**: `[fsdp] feat: project only active rows in the Torch fused LM head`  Use the existing response mask to skip vocabulary projection for rows that do not contribute to the policy loss, while preserving the packed output layout expected by downstream PPO code.  ### What does this PR do?  With remove-padd…

### #7369 — [[rollout, config] fix: make DeepSeek-V4 full weight sync work with SGLang + Megatron](https://github.com/verl-project/verl/pull/7369)
- **作者**: ChangyiYang  **时间**: 2026-08-12 11:26 CST
- **摘要**: ### What does this PR do?  Makes the full (non-delta) weight sync work end to end for **DeepSeek-V4 with SGLang rollout and a Megatron trainer**. Four commits, each fixing one thing that stopped the path from completing:  1. **`[rollout] DSv4-native fp8(ue8m0) converter for nccl full-sync`** — DSv4'…
