# verl-project/verl — 动态追踪

> 生成时间: 2026-07-31 09:05 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 📌 Issue
- **多模态 GRPO 训练速度疑问 (#7203)**：用户使用 SGLang + Megatron + Qwen3.5-9B 在 8*H800 上跑 Geo3k 数据集的多模态 GRPO 训练，稳态速度约 290s/step，希望确认该速度是否正常。
- **监督蒸馏损失归一化 Bug (#7200)**：指出在 v0.9.0.dev 版本中，监督蒸馏的损失归一化计算依赖于 micro-batch 的分区方式，可能存在逻辑问题。

### 🔧 Pull Request
**新特性：**
- **NeoProto 数据平面集成 (#7208)**：为 V0 RayPPOTrainer 引入可选的 NeoProto ref/index 数据平面（默认关闭）。
- **算法与损失函数扩展 (#7197)**：为 Tinker 增加重要性采样和 DRO 损失函数，允许传统 PPO 不进行底部裁剪，并新增 token sum 作为损失聚合方法。
- **Bypass 模式诊断 (#7196)**：为 bypass 模式添加可选的 rollout 与 actor 概率一致性诊断指标。
- **硬件与检查点支持 (#7206, #7205)**：添加 tq CI 支持；为 HCCL 检查点引擎增加 `split_weight_chunks` 功能。

**重要修复：**
- **Rollout 生成超长修复 (#7207)**：修复了部分 rollout 中止并恢复时，生成 token 数可能超出 `response_length` 预算导致生成变慢的问题。
- **Rollout 追踪解码修复 (#7204)**：修复了开启 `token2text=True` 追踪时，每轮生成遗漏 `prompt_text` 和 `response_text` 的问题。
- **FSDP PrefixGrouper 修复 (#7202)**：修复了重构后 FSDP 引擎中 PrefixGrouper 的 response 投影融合问题。
- **Megatron 生成配置保留 (#7199)**：修复了分布式 Megatron 合并模型时丢失预训练 `generation_config.json` 导致生成设置异常的问题。

**文档与重构：**
- **Ascend Docker 更名 (#7201)**：重命名最新的 ascend docker 名称。

### 🚀 Release
- 近期无新版 Release 发布。

---

## 🐛 Issues

### #7203 — [Question about grpo training speed on Geo3k(sglang + megatron + qwen3.5-9B + 8*H800)](https://github.com/verl-project/verl/issues/7203)
- **作者**: 7yzx  **时间**: 2026-07-30 21:44 CST
- **摘要**: Running Qwen3.5-9B multimodal GRPO on Geo3K dataset (geometry problems with images) using SGLang rollout + Megatron training backend. After tuning, steady-state is ~290s/step on 8x H800. Want to confirm whether this is expected performance or if there's room for improvement.   Steady-state per-step …

### #7200 — [[opd, trainer] Supervised distillation loss normalization depends on micro-batch partitioning](https://github.com/verl-project/verl/issues/7200)
- **作者**: yph22  **时间**: 2026-07-30 13:27 CST
- **标签**: bug
- **摘要**: ### System Info  verl:0.9.0.dev  ### Information  - [ ] The official example scripts - [ ] My own modified scripts  ### Tasks  - [ ] An officially supported task in the `examples` folder (such as GLUE/SQuAD, ...) - [ ] My own task or dataset (give details below)  ### Reproduction  The supervised/dir…

## 🔀 Pull Requests

### #7208 — [[neoproto] feat: integrate the ref-only data plane with V0 RayPPOTrainer](https://github.com/verl-project/verl/pull/7208)
- **作者**: alanSquirrelyz  **时间**: 2026-07-31 04:47 CST
- **摘要**: ## **What does this PR do?**  This PR adds an opt-in NeoProto ref/index data plane to the synchronous V0 `RayPPOTrainer`.  - `trainer.use_neoproto=false` remains the default. - The existing DataProto path is unchanged when NeoProto is disabled. - With NeoProto enabled, the controller manipulates sch…

### #7207 — [[rollout] fix: cap partial rollout resumes at the remaining response budget](https://github.com/verl-project/verl/pull/7207)
- **作者**: benyucong  **时间**: 2026-07-31 02:17 CST
- **摘要**: ### What does this PR do?  **Summary:** when a partial rollout is aborted and resumed, the total number of generated tokens can go past `response_length`. The extra tokens are slow to generate, and they are thrown away afterwards. This PR makes the resume path count the tokens it has already generat…

### #7206 — [[hardward] feat: add tq ci](https://github.com/verl-project/verl/pull/7206)
- **作者**: wucong25  **时间**: 2026-07-31 00:44 CST
- **摘要**: ### What does this PR do?  add tq ci  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, `sglang`,…

### #7205 — [[ckpt] feat: add hccl ckpt engine split_weight_chunks](https://github.com/verl-project/verl/pull/7205)
- **作者**: wucong25  **时间**: 2026-07-30 23:24 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  add hccl ckpt engine split_weight_chunks  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, …

### #7204 — [[rollout] fix: decode per-turn LLM tokens in traces](https://github.com/verl-project/verl/pull/7204)
- **作者**: YAO-001  **时间**: 2026-07-30 23:15 CST
- **摘要**: ### What does this PR do?  Fixes #3515.  When rollout tracing is configured with `token2text=True`, per-turn `LLMServerClient.generate` spans currently omit `prompt_text` and `response_text`. This PR makes those decoded fields available for MLflow, Weave, and Trackio while keeping the existing confi…

### #7202 — [[fsdp, perf] fix: fuse PrefixGrouper response projection](https://github.com/verl-project/verl/pull/7202)
- **作者**: supercharleszhu  **时间**: 2026-07-30 20:48 CST
- **摘要**: ### What does this PR do?  Fix the existing PrefixGrouper integration for the refactored FSDP engine.  The original integration from #4368 left PrefixGrouper utilities and attention patching in the repository, but the current actor/ref engine no longer called them. The old utility also projected eve…

### #7201 — [[doc] refactor: rename latest ascend docker name](https://github.com/verl-project/verl/pull/7201)
- **作者**: yyyy2000  **时间**: 2026-07-30 20:34 CST
- **摘要**: ### What does this PR do?   rename latest ascend docker name  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatr…

### #7199 — [[megatron] fix: preserve generation config during merge](https://github.com/verl-project/verl/pull/7199)
- **作者**: KANIKIG  **时间**: 2026-07-30 12:11 CST
- **摘要**: ### What does this PR do?  Fixes #7198.  The distributed Megatron merger now preserves the independent pretrained `generation_config.json` in merged Hugging Face outputs. This prevents generation settings that are absent from `config.json`, such as multiple EOS token IDs, from being lost.  The chang…

### #7197 — [[algo] feat: Add in tinker support for different loss funcitons and aggregation methods](https://github.com/verl-project/verl/pull/7197)
- **作者**: wyettzeng  **时间**: 2026-07-30 09:56 CST
- **摘要**: …hods  ### What does this PR do?  - Add in importance sampling and dro loss function - Allow traditional PPO to not clip bottom - Add in token sum as an aggregation method for loss values  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Form…

### #7196 — [[trainer] feat: add opt-in rollout-vs-actor probs diagnostic for bypass mode](https://github.com/verl-project/verl/pull/7196)
- **作者**: yueyiming2009  **时间**: 2026-07-30 08:27 CST
- **摘要**: ### What does this PR do?  Adds an opt-in diagnostic that restores the rollout-vs-actor consistency metrics in **bypass mode**.  In bypass mode (`algorithm.rollout_correction.bypass_mode=true`) the trainer sets `old_log_probs = rollout_log_probs` and never recomputes the actor policy π_θ, so `calcul…
