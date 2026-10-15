# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-10-15 13:02 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的中文摘要：

### 一、 Issue 动态
近期无 Issue 相关动态。

### 二、 PR 动态
近期共合并/提交 2 个 PR，主要涉及 Bug 修复与新特性支持：

*   **PR #163：修复 histoSpec 数据集和采样器辅助函数的导入问题**
    *   **作者**: jasonchen505
    *   **变更/修复**: 修复了 `specRL/histoSpec/main_ppo.py` 中的导入错误。原代码尝试从不存在的模块（`verl.trainer.histoSpec.main_ppo`）导入 `create_rl_dataset` 和 `create_rl_sampler`，现已修正该导入路径问题，以确保在指定的 verl commit 版本下正常运行。
*   **PR #162：新特性 - 将 Claude Code 子智能体作为 ThunderAgent 程序调度**
    *   **作者**: liz-badada
    *   **新特性**: 优化了 Dynamo 路由机制。此前 Claude Code 的 `x-claude-code-*` 智能体标头会被 Uni-Agent Gateway 拦截截断，导致 ThunderAgent 将每个子智能体的交互误判为无关程序。此 PR 修复了该阻塞性问题，使得 Claude Code 的子智能体能够被正确识别并作为 ThunderAgent 程序进行调度。

### 三、 Release 动态
近期无 Release 发布信息。

---

## 🔀 Pull Requests

### #163 — [Fix histoSpec import of dataset and sampler helpers](https://github.com/verl-project/verl-recipe/pull/163)
- **作者**: jasonchen505  **时间**: 2026-10-14 19:05 CST
- **摘要**: specRL/histoSpec/main_ppo.py imported create_rl_dataset and create_rl_sampler from verl.trainer.histoSpec.main_ppo  That module does not exist in the pinned verl commit ccd7d93, so importing the entry point raised ModuleNotFoundError  This change imports both helpers from verl.trainer.main_ppo, wher…

### #162 — [feat(dynamo): schedule Claude Code subagents as ThunderAgent programs](https://github.com/verl-project/verl-recipe/pull/162)
- **作者**: liz-badada  **时间**: 2026-10-14 15:01 CST
- **摘要**: co-authors: @AichenF  Claude Code's `x-claude-code-*` agent headers stop at the Uni-Agent Gateway and never reach the Dynamo router, so ThunderAgent saw every subagent turn as an unrelated program that was never closed. This PR makes the verl-recipe adapter track subagents as programs itself.  - `dy…
