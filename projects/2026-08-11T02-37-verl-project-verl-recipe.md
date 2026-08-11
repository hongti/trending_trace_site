# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-08-11 10:37 CST

## AI 总结

以下是 **verl-project/verl-recipe** 仓库近期动态的中文摘要：

### 📌 Issue
- **#132 [RFC] ECHO 终端训练方案**（作者：kylemontgomery1 | 2026-08-11）
  - **核心提议**：提议为 verl 引入新的 ECHO 训练方案。
  - **重要特性**：该方案在 GRPO 的基础上进行了增强，针对环境观察引入了基于在线策略的下一 token 交叉熵目标，旨在优化模型在环境交互下的训练效果。

### 📌 Pull Request (PR)
- 暂无近期动态

### 📌 Release
- 暂无近期动态

---

## 🐛 Issues

### #132 — [[RFC] ECHO terminal training recipe](https://github.com/verl-project/verl-recipe/issues/132)
- **作者**: kylemontgomery1  **时间**: 2026-08-11 09:56 CST
- **摘要**: ### Summary This RFC proposes an ECHO training recipe for verl. ECHO augments GRPO with an on-policy next-token cross-entropy objective over environment-observation tokens. The recipe implements this objective through a generic auxiliary-token loss interface, allowing agent loops to select arbitrary…
