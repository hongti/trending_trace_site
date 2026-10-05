# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-05 14:06 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 📌 Issue（议题）
本阶段无相关 Issue 动态。

### 🚀 Pull Request（拉取请求）
近期 PR 活动主要集中在一个 Bug 修复和一系列 CI 诊断测试上：

1. **Bug 修复：安装失败状态码修复 (#17888)**
   - **作者**: li-lizhe
   - **变更亮点**: 修复了 `csrc/build_batch_invariant_ops.sh` 脚本中的一个缺陷。此前该脚本在遇到下载或安装失败时仍会无条件以状态码 `0` 退出，导致错误被掩盖。现修正为在操作失败时以非零状态码退出，确保 CI 能正确捕获并阻断流程。

2. **CI 诊断：性能与准确率回归排查 (一组诊断 PR)**
   - **作者**: voidvelocity
   - **活动概述**: 提交了多个标记为 `[CI]` 或 `[don't merge]` 的临时诊断 PR（#17885, #17886, #17887, #17889 - #17894）。
   - **目的**: 这些 PR **请勿合并**。它们基于特定的历史提交创建空提交，主要用于触发 CI 流程，通过二分法来定位最近出现的性能回退和准确率回归问题。

### 📦 Release（版本发布）
本阶段无新的版本发布。

---

## 🔀 Pull Requests

### #17894 — [[CI] Retest good historical commit 37ca05f46](https://github.com/vllm-project/vllm-ascend/pull/17894)
- **作者**: voidvelocity  **时间**: 2026-10-05 14:05 CST
- **摘要**: Temporary diagnostic PR for good-case performance confirmation.  The branch contains one empty signed-off commit on top of 37ca05f46ff51d8e06eaea09c36915d812ac6464; its file tree exactly matches that target commit.  CI case: /nightly Qwen3-30B-QuaRot --a3-560t. - vLLM main: https://github.com/vllm-p…

### #17893 — [[CI] Test historical commit 995977208 -9](https://github.com/vllm-project/vllm-ascend/pull/17893)
- **作者**: voidvelocity  **时间**: 2026-10-05 13:45 CST
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: Temporary diagnostic PR for performance regression localization.  The branch contains one empty signed-off commit on top of 995977208dc47edc4c9b6e575d1535b191de8bfa; its file tree exactly matches that target commit.  CI case: /nightly Qwen3-30B-QuaRot --a3-560t. - vLLM main: https://github.com/vllm-…

### #17892 — [[CI] Test historical commit 31608b3f5](https://github.com/vllm-project/vllm-ascend/pull/17892)
- **作者**: voidvelocity  **时间**: 2026-10-05 13:44 CST
- **摘要**: Temporary diagnostic PR for performance regression localization.  The branch contains one empty signed-off commit on top of 31608b3f535d1dbcb244e0eea6f1363621c79139; its file tree exactly matches that target commit.  CI case: /nightly Qwen3-30B-QuaRot --a3-560t. - vLLM main: https://github.com/vllm-…

### #17891 — [[CI] Test historical commit 2280fb330](https://github.com/vllm-project/vllm-ascend/pull/17891)
- **作者**: voidvelocity  **时间**: 2026-10-05 13:44 CST
- **摘要**: Temporary diagnostic PR for performance regression localization.  The branch contains one empty signed-off commit on top of 2280fb330507ccbfc3caa3704f5a5b06dc093fe3; its file tree exactly matches that target commit.  CI case: /nightly Qwen3-30B-QuaRot --a3-560t. - vLLM main: https://github.com/vllm-…

### #17890 — [[CI] Test historical commit 7b7511a80](https://github.com/vllm-project/vllm-ascend/pull/17890)
- **作者**: voidvelocity  **时间**: 2026-10-05 13:44 CST
- **摘要**: Temporary diagnostic PR for performance regression localization.  The branch contains one empty signed-off commit on top of 7b7511a80f32b571335f7d4a5bea858f52624429; its file tree exactly matches that target commit.  CI case: /nightly Qwen3-30B-QuaRot --a3-560t. - vLLM main: https://github.com/vllm-…

### #17889 — [[CI] Test historical commit c22d832e2 -1](https://github.com/vllm-project/vllm-ascend/pull/17889)
- **作者**: voidvelocity  **时间**: 2026-10-05 13:44 CST
- **摘要**: Temporary diagnostic PR for performance regression localization.  The branch contains one empty signed-off commit on top of c22d832e28ef6f4345ca7573086d787946d6cf21; its file tree exactly matches that target commit.  CI case: /nightly Qwen3-30B-QuaRot --a3-560t. - vLLM main: https://github.com/vllm-…

### #17888 — [[BugFix] Exit non-zero when batch_invariant ops installation fails](https://github.com/vllm-project/vllm-ascend/pull/17888)
- **作者**: li-lizhe  **时间**: 2026-10-05 13:01 CST
- **摘要**: ### What this PR does  `csrc/build_batch_invariant_ops.sh` logs every download/install failure but unconditionally reaches the end of the script and exits `0`. Each failing command is wrapped in an `if`/`||` context, so `set -euo pipefail` never fires and the failure is never propagated. The caller …

### #17887 — [[don't merge] add comment to trigger pr](https://github.com/vllm-project/vllm-ascend/pull/17887)
- **作者**: voidvelocity  **时间**: 2026-10-05 11:50 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/ced6857afa0ea7b2e3f0846a62e1394e90f15607

### #17886 — [[CI] Bisect group 10/10: cumulative 37ca05f..b64b4d7 (commits 1-54)](https://github.com/vllm-project/vllm-ascend/pull/17886)
- **作者**: voidvelocity  **时间**: 2026-10-05 00:32 CST
- **标签**: documentation, module:ops, module:core, module:quantization
- **摘要**: ## Summary - **DO NOT MERGE** — diagnostic bisect PR (group 10/10) for the accuracy-regression investigation. - Head branch points at the cumulative state `37ca05f..b64b4d7` (commits 1-54 of the 57-commit range 37ca05f..b64b4d7) plus one empty checkpoint commit. - CI checks out `pull_request.head.sh…

### #17885 — [[CI] Bisect group 9/10: cumulative 37ca05f..6f74f9a (commits 1-48)](https://github.com/vllm-project/vllm-ascend/pull/17885)
- **作者**: voidvelocity  **时间**: 2026-10-05 00:32 CST
- **标签**: documentation, module:ops, module:core, module:quantization
- **摘要**: ## Summary - **DO NOT MERGE** — diagnostic bisect PR (group 9/10) for the accuracy-regression investigation. - Head branch points at the cumulative state `37ca05f..6f74f9a` (commits 1-48 of the 57-commit range 37ca05f..b64b4d7) plus one empty checkpoint commit. - CI checks out `pull_request.head.sha…
