# verl-project/verl — 动态追踪

> 生成时间: 2026-10-01 14:25 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📌 Issue 动态
- **C-GRPO 算法提案 (#8077)**：有作者提议将其 NeurIPS 2026 论文中的 **C-GRPO**（Conformal Group Relative Policy Optimization）引入 verl。该算法改进了 GRPO，使用自适应的每提示组大小，替代传统的固定 $n$ 个响应采样方式。
- **FSDP 数据类型配置失效 (#8076)**：反馈在设置 `fsdp_config.dtype=float16` 时，FSDP 引擎会忽略该配置，导致在 BF16 训练中生成 FP16 的 rollout，且不抛出任何错误或警告。

### 📌 PR 动态
**新特性**
- **vLLM 请求钩子 (#8082)**：为 `vLLMHttpServer.generate()` 增加了请求钩子，允许服务器子类在不重写方法的情况下修改请求的采样参数和输出结果。
- **Megatron 1F1b EP 重叠 (#8074)**：在 Megatron 模型引擎中重新启用了 1F1B EP all-to-all 重叠机制，提升并行效率。

**Bug 修复**
- **FSDP 混合精度修复 (#8083)**：修复了 Issue #8076，确保在未指定 `mixed_precision.param_dtype` 时，FSDP 引擎能正确遵循配置的 `dtype`。
- **vLLM Rollout 配置修复 (#8075)**：将 `max_num_batched_tokens` 设为可变，修复了在禁用 chunked prefill 且该参数小于 `max_model_len` 时触发异常的问题。
- **注释与脚本拼写修正 (#8084)**：修正了奖励循环注释和安装脚本中的拼写错误。

**依赖更新**
- 升级 `vllm` 至 0.30.0 (#8078)
- 升级 `trl` 至 1.14.0 (#8081)
- 升级 `nvidia-modelopt` 至 0.47.0 (#8080)
- 升级 `transferqueue` 至 0.1.11 (#8079)

**文档与文案**
- 修正昇腾（Ascend）产品名称表述：从 "Ascend 950 系列产品 / Ascend 950PR&DT系列产品" 更正为 "Ascend 950PR&950DT系列产品" (#8073)。

### 📌 Release 动态
- 本周期内无新的版本发布。

---

## 🐛 Issues

### #8077 — [[RFC] C-GRPO: adaptive per-prompt group size for GRPO](https://github.com/verl-project/verl/issues/8077)
- **作者**: AryaFayyazi  **时间**: 2026-10-01 00:43 CST
- **摘要**: ### Feature request  We'd like to propose C-GRPO as a verl recipe. It's from our NeurIPS 2026 paper, *C-GRPO: Conformal Group Relative Policy Optimization*.  Instead of sampling a fixed `n` responses for every prompt, C-GRPO samples in rounds (by default 2, 4, 8, 16, 32 responses) and stops early fo…

### #8076 — [FSDP engine ignores `fsdp_config.dtype`: `dtype=float16` gives FP16 rollouts with BF16 training (no error or warning)](https://github.com/verl-project/verl/issues/8076)
- **作者**: aryanyadav0402  **时间**: 2026-09-30 20:50 CST
- **摘要**: ### System Info  - verl `6093e007` (the behaviour is unchanged on `main` at `3084c261`, by code reading) - vLLM 0.29.0, torch 2.13.0+cu130, transformers 5.12.1, Python 3.12 - 1× NVIDIA RTX A6000 (48 GB), FSDP (`strategy: fsdp`), `trainer.use_v1=false` - (Versions from our environment record; `script…

## 🔀 Pull Requests

### #8084 — [[trainer, data, tool] fix: correct misspellings in reward loop comments and install script](https://github.com/verl-project/verl/pull/8084)
- **作者**: li-lizhe  **时间**: 2026-10-01 13:38 CST
- **摘要**: ### What does this PR do?  Fix misspellings in two source comments and two user-visible strings. Text-only change, no behavior change.  | File | Line | Was | Now | |---|---|---|---| | `verl/trainer/ppo/ray_trainer.py` | 912 | `initalize` | `initialize` | | `verl/experimental/separation/ray_trainer.p…

### #8083 — [[fsdp] fix: honor engine dtype for mixed precision defaults](https://github.com/verl-project/verl/pull/8083)
- **作者**: zupengwang  **时间**: 2026-10-01 10:05 CST
- **摘要**: ### What does this PR do?  Fixes #8076. Setting `actor_rollout_ref.actor.fsdp_config.dtype=float16` currently leaves FSDP compute and autocast in BF16 when `mixed_precision.param_dtype` is absent, so the FP16 gradient scaler is also omitted.  Use the engine's `dtype` as the default for both an absen…

### #8082 — [[rollout, vllm] feat: add request hooks to vLLMHttpServer.generate](https://github.com/verl-project/verl/pull/8082)
- **作者**: eteluna  **时间**: 2026-10-01 02:41 CST
- **摘要**: ### What does this PR do?  Adds two request hooks to `vLLMHttpServer.generate()`, so a server subclass can change a request's sampling parameters and its output without overriding `generate()`. Admission, pause/resume, and weight-sync coordination stay in `generate()`.  This implements the hook opti…

### #8081 — [build(deps-dev): bump trl from 0.27.0 to 1.14.0](https://github.com/verl-project/verl/pull/8081)
- **作者**: dependabot[bot]  **时间**: 2026-10-01 01:33 CST
- **标签**: dependencies, python
- **摘要**: Bumps [trl](https://github.com/huggingface/trl) from 0.27.0 to 1.14.0. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/trl/releases">trl's releases</a>.</em></p> <blockquote> <h2>v1.14.0</h2> <h2>Features</h2> <h3><code>trl.losses</code> is gone…

### #8080 — [build(deps-dev): bump nvidia-modelopt from 0.44.0 to 0.47.0](https://github.com/verl-project/verl/pull/8080)
- **作者**: dependabot[bot]  **时间**: 2026-10-01 01:33 CST
- **标签**: dependencies, python
- **摘要**: Bumps [nvidia-modelopt](https://github.com/NVIDIA/Model-Optimizer) from 0.44.0 to 0.47.0. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/NVIDIA/Model-Optimizer/releases">nvidia-modelopt's releases</a>.</em></p> <blockquote> <h2>ModelOpt 0.47.0 Release</h2>…

### #8079 — [build(deps): bump transferqueue from 0.1.10 to 0.1.11](https://github.com/verl-project/verl/pull/8079)
- **作者**: dependabot[bot]  **时间**: 2026-10-01 01:33 CST
- **标签**: dependencies, python
- **摘要**: Bumps [transferqueue](https://github.com/Ascend/TransferQueue) from 0.1.10 to 0.1.11. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/Ascend/TransferQueue/releases">transferqueue's releases</a>.</em></p> <blockquote> <h2>v0.1.11</h2> <h2>Highlight</h2> <h3>…

### #8078 — [build(deps-dev): bump vllm from 0.29.0 to 0.30.0](https://github.com/verl-project/verl/pull/8078)
- **作者**: dependabot[bot]  **时间**: 2026-10-01 01:33 CST
- **标签**: dependencies, python
- **摘要**: Bumps [vllm](https://github.com/vllm-project/vllm) from 0.29.0 to 0.30.0. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/vllm-project/vllm/releases">vllm's releases</a>.</em></p> <blockquote> <h1>v0.30.0</h1> <h2>Highlights</h2> <p>This release features 76…

### #8075 — [[rollout] fix: make max_num_batched_tokens mutable](https://github.com/verl-project/verl/pull/8075)
- **作者**: kahlun  **时间**: 2026-09-30 19:58 CST
- **摘要**: ## What does this PR do?  #7632 added a self-heal to `vLLMHttpServer._validate_configs()`. When `enable_chunked_prefill=False` and `max_num_batched_tokens < max_model_len`, it raises `max_num_batched_tokens` to `max_model_len`:  ```python self.config.max_num_batched_tokens = self.config.max_model_le…

### #8074 — [[megatron] feat: adapt 1f1b EP overlap in model engine](https://github.com/verl-project/verl/pull/8074)
- **作者**: ji-huazhong  **时间**: 2026-09-30 17:55 CST
- **摘要**: ### What does this PR do?  Follow-up to the https://github.com/verl-project/verl/pull/8058#discussion_r4132015569. This PR re-enables 1F1B EP all-to-all overlap in the Megatron model engine.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] F…

### #8073 — [[doc] fix: change  ascend product name](https://github.com/verl-project/verl/pull/8073)
- **作者**: zery00568  **时间**: 2026-09-30 16:43 CST
- **摘要**: ### What does this PR do?  Change product name. Ascend 950 系列产品 / Ascend 950PR&DT系列产品 ->Ascend 950PR&950DT系列产品  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked …
