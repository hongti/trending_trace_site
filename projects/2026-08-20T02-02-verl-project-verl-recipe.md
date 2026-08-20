# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-08-20 10:02 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的中文摘要：

### 🚀 Pull Request (PR)
*   **#138 新增 NeMo Gym 浏览器交互训练配方** (作者: waple0820)
    *   **重要新特性**：引入了 `nemo_gym/browser/` 目录，支持在 NeMo Gym 的 `interactive_browser` 环境下进行逐步 GRPO（step-wise GRPO）训练。该策略可通过 `observe`、`navigate`、`click` 等工具驱动真实的浏览器进行交互。
*   **#137 清理冗余逻辑与文档更新** (作者: wyettzeng)
    *   **优化修复**：移除了 Verl tinker 中不必要的 shutdown 逻辑，并改进了 README 文档。

### 🐛 Issue
*   近期无新增动态。

### 📦 Release
*   近期无新版本发布。

---

## 🔀 Pull Requests

### #138 — [[recipe] feat: add nemo_gym/browser — step-wise GRPO on the NeMo Gym interactive_browser environment](https://github.com/verl-project/verl-recipe/pull/138)
- **作者**: waple0820  **时间**: 2026-08-19 16:12 CST
- **摘要**: ## What this adds  `nemo_gym/browser/` — step-wise GRPO training on the NeMo Gym `interactive_browser` environment. The policy drives a live browser through one tool (`observe` / `navigate` / `click` / `type` / `finish`) and is scored per episode.  Nothing under `nemo_gym/` is modified. The only cha…

### #137 — [Verl tinker remove unnecessary shutdown logic](https://github.com/verl-project/verl-recipe/pull/137)
- **作者**: wyettzeng  **时间**: 2026-08-19 12:17 CST
- **摘要**: - Remove unnecessary shutdown logic - Improve readme
