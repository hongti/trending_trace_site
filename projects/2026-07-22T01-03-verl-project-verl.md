# verl-project/verl — 动态追踪

> 生成时间: 2026-07-22 09:03 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文简洁摘要：

### 🚨 Issue 动态
*   **[#7110] Value Loss 梯度累积缩放问题质疑**：作者 x-zb 提出，在数据并行（DP）的梯度累积场景下，`value_loss()` 可能缺少正确的缩放处理代码。相比之下，`ppo_loss()` 和 `sft_loss()` 均有相关缩放参数（如 `batch_n`），这可能是一个潜在的损失计算 Bug。

### 🛠 Pull Request 动态
近期 PR 主要聚焦于**底层性能优化、Checkpoint 同步架构升级、多引擎兼容性修复及依赖更新**，重要变更如下：

**1. 架构与性能突破（亮点）**
*   **[#7108] 引入 P2P Checkpoint 引擎 (Mooncake RDMA)**：重大新特性！为 Megatron trainer 到 SGLang rollout 的权重同步新增了 `p2p` 后端。该方案通过 Mooncake RDMA 直接推送权重，无需构建 NCCL 进程组，极大降低了跨引擎同步的开销。
*   **[#7102] V1 Trainer 支持 RemoteBackend 插件**：扩展了 `RemoteBackend` 插件抽象，使其正式支持 V1 trainer，完善了进程外 RL 后端（训练+采样）的架构 seam。
*   **[#7105] MSTX Profiler 支持调度至 Mini Batch**：性能优化特性，增强了 Profiler 对 Mini Batch 级别的调度支持。

**2. 关键 Bug 修复**
*   **[#7107] NCCL Checkpoint 桶大小广播修复**：修复了 NCCL checkpoint engine 在桶未填满（常见于最后一个桶）时仍强行广播完整 `bucket_size` 缓冲区的内存/性能浪费问题。
*   **[#7106] 保留 MoE R2 Router 决策跨 THD 批次重放**：修复了 Megatron MoE 路由决策在 THD-packed batches 中丢失的问题，并增强了嵌套多模态配置遍历的鲁棒性。
*   **[#7103] FSDP2 CPU Offload 下跳过手动 `model.to(device)`**：针对 VeOmni 引擎，修复了 FSDP2 CPU offload 模式下 DTensor storage-device 不匹配的问题（跟进 #6604）。
*   **[#7104] PrecisionDebugger 模型解析修复**：修复了 PrecisionDebugger 在接收 Megatron 模型块序列时的解析错误，并增加了多模块存在时的警告。

**3. 功能增强与依赖更新**
*   **[#7109] 保留 Tinker Forward/Backward 的模型输出**：新增特性，使得 Tinker 前向后向过程中的模型输出可被保留供服务端使用。
*   **[#7101] 升级 vLLM 与 Megatron 版本**：将核心依赖 vLLM 升级至 `0.24.0`，Megatron 升级至 `core_v0.18.0`。

### 📦 Release 动态
*   近期**无**新版本 Release 发布。

---

## 🐛 Issues

### #7110 — [Is the value loss scaled properly for gradient accumulation in value_loss() in verl/workers/utils/losses.py?](https://github.com/verl-project/verl/issues/7110)
- **作者**: x-zb  **时间**: 2026-07-22 04:26 CST
- **摘要**: I'm wondering if the value loss is correctly scaled for gradient accumulation in DP, since I cannot find any code doing this. On the contrast, in ppo_loss() and sft_loss(), parameters such as `batch_num_tokens` and `dp_size` are passed to the loss function for scaling.

## 🔀 Pull Requests

### #7109 — [[recipe] feat: Retain model output from tinker forward backward](https://github.com/verl-project/verl/pull/7109)
- **作者**: wyettzeng  **时间**: 2026-07-22 01:01 CST
- **摘要**: ### What does this PR do?  Retain model output from tinker forward backward for server use  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: https://github.com/verl-project/verl/commit/1ff76cc625e9820d2434dad1b6d9b8e5dd26a359 - [x] Format the PR title …

### #7108 — [[ckpt, sglang, megatron] feat: P2P checkpoint engine for trainer-to-rollout weight sync via Mooncake RDMA](https://github.com/verl-project/verl/pull/7108)
- **作者**: AkiRusProd  **时间**: 2026-07-22 00:18 CST
- **摘要**: ### What does this PR do?  Adds a new checkpoint-engine backend `p2p` that pushes weights from Megatron trainer ranks directly into SGLang rollout engines over Mooncake RDMA, without building an NCCL process group between trainer and rollout. Each trainer source rank exports its PP-local shard of HF…

### #7107 — [[ckpt]: nccl broadcast bucket size fix](https://github.com/verl-project/verl/pull/7107)
- **作者**: parinayc20  **时间**: 2026-07-21 23:47 CST
- **摘要**: ### What does this PR do?  The NCCL checkpoint engine always broadcasts the full bucket_size buffer per bucket, even when the bucket was only partially filled (which is the common case for the last bucket of every send_weights call, and any time the parameter stream doesn't divide evenly into bucket…

### #7106 — [[Bugfix] Preserve R2 router replay across THD Megatron batches](https://github.com/verl-project/verl/pull/7106)
- **作者**: hbhflw2000  **时间**: 2026-07-21 20:33 CST
- **摘要**: ## Summary  - preserve and replay Megatron MoE routing decisions across THD-packed batches - make nested multimodal Megatron config traversal robust - treat `routed_experts=None` as absent during R2 RECORD/REPLAY selection and fail clearly if an update receives no recorded routes - rebase onto curre…

### #7105 — [[perf] feat: mstx profiler support schedule to mini batch](https://github.com/verl-project/verl/pull/7105)
- **作者**: mengchengTang  **时间**: 2026-07-21 17:39 CST
- **摘要**: ### What does this PR do?  flow https://github.com/verl-project/verl/pull/7099   ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` incl…

### #7104 — [Fix PrecisionDebugger model resolution for Megatron](https://github.com/verl-project/verl/pull/7104)
- **作者**: Tjh-UKN  **时间**: 2026-07-21 16:07 CST
- **标签**: Ascend
- **摘要**: ## Summary - Resolve a callable Megatron model chunk when PrecisionDebugger receives `engine.module` as a sequence. - Warn when multiple chunks exist because msprobe currently binds one module per debugger invocation.  ## Duplicate-work check - No open PR exists for head `Tjh-UKN:fix/precision-debug…

### #7103 — [[veomni, fsdp] fix: skip manual model.to(device) under FSDP2 CPU offload](https://github.com/verl-project/verl/pull/7103)
- **作者**: cben484  **时间**: 2026-07-21 12:06 CST
- **摘要**: ### What does this PR do?  Follow-up to #6604 for the **VeOmni engine**.  #6604 fixed a DTensor storage-device mismatch under FSDP2 CPU offload for the **FSDP engine** (`workers/engine/fsdp/transformer_impl.py`) by guarding the manual `model.to(device)` calls with `_uses_fsdp2_cpu_offload_policy`. T…

### #7102 — [[trainer] feat: add V1 trainer support for the RemoteBackend plugin abstraction](https://github.com/verl-project/verl/pull/7102)
- **作者**: sfc-gh-kganesan  **时间**: 2026-07-21 11:22 CST
- **摘要**: # V1 trainer support for the `RemoteBackend` plugin abstraction  ## Summary  Adds V1 trainer support for the `RemoteBackend` plugin abstraction. The V0 seam for out-of-process RL backends (training + sampling that live outside verl core, e.g. Arctic-Platform's DeepSpeed/vLLM stack) lands in verl-pro…

### #7101 — [[docker] feat: upgrade vllm and megatron version](https://github.com/verl-project/verl/pull/7101)
- **作者**: ETOgaosion  **时间**: 2026-07-21 10:42 CST
- **摘要**: ### What does this PR do?  upgrade vllm (0.24.0) and megatron (core_v0.18.0) version  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}`…
