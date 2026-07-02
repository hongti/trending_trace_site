# verl-project/verl — 动态追踪

> 生成时间: 2026-07-02 12:24 UTC

## AI 总结

## verl-project/verl 近期动态摘要

---

### 🔄 Pull Request（5 项）

| PR | 核心内容 |
|---|---|
| **#6917** | **[trainer] 修复：优雅终止 tracking 和 dataloader worker** — V1 PPO Trainer 原本仅依赖 `__del__`/GC 清理资源，在 Ray/解释器关闭时执行太晚。现改为 `try/finally` 显式清理，确保正常退出与异常时都能及时释放。 |
| **#6916** | **[trainer] 修复：更新 TorchtitanEngine** — 适配最新 torchtitan nightly 的 API 兼容性，并新增端到端测试脚本，附带与 FSDP engine 的对比验证。 |
| **#6915** | **[docs] 新增 SGLang FP8 ignored layers 用法文档** — 说明如何在 SGLang FP8 rollout 权重量化时跳过指定模块，涵盖 `SGLANG_FP8_IGNORED_LAYERS` 环境变量、逗号分隔配置及 `re:` 正则模式示例。为 #6906 的后续文档补充。 |
| **#6914** | **[trainer] fully_async_opd 中添加 FuseNode** — 在 teacher forward 和 student train 流程中加入融合节点，优化异步 OPD 训练的执行效率。 |
| **#6913** | **[trainer] 修复 fused-kernel 梯度追踪 bug** — 当同时启用 `use_liger` + `use_fused_kernel` 时，内部 `.flatten(0,1)` 操作导致梯度结果被静默破坏，本次修复纠正了该追踪逻辑。 |

> **📌 重点关注：** 本轮 PR 以 **稳定性修复** 为主——#6917 修复资源泄露隐患、#6913 修复静默梯度错误；同时 #6916 推进了 torchtitan nightly 兼容性，#6914 引入异步训练融合优化。

---

### 🐛 Issue

本期无新增 Issue 动态。

---

### 🚀 Release

本期无新版本发布。

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
