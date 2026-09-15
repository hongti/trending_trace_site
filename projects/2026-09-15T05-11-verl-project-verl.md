# verl-project/verl — 动态追踪

> 生成时间: 2026-09-15 13:11 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📋 Issue
- **#7865 separate_async 请求静默搁置问题**：在 V1 的 `separate_async` 模式下，混合 vLLM 副本在首次调用 `switch_to_trainer` 之前会一直留在负载均衡器中。这导致在 `abort_all_requests` 关闭提交门控后，路由到这些副本的请求会被挂起直至验证阶段，从而静默地搁置了提示词组。

### 🔧 Pull Request (PR)
**新特性与功能支持**
- **Ascend NPU 生态支持**：
  - **#7866 支持 Kimi K3 训练**：集成 Kimi K3 多模态 GRPO 训练，支持 FSDP-Turbo 和 Megatron，注意这是一个**破坏性变更**。
  - **#7868 支持 W4A8 MXFP rollout refit**：在 Ascend 950 NPU 设备上引入 W4A8 MXFP 量化 refit 功能。
  - **#7860 GLM-5.2 全异步训练示例**：新增基于 Megatron 和 vLLM-Ascend 的 GLM-5.2 GRPO 32节点全异步训练脚本。
- **#7861 均衡数据混合采样器**：新增配置驱动的 `GroupRatioSampler`，用于 RL/PPO 训练中按指定键（如 `data_source`）进行分组，实现均衡的数据混合。

**修复与性能优化**
- **#7864 修复权重同步性能回退**：修复了从 v0.8.0 升级到 v0.9.0 后，由于在权重恢复前执行全量 GC 导致的吞吐量下降问题。
- **#7862 修复 NPU P2P 通信竞态**：解决了 Megatron 在 Ascend NPU 上开启 `variable_seq_lengths=True` 时，PP 阶段间 `batch_isend_irecv` 引发的 SymInt 报错根因。
- **#7859 隔离共置引擎端口**：修复同一节点上并发运行的 vLLM 引擎可能选中相同内部 TCP 端口的问题，避免启动冲突。

**CI 与维护**
- **#7867**：修复 Ascend docker 镜像中的 `transformers` 版本（固定为 5.10.4）。
- **#7863**：新增在 4 个 A3 节点上运行 glm5.2 (top32-experts) 的脚本。

### 🚀 Release
- 本追踪周期内无新的 Release 版本发布。

---

## 🐛 Issues

### #7865 — [separate_async: requests routed to a hybrid replica after `abort_all_requests` closes the submission gate are parked until validation, silently stranding prompt groups](https://github.com/verl-project/verl/issues/7865)
- **作者**: kocchop  **时间**: 2026-09-15 09:22 CST
- **标签**: bug
- **摘要**: ## Summary  In V1 separate_async, the hybrid vLLM replicas (engines colocated in the trainer processes) stay in the rollout load balancer from init until the first `switch_to_trainer`, which runs in on_step_begin of step 1. on_train_begin dispatches the warmup batch before that, so part of it is rou…

## 🔀 Pull Requests

### #7868 — [[vllm, rollout] feat: support W4A8 MXFP rollout refit on Ascend](https://github.com/verl-project/verl/pull/7868)
- **作者**: zaney9880  **时间**: 2026-09-15 11:34 CST
- **摘要**: Enables W4A8 MXFP rollout refit in veRL on Ascend 950 npu devices.  ### What does this PR do?  We bring W4A8 MXFP quantization refit to verl through vllm-ascend rollout. Working in tandem with the PR <[vllm-ascend#16568](https://github.com/vllm-project/vllm-ascend/pull/16568)>, one can specify `quan…

### #7867 — [[ci] chore: Fix transformers version in Ascend docker images](https://github.com/verl-project/verl/pull/7867)
- **作者**: lxb007981  **时间**: 2026-09-15 11:27 CST
- **摘要**: ### What does this PR do?  Fix transformers version `5.10.4` in Ascend docker images  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}`…

### #7866 — [[fsdp, megatron, rollout, model] feat: support Kimi K3 training on Ascend NPUs](https://github.com/verl-project/verl/pull/7866)
- **作者**: wangdongleix  **时间**: 2026-09-15 10:29 CST
- **摘要**: # Title  [BREAKING][fsdp, megatron, rollout, model] feat: support Kimi K3 training on Ascend NPUs  ### What does this PR do?  Integrates Kimi K3 multimodal GRPO training on Ascend NPUs with FSDP-Turbo and Megatron/MindSpeed-Bridge backends and vLLM-Ascend rollout. Adds shared multimodal processing, …

### #7864 — [[rollout, perf] fix: avoid full GC before weight resume](https://github.com/verl-project/verl/pull/7864)
- **作者**: lbaolin  **时间**: 2026-09-15 07:57 CST
- **摘要**: ### What does this PR do?  Our production RL training observed a throughput regression after upgrading verl v0.8.0 to v0.9.0, with colocated weight sync especially affected. Investigation found that v0.9.0 added `aggressive_empty_cache(force_sync=True)` immediately before rollout weights are resumed…

### #7863 — [[ci] feat: add a script to run glm5.2 (top32-experts) on 4 A3 nodes](https://github.com/verl-project/verl/pull/7863)
- **作者**: lxb007981  **时间**: 2026-09-14 20:07 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #7862 — [Fix/npu p2p shape race](https://github.com/verl-project/verl/pull/7862)
- **作者**: linboqiang0613  **时间**: 2026-09-14 18:40 CST
- **摘要**: # NPU 上 Megatron P2P shape 通信竞态导致 SymInt 报错的根因与修复  ## 1. Bug 描述  ### 一句话描述  Megatron 在 Ascend NPU 上开启 `variable_seq_lengths=True` 时，PP 阶段间的 shape 通信用 `batch_isend_irecv` 批量收发 shape 张量，该 API 在 NPU 上存在「`req.wait()` 已返回但 recv buffer 尚未写入」的竞态，且 `torch.npu.synchronize()` 也无法消除，导致 `.tolist()` 读到未初始化垃圾 sha…

### #7861 — [[data, trainer] feat: add config-driven GroupRatioSampler for balanced data mixing](https://github.com/verl-project/verl/pull/7861)
- **作者**: 5082459  **时间**: 2026-09-14 17:24 CST
- **摘要**: ### What does this PR do?  Add `GroupRatioSampler`, a config-driven dataloader sampler for balanced data mixing in RL/PPO training. It groups dataset rows by a configurable dot-path key (e.g. `data_source` or `extra_info.<field>`) and yields batches with per-group ratios, so minority groups are over…

### #7860 — [[recipe] feat: add GLM-5.2 GRPO fully-async training example on Ascend NPUs](https://github.com/verl-project/verl/pull/7860)
- **作者**: lxb007981  **时间**: 2026-09-14 15:54 CST
- **摘要**: ### What does this PR do?  Add a 32-node fully-async training script for GLM5.2 using Megatron and vLLM-Ascend on DAPO-Math-17k.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (Thi…

### #7859 — [[rollout, vllm] fix: isolate internal ports across colocated engines](https://github.com/verl-project/verl/pull/7859)
- **作者**: leooop-al  **时间**: 2026-09-14 14:18 CST
- **摘要**: # [rollout, vllm] fix: isolate internal ports across colocated engines  ## What does this PR do?  Prevent concurrent vLLM engines on the same node from selecting the same internal TCP port during startup.  vLLM uses `VLLM_PORT` as the starting point for its internal port scan. During multiprocessing…
