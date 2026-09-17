# verl-project/verl — 动态追踪

> 生成时间: 2026-09-17 13:10 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🚀 Pull Request (PR) 动态

**1. 新特性**
*   **DeepSeek V4 MXFP4 量化训练支持** (#7895)：为 DeepSeek-V4-Fl 新增了基于 Transformer Engine (TE) FP8 训练的路由专家权重 MXFP4 假量化支持。
*   **奖励方差感知过滤** (#7894)：在 PPO 训练中引入了可选的奖励方差感知提示组过滤机制（基于 RAGEN-2 的 SNR-Aware Filtering 方法），优化训练效果。

**2. 重要修复**
*   **GRPO 标准化标志修复** (#7893)：修复了在 `grpo_vectorized` 算法中，即使设置了 `norm_adv_by_std_in_grpo=False`，优势仍会被标准差除的 bug。
*   **蒸馏微批次处理修复** (#7887)：修复了 V1 batch balancing 在生成全 padding 的 top-k 蒸馏微批次时，导致指标计算出错的异常。
*   **CI 环境变量清理** (#7886)：从昇腾的 Dockerfile 中移除了多余的 `VERL_USE_EXTERNAL_MODULES` 环境变量。

**3. 依赖更新**
*   大规模升级了多项核心依赖：vllm (`0.24.0` → `0.29.0`)、trl (`0.27.0` → `1.13.0`)、transformers (`5.9.0` → `5.17.0`)，并放宽/更新了 tensordict 和 nvidia-modelopt 的版本要求。

---

### 📝 Issue 动态
*   本期暂无活跃的 Issue 动态。

---

### 📦 Release 动态
*   本期暂无新版本发布。

---

## 🔀 Pull Requests

### #7895 — [[megatron]feat: Deepseek v4 RL MXFP4 experts qat support](https://github.com/verl-project/verl/pull/7895)
- **作者**: sophiayyya  **时间**: 2026-09-17 11:08 CST
- **摘要**: ### What does this PR do?  ### MXFP4 QAT precision and data flow  This change adds **routed-expert weight-only MXFP4 fake quantization on top of Transformer Engine (TE) FP8 training** for DeepSeek-V4-Flash-0731. Checkpoint storage, trainable parameter dtype, and GEMM precision are separate:  | Modul…

### #7894 — [[trainer] feat: add reward-variance top-p/top-k filtering](https://github.com/verl-project/verl/pull/7894)
- **作者**: ZihanWang314  **时间**: 2026-09-17 03:13 CST
- **摘要**: ## What does this PR do?  Adds opt-in reward-variance-aware prompt-group filtering to veRL PPO training, following the SNR-Aware Filtering method introduced in [RAGEN-2](https://arxiv.org/abs/2604.06268).  - `strategy: top_p` retains the smallest stable prefix of prompt groups covering the configure…

### #7893 — [[trainer] fix: honor normalization flag in vectorized GRPO](https://github.com/verl-project/verl/pull/7893)
- **作者**: Jshipper-art  **时间**: 2026-09-17 01:47 CST
- **摘要**: ### What does this PR do?  With `algorithm.adv_estimator=grpo_vectorized`, setting `algorithm.norm_adv_by_std_in_grpo=False` still divides advantages by the group standard deviation. The trainer dispatch omits the flag, so the estimator uses its default `True`. Forward the existing flag and add a tr…

### #7892 — [build(deps-dev): bump vllm from 0.24.0 to 0.29.0](https://github.com/verl-project/verl/pull/7892)
- **作者**: dependabot[bot]  **时间**: 2026-09-17 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [vllm](https://github.com/vllm-project/vllm) from 0.24.0 to 0.29.0. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/vllm-project/vllm/releases">vllm's releases</a>.</em></p> <blockquote> <h1>v0.29.0</h1> <h2>Highlights</h2> <p>This release features 59…

### #7891 — [build(deps-dev): bump trl from 0.27.0 to 1.13.0](https://github.com/verl-project/verl/pull/7891)
- **作者**: dependabot[bot]  **时间**: 2026-09-17 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [trl](https://github.com/huggingface/trl) from 0.27.0 to 1.13.0. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/trl/releases">trl's releases</a>.</em></p> <blockquote> <h2>v1.13.0</h2> <h2>Features</h2> <h3>Training beyond 1M tokens</h3> …

### #7890 — [build(deps): bump transformers from 5.9.0 to 5.17.0](https://github.com/verl-project/verl/pull/7890)
- **作者**: dependabot[bot]  **时间**: 2026-09-17 01:33 CST
- **标签**: dependencies, python
- **摘要**: Bumps [transformers](https://github.com/huggingface/transformers) from 5.9.0 to 5.17.0. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/transformers/releases">transformers's releases</a>.</em></p> <blockquote> <h2>Release 5.17.0</h2> <h1>Release…

### #7889 — [build(deps): update tensordict requirement from !=0.9.0,<=0.10.0,>=0.8.0 to >=0.8.0,!=0.9.0,<=0.14.2](https://github.com/verl-project/verl/pull/7889)
- **作者**: dependabot[bot]  **时间**: 2026-09-17 01:33 CST
- **标签**: dependencies, python
- **摘要**: Updates the requirements on [tensordict](https://github.com/pytorch/tensordict) to permit the latest version. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/pytorch/tensordict/releases">tensordict's releases</a>.</em></p> <blockquote> <h2>TensorDict v0.14.…

### #7888 — [build(deps-dev): bump nvidia-modelopt from 0.44.0 to 0.46.1](https://github.com/verl-project/verl/pull/7888)
- **作者**: dependabot[bot]  **时间**: 2026-09-17 01:33 CST
- **标签**: dependencies, python
- **摘要**: Bumps [nvidia-modelopt](https://github.com/NVIDIA/Model-Optimizer) from 0.44.0 to 0.46.1. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/NVIDIA/Model-Optimizer/releases">nvidia-modelopt's releases</a>.</em></p> <blockquote> <h2>ModelOpt 0.46.1 Release</h2>…

### #7887 — [[algo, worker] fix: handle padding-only top-k distillation microbatches](https://github.com/verl-project/verl/pull/7887)
- **作者**: zupengwang  **时间**: 2026-09-17 00:32 CST
- **摘要**: ## What does this PR do?  V1 batch balancing can construct synthetic samples with an all-zero `response_mask`. When a top-k distillation microbatch contains only these samples, the current metric code calls `min()`/`max()` on empty tensors and aborts training before backward. Guarding only those red…

### #7886 — [[ci] fix: drop VERL_USE_EXTERNAL_MODULES env from Ascend Dockerfiles](https://github.com/verl-project/verl/pull/7886)
- **作者**: aass-79  **时间**: 2026-09-16 23:49 CST
- **摘要**: ## What this PR does  Removes `ENV VERL_USE_EXTERNAL_MODULES=megatron_adaptor` (added by #7827) from `docker/ascend/Dockerfile.ascend_9.1.0_a2` and `docker/ascend/Dockerfile.ascend_9.1.0_a3`.  ## Why  The env only takes effect in processes that `import verl` (it is read in `verl/__init__.py`), so it…
