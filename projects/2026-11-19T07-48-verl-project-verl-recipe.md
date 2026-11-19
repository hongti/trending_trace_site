# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-11-19 15:48 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的中文摘要：

### 📦 Release 动态
*本期暂无新的版本发布。*

### 🚀 PR 动态
* **PR #174：新增 Qwen3.5-35B-A3B DeepSWE 的 TPU 训练配方**
  * **作者**: askhat-g
  * **时间**: 2026-11-18
  * **重要变更**: 新增了 `deepswe_tpu/` 训练配方。该配方支持在 **TPU v7x** 上训练 `Qwen3.5-35B-A3B` 模型。
  * **核心特性**: 结合了 R2E-Gym 环境、`uni-agent` (ReAct 智能体)、TorchTitan 训练器以及异步 vLLM rollout，进一步完善了框架在 TPU 硬件上的强化学习训练能力。

### ❗ Issue 动态
*本期暂无 Issue 讨论。*

**总结**：本期仓库最主要的动态是针对 TPU 平台引入了针对 Qwen3.5-35B-A3B 模型的新训练配方，展示了其在异构硬件（TPU）上结合智能体与异步推理进行大规模训练的支持能力。

---

## 🔀 Pull Requests

### #174 — [[recipe] feat: add Qwen3.5-35B-A3B DeepSWE recipe for TPU](https://github.com/verl-project/verl-recipe/pull/174)
- **作者**: askhat-g  **时间**: 2026-11-18 18:03 CST
- **摘要**: ## Summary  Adds a `deepswe_tpu/` recipe that trains `Qwen/Qwen3.5-35B-A3B` on R2E-Gym with a ReAct agent (`uni-agent`), a TorchTitan trainer and an async vLLM rollout on TPU v7x. The recipe runs against unmodified `verl` `main` and unmodified `verl-hardware-plugin` `main`: all DeepSWE-specific logi…
