# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-11-08 14:37 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的中文摘要：

### 🛠️ Pull Request (PR) 动态
*   **#166 修复异步流性能指标名称拼写错误** (作者: li-lizhe)
    *   **变更内容**：修复了 `reduce_timing_metrics()` 函数中 AsyncFlow 吞吐量指标的键名拼写错误。
    *   **修复细节**：原先代码中使用了拼写错误的键名 `perf/throughtput`，此 PR 将其修正为文档中实际记录和规范的标准名称 `perf/throughput`。

### 📝 Issue 动态
*   近期无公开的 Issue 动态。

### 🚀 Release 动态
*   近期无新的版本发布。

---

## 🔀 Pull Requests

### #169 — [fix(async_flow): emit the documented perf/throughput metric name](https://github.com/verl-project/verl-recipe/pull/169)
- **作者**: li-lizhe  **时间**: 2026-11-07 21:45 CST
- **摘要**: ## What this fixes  `reduce_timing_metrics()` publishes the two AsyncFlow throughput numbers under misspelled keys:  ```python timing_metrics.update({"perf/throughtput": e2e_tps}) timing_metrics.update({"compute/throughtput": compute_tps}) ```  Everything on the reading side uses the correct spellin…
