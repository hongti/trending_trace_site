# verl-project/verl — 动态追踪

> 生成时间: 2026-08-24 10:09 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 🔄 Issue
本次动态未包含 Issue 相关信息。

### 🚀 Release
本次动态未包含 Release 相关信息。

### 🔧 Pull Request (PR)

**新特性：**
*   **支持加速器亲和资源池** (#7531)：为异构 Ray 集群引入了向后兼容的加速器感知资源调度，新增两个可选（opt-in）配置字段。
*   **训练引擎新增磁盘卸载** (#7530)：针对 FSDP/Megatron/VeOmni，在共置 RL（Colocated RL）场景下支持将非活跃的训练引擎状态卸载至磁盘，从而为其他角色释放宝贵的 HBM 显存。

**重要修复：**
*   **[破坏性变更] TransferQueue 存储单元默认值调整** (#7525)：存储单元数量现在基于 `trainer.nnodes` 动态推导（默认 `max(2, 2 * trainer.nnodes)`），修复了默认值为空引发的问题。
*   **修复 remove-padding 批次的有效长度验证** (#7532)：修复了 `rearrange_micro_batches` 在 padding 移除后，仍按填充前的 padded width 拒绝有效批次的问题。
*   **修复 TransferQueue 元数据静默丢失** (#7529)：修复了 `tqbridge` 丢弃仅含元数据的 `TensorDict` 输出，导致 TrainerV1 训练指标丢失的问题。
*   **修复 Megatron R2 重放填充分布问题** (#7524)：解决了 R2 重放 MoE 路由时，zero-filled 的对齐行未在 EP ranks 间正确分布的问题。
*   **修复 JSON Schema 约束被误剔除** (#7523)：修复了 `@function_tool` 中，Pydantic 模型验证导致显式传入的有效 JSON Schema 约束被意外丢弃的问题。
*   **修复 Slurm 主机名解析** (#7527)：正确将 `scontrol show hostnames` 的输出解析为数组，并增加了空结果或数量不匹配的校验。

**文档更新：**
*   **更新 Agentic RL 训练指南** (#7528)：将文档中已废弃的示例替换为当前 `main` 分支中维护的示例。
*   **补充 ARM64 Apptainer 工作流文档** (#7526)：替换了过时的 amd64-only Docker 镜像说明，增加了多架构感知的容器指导。

---

## 🔀 Pull Requests

### #7532 — [[training_utils] fix: validate remove-padding batches by effective length](https://github.com/verl-project/verl/pull/7532)
- **作者**: liuhao-labs  **时间**: 2026-08-24 09:09 CST
- **摘要**: ### What does this PR do?  Fixes #3088.  `rearrange_micro_batches` currently rejects a dense padded batch when its padded width exceeds `max_token_len`, even when padding is removed and every sample's effective token count fits the limit. The same function then uses effective lengths for micro-batch…

### #7531 — [[ray, trainer, rollout] feat: support accelerator-affine resource pools](https://github.com/verl-project/verl/pull/7531)
- **作者**: zhang-cheng-hao  **时间**: 2026-08-24 00:15 CST
- **摘要**: ### What does this PR do?  This PR adds a minimal, backward-compatible bridge for accelerator-aware resource placement in heterogeneous Ray clusters.  It introduces two opt-in configuration fields:  - `trainer.accelerator_resource_key` - `actor_rollout_ref.rollout.accelerator_resource_key`  The trai…

### #7530 — [[fsdp, megatron, veomni] feat: add disk offload for training engine](https://github.com/verl-project/verl/pull/7530)
- **作者**: ji-huazhong  **时间**: 2026-08-23 23:43 CST
- **摘要**: ### What does this PR do?  Colocated RL reuses accelerators across training and inference phases, but inactive training-engine state must leave HBM before another role can use the device. CPU offload is the preferred tier w dihen host memory is available; on hosts where DRAM is tight relative to agg…

### #7529 — [[worker] fix: preserve metadata-only TransferQueue output](https://github.com/verl-project/verl/pull/7529)
- **作者**: YZJF  **时间**: 2026-08-23 22:33 CST
- **摘要**: ### What does this PR do?  Fixes `tqbridge` silently dropping a metadata-only `TensorDict` output. The TrainerV1 `train_mini_batch` path returns metrics in this form, so those metrics could be lost before reaching the controller.  The root cause was that `_async_update_meta_with_output` assigned `me…

### #7528 — [[doc] fix: refresh agentic RL training guide](https://github.com/verl-project/verl/pull/7528)
- **作者**: liuhao-labs  **时间**: 2026-08-23 20:43 CST
- **摘要**: ### What does this PR do?  Refresh the Agentic RL training guide so that it points to examples that exist and are maintained on the current `main` branch.  - Replace the retired `examples/sglang_multiturn` launcher with the Agent Loop get-started notebook. - Document the current `agent_name` fallbac…

### #7527 — [[deployment] fix: parse Slurm hostnames into node array](https://github.com/verl-project/verl/pull/7527)
- **作者**: liuhao-labs  **时间**: 2026-08-23 19:50 CST
- **摘要**: ### What does this PR do?  Fixes #548 by reading `scontrol show hostnames` output into one Bash array element per allocated node. The Slurm Ray launcher now rejects an empty result or a hostname count that differs from `SLURM_JOB_NUM_NODES` before passing an invalid node constraint to `srun`.  A CPU…

### #7526 — [[doc] fix: document ARM64 Apptainer workflow for Slurm](https://github.com/verl-project/verl/pull/7526)
- **作者**: liuhao-labs  **时间**: 2026-08-23 19:50 CST
- **摘要**: ### What does this PR do?  Fixes #613 by replacing the stale amd64-only Docker image in the Slurm documentation with architecture-aware container guidance.  The canonical multi-node guide keeps a generic registry conversion path for images that publish the target architecture and requires the Apptai…

### #7525 — [[BREAKING][trainer, cfg] fix: derive TransferQueue storage units from node count](https://github.com/verl-project/verl/pull/7525)
- **作者**: liuhao-labs  **时间**: 2026-08-23 19:50 CST
- **摘要**: ### What does this PR do?  Fixes #7398 by making the default number of TransferQueue `SimpleStorage` units depend on `trainer.nnodes`. A null default now resolves to `max(2, 2 * trainer.nnodes)` immediately before TransferQueue initialization, while an explicit `num_data_storage_units` override is p…

### #7524 — [[megatron] fix: distribute R2 replay padding across EP ranks](https://github.com/verl-project/verl/pull/7524)
- **作者**: EazyReal  **时间**: 2026-08-23 17:00 CST
- **摘要**: ### What does this PR do?  Megatron R2 replays the recorded MoE routes during the actor update. THD/BSHD preprocessing can add loss-masked alignment rows, but their zero-filled routes still enter MoE dispatch because R2 has no replay mask. Every top-k slot on those rows therefore selects expert 0.  …

### #7523 — [[tool] fix: preserve explicit JSON Schema constraints](https://github.com/verl-project/verl/pull/7523)
- **作者**: YZJF  **时间**: 2026-08-23 15:25 CST
- **摘要**: ### What does this PR do?  Fixes #7505. Explicit dictionary schemas passed to `@function_tool(..., schema=...)` were validated by narrow Pydantic carrier models, causing valid JSON Schema constraints to disappear before prompt rendering. This change preserves valid extra keywords on parameter and pr…
