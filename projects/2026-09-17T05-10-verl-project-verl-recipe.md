# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-09-17 13:10 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的中文摘要：

### 📋 Issue 动态
*近期暂无相关活动。*

### 🔧 PR 动态
* **PR #153: 为 Dynamo 后端添加 SGLang 引擎及异步 RL 支持** (作者: sophiayyya)
  * **新特性**：在现有的 `dynamo` rollout 后端中引入了 SGLang 作为可选的底层推理引擎，并支持了异步强化学习（async RL）。
  * **使用方式**：用户可通过配置 `actor_rollout_ref.rollout.engine_kwargs.dynamo.engine=sglang` 来启用 SGLang 引擎。如果不特别指定，系统默认仍使用 vLLM 引擎。

### 🚀 Release 动态
*近期暂无新版本发布。*

*(注：本次动态主要聚焦于推理后端的扩展，通过引入 SGLang 为 Dynamo 提供了更多的引擎选择与异步能力。)*

---

## 🔀 Pull Requests

### #153 — [feat(dynamo): add SGLang engine backend and async RL for Dynamo backend](https://github.com/verl-project/verl-recipe/pull/153)
- **作者**: sophiayyya  **时间**: 2026-09-17 10:34 CST
- **摘要**: ## What this PR adds  Add SGLang as an engine behind the existing `rollout.name=dynamo` backend. Select it with `actor_rollout_ref.rollout.engine_kwargs.dynamo.engine=sglang`; vLLM remains the default.  The branch provides SGLang worker launch/configuration. It also carries V1 colocate/separate trai…
