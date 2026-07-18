# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-18 09:04 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库最近的动态摘要：

### 🚀 Release (发布)
近期暂无新的版本发布动态。

### 🐛 Issue (问题)
近期暂无公开的 Issue 动态。

### 🔀 Pull Request (拉取请求)
近期 PR 活动主要集中在**新特性支持、性能优化、Bug 修复及分支代码清理**，以下是核心亮点：

**1. 新特性与性能优化**
*   **Qwen3.5 LoRA 支持** (#12295)：为 Qwen3.5 系列（hybrid-dense + shared-experts MoE）启用并稳定了 LoRA 功能，通过隔离 base-model 与 adapter 的 ACL 编译图来避免冲突，兼容 eager 和 full-graph 模式。
*   **DSSpark FULL Graph 支持** (#12296)：为 DSpark 投机执行草稿模型增加了正确的 ACL/CUDA Graph 支持，包括 `FULL_DECODE_ONLY` 执行模式。
*   **跨 DP 通信优化** (#12292)：针对数据并行部署下的 Decoder Serving 场景，引入了 NonBSP 负载感知调度，以优化跨 Rank 的工作负载并减少通信气泡。

**2. Bug 修复**
*   **Qwen3.5 GDN 正确性修复** (#12297)：修复了 v0.23.0 版本中，Qwen3.5 GDN 在 `FULL_DECODE_ONLY` 模式下因基于形状的图分发导致的正确性边界问题，通过绕过仅解码的重放逻辑解决。

**3. 代码重构与分支 Cherry-pick**
*   **v0.24.0rc 分支清理**：移除了过时的 parser 兼容性补丁及其注册和文档 (#12294)；移除了陈旧的 MiniMax-M2 tool-call parser 与用量计算补丁，并替换为新的 patch 机制 (#12293)。

**4. 文档与 CI 调整**
*   **文档修复**：修复了 glm5.2 1M PD 脚本的文档错误，并同步 cherry-pick 至 v0.23.0 分支 (#12289, #12290)。
*   **CI 维护**：增加了 MiniMax 模型的 CI 测试 (#12291)；修改了 a3 560t 环境的 PVC 名称 (#12288)。

---

## 🔀 Pull Requests

### #12297 — [[v0.23.0][BugFix][Graph] Bypass decode-only replay for GDN prefill](https://github.com/vllm-project/vllm-ascend/pull/12297)
- **作者**: maoxx241  **时间**: 2026-07-18 05:08 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This is a minimal `releases/v0.23.0` fix for a Qwen3.5 GDN correctness boundary under `FULL_DECODE_ONLY`.  Graph dispatch is shape based. A final one-token chunk can therefore look like uniform decode even when `CommonAttentionMetadata.is_prefilling` still id…

### #12296 — [Dspark FULL Graph support](https://github.com/vllm-project/vllm-ascend/pull/12296)
- **作者**: StanislavII  **时间**: 2026-07-18 00:32 CST
- **摘要**: # Add DSpark FULL Decode Graph Support on vLLM Ascend  ## Summary  This PR adds correct ACL/CUDA graph support for the DSpark speculative drafter, including `FULL_DECODE_ONLY` execution.  DSpark differs from existing EAGLE/DFlash proposers because the target and draft forwards use different per-requ…

### #12295 — [[LoRA] Qwen3.5 hybrid-dense + shared-experts MoE LoRA with isolated ACL compile graphs](https://github.com/vllm-project/vllm-ascend/pull/12295)
- **作者**: Liuchenbing-2026  **时间**: 2026-07-17 23:50 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Enable and stabilize LoRA on the Qwen3.5 family in vllm-ascend across both eager and full-graph modes, and isolate base-model / adapter ACL compile graphs to avoid replay explosions when the adapter set changes.  The PR is three stacked commits:  1. **`[LoRA]…

### #12294 — [[Cherry-pick][releases/v0.24.0rc][Refactor][Patch] Remove obsolete parser patches (from #11453)](https://github.com/vllm-project/vllm-ascend/pull/12294)
- **作者**: QwertyJack  **时间**: 2026-07-17 21:46 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Draft cherry-pick of #11453 onto `releases/v0.24.0rc`.  This PR removes these compatibility patches, their registrations, patch documentation, and dedicated unit tests:  - `patch_glm47_tool_call_parser.py` - `patch_glm_tool_call_streaming.py` - `patch_tool_ch…

### #12293 — [[Cherry-pick][releases/v0.24.0rc][Refactor][Patch] Replace MiniMax parser backports (from #11384)](https://github.com/vllm-project/vllm-ascend/pull/12293)
- **作者**: QwertyJack  **时间**: 2026-07-17 21:43 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Cherry-pick of #11384 onto `releases/v0.24.0rc`.  This PR:  - removes the stale MiniMax-M2 tool-call parser and usage-accounting patches - adds `patch_parser_reasoning_usage.py` as a backport of https://github.com/vllm-project/vllm/pull/45802 - preserves `com…

### #12292 — [[Feature] Cross-DP Communication Bubble Optimization for Decoder Serving (NonBSP)](https://github.com/vllm-project/vllm-ascend/pull/12292)
- **作者**: Ydr-pku  **时间**: 2026-07-17 21:15 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  This PR introduces NonBSP load-aware scheduling for data-parallel vLLM Ascend deployments.  The main changes include:  - Add a NonBSP scheduler with cross-rank workload balancing. - Run load balancing before scheduling so modifications are applied   to the cu…

### #12291 — [[CI] minimax](https://github.com/vllm-project/vllm-ascend/pull/12291)
- **作者**: chen-commits  **时间**: 2026-07-17 20:39 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389

### #12290 — [[Cherry-pick][v0.23.0][Doc] Fix glm5.2 1M PD script](https://github.com/vllm-project/vllm-ascend/pull/12290)
- **作者**: nwpu-zxr  **时间**: 2026-07-17 20:39 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? Fix glm5.2 1M PD script. Cherry pick from #12289   ### Does this PR introduce _any_ user-facing change? No.  ### How was this patch tested? By CI.  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c3…

### #12289 — [[Doc] Fix glm5.2 1M PD script](https://github.com/vllm-project/vllm-ascend/pull/12289)
- **作者**: nwpu-zxr  **时间**: 2026-07-17 20:32 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? Fix glm5.2 1M PD script.  ### Does this PR introduce _any_ user-facing change? No.  ### How was this patch tested? By CI.  - vLLM version: v0.24.0 - vLLM main: https://github.com/vllm-project/vllm/commit/85c09e9885e346ea1612da30ebff5a75f67d2350

### #12288 — [[CI] change pvc name for a3 560t](https://github.com/vllm-project/vllm-ascend/pull/12288)
- **作者**: zhangxinyuehfad  **时间**: 2026-07-17 19:14 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? change pvc name for a3 560t ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.24.0 - vLLM main: https://github.com/vllm-project/vllm/commit/85c09e9885e346ea1612da30ebff5a75f67d2350
