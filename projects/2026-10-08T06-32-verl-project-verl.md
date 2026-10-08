# verl-project/verl — 动态追踪

> 生成时间: 2026-10-08 14:32 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📌 Issue
本期未提供相关 Issue 动态。

### 🚀 Release
本期未提供相关 Release 动态。

### 🔧 Pull Request
本期共有 8 个 PR，主要涵盖新硬件支持、量化检查优化、多轮 Agent 调度及依赖升级等重要变更：

**1. 新硬件支持与文档**
*   **新增 TPU 支持 (#8138)**：新增在 64 个 TPU 上运行 Qwen3-4B-Base 的 GRPO 启动器，并通过 Kueue 实现自动拉起 Ray 集群。
*   **修复昇腾安装文档 (#8139)**：修正了 Ascend 硬件的安装说明文档。

**2. 量化与权重同步审计 (PR #8130, #8131, #8132)**
这部分是一组连续的 MXFP8 量化功能增强（5/7 至 7/7）：
*   **SGLang MXFP8 Rollout (#8130)**：在 SGLang 上新增 MXFP8 rollout 支持，通过 refit loader 在训练端使用 TE 量化器处理好权重后再流式传输给采样器。
*   **内核布局自检 (#8131)**：每次 MXFP8 权重同步后，自动运行引擎内核并与 BF16 参考值对比，以捕获过期或布局错误的权重。
*   **量化层一致性审计 (#8132)**：在训练步首次权重同步后，审计 Learner 和 Rollout 引擎是否按参数名量化了相同的层，防止不一致。

**3. 多轮 Agent 推理调度优化 (PR #8135, #8136)**
*   **恢复感知调度 (Resume-aware Scheduling)**：为多轮 Agent rollout 新增可选的调度策略，将模型请求分类为 `fresh`、`continuation` 或 `retry`，以优先恢复被中断的 Agent 请求，支持 vLLM 和 SGLang，提升多轮推理性能。

**4. 性能重构与依赖更新**
*   **性能优化 (#8137)**：重构 Karmarkar-Karp 算法，在构造后缓存 `State.spread` 并在堆重新插入前刷新，消除了重复的堆比较计算。
*   **依赖升级 (#8133, #8134)**：将 `sglang` 从 0.5.20 升至 0.5.21，`trl` 从 0.27.0 升至 1.14.1。

---

## 🔀 Pull Requests

### #8139 — [[doc] fix: Fix Ascend install doc](https://github.com/verl-project/verl/pull/8139)
- **作者**: jackon-creator  **时间**: 2026-10-08 12:45 CST
- **摘要**: Fix Ascend install doc

### #8138 — [[doc, hardware, worker, trainer] feat: add GRPO support for TPU](https://github.com/verl-project/verl/pull/8138)
- **作者**: jialei777  **时间**: 2026-10-08 11:39 CST
- **摘要**: ### What does this PR do?  Adds a Qwen3-4B-Base GRPO launcher for 250 steps on 64 TPUs, with automatic Ray cluster startup through Kueue.  Complements [verl-hardware-plugin#38](https://github.com/verl-project/verl-hardware-plugin/pull/38) with FP32 trainer log-probabilities and TorchTitan configurat…

### #8137 — [[training_utils, perf] refactor: cache Karmarkar-Karp state spread](https://github.com/verl-project/verl/pull/8137)
- **作者**: IsaacLi74  **时间**: 2026-10-08 07:09 CST
- **摘要**: ### What does this PR do?  Cache `State.spread` after construction and refresh it after each merge, before heap reinsertion. This removes repeated heap-comparison calculations while preserving accumulation order, numeric types and tie-breaking. No public API changes.  Extends `check_license.py` to a…

### #8136 — [[rollout, vllm, sglang, perf] feat: prioritize resumed agent requests](https://github.com/verl-project/verl/pull/8136)
- **作者**: jiacao-amd  **时间**: 2026-10-08 06:47 CST
- **摘要**: ### What does this PR do?  Adds opt-in, resume-aware scheduling for multi-turn agent rollouts:  - `ToolAgentLoop` classifies model requests as `fresh`, `continuation`, or   `retry` and records prompt, estimated uncached-token, expected-output, and   enqueue metadata. - A pluggable priority policy co…

### #8135 — [[rollout, vllm, perf] feat: prioritize resumed agent requests](https://github.com/verl-project/verl/pull/8135)
- **作者**: jiacao-amd  **时间**: 2026-10-08 05:11 CST
- **摘要**: ### What does this PR do?  Adds opt-in, resume-aware scheduling for multi-turn agent rollouts:  - `ToolAgentLoop` classifies model requests as `fresh`, `continuation`, or   `retry` and records prompt, estimated uncached-token, expected-output, and   enqueue metadata. - A pluggable priority policy co…

### #8134 — [build(deps-dev): bump trl from 0.27.0 to 1.14.1](https://github.com/verl-project/verl/pull/8134)
- **作者**: dependabot[bot]  **时间**: 2026-10-08 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [trl](https://github.com/huggingface/trl) from 0.27.0 to 1.14.1. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/trl/releases">trl's releases</a>.</em></p> <blockquote> <h2>v1.14.1</h2> <h2>What's Changed</h2> <ul> <li>Fix the server-mode …

### #8133 — [build(deps-dev): bump sglang from 0.5.20 to 0.5.21](https://github.com/verl-project/verl/pull/8133)
- **作者**: dependabot[bot]  **时间**: 2026-10-08 01:33 CST
- **标签**: dependencies, python
- **摘要**: Bumps [sglang](https://github.com/sgl-project/sglang) from 0.5.20 to 0.5.21. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/sgl-project/sglang/releases">sglang's releases</a>.</em></p> <blockquote> <h2>v0.5.21</h2> <h1>Highlights</h1> <p><em>779 PRs from 2…

### #8132 — [[7/7] [rollout] feat: audit that training and rollout quantize the same layers](https://github.com/verl-project/verl/pull/8132)
- **作者**: wengeezhang  **时间**: 2026-10-07 20:40 CST
- **摘要**: ### What does this PR do?  Audits, at the first weight sync after a training step, that **the learner and the rollout engine quantize the same layers**, per parameter name. On vLLM the audit asks the live engine. On SGLang, the refit loader refuses a sync whose dtypes disagree with the engine's para…

### #8131 — [[6/7] [rollout] feat: self-check MXFP8 kernel layouts after every weight sync](https://github.com/verl-project/verl/pull/8131)
- **作者**: wengeezhang  **时间**: 2026-10-07 20:40 CST
- **摘要**: ### What does this PR do?  After every MXFP8 weight sync, **runs the engine's own kernel on one layer and one MoE expert** and compares the result with a dequantized BF16 reference. A stale or mis-laid-out kernel layout raises instead of silently serving garbage. Works on both vLLM and SGLang.  This…

### #8130 — [[5/7] [rollout, sglang] feat: MXFP8 SGLang rollout with a refit loader](https://github.com/verl-project/verl/pull/8130)
- **作者**: wengeezhang  **时间**: 2026-10-07 20:39 CST
- **摘要**: ### What does this PR do?  Adds **MXFP8 rollout on SGLang**: - at every weight sync, the learner's own TE quantizer quantizes the weights on the trainer side before they are streamed, so the sampler serves the learner's FP8 weight grid; - a **refit loader** re-derives SGLang's kernel-specific scale …
