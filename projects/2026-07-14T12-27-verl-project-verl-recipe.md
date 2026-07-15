# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-07-14 20:27 CST

## AI 总结

以下是 **verl-project/verl-recipe** 仓库最近动态的中文简洁摘要：

### 🚀 Pull Request (PR)
*   **PR #120: Verl tinker profiler 及卸载逻辑重构** | 作者: wyettzeng | 2026-07-13
    *   **新特性/重要变更**：
        1. **新增 Profiler 示例**：添加了启用 profiler 的示例代码，方便用户进行性能分析。
        2. **卸载逻辑迁移**：将 offloading（卸载）逻辑从 verl 核心移出至 tinker server 中，实现了逻辑的解耦。
        3. **支持惰性卸载**：如果启用该功能，系统将执行惰性卸载，有助于优化资源调度与性能。

### 🐛 Issue
*   暂无近期动态。

### 📦 Release
*   暂无新版本发布。

---

## 🔀 Pull Requests

### #120 — [Verl tinker profiler + offloading logic](https://github.com/verl-project/verl-recipe/pull/120)
- **作者**: wyettzeng  **时间**: 2026-07-14 02:36 CST
- **摘要**: - Add in example code to enable profiler - Bring offloading logic out of verl and into the tinker server, do laze offloading if enabled
