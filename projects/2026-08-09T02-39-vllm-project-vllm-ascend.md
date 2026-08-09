# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-09 10:39 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的简洁摘要：

### 📘 Issue (问题)
*   **异步 MTP 投机解码导致输出乱码 (#13863)**：在异步调度和 MTP 投机解码场景下，Qwen3.5/Qwen3.6 混合模型会间歇性出现 token 重复、丢失或乱码现象。原因与 batch condense 后复用过期的 accepted-token 计数有关。
*   **CI 自动化流程失败 (#13862)**：`main2main` 自动化流程未完成所有计划步骤即停止，状态显示为 `failed`，需要人工审查介入。

### 📙 Pull Request (代码合并)
**核心 Bug 修复：**
*   **保留异步 accepted-token 快照 (#13864, #13865)**：针对上述 Issue #13863 的修复。在异步调度压缩并重用 `InputBatch` 行之前，保留 accepted-token 的快照，防止下一次模型迭代重映射时出现错位。#13864 针对主分支，#13865 针对发布分支。
*   **修复 PD 分离架构下 EOS 停止时的 KV 参数返回 (#13870)**：修复了在 PD 部署中，当 P 端合成的单 token 请求遇到 EOS 导致请求被标记为 `FINISHED_STOPPED` 时，MooncakeHybridConnector 无法正确返回 KV 参数的问题。
*   **SFA DSA-CP 图填充序列长度保护 (#13861)**：将 DSA-CP 图填充序列长度的安全保护机制反向移植到 `releases/v0.23.0` 分支的 SFA v1 元数据构建器中。

**构建与打包：**
*   **Docker 构建 Rust 工具解析器 (#13860)**：为 `v0.24.0rc` 版本的 Docker 镜像构建并打包了 vLLM 新引入的 Rust 版 Python 工具解析器扩展。

**CI 与测试：**
*   **每周例行测试 (#13869, #13868, #13866)**：针对 v0.23.0 和 v0.26.0 版本运行每周例行测试用例。
*   **CI 工作流参数修改 (#13867)**：修改了 `workflow_dispatch` 中的 `vllm_ascend_branch` 参数。
*   **清理测试环境变量 (#13859)**：从端到端（e2e）每周测试配置中移除了不再需要的 PD 分离环境变量（如 `ASCEND_ENABLE_USE_FABRIC_MEM` 和 `HCCL_INTRA_ROCE_ENABLE`）。

### 📗 Release (发布)
*   近期无新版本发布记录。

---

## 🐛 Issues

### #13863 — [[Bug]: Async MTP can reuse stale accepted-token counts after batch condense](https://github.com/vllm-project/vllm-ascend/issues/13863)
- **作者**: QwertyJack  **时间**: 2026-08-09 09:00 CST
- **标签**: advanced-features, mtp/speculative-decode
- **摘要**: ### Your current environment  <details> <summary>Environment used for the controlled reproduction</summary>  ```text Hardware: Ascend A3, 2 NPUs (TP2) Model: /models/Qwen3.6-35B-A3B-w8a8 (real weights) vLLM: 0.23.0 vLLM-Ascend: 0.23.0rc1 plus #12255 torch-npu: 2.10.0.post2 CANN: 9.0.1 ```  </details…

### #13862 — [[main2main] main2main manual review required ()](https://github.com/vllm-project/vllm-ascend/issues/13862)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-09 00:03 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR:  - Commit range: `0351e9aa1fdf1a51329d1906881528dfe61fc88e`...`` - Status: `failed`  ## Final Summary  main2main adaptation failed — no steps completed.

## 🔀 Pull Requests

### #13870 — [[BugFix][P/D] Return KV params when P stops on EOS](https://github.com/vllm-project/vllm-ascend/pull/13870)
- **作者**: hukongyi  **时间**: 2026-08-09 10:16 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  For the synthetic P-side one-token request used by PD deployment, an EOS token can cause vLLM to mark the request as `FINISHED_STOPPED`. MooncakeHybridConnector previously generated KV transfer parameters only for `FINISHED_LENGTH_CAPPED`, so the PD proxy cou…

### #13869 — [[CI]weekly temp test](https://github.com/vllm-project/vllm-ascend/pull/13869)
- **作者**: guxin108  **时间**: 2026-08-09 09:57 CST
- **摘要**: ### What this PR does / why we need it? we test weekly cases   ### Does this PR introduce _any_ user-facing change? no  ### How was this patch tested? run the cases  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389

### #13868 — [[CI] weekly temp test](https://github.com/vllm-project/vllm-ascend/pull/13868)
- **作者**: guxin108  **时间**: 2026-08-09 09:56 CST
- **标签**: documentation, ci/build, module:tests, module:ops, module:core, module:quantization, module:tools
- **摘要**: ### What this PR does / why we need it? we test weekly cases   ### Does this PR introduce _any_ user-facing change? no  ### How was this patch tested? run the cases  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/0351e9aa1fdf1a51329d1906881528dfe61fc88e

### #13867 — [[CI] Modify vllm_ascend_branch for workflow_dispatch](https://github.com/vllm-project/vllm-ascend/pull/13867)
- **作者**: weixinAc  **时间**: 2026-08-09 09:53 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it?  Modify vllm_ascend_branch for workflow_dispatch  ### Does this PR introduce _any_ user-facing change? NO ### How was this patch tested? by the running the test  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac76…

### #13866 — [[CI] weekly  temp test](https://github.com/vllm-project/vllm-ascend/pull/13866)
- **作者**: guxin108  **时间**: 2026-08-09 09:42 CST
- **标签**: documentation, ci/build, module:tests, module:ops, module:core, module:quantization, module:tools
- **摘要**: ### What this PR does / why we need it? we test weekly cases  ### Does this PR introduce _any_ user-facing change? no  ### How was this patch tested? run the cases weekly  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/0351e9aa1fdf1a51329d1906881528dfe61fc88e

### #13865 — [[v0.23.0][BugFix][Worker] Preserve async accepted-token snapshot](https://github.com/vllm-project/vllm-ascend/pull/13865)
- **作者**: QwertyJack  **时间**: 2026-08-09 09:12 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Fixes #13863. Release-branch counterpart of #13864.  Async scheduling can condense and reuse `InputBatch` rows before the next model iteration remaps accepted-token counts through `prev_positions`. Because the previous counts are currently stored in those mut…

### #13864 — [[BugFix][Worker] Preserve async accepted-token snapshot](https://github.com/vllm-project/vllm-ascend/pull/13864)
- **作者**: QwertyJack  **时间**: 2026-08-09 09:11 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Fixes #13863.  Async scheduling can condense and reuse `InputBatch` rows before the next model iteration remaps accepted-token counts through `prev_positions`. Because the previous counts are currently stored in those mutable rows, a new request can overwrite…

### #13861 — [[BugFix][0.23.0] Guard SFA DSA-CP graph padding sequence lengths](https://github.com/vllm-project/vllm-ascend/pull/13861)
- **作者**: pisceskkk  **时间**: 2026-08-08 23:54 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it?  This adds the DSA-CP graph-padding sequence-length safeguard to the SFA v1 metadata builder on the `releases/v0.23.0` branch. It is the v0.23.0 counterpart of the fix proposed for main in #13779.  In graph mode, padded request slots can keep fixed query posit…

### #13860 — [[BugFix][Docker][v0.24.0rc] Build and package vLLM Rust tool parser artifacts](https://github.com/vllm-project/vllm-ascend/pull/13860)
- **作者**: lvhua6352  **时间**: 2026-08-08 23:26 CST
- **摘要**: ### What this PR does / why we need it?    vLLM v0.24.0 includes the Rust-backed Python tool parser extension introduced by [vllm-project/vllm#44624](https://github.com/vllm-project/vllm/pull/44624).    The vLLM-Ascend Dockerfiles clone and install vLLM in editable mode, but they do not build or cop…

### #13859 — [[Test][Misc] Remove ASCEND_BUFFER_POOL replacements from e2e](https://github.com/vllm-project/vllm-ascend/pull/13859)
- **作者**: lizy124  **时间**: 2026-08-08 23:11 CST
- **标签**: documentation, module:tests, ready
- **摘要**: Remove ASCEND_ENABLE_USE_FABRIC_MEM and HCCL_INTRA_ROCE_ENABLE lines that were added as replacements for ASCEND_BUFFER_POOL in e2e weekly test configurations. These PD separation configs do not need the replacement variable; the original ASCEND_BUFFER_POOL is simply removed.  ### What this PR does /…
