# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-10-17 12:52 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的简洁摘要：

### 🛠️ Pull Request (拉取请求)
* **#164 bugfix(EPLB): 保留 NPU 计算图中的专家映射缓冲区**
  * **作者**: LeeHR0930
  * **时间**: 2026-10-16 09:08 UTC
  * **重要修复**: 针对 NPU 环境下的 EPLB（专家负载均衡）进行了 Bug 修复。更新了 EPLB 核心补丁，确保在图捕获前初始化路由缓冲区，并在专家放置更新期间保留其存储。该修复解决了此前 `log2phy` 可能在图捕获后才被初始化所导致的潜在问题，保障了计算图的正确运行。

### 📝 Issue (议题)
* 近期暂无 Issue 动态。

### 🚀 Release (版本发布)
* 近期暂无新版本发布。

---

## 🔀 Pull Requests

### #164 — [bugfix(EPLB): preserve expert mapping buffers for NPU graphs](https://github.com/verl-project/verl-recipe/pull/164)
- **作者**: LeeHR0930  **时间**: 2026-10-16 17:08 CST
- **摘要**: ## Summary  Update the EPLB core patch to initialize routing buffers before graph capture and preserve their storage during expert placement updates.  Previously, `log2phy` could be initialized after graph capture, and subsequent mapping updates could replace the tensors referenced by the captured g…
