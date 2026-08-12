# verl-project/verl — 动态追踪

> 生成时间: 2026-08-12 11:06 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 🐛 Issue 动态
*   **高严重度缺陷 (#7368)**：启用独立 DisRM 模式时，若缺少 `reward_extra_info`，会导致异步 AgentLoop 评分在请求成功后直接终止，致使训练无法继续。
*   **新模型支持请求 (#7365)**：用户提出希望支持新推出的 Agent 模型 `meta-models/Muse-Glimmer-30B` 的训练。
*   **架构设计讨论 (#7360)**：提出 RFC，讨论在共享异构 Ray 集群中，为 VERL 派生的 Ray actors 增加可选的通用节点资源调度约束，避免 Pod 被调度到无运行时镜像的节点。

### 🔀 PR 动态
**新特性与功能增强：**
*   **新增 FSDPTurbo 后端支持 (#7362)**：在 CI 和 trainer 中引入了 `fsdpturbo` 后端引擎。
*   **Triton 适配 MLU (#7367)**：使 Triton 能够适配寒武纪 MLU 硬件。
*   **Megatron 序列长度分桶 (#7358)**：引入 FSDP 兼容的 `pad_to_length` 控制，可选将打包的微批次舍入到 512-token 的桶中，以减少形状变体，提升性能。
*   **Rollout 端性能分析 (#7364)**：修改 v1 trainer，支持启用 rollout 侧的 profiling。

**Bug 修复：**
*   **vLLM 分桶接收失败暴露 (#7366)**：使分桶 ACK 等待时间可配置（默认30秒），并增加正数和有限性校验，暴露接收器失败问题。
*   **Megatron 异步检查点修复 (#7363)**：修复开启异步保存时，SFT 未驱动或排空异步队列导致的问题，现确保在每个 SFT rank 上终结队列。
*   **指标聚合崩溃修复 (#7361)**：修复 `reduce_metrics` 在空列表上的崩溃问题，并收紧了 max/min 匹配逻辑（从子串匹配改为最终路径段匹配）。
*   **3D position_ids 索引修复 (#7355)**：修复 `index_select_tensor_dict` 中 3D jagged `position_ids` 的 `_ragged_idx` 选择错误。
*   **NPU vLLM 环境修复 (#7359)**：在 NPU 安装脚本中为 vLLM 添加 `--no-build-isolation`，并预装依赖，防止破坏现有 PyTorch 环境。

**CI 与测试：**
*   **工作流迁移 (#7357)**：将 CI 从 `fully_async/one_step_off_policy` 迁移至 `v1 separate_async`，为将 `fully_async` 移入 recipe 做准备。

### 🚀 Release 动态
*   近期暂无新版发布。

---

## 🐛 Issues

### #7368 — [Standalone DisRM crashes async agent reward scoring when reward_extra_info is missing](https://github.com/verl-project/verl/issues/7368)
- **作者**: ai-yang  **时间**: 2026-08-12 10:35 CST
- **摘要**: ## Severity  High. Enabling the documented standalone DisRM mode deterministically terminates async AgentLoop scoring after the reward-model request succeeds, so training cannot advance.  ## System Info  - verl main commit: `060bebc565e07b0341465894716b92ac0903f6e1` - Linux x86_64, kernel 5.15 - Pyt…

### #7365 — [meta-models/Muse-Glimmer-30B support](https://github.com/verl-project/verl/issues/7365)
- **作者**: Yunhai-Hu  **时间**: 2026-08-12 08:28 CST
- **摘要**: ### Feature request  Will we consider to support meta-models/Muse-Glimmer-30B training?  ### Motivation  meta-models/Muse-Glimmer-30B is a new model for Agent  ### Your contribution  N/A

### #7360 — [[RFC] Optional generic node-resource scheduling constraint for VERL-derived Ray actors on shared clusters](https://github.com/verl-project/verl/issues/7360)
- **作者**: nataliekung  **时间**: 2026-08-11 18:33 CST
- **摘要**: ## Summary  This is a design question / RFC, not a change request. On shared, heterogeneous Ray clusters, VERL-derived Ray actors can be scheduled onto nodes that do not carry the job's runtime image, causing `ModuleNotFoundError` crashes. #7343 addresses this for the agent-loop and reward-loop work…

## 🔀 Pull Requests

### #7367 — [[mlu] feat: triton adapted for mlu](https://github.com/verl-project/verl/pull/7367)
- **作者**: guleo  **时间**: 2026-08-12 10:22 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #7366 — [[vllm] fix: surface bucketed receiver failures](https://github.com/verl-project/verl/pull/7366)
- **作者**: nataliekung  **时间**: 2026-08-12 09:01 CST
- **摘要**: ## Summary - Make bucketed ACK waits configurable through `rollout.checkpoint_engine.update_weights_ack_timeout_seconds` (default: 30 seconds), validate it as positive and finite, and pass it to the vLLM sender. - Bound real ZMQ ACK waits while retaining compatibility with recv-only socket doubles; …

### #7364 — [[rollout] fix: Enable rollout-side profiling](https://github.com/verl-project/verl/pull/7364)
- **作者**: rao-ashish  **时间**: 2026-08-11 22:32 CST
- **摘要**: ### What does this PR do?  This PR modifies the v1 trainer to enable rollout-side profiling.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: "rollout profiling" - [x] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by…

### #7363 — [[ckpt, sft, megatron] fix: finalize Megatron async checkpoint queue on every SFT rank](https://github.com/verl-project/verl/pull/7363)
- **作者**: ewan0x79  **时间**: 2026-08-11 20:28 CST
- **摘要**: ### What does this PR do?  When `checkpoint.async_save=true`, SFT scheduled Megatron distributed checkpoint writes but did not drive or drain the associated async queue from the training loop. Consequently, pending async checkpoint requests were not guaranteed to finalize before training exited, esp…

### #7362 — [[ci, trainer] feat: add fsdpturbo backend engine support.](https://github.com/verl-project/verl/pull/7362)
- **作者**: pengnuoheng  **时间**: 2026-08-11 20:14 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  add fsdpturbo backend engine support.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `me…

### #7361 — [[misc] fix: reduce_metrics crash on empty lists and tighten max/min k…](https://github.com/verl-project/verl/pull/7361)
- **作者**: zacheryasc  **时间**: 2026-08-11 19:13 CST
- **摘要**: ### What does this PR do?  Fix `reduce_metrics` so metric reduction is selected from the final path segment (`/max` or `/min`) instead of any `"max"`/`"min"` substring in the key, and so empty metric lists reduce to `NaN` instead of crashing under max/min reductions. This prevents keys such as `glob…

### #7359 — [[npu] fix: preserve PyTorch environment when installing vLLM](https://github.com/verl-project/verl/pull/7359)
- **作者**: Ray-RP  **时间**: 2026-08-11 18:23 CST
- **摘要**: ## Summary  - Add `--no-build-isolation` to the vLLM editable install in `scripts/install_vllm_mcore_npu.sh`. - Pre-install `setuptools-rust` and `wheel` alongside the existing `setuptools-scm`. - Make the vLLM source install consistent with the following vLLM-Ascend install, which already uses `--n…

### #7358 — [[megatron] feat: bucket packed sequence lengths](https://github.com/verl-project/verl/pull/7358)
- **作者**: ISEEKYAN  **时间**: 2026-08-11 18:05 CST
- **摘要**: ### What does this PR do?  Adds FSDP-compatible `pad_to_length` controls to the Megatron engine and optionally rounds each packed THD micro-batch to a 512-token bucket. This reduces packed-shape variability and associated JIT/compilation overhead while leaving existing behavior unchanged by default …

### #7357 — [[ci] test: migrate workflows from fully_async/one_step_off_policy to v1 separate_async](https://github.com/verl-project/verl/pull/7357)
- **作者**: Begunner  **时间**: 2026-08-11 17:41 CST
- **摘要**: ### What does this PR do?  Add v1 separate_async workflow and disable fully_async/one_step_off_policy workflows, preparing to move fully_async into recipe.

### #7355 — [fix: repair 3D position_ids ragged index selection](https://github.com/verl-project/verl/pull/7355)
- **作者**: oops-debug  **时间**: 2026-08-11 14:51 CST
- **摘要**: ### What does this PR do?  Fixes `index_select_tensor_dict` for 3D jagged `position_ids`.  After TensorDict consolidation and serialization/deserialization, the internal `_ragged_idx` of 3D `position_ids` may incorrectly become `1` instead of `2`. Calling `unbind()` then raises a runtime error or pr…
