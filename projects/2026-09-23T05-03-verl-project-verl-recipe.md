# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-09-23 13:03 CST

## AI 总结

以下是 **verl-project/verl-recipe** 仓库近期动态的简洁摘要：

### 📝 Issue（议题）
近期无公开的 Issue 动态。

### 🔀 Pull Request（拉取请求）
* **PR #156：支持 Uni-Agent ThunderAgent 异步训练** (作者: sophiayyya)
  * **核心变更**：引入了对 Uni-Agent ThunderAgent 异步训练的支持，允许固定的 Uni-Agent Gateway 结合 vLLM 0.24 运行 Dynamo。
  * **新特性**：支持两种异步训练模式，分别是 V1 架构下的 `colocate_async`（混合异步）以及完全分离的 `separate_async`（完全异步）模式。
  * **关联动态**：此 PR 是在 #153 基础上的后续集成工作。

### 🚀 Release（版本发布）
近期无新的版本发布动态。

---

## 🔀 Pull Requests

### #156 — [feat(dynamo): support Uni-Agent ThunderAgent async training ](https://github.com/verl-project/verl-recipe/pull/156)
- **作者**: sophiayyya  **时间**: 2026-09-22 16:49 CST
- **摘要**: Enable the pinned Uni-Agent Gateway to run Dynamo ThunderAgent with vLLM 0.24 in both V1 `colocate_async` and fully async `separate_async`. This follows #153 and contains the subsequent integration changes.  ## Changes  - Add a recipe-owned Gateway adapter that keeps multi-turn program state across …
