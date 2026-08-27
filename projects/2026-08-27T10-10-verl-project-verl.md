# verl-project/verl — 动态追踪

> 生成时间: 2026-08-27 18:10 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📌 Issue
- **[RFC] V1 Trainer 崩溃可恢复重放缓冲区** (#7579)：提出基于 `TransferQueue` 为 V1 Trainer 实现崩溃容错重放缓冲区的设计提案，并提供了原型分支及复现性能开销的测试集。
- **Qwen3.5-35B-A3B GRPO 训练失败** (#7578)：用户反馈在特定硬件环境（910B2 & 25.2.3HDK）下，运行 Qwen3.5-35B-A3B GRPO 训练脚本时遇到报错。

---

### 🔧 Pull Request
**✨ 新特性与性能优化**
- **支持加权多输出 Agent-loop 轨迹** (#7580)：新增一等公民支持，允许将单条逻辑轨迹展开为多个有序训练段，并支持为各段分配权重，极大增强了智能体循环训练的灵活性。
- **支持 DeepSeek V4 QAT bf16 伪量化训练** (#7577)：集成 VeOmni 项目的最新进展，引入 DeepSeek V4 量化感知训练（QAT）的 bf16 伪量化支持。
- **跳过无用的旧策略熵计算** (#7576)：性能优化，当不需要熵诊断或熵正则化时，跳过旧策略重算期间的 token 级熵计算与传输，有效降低开销。

**🐛 Bug 修复**
- **修复分块熵计算配置失效问题** (#7582)：修复了当 `remove_padding` 关闭时，低内存分块配置 `entropy_from_logits_with_chunking` 未被正确遵循的问题。
- **修复 RLOO 算法单样本组零路径问题** (#7581)：确保单样本组不会进入 `rloo_vectorized` 的零路径，从而保证其与 `rloo` 计算结果的等效性。

**🧪 测试与依赖更新**
- **增加动态资源调度测试** (#7583)：为 fully_async 模式下的动态资源调度增加 CPU 测试，覆盖固定比例策略及混合副本生命周期排序。
- **依赖项升级** (#7572~#7575)：批量升级开发依赖，包括 `cupy-cuda13x` (至 14.2.0)、`flash-attn` (至 2.8.3.post1)、`sglang` (至 0.5.18)，并放宽了 `pyarrow` 的版本上限限制（支持至 25.0.1）。

---

### 🚀 Release
- 近期无新版本发布。

---

## 🐛 Issues

### #7579 — [[RFC] A crash-durable replay buffer for the V1 trainer, behind TransferQueue](https://github.com/verl-project/verl/issues/7579)
- **作者**: savaresejeremy  **时间**: 2026-08-27 11:13 CST
- **摘要**: Prototype branch: https://github.com/savaresejeremy/verl/tree/feat/durable-replay-buffer. The accounting and overhead numbers in this RFC can be reproduced by running the harnesses in [`experiments/durable_buffer/`](https://github.com/savaresejeremy/verl/tree/feat/durable-replay-buffer/experiments/d…

### #7578 — [Qwen3.5-35B-A3B GRPO FAILED](https://github.com/verl-project/verl/issues/7578)
- **作者**: ChenBingqjl  **时间**: 2026-08-27 10:35 CST
- **标签**: bug
- **摘要**: ### System Info  I refer the follow link:https://github.com/verl-project/verl/blob/main/examples/grpo_trainer/run_qwen3_5_35b_megatron.sh when i start task, error occurred 910B2 & 25.2.3HDK  <img width="2526" height="1200" alt="Image" src="https://github.com/user-attachments/assets/f261744a-6515-438…

## 🔀 Pull Requests

### #7583 — [[fully_async] test: cover dynamic resource scheduling](https://github.com/verl-project/verl/pull/7583)
- **作者**: nuerxiati  **时间**: 2026-08-27 16:40 CST
- **摘要**: Register the documented fixed-ratio policy and protect policy decisions and hybrid replica lifecycle ordering with CPU tests.  ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  - Ad…

### #7582 — [[engine] fix: honor entropy_from_logits_with_chunking when remove_padding is off](https://github.com/verl-project/verl/pull/7582)
- **作者**: alanhuangyoo  **时间**: 2026-08-27 14:07 CST
- **摘要**: ### What does this PR do?  `entropy_from_logits_with_chunking` is a config flag (`actor_rollout_ref.actor.entropy_from_logits_with_chunking`) that swaps `entropy_from_logits` for a lower-memory chunked version. The engines resolve it once in `__init__`:  ```python if self.engine_config.entropy_from_…

### #7581 — [[algo] fix: keep single-sample groups out of the rloo_vectorized zero path](https://github.com/verl-project/verl/pull/7581)
- **作者**: alanhuangyoo  **时间**: 2026-08-27 13:55 CST
- **摘要**: ### What does this PR do?  `rloo` and `rloo_vectorized` are documented as interchangeable `adv_estimator` values (`docs/examples/config.rst`), and `test_rloo_and_vectorized_equivalence` asserts they agree. They don't, on groups of one.  `compute_rloo_outcome_advantage` only rewrites a score when its…

### #7580 — [[rollout, trainer, training_utils, doc] feat: support weighted multi-output agent-loop trajectories](https://github.com/verl-project/verl/pull/7580)
- **作者**: dafu-wu  **时间**: 2026-08-27 13:46 CST
- **摘要**: ## What does this PR do?    Add first-class support for weighted multi-output agent-loop trajectories.    A single logical trajectory can now be expanded into multiple ordered training segments. Each segment carries an explicit   policy-gradient weight, typically `1/N`, so trajectory contribution re…

### #7577 — [[veomni] feat: DeepSeek V4 QAT bf16 fake quant training](https://github.com/verl-project/verl/pull/7577)
- **作者**: wuxibin89  **时间**: 2026-08-27 10:07 CST
- **摘要**: ### What does this PR do?  veomni DeepSeek V4 QAT: https://github.com/ByteDance-Seed/VeOmni/pull/1089

### #7576 — [[trainer, perf] feat: skip unused old-policy entropy computation](https://github.com/verl-project/verl/pull/7576)
- **作者**: eteluna  **时间**: 2026-08-27 05:13 CST
- **摘要**: ### What does this PR do?  Avoid computing and transferring token-level entropy during old-policy log-probability recomputation when neither entropy diagnostics nor entropy regularization requires it.  The classic and v1 PPO trainers currently force `calculate_entropy=True` for the old-policy forwar…

### #7575 — [build(deps-dev): bump cupy-cuda13x from 14.0.1 to 14.2.0](https://github.com/verl-project/verl/pull/7575)
- **作者**: dependabot[bot]  **时间**: 2026-08-27 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [cupy-cuda13x](https://github.com/cupy/cupy) from 14.0.1 to 14.2.0. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/cupy/cupy/releases">cupy-cuda13x's releases</a>.</em></p> <blockquote> <h2>v14.2.0</h2> <h1>CuPy v14.2.0 Release Note</h1> <p>This rele…

### #7574 — [build(deps-dev): bump flash-attn from 2.8.3 to 2.8.3.post1](https://github.com/verl-project/verl/pull/7574)
- **作者**: dependabot[bot]  **时间**: 2026-08-27 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [flash-attn](https://github.com/Dao-AILab/flash-attention) from 2.8.3 to 2.8.3.post1. <details> <summary>Commits</summary> <ul> <li><a href="https://github.com/Dao-AILab/flash-attention/commit/a8aa52b1ab3e9ca574c8a33b3f35afc017ffa2e2"><code>a8aa52b</code></a> fix: bump <strong>version</strong>…

### #7573 — [build(deps-dev): bump sglang from 0.5.12 to 0.5.18](https://github.com/verl-project/verl/pull/7573)
- **作者**: dependabot[bot]  **时间**: 2026-08-27 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [sglang](https://github.com/sgl-project/sglang) from 0.5.12 to 0.5.18. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/sgl-project/sglang/releases">sglang's releases</a>.</em></p> <blockquote> <h2>v0.5.18</h2> <h1>Highlights</h1> <p><em>710 PRs from 2…

### #7572 — [build(deps): update pyarrow requirement from <=24.0.0,>=15.0.0 to >=15.0.0,<=25.0.1](https://github.com/verl-project/verl/pull/7572)
- **作者**: dependabot[bot]  **时间**: 2026-08-27 01:34 CST
- **标签**: dependencies, python
- **摘要**: Updates the requirements on [pyarrow](https://github.com/apache/arrow) to permit the latest version. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/apache/arrow/releases">pyarrow's releases</a>.</em></p> <blockquote> <h2>Apache Arrow 25.0.1</h2> <p>Release…
