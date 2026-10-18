# verl-project/verl — 动态追踪

> 生成时间: 2026-10-18 12:59 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📥 Pull Request (PR)
* **#8273 [vllm, rollout, ckpt] feat: preserve requests across weight updates**
  * **作者:** cr-gao | **时间:** 2026-10-18
  * **重要变更/新特性:** 针对大模型推理与训练交互环节进行了优化。引入了一项新特性，使得在**模型权重更新期间，能够保留正在处理中的 vLLM 请求**。该功能调用了 vLLM 原生的 `pause_generation(mode="keep", clear_cache=False)` 接口，确保未完成的请求在权重更新后能够无缝继续，且原有的 KV 缓存、已生成的 token 以及 logprobs 等上下文信息均不会丢失。

### 🐞 Issue
* 本次提供的数据中暂无 Issue 相关动态。

### 🚀 Release
* 本次提供的数据中暂无 Release 相关动态。

---

## 🔀 Pull Requests

### #8273 — [[vllm, rollout, ckpt] feat: preserve requests across weight updates](https://github.com/verl-project/verl/pull/8273)
- **作者**: cr-gao  **时间**: 2026-10-18 10:29 CST
- **摘要**: ### What does this PR do?  Preserve in-flight vLLM requests across weight updates using native `pause_generation(mode="keep", clear_cache=False)`. Requests retain KV cache, generated tokens, logprobs and MoE routing, then resume without re-prefilling their history.  Implements the token-level interr…
