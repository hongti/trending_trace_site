# verl-project/verl — 动态追踪

> 生成时间: 2026-08-08 10:30 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
近期主要反馈了两个运行与配置问题：
*   **更新后报错** (#7320)：用户拉取最新代码后，在设置 `VLLM_USE_V1=1` 运行时出现错误。
*   **Agent 训练配置覆盖失败** (#7319)：使用 sandbox tool 训练 agent 时，无法覆盖 `data.apply_chat_template_kwargs.enable_thinking` 参数。

---

### 🔀 PR 动态
近期合并了多个修复与功能增强，重点改善了检查点、PPO 训练配置及 Ray 运行时环境：

**✨ 新特性**
*   **支持 Worker 级环境变量** (#7312)：新增 `trainer.worker_env` 配置项，允许为 PPO trainer 管理的 Ray worker 组单独设置环境变量（兼容新旧版 PPO trainer）。

**🛠️ 核心修复**
*   **修复混合精度检查点崩溃** (#7318)：修复了 `nccl`/`hccl`/`nixl` 检查点引擎在打包混合精度权重流时，因偏移量未对齐 `dtype itemsize` 导致的潜在崩溃。
*   **修复 HFRollout 重复惩罚参数失效** (#7314)：修复了 `HFRollout` 忽略配置中的 `repetition_penalty`，错误地继承 checkpoint 默认值的问题。
*   **修复本地检查点未清理问题** (#7315)：修复了 `del_local_ckpt_after_load` 参数生效时，未能从本地磁盘删除已加载检查点目录的问题。
*   **修复 PPO 配置缺失报错** (#7316)：在 PPO YAML 配置中补充了 `trainer.max_ckpt_to_keep` 键，修复了因该键缺失导致的 Hydra struct-mode 错误。
*   **修复 FSDP1 保存/加载路由** (#7317)：为 `DetachActorWorker` 中的 FSDP1 分配了专用的保存/加载处理器（注：已被 #7306 替代并合并）。
*   **移除冲突的 PYTHONPATH** (#7313)：移除了 Ray 运行时环境自动转发的 `PYTHONPATH`，避免与用户通过 `ray job submit` 设置的路径发生冲突。

**📝 文档与 CI**
*   **修复 CI 镜像上传** (#7311)：修复了 Docker 镜像上传失败的问题。
*   **更新 NPU 相关文档** (#7310, #7309)：修复了 NPU 依赖安装说明，并更新了针对 verl v0.8.0 的 NPU 使用指南。

---

### 🚀 Release 动态
*   近期**无**新版本发布记录。

---

## 🐛 Issues

### #7320 — [after update latest code , raise errors](https://github.com/verl-project/verl/issues/7320)
- **作者**: cqray1990  **时间**: 2026-08-08 10:07 CST
- **标签**: bug
- **摘要**: ### System Info                set -x                              export VLLM_USE_V1=1                              # ================= data/model/tool =================               HDFS_ROOT=${HDFS_ROOT:-$PWD}               DATA_ROOT=${DATA_ROOT:-$PWD}                              #dapo_math_17k…

### #7319 — [Could not override 'data.apply_chat_template_kwargs.enable_thinking'.](https://github.com/verl-project/verl/issues/7319)
- **作者**: cqray1990  **时间**: 2026-08-08 09:09 CST
- **标签**: bug
- **摘要**: ### System Info  verl code is latest,when  training agent with sandbox tool,set data.apply_chat_template_kwargs.enable_thinking is not right, Could not override 'data.apply_chat_template_kwargs.enable_thinking'. To append to your config use +data.apply_chat_template_kwargs.enable_thinking=False Key …

## 🔀 Pull Requests

### #7318 — [[ckpt] fix: align packed tensor offsets to dtype itemsize for mixed-dtype streams](https://github.com/verl-project/verl/pull/7318)
- **作者**: savaresejeremy  **时间**: 2026-08-08 05:32 CST
- **摘要**: ### What does this PR do?  Fixes a latent crash in the checkpoint-engine wire for mixed-dtype weight streams.  The `nccl`/`hccl`/`nixl` checkpoint engines pack tensors back-to-back into a uint8 bucket and record each start in `TensorMeta.offset`. On receive, each single-chunk tensor is materialized …

### #7317 — [fix: route FSDP1 to dedicated save/load handlers in DetachActorWorker (#7249)](https://github.com/verl-project/verl/pull/7317)
- **作者**: shotsan  **时间**: 2026-08-08 02:11 CST
- **摘要**: Superseded by #7306 which was merged upstream with the same fix.

### #7316 — [fix: add trainer.max_ckpt_to_keep to PPO config (#7295)](https://github.com/verl-project/verl/pull/7316)
- **作者**: shotsan  **时间**: 2026-08-08 02:11 CST
- **摘要**: ## Summary  Fixes #7295 — `trainer.max_ckpt_to_keep` cannot be set on PPO training runs because the key does not exist in the PPO YAML config, causing a Hydra struct-mode error.  Supersedes #7301 (closed due to unsigned CLA, now signed).  ## Root Cause  The PPO trainer YAML (`ppo_trainer.yaml`) defi…

### #7315 — [fix: del_local_ckpt_after_load now removes local checkpoint directories (#7213)](https://github.com/verl-project/verl/pull/7315)
- **作者**: shotsan  **时间**: 2026-08-08 02:11 CST
- **摘要**: ## Summary  Fixes #7213 — `del_local_ckpt_after_load` does not remove loaded checkpoints from local disk.  Supersedes #7299 (closed due to unsigned CLA, now signed).  ## Root Cause  The cleanup code in both FSDP and Megatron checkpoint managers had the same two bugs:  ### Bug 1: `is_non_local()` gat…

### #7314 — [fix: pass repetition_penalty from config to GenerationConfig in HFRollout (#7270)](https://github.com/verl-project/verl/pull/7314)
- **作者**: shotsan  **时间**: 2026-08-08 02:10 CST
- **摘要**: ## Summary  Fixes #7270 — `HFRollout` ignores `rollout.repetition_penalty`, silently inheriting the checkpoint's `generation_config.json` value instead of using the user's configured value.  Supersedes #7298 (closed due to unsigned CLA, now signed).  ## Root Cause  `HFRollout._generate_minibatch` (`…

### #7313 — [[ray] fix: remove conflicting PYTHONPATH forwarding](https://github.com/verl-project/verl/pull/7313)
- **作者**: yyDing1  **时间**: 2026-08-07 18:44 CST
- **摘要**: ### What does this PR do?  Removes automatic `PYTHONPATH` forwarding from `get_ppo_ray_runtime_env()`.  When a job submitted through `ray job submit --runtime-env` already defines `PYTHONPATH`, verl adds the same key again to `ray.init(runtime_env=...)`. Ray rejects the merge and training fails befo…

### #7312 — [[trainer, ray] feat: support worker-scoped environment variables](https://github.com/verl-project/verl/pull/7312)
- **作者**: YolandaLyj  **时间**: 2026-08-07 18:31 CST
- **摘要**: ### What does this PR do?  This PR adds `trainer.worker_env`, allowing environment variables to be applied specifically to Ray worker groups managed by the PPO trainer.  Both the legacy and V1 PPO trainers resolve and forward this mapping to `RayWorkerGroup(worker_env=...)`. No environment variables…

### #7311 — [[ci] chore: Fix docker image upload](https://github.com/verl-project/verl/pull/7311)
- **作者**: LeoYao123  **时间**: 2026-08-07 17:58 CST
- **摘要**: ### What does this PR do? Fix docker image upload > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] F…

### #7310 — [[doc] fix: install NPU requirements](https://github.com/verl-project/verl/pull/7310)
- **作者**: Mengyuyang  **时间**: 2026-08-07 17:32 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  fix: install NPU requirements  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ...…

### #7309 — [[doc] fix: update NPU guidance verl0.8.0](https://github.com/verl-project/verl/pull/7309)
- **作者**: Mengyuyang  **时间**: 2026-08-07 17:29 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review. update NPU guidance verl0.8.0  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... …
