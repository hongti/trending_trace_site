# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-08-12 11:07 CST

## AI 总结

以下是 **verl-project/verl-recipe** 仓库最近的动态摘要：

### 📋 Issue 动态
* **#133 [RFC] TMax 终端训练方案**
  * **重点**：作者 kylemontgomery1 提议新增 TMax 训练配方。TMax 是一种专为长周期终端智能体设计的强化学习（RL）方案，其核心亮点在于结合了 TMax-15K 环境与全异步训练机制。

### 🔀 PR 动态
* **#134 feat: 新增 Ascend INT8 W8A8 QAT rollout 配方**
  * **重点**：作者 sunsunsun98 引入了一项实验性的昇腾 INT8 W8A8 rollout 配方。该 PR 的核心特性是支持 FSDP 量化感知训练（QAT），并适配了 GRPO 算法，支持模型包括 Qwen3-8B、Qwen3-30B MoE 以及 openPangu-7B。

### 🚀 Release 动态
* 近期暂无新版本发布。

---

## 🐛 Issues

### #133 — [[RFC] TMax terminal training recipe](https://github.com/verl-project/verl-recipe/issues/133)
- **作者**: kylemontgomery1  **时间**: 2026-08-11 11:30 CST
- **摘要**: ### Summary This RFC proposes a TMax training recipe for verl. TMax is an RL recipe for long-horizon terminal agents that combines the released TMax-15K environments with fully asynchronous training on long contexts. The recipe targets the paper's main Qwen3.5-9B experiment.  ### Motivation TMax pro…

## 🔀 Pull Requests

### #134 — [feat(recipe): add Ascend INT8 W8A8 QAT rollout](https://github.com/verl-project/verl-recipe/pull/134)
- **作者**: sunsunsun98  **时间**: 2026-08-11 20:26 CST
- **摘要**: ## What does this PR do?  Adds an experimental Ascend INT8 W8A8 rollout recipe with FSDP quantization-aware training for Qwen3-8B/Qwen3-30B MoE/openPangu-7B GRPO .  The implementation is maintained in: https://github.com/sunsunsun98/verl-int8-w8a8  The recipe pins the custom veRL v0.7.0-based implem…
