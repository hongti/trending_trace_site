# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-07-26 09:06 CST

## AI 总结

以下是 **verl-project/verl-recipe** 仓库近期动态的中文摘要：

### 📌 Issue 动态
暂无近期动态。

### 🚀 Pull Request 动态
*   **PR #125** `[recipe] feat: add gsm8k_outcome_gating` (作者: yuyu0529nya)
    *   **新特性**：新增 `gsm8k_outcome_gating` 配方。这是一个关于 GRPO 算法中“群体过滤何时真正起作用”的完全可复现研究。
    *   **核心亮点**：深入探讨了受塑形奖励影响的“虚幻优势”问题，并引入了抑制该问题的门控机制。
    *   **复现门槛低**：仅需单张 RTX 5090 显卡，采用 Qwen2.5-1.5B-Instruct + LoRA(r=32) 模型在 GSM8K 数据集上运行，每 40 步耗时约 50 分钟。

### 📦 Release 动态
暂无新版本发布。

---

## 🔀 Pull Requests

### #125 — [[recipe] feat: add gsm8k_outcome_gating — phantom advantages under shaped rewards, and the gate that kills them](https://github.com/verl-project/verl-recipe/pull/125)
- **作者**: yuyu0529nya  **时间**: 2026-07-26 02:07 CST
- **摘要**: ## What  A small, fully-reproducible study of **when group filtering actually matters in GRPO**: `gsm8k_outcome_gating`. Single RTX 5090, Qwen2.5-1.5B-Instruct + LoRA(r=32), GSM8K, ~50 min per 40-step arm, pinned verl commit in `REQUIRED_VERL.txt`.  **TL;DR** — under a shaped reward with a mild leng…
