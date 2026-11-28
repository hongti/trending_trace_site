# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-11-28 16:19 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的中文摘要：

### 📝 Pull Request (PR) 动态
* **PR #183**（作者: jialei777 | 时间: 2026-11-28）
  * **核心变更**：更新了 `deepswe_tpu` 的 **Qwen3.5-397B 64K** 训练配方默认配置。
  * **新特性与优化**：采用了目前实测最快的 **no-drop（无丢弃）** 训练器配置，默认集成了分块专家并行、全局打包以及 GDN `all_to_all` 机制。
  * **重要修复**：修复了 fp32-master 优化器路径的问题，保障了 397B 模型训练流程的正确执行。

### 🐛 Issue 动态
* 近期暂无公开的 Issue 动态。

### 🚀 Release 动态
* 近期暂无新版本发布。

---

## 🔀 Pull Requests

### #183 — [[deepswe_tpu] 397B 64K: no-drop chunked EP + global packing + GDN all_to_all defaults; fix fp32-master optimizer path](https://github.com/verl-project/verl-recipe/pull/183)
- **作者**: jialei777  **时间**: 2026-11-28 14:01 CST
- **摘要**: ### What does this PR do?  Updates the `deepswe_tpu` **Qwen3.5-397B 64K** recipe defaults to the fastest **no-drop** trainer configuration measured so far. It also fixes the optimizer path, so the 397B recipe now runs the fp32-master + host AdamW step it advertises.  > [!WARNING] > These results com…
