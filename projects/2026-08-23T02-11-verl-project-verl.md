# verl-project/verl — 动态追踪

> 生成时间: 2026-08-23 10:11 CST

## AI 总结

## verl-project/verl 近期动态摘要

---

### 🐛 Issue

- **#7520** — FSDP2 下 fused untied `lm_head` 在梯度累积时崩溃（缺失 `_unsharded_param`）。环境：verl 0.10.0.dev + Qwen3.5-9B + 8×A800。属于较严重的训练阻断性 Bug。

---

### 🔀 Pull Request

**🚀 新特性**
- **#7519** — **MXFP8 训练 + SGLang 推理组合**（Megatron + SGLang），面向 Blackwell 架构，从设计上保证训练-推理一致性，端到端解决 #5972。**重要特性。**

**🔧 修复**
- **#7521** — Megatron BSHD 默认对每个 micro-batch 做 padding 改为 **opt-in**，避免不必要的性能损耗。
- **#7516** — 当 `total_epochs` 先于 `total_training_steps` 耗尽时，`is_last_step` 永不为 True，导致最终 checkpoint 丢失。现已修复。
- **#7515** — 纯文本 batch 下 VL processor 仍调用 `get_rope_index` 导致 `next(None)` 报错，现已跳过。
- **#7514** — 蒸馏任务中 `response_mask` 与 padded log-probs 长度不一致触发 shape assert，已对齐修复。
- **#7517** — Ascend E2E 测试中启用 `fully_sharded_loras`，绕过 vllm LoRA TP shape Bug。
- **#7518** — Agent-loop 测试引入确定性采样/调度、固定 Python hash seed、稳定 request ID 生成，消除测试不稳定。

---

### 📦 Release

- 本周期内无新版本发布。

---

> **总结**：本期核心亮点是 **MXFP8 训练+推理全链路支持（#7519）**；修复集中在 FSDP2 梯度累积崩溃、checkpoint 保存遗漏、蒸馏 shape 对齐等关键训练稳定性问题。

---

## 🐛 Issues

### #7520 — [[Bug][FSDP2] Fused untied lm_head crashes during gradient accumulation with missing _unsharded_param](https://github.com/verl-project/verl/issues/7520)
- **作者**: Zhaoyi-Tian  **时间**: 2026-08-23 00:52 CST
- **标签**: bug
- **摘要**: ### System Info  verl: 0.10.0.dev, source checkout from main Model: Qwen3.5-9B Architecture: Qwen3_5ForConditionalGeneration Hardware: 8 × NVIDIA A800-SXM4-80GB NVIDIA driver: 570.158.01 CUDA runtime: 12.8 OS: Linux x86_64, glibc 2.31 Python: 3.12.13 PyTorch: 2.10.0+cu128 Transformers: 5.3.0.dev0 vL…

## 🔀 Pull Requests

### #7521 — [[megatron, perf] fix: make mini-batch BSHD padding opt-in](https://github.com/verl-project/verl/pull/7521)
- **作者**: EazyReal  **时间**: 2026-08-23 06:54 CST
- **摘要**: ### What does this PR do?  Megatron BSHD currently defaults to padding every final micro-batch to the longest sequence in the surrounding mini-batch. That target is computed before dynamic micro-batch membership is finalized. For two final micro-batches containing 14,010 and 512 tokens, the default …

### #7519 — [[megatron, rollout] feat: MXFP8 training and SGLang rollout](https://github.com/verl-project/verl/pull/7519)
- **作者**: wengeezhang  **时间**: 2026-08-22 22:38 CST
- **摘要**: ### What does this PR do?  Enables the **MXFP8 train + rollout combo (Megatron + SGLang)** on Blackwell, addressing #5972 end-to-end with a single design goal: **train-inference consistency by construction**.  Two parts:  **1. Training (`fp8_recipe: "mxfp8"`)** — FP8 training with the `blockwise` re…

### #7518 — [[rollout, ci] fix: make agent-loop tests fully deterministic](https://github.com/verl-project/verl/pull/7518)
- **作者**: lxb007981  **时间**: 2026-08-22 19:24 CST
- **摘要**: ### What does this PR do? Configure deterministic sampling and scheduling, seed Python hashing, and generate stable request IDs from sample priority when full determinism is enabled for agent loops. ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... …

### #7517 — [[ci] fix: enable fully sharded LoRA in Ascend E2E tests](https://github.com/verl-project/verl/pull/7517)
- **作者**: lxb007981  **时间**: 2026-08-22 12:43 CST
- **摘要**: ### What does this PR do? Enable fully_sharded_loras to work around a vllm LoRA tensor-parallel shape bug. Details at https://github.com/vllm-project/vllm/issues/47650  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #7516 — [[trainer, ckpt] fix: save final checkpoint when epochs exhaust](https://github.com/verl-project/verl/pull/7516)
- **作者**: kekellllll  **时间**: 2026-08-22 12:03 CST
- **摘要**: ### What does this PR do?  When `total_epochs` runs out before `total_training_steps`, `is_last_step` never becomes true. A final step that is not on `save_freq` (for example step 237 with `save_freq=50`) is then discarded. Force-save that last completed step after the train loop exits.  ### Checkli…

### #7515 — [[rollout, tests] fix: skip VL get_rope_index on text-only batches](https://github.com/verl-project/verl/pull/7515)
- **作者**: kekellllll  **时间**: 2026-08-22 12:03 CST
- **摘要**: ### What does this PR do?  VL processors still expose `get_rope_index` when the batch has no image/video grid. Text-only training (or leftover `image_pad` / `video_pad` tokens) then calls `next(None)` inside `get_rope_index`. Fall back to 1D `compute_position_id_with_mask` when both grids are missin…

### #7514 — [[algo, trainer, tests] fix: align distillation response masks to padded log-probs](https://github.com/verl-project/verl/pull/7514)
- **作者**: kekellllll  **时间**: 2026-08-22 12:03 CST
- **摘要**: ### What does this PR do?  Nested `response_mask` and `no_padding_2_padding` log-probs can pad to different lengths, so reverse-KL / top-k distillation hits a shape assert on some batches. This also refreshes `global_batch_info` from the current micro-batch instead of inheriting stale normalization …
