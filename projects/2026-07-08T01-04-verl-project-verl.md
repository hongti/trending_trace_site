# verl-project/verl — 动态追踪

> 生成时间: 2026-07-08 09:04 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 🛠 Issue 概要
*   **NCCL 初始化挂起（竞态条件）** [#6967]：在 `fully_async` 模式下，首个 NCCL checkpoint-engine 组初始化时发生挂起。该问题出现在生成前的首次权重同步阶段，且在无工具的单轮对话中即可复现，作者认为这与已关闭的 #5321 是不同的问题。

### 🚀 PR 概要
本次 PR 主要围绕 **Qwen3.5 模型支持**、**Fully Async 修复** 及 **硬件适配**，已合并重复的同名 PR：

*   **模型与性能优化（Qwen 系列）**
    *   **Qwen3.5 适配与 v0.8.0 准备** [#6970 / #6969]：将 Qwen3.5 适配至 Docker 容器，新增 397B 参数训练脚本，并**修复了多模态模型在 GRPO 训练中 vLLM 推理缓存复用的 `mm_hash` 问题**。
    *   **Qwen3.5 MFU 估算** [#6965]：为 `qwen3_5` 模型新增 MFU FLOPs 估算逻辑，避免因 fallback 到未知模型估算导致性能指标（MFU）返回 0 的问题。
    *   **Qwen3 Next 80B NPU 稳定性** [#6972 / #6971]：调整 Qwen3 Next 80B FSDP Ascend GRPO 示例，缓解因引入 NPU expandable segment 支持而导致的 rollout 内存压力。

*   **Fully Async 修复**
    *   **Log-probs 丢失报错** [#6963]：增加 fail-loud 严格校验，当请求的 rollout log-probs 静默丢失时直接报错，避免训练出现隐蔽问题。
    *   **Rollout Segments Merge 修复** [#6962]：修复 `FullyAsyncLLMServerClient` 合并部分 rollout 片段时丢弃 `TokenOutput.extra_fields` 的问题，并强化了 `routed_experts` 的合并逻辑。

*   **文档与硬件支持**
    *   **Atlas 950DT A5 安装指南** [#6968 / #6966]：为文档新增 Atlas 950DT A5 硬件的软件版本、依赖及环境设置安装说明。

*   **其他**
    *   [#6964]：PR 描述暂缺具体内容，疑似常规代码提交或分支同步。

### 📦 Release 动态
*   本次记录虽无官方 Release 发布条目，但从 PR 动态可见，团队正在积极为 **Verl 0.8.0 版本**做准备（特别是 Qwen3.5 的 Docker 容器化适配及 397B 大规模脚本支持），预计新版本将对多模态推理缓存和大参数模型训练有重要改进。

---

## 🐛 Issues

### #6967 — [[fully_async] First NCCL checkpoint-engine group init hangs (timing race) — reproduces single-turn without tools](https://github.com/verl-project/verl/issues/6967)
- **作者**: chengcuiping  **时间**: 2026-07-07 16:51 CST
- **标签**: bug
- **摘要**: ### System Info   **Possibly related to the (now closed) #5321 — but our hang is at the first weight sync before any generation, and reproduces single-turn without tools, so we believe it is a distinct/lower-level problem.**  We root-caused this during a profiling effort on `recipe/fully_async_polic…

## 🔀 Pull Requests

### #6972 — [[trainer] fix: stabilize Qwen3 Next 80B FSDP rollout on NPU](https://github.com/verl-project/verl/pull/6972)
- **作者**: zjchenn  **时间**: 2026-07-07 22:45 CST
- **摘要**: ### What does this PR do?  Adjust the Qwen3 Next 80B FSDP Ascend GRPO example to reduce rollout memory pressure after verl-project/verl#6346.  This is the `main` branch counterpart of verl-project/verl#6971.  That PR added NPU expandable segment support. In this example, the `update_weights` stage c…

### #6971 — [[trainer] fix: stabilize Qwen3 Next 80B FSDP rollout on NPU](https://github.com/verl-project/verl/pull/6971)
- **作者**: zjchenn  **时间**: 2026-07-07 22:27 CST
- **摘要**: ### What does this PR do?  Adjust the Qwen3 Next 80B FSDP Ascend GRPO example to reduce rollout memory pressure after verl-project/verl#6346.  That PR added NPU expandable segment support. In this example, the `update_weights` stage can now see a higher peak NPU memory footprint, so this PR makes th…

### #6970 — [[env,model] Adapt qwen3.5 to Docker containers for Verl release 0.8.0 and add 397B scripts](https://github.com/verl-project/verl/pull/6970)
- **作者**: ruanhao566  **时间**: 2026-07-07 19:56 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Adapt qwen3.5 to Docker containers for Verl release 0.8.0 and add 397B scripts. Fix mm_hash issue in vLLM inference cache reuse during GRPO training for multimodal models.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ...…

### #6969 — [[env,model] Adapt qwen3.5 to Docker containers for Verl release 0.8.0 and add 397B scripts](https://github.com/verl-project/verl/pull/6969)
- **作者**: ruanhao566  **时间**: 2026-07-07 19:54 CST
- **摘要**: ### What does this PR do?  Adapt qwen3.5 to Docker containers for Verl release 0.8.0 and add 397B scripts. Fix mm_hash issue in vLLM inference cache reuse during GRPO training for multimodal models.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ...…

### #6968 — [[doc] feat: add installation instructions for Atlas 950DT A5](https://github.com/verl-project/verl/pull/6968)
- **作者**: fh188  **时间**: 2026-07-07 17:02 CST
- **摘要**: ### What does this PR do?  This PR adds installation instructions for Atlas 950DT A5 to the documentation, including the required software versions, dependencies, and environment setup steps. It helps users set up the development environment for the Atlas 950DT A5 platform more easily.  ### Checklis…

### #6966 — [[doc] feat: add installation instructions for Atlas 950DT A5](https://github.com/verl-project/verl/pull/6966)
- **作者**: fh188  **时间**: 2026-07-07 15:57 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  This PR adds installation instructions for Atlas 950DT A5 to the documentation, including the required software versions, dependencies, and environment setup steps. It helps users set up the development environment for the Atlas 950DT A5 platform more easily.  ### Checklis…

### #6965 — [[perf, model] feat: add MFU flops estimation for Qwen3.5 (qwen3_5)](https://github.com/verl-project/verl/pull/6965)
- **作者**: ldemon2333  **时间**: 2026-07-07 15:36 CST
- **摘要**: ### What does this PR do?  Adds MFU FLOPs estimation for **Qwen3.5** (`model_type: "qwen3_5"`). Without it, `qwen3_5` falls through to `_estimate_unknown_flops`, which returns `0`, so `perf/mfu/*` is always zero for Qwen3.5 training runs.  Qwen3.5 differs from the models already covered by `flops_co…

### #6964 — [Zchen](https://github.com/verl-project/verl/pull/6964)
- **作者**: DoYangTan  **时间**: 2026-07-07 15:33 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #6963 — [[fully_async, trainer] fix: fail loud when requested rollout log-probs are missing](https://github.com/verl-project/verl/pull/6963)
- **作者**: rawsh  **时间**: 2026-07-07 15:12 CST
- **摘要**: ### What does this PR do?  Adds fail-loud invariants so that when rollout log-probs are **requested** (`rollout.calculate_log_probs` or `actor.use_rollout_log_probs`), a batch that silently loses `rollout_log_probs` raises instead of quietly degrading importance-correction / debug metrics to recompu…

### #6962 — [[fully_async, rollout] fix: preserve extra_fields when merging partial rollout segments](https://github.com/verl-project/verl/pull/6962)
- **作者**: rawsh  **时间**: 2026-07-07 15:12 CST
- **摘要**: ### What does this PR do?  Fixes `FullyAsyncLLMServerClient` partial-rollout merge (`verl/workers/rollout/llm_server.py`) dropping `TokenOutput.extra_fields`, and hardens the `routed_experts` merge.  **Problem / root cause.** When a request is resumed across engine versions (partial rollout), the cl…
