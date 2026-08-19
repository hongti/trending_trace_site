# verl-project/verl — 动态追踪

> 生成时间: 2026-08-19 10:04 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 🐛 Issue 摘要
近期 Issue 主要集中在**训练器配置失效**与**数据加载逻辑错误**，影响了 SFT 和 PPO 的核心训练流程：
1. **SFT 验证集数据丢失导致 NaN (#7464)**：SFT 验证阶段错误使用了 `drop_last=True` 和训练批次大小，可能导致整个验证集被丢弃并记录出 NaN 的 val/loss。
2. **MoE 路由配置失效 (#7463)**：`actor.router_replay.mode` 配置项被系统忽略，官方文档指向了无效的配置键（dead key）。
3. **Critic 配置传递失败与预算覆盖 (#7462, #7460)**：Critic 的 batching keys 无法正确传递至 `TrainingWorkerConfig`；V1 训练器会错误覆盖 train token budget；同时 `critic.model.model_type` 在构建配置时也被忽略。

---

### 🔀 PR 摘要
近期 PR 包含重要的内核替换、多硬件架构修复及多模态数据处理修复：

**🚀 新特性与重要变更**
* **引入 Liger 融合内核 (#7461)**：当 Liger 可用时，使用 Liger Kernel v0.8.2 的 `LigerFusedLinearScaledCrossEntropyFunction` 替换 verl 原有的实验性融合线性 PPO 输出头实现，提升性能。

**🛠️ 关键修复**
* **FSDP 延迟梯度同步可配置化 (#7458)**：完善了此前合并的 FSDP 延迟梯度同步功能，补全了其配置项契约。
* **保留多模态 Token 类型 ID (#7457)**：修复 `MultiTurnSFTDataset` 错误删除 `mm_token_type_ids` 的问题，该数组对 Gemma-4 构建双向注意力掩码至关重要。
* **修复 ROCm/vLLM CUDA Graphs 缓存问题 (#7455)**：修复了启用 CUDA/HIP Graph Replay 时，ROCm 下 DeepSeek-V4 vLLM 权重重构导致的注意力缓存地址丢失问题。
* **修复 Ascend 镜像构建流程 (#7459)**：解决 QUAY 镜像站自动回收机制导致 ARM 镜像未构建完就被销毁的问题，改为先推送到临时仓库再推至正式仓库。

**🔧 CI 与杂项**
* **更新 Ascend CI 镜像 (#7456)**：适配 A2 机器单节点 8 NPU 的配置。
* **添加 verl-Tinker 新闻 (#7454)**：在文档/仓库中添加了 verl-Tinker 的相关资讯。

---

### 📦 Release 摘要
* 近期无新版 Release 发布。

---

## 🐛 Issues

### #7464 — [[Bug][SFT] Validation uses drop_last=True and train_batch_size, which can drop the entire val set and log NaN val/loss](https://github.com/verl-project/verl/issues/7464)
- **作者**: YeonwooSung  **时间**: 2026-08-19 09:50 CST
- **摘要**: ### System Info  Reproduced by reading current main (`cfacd76a`).  Affects both SFT entry points: - `verl/trainer/sft_trainer.py` (SPMD / torchrun) - `verl/trainer/sft_trainer_ray.py` (Ray)  Default config: `verl/trainer/config/sft_trainer_engine.yaml`  ```yaml data.train_batch_size: 256 data.micro_…

### #7463 — [[config][MoE] actor.router_replay.mode is ignored; official docs point at a no-op key](https://github.com/verl-project/verl/issues/7463)
- **作者**: YeonwooSung  **时间**: 2026-08-19 09:50 CST
- **摘要**: ### System Info  Reproduced on current main (`cfacd76a`).  Affects: - `verl/trainer/config/actor/actor.yaml` (dead key) - `verl/trainer/config/engine/megatron.yaml` and `veomni.yaml` (live key) - `verl/workers/engine_workers.py` (reads only the engine copy) - `docs/ascend_tutorial/dev_guide/model_de…

### #7462 — [[trainer][critic] Critic batching keys never reach TrainingWorkerConfig; V1 also overwrites the train token budget](https://github.com/verl-project/verl/issues/7462)
- **作者**: YeonwooSung  **时间**: 2026-08-19 09:50 CST
- **摘要**: ### System Info  Reproduced by reading current main (`cfacd76a`).  Affects: - `verl/trainer/ppo/ray_trainer.py` (v0) - `verl/trainer/ppo/v1/trainer_base.py` (default `trainer.use_v1=true`) - `verl/experimental/separation/ray_trainer.py` - `verl/workers/engine_workers.py` - `verl/workers/config/engin…

### #7460 — [[trainer] critic.model.model_type is ignored when building TrainingWorkerConfig](https://github.com/verl-project/verl/issues/7460)
- **作者**: liushaohuai5  **时间**: 2026-08-19 07:20 CST
- **标签**: bug
- **摘要**: ### System Info  suggested_fix: model_type=orig_critic_cfg.model.get("model_type", "value_model")  ### Information  - [ ] The official example scripts - [ ] My own modified scripts  ### Tasks  - [ ] An officially supported task in the `examples` folder (such as GLUE/SQuAD, ...) - [ ] My own task or …

## 🔀 Pull Requests

### #7461 — [[training_utils, env, doc] feat: use Liger fused linear PPO kernel](https://github.com/verl-project/verl/pull/7461)
- **作者**: kolehma8  **时间**: 2026-08-19 07:37 CST
- **摘要**: ### What does this PR do?  Closes #7424.  Replaces verl's experimental fused linear PPO output-head implementation with Liger Kernel v0.8.2's `LigerFusedLinearScaledCrossEntropyFunction` when Liger is installed. The wrapper preserves log-probability sign, entropy output, 2D/3D shapes, and gradients;…

### #7459 — [[env] fix: Update ascend image build workflow](https://github.com/verl-project/verl/pull/7459)
- **作者**: yyyy2000  **时间**: 2026-08-18 22:18 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  QUAY镜像网站自动回收机制导致ascend上，amd镜像没有构建完，arm镜像就被销毁了。 pr将单架构镜像首先推送到临时仓库，merge后推送正式仓库  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the…

### #7458 — [[fsdp] fix: make deferred gradient sync configurable](https://github.com/verl-project/verl/pull/7458)
- **作者**: Mengyuyang  **时间**: 2026-08-18 19:49 CST
- **摘要**: <!-- Suggested PR title: [fsdp] fix: make deferred gradient sync configurable -->  ### What does this PR do?  Complete the configuration contract for deferred FSDP gradient synchronization introduced by [#7095](https://github.com/verl-project/verl/pull/7095).  The merged implementation currently def…

### #7457 — [[data] fix: keep mm_token_type_ids in MultiTurnSFTDataset](https://github.com/verl-project/verl/pull/7457)
- **作者**: ZhiliangWu  **时间**: 2026-08-18 17:11 CST
- **摘要**: ### What does this PR do?  `MultiTurnSFTDataset` deletes `mm_token_type_ids` from the processor output. That array carries one entry per token — 0 text, 1 image, 2 video. Gemma-4 uses it to build a bidirectional attention mask for vision inputs, as the patches of one picture have no left-to-right or…

### #7456 — [[ci] chore: Update ascend ci image](https://github.com/verl-project/verl/pull/7456)
- **作者**: lxb007981  **时间**: 2026-08-18 15:48 CST
- **摘要**: ### What does this PR do? A2 machines have 8 npus per node.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI) - `{modules}` include `fsdp`, `megatron`…

### #7455 — [[vllm] fix: preserve ROCm attention cache for CUDA graphs](https://github.com/verl-project/verl/pull/7455)
- **作者**: PeterYang12  **时间**: 2026-08-18 15:45 CST
- **摘要**: ### What does this PR do?  Fixes ROCm DeepSeek-V4 vLLM weight refits when CUDA/HIP graph replay is enabled.  The captured attention graph keeps the address of the cached bf16 `wo_a` tensor. Deleting that cache after a weight refit and rebuilding it lazily allocates new storage, leaving the captured …

### #7454 — [[recipe] chore: Add in news for verl-Tinker](https://github.com/verl-project/verl/pull/7454)
- **作者**: wyettzeng  **时间**: 2026-08-18 09:09 CST
- **摘要**: ### What does this PR do?  Add in news for verl-Tinker  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `…
