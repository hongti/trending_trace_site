# verl-project/verl — 动态追踪

> 生成时间: 2026-07-29 09:03 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🐛 Issue
*   **#7180 程序在运行随机次数后挂起**：用户反映在 rollout 生成阶段，程序会在运行几次后间歇性卡死挂起。
*   **#7177 训练 GSM8K Agent RL 时持续输出重复日志**：用户在使用 vLLM V1 训练 GSM8K Agent 时，遇到不断输出重复日志的问题。

---

### 🔀 Pull Request
**核心修复与回退**
*   **#7182 [FSDP] 修复 ref model CPUOffload 硬编码问题**：重要修复。将 FSDP 对非 actor 模型（如 reference/critic 等）的 `CPUOffload(offload_params=True)` 从硬编码改为可配置，提升灵活性。
*   **#7174 [Experimental] 修复多轮验证对齐的 AssertionError**：修复了在完全异步 PPO 中运行验证时的报错，通过正确保留 `__sample_index_in_batch` 解决了 Uni-Agent 上下文管理组件的问题。
*   **#7173 回退 Megatron Qwen3.5 LoRA & MTP 支持的 PR**：回退了之前关于 Qwen3.5 LoRA 与 MTP 支持的 PR (#5599)。

**架构与重构**
*   **#7181 [Megatron] delta_sharded 支持 Megatron-Bridge 参数映射**：重要特性。让 mcore 后端加入了 sharded-delta contract，支持 TP+EP 和 hybrid-Mamba 架构。
*   **#7179 [vLLM] 重构权重同步逻辑**：重构了 `update_weights_from_ipc`，使 vLLM 的权重同步代码逻辑更加清晰。

**硬件适配与 CI/文档**
*   **#7176 [CI] 修复 Ascend nightly CI 与 Docker 失败**：更新并修复了昇腾环境的夜间 CI 和 Docker 构建失败问题。
*   **#7178 / #7175 [Doc] 补充 NPU 多机任务启动文档**：为 NPU 环境添加了多机任务启动的实践操作文档。
*   **#7172 [training_utils] 统一 attention utilities 导入语句**：统一了不同硬件/测试文件中 `attention_util` 的导入方式，解决代码不一致问题。
*   **#7171 误操作 PR**：误开向错误仓库的 PR，已关闭。

---

### 🚀 Release
*   近期无新版本发布。

---

## 🐛 Issues

### #7180 — [Program hang after random number of runs](https://github.com/verl-project/verl/issues/7180)
- **作者**: YushuYan064  **时间**: 2026-07-29 03:40 CST
- **标签**: bug
- **摘要**: ### System Info  Hi,  I am new to VERL and recently encountered an issue where the program hangs during rollout generation after a few runs. The issue occurs intermittently: sometimes it happens during validation, and sometimes during training. I have not observed any error messages in the logs. The…

### #7177 — [train gsm8k agent RL ,  Continuously outputting the following logs:](https://github.com/verl-project/verl/issues/7177)
- **作者**: cqray1990  **时间**: 2026-07-28 18:42 CST
- **标签**: bug
- **摘要**: ### System Info  training scripts：                             set -x                                          export VLLM_USE_V1=1                                          # ================= data/model/tool =================                     HDFS_ROOT=${HDFS_ROOT:-$PWD}                     DATA…

## 🔀 Pull Requests

### #7182 — [Fix: Make ref model CPUOffload configurable instead of hardcoded](https://github.com/verl-project/verl/pull/7182)
- **作者**: Shadowking912  **时间**: 2026-07-29 04:29 CST
- **摘要**: ## [fsdp] fix: Make ref model CPUOffload configurable instead of hardcoded  ### What does this PR do?  Fixes an issue where FSDP hardcodes `CPUOffload(offload_params=True)` for all non-actor (reference/reward) models, ignoring the user-configured `param_offload` setting in `verl/workers/engine/fsdp/…

### #7181 — [[megatron] delta_sharded on Megatron-Bridge param mappings (TP+EP, hybrid-Mamba)](https://github.com/verl-project/verl/pull/7181)
- **作者**: ChangyiYang  **时间**: 2026-07-29 04:25 CST
- **摘要**: ### What does this PR do?  The mcore backend joins the sharded-delta contract (#7144). **Stacked on #7085** — the first 8 commits are that PR (veomni EP); review scope here is the 23 megatron/engine commits. Everything rides Megatron-Bridge's own per-param machinery — no legacy converters, no hand-w…

### #7179 — [[vllm] refactor: clean up weight sync](https://github.com/verl-project/verl/pull/7179)
- **作者**: wuxibin89  **时间**: 2026-07-28 22:20 CST
- **摘要**: ### What does this PR do?  Restructure `update_weights_from_ipc` to make vllm weigt sync more clear.

### #7178 — [[doc] chore: add multi machine task startup document for npu](https://github.com/verl-project/verl/pull/7178)
- **作者**: zhouhengan1211  **时间**: 2026-07-28 19:34 CST
- **摘要**: ### What does this PR do?  Add multi machine task startup practice document for npu usage. See https://github.com/verl-project/verl/pull/7175  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist B…

### #7176 — [[ci] fix: update Ascend nightly CI & docker](https://github.com/verl-project/verl/pull/7176)
- **作者**: kyle-zhangchi  **时间**: 2026-07-28 17:36 CST
- **摘要**: ### What does this PR do?  Fixed the nightly CI and Docker failures.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`,…

### #7175 — [[doc] chore: add multi machine task startup document for npu](https://github.com/verl-project/verl/pull/7175)
- **作者**: zhouhengan1211  **时间**: 2026-07-28 17:15 CST
- **摘要**: ### What does this PR do?  Add multi machine task startup practice document for npu usage.  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste…

### #7174 — [[experimental] fix: preserve __sample_index_in_batch for multi-turn validation alignment](https://github.com/verl-project/verl/pull/7174)
- **作者**: iiGray  **时间**: 2026-07-28 15:38 CST
- **摘要**: **Bug fixes related to the Uni-Agent context management component**  This fixes the AssertionError when running validation in fully async PPO by properly tracking the original sample indices through the agent loop and aligning them before DataProto union.

### #7173 — [Revert "[megatron] fix: Qwen3.5 LoRA & MTP support (with Megatron-Bridge) (#5599)"](https://github.com/verl-project/verl/pull/7173)
- **作者**: wuxibin89  **时间**: 2026-07-28 15:16 CST
- **摘要**: ### What does this PR do?  As title.

### #7172 — [[training_utils, hardware] fix: unify import statements for attention utilities across test files](https://github.com/verl-project/verl/pull/7172)
- **作者**: kahlun  **时间**: 2026-07-28 11:31 CST
- **摘要**: ### What does this PR do?  Would want to unify the one more attention_util file in torch_functional.py After this [#7098] merged, then this PR should be related to merge in. To resolve the different hardware encounter bug  to tconvert the code of  ``` python elif get_device_name() == "XXX": from npu…

### #7171 — [Closed](https://github.com/verl-project/verl/pull/7171)
- **作者**: hagiss  **时间**: 2026-07-28 08:55 CST
- **摘要**: Opened this pull request against the wrong repository by mistake.
