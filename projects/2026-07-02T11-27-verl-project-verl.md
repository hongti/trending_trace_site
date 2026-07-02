# verl-project/verl — 动态追踪

> 生成时间: 2026-07-02 11:27 UTC

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 🛠️ Pull Request (PR)
本次共有 5 个 PR，主要涉及**关键 Bug 修复、引擎兼容性更新及文档完善**：

*   **Bug 修复**
    *   **#6917**: 修复 V1 PPO trainer 资源释放延迟问题。原逻辑依赖 `__del__`/垃圾回收清理 tracking 和 dataloader workers，在 Ray/解释器销毁时执行过晚；现改为在训练循环的 `try/finally` 中显式、优雅地清理，确保正常退出时也能及时释放资源。
    *   **#6913**: 修复 fused-kernel 梯度追踪 Bug。此前同时启用 `use_liger` 和 `use_fused_kernel` 会产生静默的错误结果，根因是内部 `.flatten(0,1)` 操作破坏了梯度追踪，本 PR 予以修正。

*   **功能与优化**
    *   **#6916**: 更新 TorchtitanEngine，适配最新的 torchtitan nightly API，保证兼容性，并新增了端到端（e2e）测试脚本。
    *   **#6914**: 在 `fully_async_opd` 的 teacher forward（前向推理）和 student train（训练）流程中添加了 `fusenode` 节点，提升运行效率。

*   **文档更新**
    *   **#6915**: 补充 SGLang FP8 ignored layers 的使用文档。说明如何在 rollout 权重量化时跳过指定模块，并给出了 `SGLANG_FP8_IGNORED_LAYERS` 的逗号分隔配置及 `quantization_config.ignored_layers` 的正则匹配（`re:`）示例（跟进 #6906）。

### 🐛 Issue
本次输入数据中**无** Issue 动态。

### 🚀 Release
本次输入数据中**无** Release 动态。

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
