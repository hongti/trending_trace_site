# verl-project/verl — 动态追踪

> 生成时间: 2026-09-22 13:18 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 🐛 Issue 提要
1. **Blackwell 架构 MoE 权重同步异常 (#7978)**：在 B200 GPU 上使用 vLLM 0.24 的 TRT-LLM MoE 后端时，bf16 权重同步因 4-D w13/w2 布局在首次同步时触发 `shard_dim` ValueError 报错。
2. **OPD 算法文档需完善 (#7975)**：用户建议在主分支 OPD 文档中增加与 `verl-recipe` 中 GKD 配方的 loss 语义及配置差异对比说明。

### 🚀 Pull Request 提要
**1. 新特性与性能优化**
* **TPU P2P 权重同步引擎 (#7982)**：引入 `RaidenCheckpointEngine`，在 TPU v6e 上通过直接 socket 实现训练器与 vLLM rollout worker 间的点对点权重同步，取代了旧的 Ray Plasma 和 /tmp 磁盘缓存方案。
* **NPU 性能翻倍 (#7980)**：针对 NPU 上的 Qwen3.5 122B 模型提出性能优化方案，在当前序列长度下实现性能翻倍。

**2. 核心依赖与版本升级**
* **Rollout 引擎大版本升级 (#7973)**：将 vLLM 升级至 0.29.0，SGLang 升级至 0.5.20，同时升级 torch (2.13.0) 和 transformers (5.12.1)。

**3. 重要 Bug 修复**
* **Checkpoint 恢复跳过 Epoch 修复 (#7974)**：修复了在 epoch 边界保存 checkpoint时，恢复训练后会静默跳过下一个 epoch 数据加载的问题。

**4. CI 与文档修复 (Ascend 专项)**
* **CI 流水线修复**：修正 Ascend SGLang 工作流中的 DeepSeek 模型路径 (#7979)；在 vLLM Ascend 测试前加载 megatron_adaptor (#7976)；并在 CI 中启用持久化缓存 (#7972)。
* **文档更新**：修复 Ascend 950 安装文档 (#7981)，并同步更新 Ascend 模型支持矩阵 (#7977)。

### 📦 Release 提要
* 近期无新版本发布。

---

## 🐛 Issues

### #7978 — [[rollout, vllm] bf16 MoE weight sync breaks on Blackwell with vLLM 0.24's TRT-LLM MoE backend (4-D w13/w2 layout → shard_dim ValueError at first sync)](https://github.com/verl-project/verl/issues/7978)
- **作者**: wengeezhang  **时间**: 2026-09-21 20:54 CST
- **摘要**: ### System Info  Observed on 1×NVIDIA B200 (SM100, 183 GB, driver 595.91.07, CUDA 13.2 host), image `verlai/verl:vllm024.dev2`:  - verl: `feat/megatron-mxfp8-training` @ `79caec3c` (fork of PR #7519); the standard weight-sync path in question (`verl/workers/rollout/vllm_rollout/utils.py::update_weig…

### #7975 — [[docs, OPD] Clarify loss semantics and configuration differences between mainline OPD and the GKD recipe](https://github.com/verl-project/verl/issues/7975)
- **作者**: XuMeng00124  **时间**: 2026-09-21 16:28 CST
- **摘要**: ### Feature request  Could the [mainline OPD documentation](https://github.com/verl-project/verl/blob/main/docs/algo/opd.md) include a short comparison with the GKD recipe in `verl-recipe`?  While investigating [verl-recipe#151](https://github.com/verl-project/verl-recipe/issues/151), I found that s…

## 🔀 Pull Requests

### #7982 — [feat(tpu): add Raiden P2P weight synchronization engine for multi-host TPU RL](https://github.com/verl-project/verl/pull/7982)
- **作者**: wenjung2007  **时间**: 2026-09-22 11:47 CST
- **摘要**: Introduce RaidenCheckpointEngine for direct socket-based peer-to-peer weight synchronization between TorchTitan trainer and vLLM rollout workers on TPU v6e.  - Replaces Ray Plasma and /tmp disk caching with direct host-to-host TCP streaming,   eliminating disk I/O bottlenecks and NVMe exhaustion ris…

### #7981 — [[doc] fix: fix ascend 950 install readme](https://github.com/verl-project/verl/pull/7981)
- **作者**: yyyy2000  **时间**: 2026-09-22 11:45 CST
- **摘要**: ### What does this PR do?  fix ascend 950 install readme  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`,…

### #7980 — [[perf] chore: qwen3.5 122B performance optimization solution for NPU](https://github.com/verl-project/verl/pull/7980)
- **作者**: zhouhengan1211  **时间**: 2026-09-22 10:47 CST
- **摘要**: ### What does this PR do?  - We have verified that the performance of Qwen3.5 122B can be doubled with the current sequence length on NPU.  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.   ### Checklist Bef…

### #7979 — [[ci] fix: correct DeepSeek model path in Ascend SGLang workflow](https://github.com/verl-project/verl/pull/7979)
- **作者**: lxb007981  **时间**: 2026-09-22 10:44 CST
- **摘要**: ### What does this PR do?  Correct DeepSeek model path in Ascend SGLang workflow  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` inc…

### #7977 — [[doc] fix: sync Ascend model support matrix](https://github.com/verl-project/verl/pull/7977)
- **作者**: LZC-BELIEVER  **时间**: 2026-09-21 20:17 CST
- **标签**: Ascend
- **摘要**: Fixes #5529  ## What changed - update the Ascend model support matrix date - point the existing Qwen3.5-122B-A10B and Qwen3-Next-80B entries to their current Ascend-specific examples  The change is limited to `docs/ascend_tutorial/zh/model_support/model_and_algorithm_support.md`. The referenced Asce…

### #7976 — [[ci] fix: load megatron adaptor in vllm Ascend workflow](https://github.com/verl-project/verl/pull/7976)
- **作者**: lxb007981  **时间**: 2026-09-21 17:00 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Load megatron_adaptor before vLLM rollout tests.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include …

### #7974 — [[trainer, ckpt] fix: skip dataloader state restore when resuming at epoch boundary](https://github.com/verl-project/verl/pull/7974)
- **作者**: ewan0x79  **时间**: 2026-09-21 15:01 CST
- **摘要**: ### What does this PR do?  Fixes #7401. When a checkpoint is saved exactly at an epoch boundary (e.g. `save_freq` is a multiple of `steps_per_epoch`), resuming silently skips the next epoch: the job can exit with status 0 having performed none of the remaining updates, leaving an under-trained model…

### #7973 — [[rollout] chore: upgrade vllm==0.29.0/sglang==0.5.20](https://github.com/verl-project/verl/pull/7973)
- **作者**: wuxibin89  **时间**: 2026-09-21 14:25 CST
- **摘要**: ### What does this PR do?  Upgrade rollout engine version: - torch==2.13.0 - vllm==0.29.0 - sglang==0.5.20 - transformers==5.12.1

### #7972 — [[uv, ci] fix: use persistent cache in CI](https://github.com/verl-project/verl/pull/7972)
- **作者**: ETOgaosion  **时间**: 2026-09-21 12:25 CST
- **摘要**: ### What does this PR do?  As title  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, `sglang`, …
