# verl-project/verl — 动态追踪

> 生成时间: 2026-07-14 09:01 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📋 Issue 动态
- **[#7029] [Bug] 权重同步严重降低 H100 解码吞吐量**：报告了 Pre-rollout 阶段的权重同步 (`update_weights`) 在 H100 80GB 显卡上会导致 rollout 解码吞吐量严重下降的性能问题；值得注意的是，相同的软件栈在 H20-3e 141GB 显卡上该问题几乎可忽略，属于显著的跨硬件性能差异 Bug。
- **[#7030] [RFC] 征求意见稿**：内容暂为 TODO，尚无实质性讨论细节。

### 🔀 Pull Request 动态

#### 🚀 新特性
- **[#7022] 引入 UP-GRPO 算法**：新增 UP-GRPO（无界正不对称策略优化）作为即插即用的策略损失模式 `up`，丰富了算法支持。
- **[#7032] Skip Manager 支持参数同步步骤**：在 Skip Manager 中增加了参数同步步骤支持（此改动很可能与解决 #7029 的吞吐量下降问题相关）。
- **[#7027] 生成式奖励模型支持确定性计算**：为用户自定义的生成式奖励模型 (GRM) 路径提供确定性 (deterministic) 计算支持，确保结果可复现。

#### 🛠️ Bug 修复与兼容性
- **[#7033] 修复 FA3 环境下的注意力 fallback 逻辑**：在仅有 FlashAttention-3 (FA3) 的环境中，修复了 fallback 到 `transformers` padding utils 时引发的报错，确保无 `flash_attn` 包时的兼容性。
- **[#7031] 修复 Qwen3VL Megatron 后端错误**：解决了 Qwen3VL 模型在 Megatron 后端运行时的报错问题。
- **[#7026] 修复 FSDP 数据类型冲突**：解决了 FSDP 框架下 `torch.Tensor` 与 `DTensor` 之间因数据类型不同导致的冲突。
- **[#7028] 同步 Qwen3.5 Dockerfile 及锁定依赖版本**：将 0.8.0 分支的 Qwen3.5 Dockerfile 同步至主线，并将 PyArrow 版本锁定至 `<=24.0.0`，以规避 v25.0.0 引起的兼容性问题。

#### 📝 工程与文档
- **[#7023 / #7024] 重构 NPU Profiling 文档**：优化 NPU 性能分析文档的易用性，新增快速入门案例，并修改了细粒度采集部分的常用参数说明。
- **[#7025] 临时禁用 torchtitan CI**：因当前 PyTorch 版本不兼容，暂时禁用了 torchtitan 的 CI 流水线。

### 🚀 Release 动态
- 本次统计周期内**无新版本发布**。

---

## 🐛 Issues

### #7030 — [[RFC]](https://github.com/verl-project/verl/issues/7030)
- **作者**: Jackie2049  **时间**: 2026-07-13 15:20 CST
- **摘要**: TODO

### #7029 — [[Bug] Pre-rollout weight sync (update_weights) severely degrades rollout decode throughput on H800](https://github.com/verl-project/verl/issues/7029)
- **作者**: kfzshere  **时间**: 2026-07-13 15:13 CST
- **标签**: bug
- **摘要**: ### System Info  This bug is a **cross-hardware comparison** (severe on H100 80GB, negligible on H20-3e 141GB), so I report both machines. The software stack is byte-identical on both (same docker image); the only variables are the **GPU and the NVIDIA driver**.  **Common software stack (identical o…

## 🔀 Pull Requests

### #7033 — [[utils] fix: guard transformers fallback for flash_attn padding utils in FA3-only envs](https://github.com/verl-project/verl/pull/7033)
- **作者**: ArdalanM  **时间**: 2026-07-14 06:19 CST
- **摘要**: ## Summary `verl/utils/attention_utils.py` falls back to `transformers.modeling_flash_attention_utils` when `flash_attn` isn't installed, which is how we support running with FlashAttention-3 (`flash_attn_interface`) instead of FA2. That fallback needs `transformers>=4.56.0` (`_pad_input`/`_unpad_in…

### #7032 — [[tool, rollout] feat: support parameter sync steps in Skip Manager](https://github.com/verl-project/verl/pull/7032)
- **作者**: mikequan0425  **时间**: 2026-07-13 22:04 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  Related issue：https://github.com/verl-project/verl/issues/7007 Save the skipped rollout data in the format of global step + param s…

### #7031 — [[megatron] fix: Fix the error in the Qwen3VL Megatron backend](https://github.com/verl-project/verl/pull/7031)
- **作者**: Seren-hao  **时间**: 2026-07-13 20:27 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Fix the error in the Qwen3VL Megatron backend  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fs…

### #7028 — [[env] fix: Sync Qwen3.5 Dockerfile from 0.8.0 Branch to Main and Pin PyArrow Version](https://github.com/verl-project/verl/pull/7028)
- **作者**: ruanhao566  **时间**: 2026-07-13 15:09 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Sync the Qwen3.5 Dockerfile from the 0.8.0 branch to the main branch, and pin the PyArrow version to <=24.0.0 to avoid issues with version 25.0.0.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR titl…

### #7027 — [[reward] feat: support deterministic reward for user-defined generative RM paths](https://github.com/verl-project/verl/pull/7027)
- **作者**: KaisennHu  **时间**: 2026-07-13 11:40 CST
- **摘要**: ### What does this PR do?  Builds on `df35e73` (full determinism for vLLM rollout + reward model, PR #6572) to make the reward path deterministic for **generative RM (GRM)** — the user-defined reward path used by verl-omni's VLM scoring. `df35e73` forced `max_num_seqs=1` on all RM (discriminative `/…

### #7026 — [[fsdp] fix: Fix the conflict caused by different data types between to…](https://github.com/verl-project/verl/pull/7026)
- **作者**: Seren-hao  **时间**: 2026-07-13 11:23 CST
- **摘要**: …rch.Tensorand DTensor.  ### What does this PR do?  Fix the conflict caused by different data types between torch.Tensorand DTensor.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` …

### #7025 — [[ci] chore: temporary disable torchtitan ci due to im-compatible torchversion](https://github.com/verl-project/verl/pull/7025)
- **作者**: wuxibin89  **时间**: 2026-07-13 10:48 CST
- **摘要**: ### What does this PR do?  As title.

### #7024 — [[doc] refactor: edit profiling docs of npu for usability](https://github.com/verl-project/verl/pull/7024)
- **作者**: zhouhengan1211  **时间**: 2026-07-13 10:37 CST
- **摘要**: ### What does this PR do?  The main modifications are as follows:  - Added a quick start case to facilitate direct use by users. - The fine-grained collection section has been modified, and common parameter settings are extracted to facilitate user operations.  > Add **concise** overview of what thi…

### #7023 — [[doc] refactor: edit profiling docs of npu for usability](https://github.com/verl-project/verl/pull/7023)
- **作者**: zhouhengan1211  **时间**: 2026-07-13 10:02 CST
- **摘要**: ### What does this PR do?  The main modifications are as follows: - Added a quick start case to facilitate direct use by users. - The fine-grained collection section has been modified, and common parameter settings are extracted to facilitate user operations.  > Add **concise** overview of what this…

### #7022 — [[algo, doc] feat: add UP-GRPO unbounded positive asymmetric policy loss](https://github.com/verl-project/verl/pull/7022)
- **作者**: a-F1  **时间**: 2026-07-13 08:53 CST
- **摘要**: ### What does this PR do?  Adds **UP-GRPO** (Unbounded Positive Asymmetric Optimization, [arXiv:2607.06987](https://arxiv.org/pdf/2607.06987)) as a new, plug-and-play policy-loss mode `up`.  **Motivation.** Standard GRPO/PPO combines a symmetric clip (single `ε`) with importance sampling against `π_…
