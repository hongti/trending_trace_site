# verl-project/verl — 动态追踪

> 生成时间: 2026-07-02 12:55 UTC

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🚀 Release（版本发布）
近期无新版本发布。

### 🐛 Issue（问题讨论）
近期无新 Issue 记录。

### 🔧 Pull Request（代码合并请求）
近期的 PR 主要集中在 **训练器资源清理与修复**、**内核/训练逻辑优化** 以及 **文档补充**，具体亮点如下：

*   **训练器修复与兼容性更新**
    *   **#6917**：修复 V1 PPO trainer 在训练结束时资源释放过晚的问题。原先依赖 `__del__`/垃圾回收清理 tracking 和 dataloader workers，在 Ray 环境下容易延迟；现改为在训练循环外层使用 `try/finally` 显式、优雅地终止资源，确保正常退出或异常时都能及时清理。
    *   **#6916**：更新 TorchtitanEngine 以兼容最新的 torchtitan nightly 版本，进行了 API 兼容性修复，并新增了端到端（e2e）测试脚本（附与 FSDP engine 的对比测试）。

*   **内核与训练逻辑 Bug 修复 / 增强**
    *   **#6913**：修复 fused-kernel 梯度追踪 Bug。此前同时使用 `use_liger` 和 `use_fused_kernel` 标志会导致计算结果被悄无声息地损坏，该 PR 解决了内部的 `.flatten(0,1)` 引起的梯度追踪问题。
    *   **#6914**：在 `fully_async_opd` 流程中，为 teacher forward 和 student train 添加了融合节点，以优化异步训练性能。

*   **文档更新**
    *   **#6915**：补充 SGLang FP8 rollout 的使用文档，说明如何在权重量化时跳过指定模块。文档增加了 `SGLANG_FP8_IGNORED_LAYERS` 的逗号分隔配置示例，以及模型 `quantization_config.ignored_layers` 的正则（`re:`）匹配模式示例（跟进 #6906 功能）。

---

## 🔀 Pull Requests

### #6917 — [[trainer] fix: gracefully finalize tracking and dataloader workers at fit() exits](https://github.com/verl-project/verl/pull/6917)
- **作者**: seokhyunan  **时间**: 2026-07-02 07:34 UTC
- **摘要**: ### What does this PR do?  The V1 PPO trainer finalizes two end-of-run resources only via `__del__` / garbage collection, which under Ray / interpreter teardown runs too late. This PR finalizes both explicitly, in a single `try/finally` around the training loop, so cleanup runs on normal completion,…

### #6916 — [[trainer] fix: Update latest TorchtitanEngine](https://github.com/verl-project/verl/pull/6916)
- **作者**: acisseJZhong  **时间**: 2026-07-02 07:18 UTC
- **摘要**: ### What does this PR do?  API-compatibility fixes so the TorchTitan training engine works with latest torchtitan nightly, added an e2e test script.  ### Test Ran comparison to FSDP engine([config](https://gist.github.com/acisseJZhong/0bf41bc1a7c624529b7f0c259b3d664a)).  ### Checklist Before Submitt…

### #6915 — [[docs] add SGLang FP8 ignored layers usage](https://github.com/verl-project/verl/pull/6915)
- **作者**: gem-mint  **时间**: 2026-07-02 03:33 UTC
- **摘要**: ﻿## Summary  - Document how to skip selected modules during SGLang FP8 rollout weight quantization. - Add examples for `SGLANG_FP8_IGNORED_LAYERS`, comma-separated entries, and model `quantization_config.ignored_layers` with `re:` patterns.  ## Context  This docs-only PR follows up on #6906, which a…

### #6914 — [add fusenode on teacher forward and student train in fully_async_opd](https://github.com/verl-project/verl/pull/6914)
- **作者**: msz12345  **时间**: 2026-07-02 02:23 UTC
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #6913 — [Fixing fused-kernel gradient tracking bug.](https://github.com/verl-project/verl/pull/6913)
- **作者**: kolehma8  **时间**: 2026-07-01 22:21 UTC
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  Currently using use_liger and use_fused_kernel flags create silently corrupted result. The issue is that the .flatten(0,1) inside t…
