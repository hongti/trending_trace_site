# verl-project/verl — 动态追踪

> 生成时间: 2026-07-02 14:08 UTC

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🚀 Pull Request (PR) 活动

近期 PR 主要集中在**关键 Bug 修复、引擎兼容性更新及文档完善**：

*   **重要修复**：
    *   **Trainer 资源清理修复 (#6917)**：修复了 V1 PPO trainer 在训练结束时仅依赖 `__del__`/垃圾回收来清理 tracking 和 dataloader workers 导致的延迟问题。现改为在训练循环中使用 `try/finally` 显式、优雅地终结资源，确保正常或异常退出时都能及时清理。
    *   **Fused-kernel 梯度跟踪 Bug 修复 (#6913)**：修复了同时使用 `use_liger` 和 `use_fused_kernel` 标志时，内部 `.flatten(0,1)` 操作会导致梯度静默损坏的问题，保障了训练结果的正确性。
    *   **TorchtitanEngine 兼容性更新 (#6916)**：适配了最新版的 torchtitan nightly API，修复了兼容性问题，并新增了端到端（e2e）测试脚本以确保 FSDP 引擎的对比验证。

*   **功能增强**：
    *   **Fully async OPD 优化 (#6914)**：在 `fully_async_opd` 模式下的 teacher forward 和 student train 流程中添加了 fuse node（算子融合节点），以进一步提升性能。

*   **文档更新**：
    *   **SGLang FP8 忽略层用法补充 (#6915)**：新增文档说明如何在 SGLang FP8 rollout 权重量化时跳过指定模块，提供了 `SGLANG_FP8_IGNORED_LAYERS` 及 `quantization_config.ignored_layers` 正则匹配的示例。
    *   **Ascend 启动文档修正 (#6918)**：更新了针对 Ascend 硬件的 `quick_start.rst` 快速入门指南。

---

### 🐛 Issue 活动
近期暂无列入摘要的 Issue 动态。

---

### 📦 Release 活动
近期暂无新版本 Release 发布。

---

## 🔀 Pull Requests

### #6918 — [[doc] fix: Update ascend quick_start.rst](https://github.com/verl-project/verl/pull/6918)
- **作者**: yyyy2000  **时间**: 2026-07-02 13:55 UTC
- **摘要**: ### What does this PR do?  Update ascend quick_start.rst  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`,…

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
