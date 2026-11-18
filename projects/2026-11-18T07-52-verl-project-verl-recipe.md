# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-11-18 15:52 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的中文摘要：

### 📋 Issue 动态
*   **#173 [RFC] 异步训练引入可选的分组过滤与预算受限补充机制**
    *   **作者:** volit10nzzzz
    *   **亮点:** 提出了一个针对“完全异步训练”的新设计方案（RFC）。建议在训练过程中利用闲置的 rollout（推演）时间，对已完成的 group 进行过滤和预算受限的补充。此举旨在优化异步训练效率，目前正向社区征集反馈。

### 🔧 Pull Request 动态
*   **#172 为 ReTool 新增 E2B 沙箱工具及配置**
    *   **作者:** max-sudolabs
    *   **新特性:** 为 ReTool 引入了 [E2B](https://e2b.dev) 作为托管的代码执行后端，与现有的 Sandbox Fusion 和 Daytona 并列。
    *   **实现细节:** 新增了 `retool/e2b_sandbox_tool.py` 文件，实现了基于 `BaseTool` 的 `CustomE2BSandboxTool`，为代码执行提供了新的沙箱环境选项。

### 🚀 Release 动态
*   **暂无新版本发布**
    *   近期仓库未发布新的 Release 版本。

---

## 🐛 Issues

### #173 — [[RFC][fully_async] Optional group filtering with budget-bounded refill](https://github.com/verl-project/verl-recipe/issues/173)
- **作者**: volit10nzzzz  **时间**: 2026-11-18 00:23 CST
- **摘要**: ## English  ### Summary and motivation  I would like feedback on **group filtering designed for asynchronous training: using otherwise idle rollout time to filter and refill complete groups, potentially without extending the training critical path under suitable conditions**. This is the primary mot…

## 🔀 Pull Requests

### #172 — [feat(retool): Add E2B sandbox tool and config](https://github.com/verl-project/verl-recipe/pull/172)
- **作者**: max-sudolabs  **时间**: 2026-11-18 00:13 CST
- **摘要**: ## Summary  Adds [E2B](https://e2b.dev) as a hosted code execution backend for ReTool, next to Sandbox Fusion and Daytona.  - `retool/e2b_sandbox_tool.py`: `CustomE2BSandboxTool`, a `BaseTool` on the E2B async SDK. - `retool/e2b_tool_config.yaml`: drop-in tool config. - `retool/tests/test_e2b_sandbo…
