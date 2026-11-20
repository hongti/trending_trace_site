# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-11-20 15:40 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 最近的动态摘要：

### 🛠️ Pull Requests (PR)
*   **PR #175: 引入 Dynamo 的 SGLang Sidecar 模式及相关高级特性** (作者: sophiayyya)
    *   **新特性**：为 `recipe/dynamo` 模块添加了一种全新的 SGLang rollout 分片运行方式。除了现有的运行方式外，现在支持通过 Dynamo 的 SGLang sidecar（执行 `python -m dynamo.sglang.sidecar`）来运行原生的 SGLang。
    *   **重要变更**：该 PR 带来了对原生 `/generate` 接口、R3 以及 PD（Prefill-Decode）分离架构的支持，进一步丰富了部署和推理的灵活性。

### 🐛 Issues
*   近期暂无公开的 Issue 动态。

### 🚀 Releases
*   近期暂无新的版本发布。 

*(注：当前周期内的核心动态集中在 PR #175，重点是对 Dynamo 模块下 SGLang 推理后端的架构扩展与功能增强。)*

---

## 🔀 Pull Requests

### #175 — [[recipe] feat(dynamo): SGLang sidecar mode, native /generate, R3 and PD disaggregation](https://github.com/verl-project/verl-recipe/pull/175)
- **作者**: sophiayyya  **时间**: 2026-11-20 01:20 CST
- **摘要**: ### What does this PR do?    Adds a second way for `recipe/dynamo` to run SGLang rollout shards: **stock SGLang behind Dynamo's SGLang sidecar** (`python -m dynamo.sglang.sidecar`), next to the existing `dynamo.sglang` worker. On top of that it adds the SGLang-native `/generate` path, rollout routin…
