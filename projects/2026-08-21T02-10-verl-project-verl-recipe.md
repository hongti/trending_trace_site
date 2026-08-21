# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-08-21 10:10 CST

## AI 总结

以下是 **verl-project/verl-recipe** 仓库近期动态的中文摘要：

### 🐛 Issue
*   **#139**：报告 `langgraph_agent` 存在崩溃 Bug。当 chat template 返回 `BatchEncoding` 对象时，会导致程序运行异常。

### 🔧 Pull Request (PR)
*   **#140**：**[修复]** 修复 Issue #139，对 LangGraph 中 `tokenizer.apply_chat_template` 返回的 token IDs 进行规范化处理，确保在后续的切片、追加及发送至 rollout 之前数据格式正确，避免崩溃。
*   **#141**：**[新特性]** 新增 **TMax** terminal-agent 端到端复现配方。TMax 是一种用于训练长视野（long-horizon）终端智能体的 RL 配方，结合了 TMax-15K 环境进行训练。
*   **#142**：**[新特性]** 新增 **ECHO** terminal-agent 端到端复现配方。ECHO 是一种针对终端智能体的 RL 配方，它在 GRPO 算法的基础上，增加了针对环境观察 tokens 的 on-policy 交叉熵损失。

### 🚀 Release
*   近期无新版本发布。

---

## 🐛 Issues

### #139 — [[BUG] langgraph_agent crashes when chat templates return BatchEncoding](https://github.com/verl-project/verl-recipe/issues/139)
- **作者**: le-czs  **时间**: 2026-08-20 11:14 CST
- **摘要**: ## System Info  - `verl-recipe`: `main` at `ab641fecadad7bc34b2d6b3e25a38a22d85801c5` - `verl`: the commit pinned by `langgraph_agent/REQUIRED_VERL.txt` (`bcb638649a50e58494a8ddd92085ad1174f674b8`) - Python: 3.12 - Transformers: reproduced with 5.5.3 - Recipe: official `langgraph_agent`  `python scr…

## 🔀 Pull Requests

### #142 — [feat(recipe): add ECHO terminal-agent reproduction](https://github.com/verl-project/verl-recipe/pull/142)
- **作者**: kylemontgomery1  **时间**: 2026-08-21 07:41 CST
- **摘要**: ## Background & Motivation  ECHO is an RL recipe for terminal agents that augments GRPO with an on-policy cross-entropy loss over environment-observation tokens. This PR adds an end-to-end reproduction of the paper's main Qwen3-8B experiment to verl-recipe. The motivation, design, and experimental s…

### #141 — [feat(recipe): add TMax terminal-agent reproduction](https://github.com/verl-project/verl-recipe/pull/141)
- **作者**: kylemontgomery1  **时间**: 2026-08-21 05:34 CST
- **摘要**: ## Background & Motivation  [TMax](https://arxiv.org/abs/2606.23321) is an RL recipe for training long-horizon terminal agents using the released TMax-15K environments. This PR adds an end-to-end reproduction of the paper’s main Qwen3.5-9B experiment to `verl-recipe`. The motivation, design, and exp…

### #140 — [[recipe] fix: normalize LangGraph chat template token IDs](https://github.com/verl-project/verl-recipe/pull/140)
- **作者**: le-czs  **时间**: 2026-08-20 15:36 CST
- **摘要**: ### What does this PR do?  Fixes #139 by normalizing every `tokenizer.apply_chat_template(..., tokenize=True)` result used by the LangGraph recipe before it is sliced, appended to, sent to the rollout server, or stored in response metadata. This makes tokenizers returning a Transformers `BatchEncodi…
