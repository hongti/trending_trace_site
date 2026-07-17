# verl-project/verl — 动态追踪

> 生成时间: 2026-07-17 09:04 CST

## AI 总结

以下是 **verl-project/verl** 仓库最近动态的中文简洁摘要：

### 🚫 Issue 动态
本次统计周期内无相关 Issue 动态。

### 🚫 Release 动态
本次统计周期内无版本发布动态。

### ✅ PR 动态
近期 PR 主要集中在**分布式训练架构扩展、关键计算与存储 Bug 修复，以及 CI/文档维护**：

🌟 **重要新特性 / 架构扩展**
*   **#7080 [ckpt,trainer] Block-placement 分片增量同步及 FSDP2+EP 支持**：将分片增量权重同步（delta sync）从原本的平面 FSDP2 `Shard(0)` 扩展至 block-placement（`Shard(k)` 及多维网格），并新增 Automodel 的 FSDP2+EP 支持，大幅提升大模型 checkpoint 同步的灵活性。
*   **#7082 [trainer] V1 Replay Buffer 驱逐/重填充统一处理**：统一了 V1 ReplayBuffer 对过时、DAPO 过滤及失败 rollout 组的处理逻辑，并为 V1 trainer 增加了针对 DAPO 组的模式特定重填充功能。

🛠️ **关键 Bug 修复**
*   **#7083 [checkpoint_engine] CUDA 流同步修复**：修复了 NCCL broadcast 仅将内核入队而不等待 GPU 执行完成的问题，加入了 CUDA stream 同步机制。
*   **#7077 [ckpt] FSDP2 CPUOffload Checkpoint 保存修复**：修复了 FSDP2 使用 `CPUOffloadPolicy` 时，`save_checkpoint` 错误调用 GPU 加载函数导致 DTensor 序列化异常的问题。
*   **#7075 [trainer] MoE 负载均衡指标偏差修复**：排除了从未被路由的最终响应 token（EOS/停止符），修复了 rollout MoE load-balance metrics 的系统性偏差。
*   **#7079 [trainer] 投机解码指标规范化**：统一支持从 NumPy-like 容器和 `tensordict` LinkedList 中提取并规范化投机解码指标。
*   **#7076 [vllm] 恢复 vllm patch**：修复并重新启用了 vllm 的补丁代码。

📦 **CI / 文档 / 维护**
*   **#7081 [ci] 修复 NPU nightly CI**：调整了 `save_freq` 解决异常退出问题，并更新了 bridge 版本以适应默认值变更。
*   **#7078 [ci] 增加 megatron-bridge 测试文档**。
*   **#7074 [doc] 修复 Ascend 文档链接**：修复了路径迁移后失效的 Ascend 教程链接，并清理了包含不可见换行符的 shell 脚本文件名。

---

## 🔀 Pull Requests

### #7083 — [[checkpoint_engine] fix: added cuda stream synchronization in NCCL broadcast wait for complete](https://github.com/verl-project/verl/pull/7083)
- **作者**: parinayc20  **时间**: 2026-07-16 23:12 CST
- **摘要**: ### What does this PR do?  `ray.util.collective.broadcast()` only *enqueues* the NCCL kernel on ray's internal stream pool and returns before it finishes on the GPU (ray never calls `record_stream()` on the buffer for this path either). `BroadcastOperation.wait_for_complete()` awaited only that enqu…

### #7082 — [[trainer] V1 replay buffer eviction/refill handling for stale, DAPO-filtered, and failed rollout groups](https://github.com/verl-project/verl/pull/7082)
- **作者**: Begunner  **时间**: 2026-07-16 22:11 CST
- **摘要**: ### What does this PR do?  1. Unify V1 ReplayBuffer eviction/refill handling for stale, DAPO-filtered, and failed rollout groups. 2. Add DAPO group filtering for V1 trainers with mode-specific refill behavior. 3. Prevent late trajectory writes by publishing terminal prompt status only after all sess…

### #7081 — [[ci] chore: fix nightly ci of npu](https://github.com/verl-project/verl/pull/7081)
- **作者**: zhouhengan1211  **时间**: 2026-07-16 20:52 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  - Editing save_freq to solve the problem of abnormal exit. - The version of the bridge used is changed due to the change of the default value of vanilla_mbridge. This problem needs to be fixed. - The megatron-bridge is deleted from the Docker environment. Therefore, we nee…

### #7080 — [[ckpt,trainer] feat: block-placement sharded delta sync + Automodel FSDP2+EP support](https://github.com/verl-project/verl/pull/7080)
- **作者**: ChangyiYang  **时间**: 2026-07-16 19:51 CST
- **摘要**: ## What does this PR do?  Extends the sharded delta weight sync (`delta_sharded`, #6974) beyond the flat FSDP2 `Shard(0)` fast path to **block placements** — `Shard(k)` and multi-Shard-dim meshes — and wires the **Automodel (nemo_automodel) engine with expert parallelism** into it end to end. Item 1…

### #7079 — [[trainer] fix: support linked-list speculative metrics](https://github.com/verl-project/verl/pull/7079)
- **作者**: 1ring2rta  **时间**: 2026-07-16 19:43 CST
- **摘要**: ## What  - Normalize speculative-decoding rollout metrics from both NumPy-like containers and `tensordict` `LinkedList` values. - Reuse the normalization helper in both PPO trainer implementations. - Add a CPU-only regression test covering the linked-list path and metric aggregation.  ## Why  The V1…

### #7078 — [[ci] doc: test meagtron-bridge](https://github.com/verl-project/verl/pull/7078)
- **作者**: yyyy2000  **时间**: 2026-07-16 18:28 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #7077 — [fix/support-checkpointsave_cpuoffload](https://github.com/verl-project/verl/pull/7077)
- **作者**: Lzw-lukeskywalker  **时间**: 2026-07-16 17:51 CST
- **摘要**: ### What does this PR do?  >When using FSDP2 with `CPUOffloadPolicy`, `save_checkpoint()` incorrectly calls `load_fsdp_model_to_gpu()`, which triggers `model.state_dict()` → DTensor serialization → device mismatch crash on Ascend NPU.  ### Checklist Before Starting  - [ ] Search for similar PRs. Pas…

### #7076 — [[vllm] fix: restore vllm patch](https://github.com/verl-project/verl/pull/7076)
- **作者**: tardis-key  **时间**: 2026-07-16 17:43 CST
- **摘要**: ### What does this PR do?  enable vllm patch  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, `…

### #7075 — [[trainer] fix: exclude the never-routed final response token from rollout MoE load-balance metrics](https://github.com/verl-project/verl/pull/7075)
- **作者**: zsnoob  **时间**: 2026-07-16 17:36 CST
- **摘要**: ### What does this PR do?  Fixes a systematic bias in the rollout MoE load-balance metrics introduced in #6853.  The last generated token of each sequence (EOS / max-length stop) is sampled but never fed back through the model, so it has **no routing record**: SGLang's capturer intentionally returns…

### #7074 — [[doc] fix: repair broken Ascend tutorial links and shell filename](https://github.com/verl-project/verl/pull/7074)
- **作者**: chengminhua  **时间**: 2026-07-16 17:26 CST
- **摘要**: ### What does this PR do?  Fix broken Ascend documentation links after the quickstart path migration, and clean up related invalid example links / a shell script filename that contained an invisible LRM character.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query…
