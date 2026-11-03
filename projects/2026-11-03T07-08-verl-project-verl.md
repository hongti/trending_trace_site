# verl-project/verl — 动态追踪

> 生成时间: 2026-11-03 15:08 CST

## AI 总结

## verl-project/verl 近期动态摘要

### 📌 Issue
- **#8504** — V1 `MetricsAggregator` 存在错位 Bug：rollout 概率差异均值与有效性标志（validity flags）未正确对齐，导致 PPO 指标计算可能出错。已在主分支 `1186e32` 上复现，待修复。

### 🔧 Pull Requests

**性能与新特性**
- **#8506**：为 `deepseek_v4` 添加 MFU/FLOPs 估算，解决 DSV4 运行时日志输出 `mfu = 0` 的问题。
- **#8500**：引入分组 TileLang 量化（支持 FP8 / MXFP8 / MXFP4），覆盖 QAT 与权重导出路径，适用于 Megatron + Transformer Engine 专家权重。
- **#8508**：利用 fused kernel 优化 Qwen3.5 的 FSDP-turbo 训练性能。
- **#8505**：新增 ROCm（MI355X, gfx950）平台下 DeepSeek-V4-Flash 真·on-policy GRPO 配方与镜像，实现 vLLM rollout 与 trainer 的 log-probs 位级一致。
- **#8510**：CI 增加精确测试的 coverage 收集能力。

**Bug 修复**
- **#8509**：修复 `use_kl_in_reward=false` 时 PPO 指标采集错误——仅在产生 token-level rewards 时才去获取，保留原有数据写入行为。
- **#8503**：修复 Ray worker 启动时 master port 竞争导致的 `EADDRINUSE` 错误，消除端口探测与 PyTorch TCPStore 绑定之间的间隙。
- **#8502**：清理 `setup.py` 中不再支持的遗留 extras（`gpu`、`geo`、`prime`、`trl`、`mcore`、`trtllm`），与 `pyproject.toml` 保持一致。

**重构与清理**
- **#8507**：移除 vLLM-Ascend v0.30.0 已废弃的 NPU patch shim 代码。
- **#8501**：为 V1 PPO 的 `TransferQueue` 字段选择添加 CPU 回归测试，确保仅在启用 critic 时才请求 `values`。

### 📦 Release
本轮动态中**无新版本发布**。

---

**总体趋势**：本轮活动集中在 V1 PPO trainer 的稳定性修复（指标对齐、KL 开关、TransferQueue 测试），同时持续推进新模型（DeepSeek-V4 / V4-Flash、Qwen3.5）与新硬件（ROCm MI355X、Ascend NPU）的适配与性能优化，并完善 CI 测试覆盖率。

---

## 🐛 Issues

### #8504 — [[trainer] V1 MetricsAggregator misaligns rollout probability-difference means and validity flags](https://github.com/verl-project/verl/issues/8504)
- **作者**: wengeezhang  **时间**: 2026-11-03 11:20 CST
- **摘要**: ### System Info  - Verified against `verl-project/verl` main at [`1186e32f6ec81da21bd827f1178d39227f6b5441`](https://github.com/verl-project/verl/commit/1186e32f6ec81da21bd827f1178d39227f6b5441). - CPU reproduction: macOS/Darwin 25.6.0 arm64, Python 3.12.13, PyTorch 2.9.1, NumPy 2.5.1; CUDA unavaila…

## 🔀 Pull Requests

### #8510 — [[ci] feat: add coverage collect for precise tests](https://github.com/verl-project/verl/pull/8510)
- **作者**: lxb007981  **时间**: 2026-11-03 15:01 CST
- **摘要**: ### What does this PR do?  [ci] feat: add coverage collect for precise tests  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include…

### #8509 — [[trainer] fix: conditionally fetch token-level rewards](https://github.com/verl-project/verl/pull/8509)
- **作者**: ChibiQuest  **时间**: 2026-11-03 14:51 CST
- **摘要**: ### What does this PR do?  Fix PPO metrics collection when algorithm.use_kl_in_reward=false by only fetching token_level_rewards when they are produced. This preserves the original data-writing behavior and prevents missing-field errors without affecting KL-in-reward workflows.  ### Checklist Before…

### #8508 — [[fsdp, ci] fix: optimize Qwen3.5 fsdp-turbo training with fused kernels](https://github.com/verl-project/verl/pull/8508)
- **作者**: pengnuoheng  **时间**: 2026-11-03 11:56 CST
- **摘要**: ### What does this PR do?  optimize Qwen3.5 fsdp-turbo training with fused kernels.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` …

### #8507 — [[vllm, hardware] refactor: drop obsolete vLLM-Ascend NPU patch shims for vllm-ascend v0.30.0](https://github.com/verl-project/verl/pull/8507)
- **作者**: floatlibai  **时间**: 2026-11-03 11:53 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  Narrow `verl/utils/vllm/npu_vllm_patch.py` to the patches that are still version-gated and applicable, removing shims that targeted…

### #8506 — [[perf, model] feat: add DeepSeek-V4 MFU / FLOPs estimation](https://github.com/verl-project/verl/pull/8506)
- **作者**: arvyanh  **时间**: 2026-11-03 11:36 CST
- **摘要**: ### What does this PR do?  - Add MFU/FLOPs estimation for `deepseek_v4` in `verl/utils/flops_counter.py`, so DSV4 runs no longer log `mfu = 0` after `Only support config type of {...}, but got deepseek_v4`. - Account for DSV4 hybrid attention: sliding-window layers, CSA layers (2x compressor + light…

### #8505 — [[hardware, recipe, rollout, worker] feat: DeepSeek-V4-Flash true on-policy GRPO on ROCm (MI355X)](https://github.com/verl-project/verl/pull/8505)
- **作者**: indianspeedster  **时间**: 2026-11-03 11:21 CST
- **摘要**: ### What does this PR do?  Adds a ROCm (MI355X, gfx950) recipe and image for DeepSeek-V4-Flash GRPO where the vLLM rollout log-probs and the trainer log-probs of the same tokens are **bitwise equal** (`rollout_probs_diff_max` = 0.0). It follows the pattern of #7050 and #8146: a Dockerfile on `vllm/v…

### #8503 — [[ray] fix: prevent master port races during worker startup](https://github.com/verl-project/verl/pull/8503)
- **作者**: SeptPonts  **时间**: 2026-11-03 10:15 CST
- **摘要**: ### What does this PR do?  Prevent `EADDRINUSE` during Ray worker startup caused by the gap between selecting a master port and binding PyTorch's TCPStore.  Previously, `RayWorkerGroup` probed a free port with a temporary socket, closed it, and passed the port to workers. Another process could claim…

### #8502 — [[packaging, doc] fix: remove unsupported legacy extras](https://github.com/verl-project/verl/pull/8502)
- **作者**: joshuaburgen  **时间**: 2026-11-03 01:58 CST
- **摘要**: ### What does this PR do?  Removes the unsupported legacy `gpu`, `geo`, `prime`, `trl`, `mcore`, and `trtllm` extras from `setup.py`, leaving extras that are still declared in `pyproject.toml`. Updates the TensorRT-LLM installation instructions to install verl without the unavailable `.[trtllm]` ext…

### #8501 — [[trainer] test: cover V1 TransferQueue critic field selection](https://github.com/verl-project/verl/pull/8501)
- **作者**: joshuaburgen  **时间**: 2026-11-03 01:13 CST
- **摘要**: ### What does this PR do?  Adds CPU regression tests for V1 PPO's TransferQueue field selection. Both `_compute_advantage` and `_compute_metrics` must request `values` only when the critic is enabled, while preserving the critic-enabled request.  ### Why is this not duplicating an existing PR?  Sear…

### #8500 — [[megatron, rollout, perf] feat: add grouped TileLang quantization for QAT and export](https://github.com/verl-project/verl/pull/8500)
- **作者**: wuxibin89  **时间**: 2026-11-02 23:12 CST
- **摘要**: ### What does this PR do?  Adds grouped TileLang fake-quantization and weight-export paths for FP8, MXFP8, and MXFP4. Transformer Engine grouped expert weights are quantized through pointer tables without materializing a stacked copy, while fused 3D Megatron-Bridge expert tensors are quantized once …
