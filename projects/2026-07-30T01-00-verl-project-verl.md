# verl-project/verl — 动态追踪

> 生成时间: 2026-07-30 09:00 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 📌 Issue 动态
*   **#7191 模型合并器成功路径缺少最终的 Hugging Face 输出校验**
    作者指出 `verl.model_merger` 在完成模型序列化后，缺乏文件系统级别的检查，无法确认输出是否为符合标准 Transformers 格式的有效模型。

---

### 🔀 Pull Request 动态

**🚀 重要新特性**
*   **#7186 支持 MXFP8 推理量化**：为 SGLang 后端添加了 MXFP8 权重量化支持，启用后可动态量化更新后的权重。
*   **#7187 FSDP 支持 LoRA 结合全秩 VLM 模块**：引入 PEFT 的 `modules_to_save` 语义，允许视觉语言模型（VLM）在语言模型使用 LoRA 微调的同时，对非 LoRA 子模块（如多模态投影层）进行全参数训练。
*   **#7188 支持 v1 separate_async 的解耦 PPO**：为 v1 separate_async 模式添加了解耦 PPO（decouple ppo）支持。
*   **#7192 原生 HybridModel MTP 支持**：为 Nemotron 3 Super 添加了原生 MCore MTP（Multi-Token Prediction）支持，并增加了一个可复现的四节点 SFT 启动器。
*   **#7196 Bypass 模式下新增 Rollout-Actor 概率诊断**：添加了可选的诊断功能，用于在 bypass 模式下恢复 rollout 与 actor 的一致性指标。

**🐛 Bug 修复**
*   **#7193 修复模型合并器输出校验缺失（修复 #7191）**：在模型序列化后添加了终端文件系统级别的有效性检查，确保写入的权重和模型结构正确。
*   **#7195 修复 Veomni EP 专家传输错误**：修复了当 EP（专家并行）> 1 时，各 rank 仅发送本地专家导致 vLLM gpt-oss gather 失败的问题（现改为一起发送所有 expert tensors）。

**🛠️ 重构、依赖与清理**
*   **#7190 移除对 vLLM 0.18.0 以下版本的支持**：将最低支持的 vLLM 版本提升至 0.18.0，并清理了所有旧版本的兼容性代码。
*   **#7194 依赖更新**：将 `peft` 库的版本要求从 `>=0.15.2` 升级至 `>=0.20.0`。
*   **#7189 代码清理**：移除了 vllm rollout 中未使用的 `get_device_uuid` 函数。

---

### 🚀 Release 动态
*   近期无新版本发布。

---

## 🐛 Issues

### #7191 — [Model merger success path lacks a terminal Hugging Face output check](https://github.com/verl-project/verl/issues/7191)
- **作者**: kaining-never-stop  **时间**: 2026-07-29 23:38 CST
- **摘要**: # Model merger success path lacks a terminal Hugging Face output check  ## System Info  - Repository: `verl-project/verl` - Current main checked: `bc72e38e` - Model-merger baseline tested: `7fbc00a9` - Python: 3.11.6 - PyTorch: 2.12.0+cu130 - Transformers: 5.8.1 - Accelerate: 1.13.0 - Ray: 2.55.1 - …

## 🔀 Pull Requests

### #7196 — [[trainer] feat: add opt-in rollout-vs-actor probs diagnostic for bypass mode](https://github.com/verl-project/verl/pull/7196)
- **作者**: yueyiming2009  **时间**: 2026-07-30 08:27 CST
- **摘要**: ### What does this PR do?  Adds an opt-in diagnostic that restores the rollout-vs-actor consistency metrics in **bypass mode**.  In bypass mode (`algorithm.rollout_correction.bypass_mode=true`) the trainer sets `old_log_probs = rollout_log_probs` and never recomputes the actor policy π_θ, so `calcul…

### #7195 — [[veomni] fix: Fix gpt oss gather with ep expert transfer error](https://github.com/verl-project/verl/pull/7195)
- **作者**: wyettzeng  **时间**: 2026-07-30 02:06 CST
- **摘要**: ### What does this PR do?  vLLM expects all expert tensors to be received together for gpt-oss. However, for Veomni with ep > 1, each rank will only send over the experts that it has. Therefore, we will need to gather all the experts before sending it to vLLM during actor-rollout sync.  ### Checklis…

### #7194 — [build(deps): update peft requirement from >=0.15.2 to >=0.20.0](https://github.com/verl-project/verl/pull/7194)
- **作者**: dependabot[bot]  **时间**: 2026-07-30 01:33 CST
- **标签**: dependencies, python
- **摘要**: Updates the requirements on [peft](https://github.com/huggingface/peft) to permit the latest version. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/peft/releases">peft's releases</a>.</em></p> <blockquote> <h2>v0.20.0</h2> <h1>Highlights</h1> …

### #7193 — [[ckpt, model] fix: validate model merger outputs](https://github.com/verl-project/verl/pull/7193)
- **作者**: kaining-never-stop  **时间**: 2026-07-30 00:26 CST
- **摘要**: ### What does this PR do?  Fixes #7191.  `verl.model_merger` currently has no terminal filesystem-level check after model serialization. A standard Transformers model normally writes its weights, but the merger supports pluggable/custom model implementations and multiple save paths, so callers have …

### #7192 — [Add native HybridModel MTP support](https://github.com/verl-project/verl/pull/7192)
- **作者**: Phlip79  **时间**: 2026-07-30 00:17 CST
- **摘要**: Support native MCore MTP for Nemotron 3 Super and add a reproducible four-node SFT launcher.  ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] …

### #7190 — [[vllm] refactor: drop support for vLLM older than 0.18.0](https://github.com/verl-project/verl/pull/7190)
- **作者**: aoshen02  **时间**: 2026-07-29 23:19 CST
- **标签**: vllm related
- **摘要**: Raises the minimum supported vLLM to 0.18.0 and removes every compatibility branch at or below that version, across `verl/workers/rollout/vllm_rollout/`, `verl/utils/vllm/`, and `verl/third_party/vllm/`.

### #7189 — [[vllm] chore: remove unused get_device_uuid from vllm rollout](https://github.com/verl-project/verl/pull/7189)
- **作者**: aoshen02  **时间**: 2026-07-29 22:27 CST
- **摘要**: Delete the useless code.

### #7188 — [[trainer] feat: support decouple ppo for v1 separate_async](https://github.com/verl-project/verl/pull/7188)
- **作者**: zpltys  **时间**: 2026-07-29 16:36 CST
- **摘要**: ### What does this PR do?  as the title  test result （mimo 7b， dapo dataset）  <img width="1741" height="467" alt="image" src="https://github.com/user-attachments/assets/e4c68ee8-7269-4b82-b191-f31ed180d919" /> <img width="1744" height="477" alt="image" src="https://github.com/user-attachments/assets…

### #7187 — [[fsdp, model] feat: support full-rank VLM modules with LoRA](https://github.com/verl-project/verl/pull/7187)
- **作者**: WenZheWang  **时间**: 2026-07-29 14:14 CST
- **摘要**: ### What does this PR do?  Adds PEFT modules_to_save semantics to the FSDP Hugging Face model path so a VLM can combine language-model LoRA with fully trainable non-LoRA submodules such as a multimodal projector.  - exposes actor_rollout_ref.model.modules_to_save and passes it to LoraConfig; - keeps…

### #7186 — [  [rollout] feat: support MXFP8 rollout quantization](https://github.com/verl-project/verl/pull/7186)
- **作者**: jQizhang  **时间**: 2026-07-29 14:13 CST
- **摘要**: ### What does this PR do?  This PR adds MXFP8 rollout weight quantization support for the SGLang backend.    When `actor_rollout_ref.rollout.quantization=mxfp8` is enabled, verl dynamically quantizes updated higher-precision rollout weights into MXFP8 on the SGLang side. Selected linear and MoE expe…
