# verl-project/verl — 动态追踪

> 生成时间: 2026-11-15 15:00 CST

## AI 总结

# verl-project/verl 近期动态摘要

## Issue

- **#8691 [RFC] Rollout replica 重启不中断 Trainer**
  作者 HaoWang113 提出 RFC，讨论在不打断训练流程的情况下实现 rollout 副本重启。该 RFC 是 #8380 的子方案（rollout 侧），对应 26Q4 路线图中的相关目标，当前处于 Draft 状态，并已有 demo 验证。

## Pull Request

**性能优化**

- **#8689** 重构 vLLM 采样 token 的 log-prob 读取方式，改为从扁平列表读取，替代逐 token 的 `Logprob` 字典，同时让 prompt log-prob 读取器兼容扁平列表，降低开销。
- **#8687** 为 mstx profiler 新增调度配置（skip_first/wait/warmup/active/repeat），用于在 Actor/Critic 更新存在大量 mini-batch 时控制 trace 体积，避免生成过大 trace。

**Bug 修复**

- **#8693** 修复 vLLM 在 IPC 分桶权重同步期间的 MTP 完整性校验问题：单桶可能不含 MTP 层权重，因此跳过逐 load 校验。
- **#8690** 修复 RL-Insight 初始化时机问题：`RLInsightLogger.trace_state()` 可能在 logger 提供 project/experiment 名称之前调用，导致提前初始化；改为延迟初始化。
- **#8688** 当系统缺少 `tilelang` 时，FP8 导出自动回退到 PyTorch 实现（数值等价），使 `tilelang` 成为可选依赖。

**新特性**

- **#8686** 启用 Ascend PD（prefill/decode 分离）架构，通过 `MooncakeConnectorV1` 在 verl 现有单节点 1P+ND 与非对称 TP 实现之上支持昇腾平台。

**CI / 基础设施**

- **#8692** 为 ROCm runner 添加 Python bootstrap hook（sitecustomize），在 e2e 容器启动训练前预先初始化 ROCm 设备清单。

## Release

近期无新的 Release。

---

**总体要点**：本周活动集中在 rollout 稳定性与性能（vLLM log-prob 重构、MTP 校验修复、FP8 回退）、昇腾 PD 分离部署支持，以及训练侧 profiler 调度与 RL-Insight 初始化修复。#8691 的 RFC 为后续"训练不中断重启 rollout"奠定设计基础。

---

## 🐛 Issues

### #8691 — [[RFC] Rollout replica restart without interrupting the trainer (sub-RFC of #8380, demo-validated)](https://github.com/verl-project/verl/issues/8691)
- **作者**: HaoWang113  **时间**: 2026-11-14 23:49 CST
- **摘要**: > **Status**: Draft | > **Related**: sub-RFC of #8380 (rollout side), covering the 26Q4 roadmap #8186 item "Rollout replica restart without interrupting trainer" | > **Positioning**: the detailed rollout-side design, refining the rollout-recovery section of #8492. Route A is the store-compatible col…

## 🔀 Pull Requests

### #8693 — [[vllm] fix: skip MTP completeness checks during bucketed weight sync](https://github.com/verl-project/verl/pull/8693)
- **作者**: lxb007981  **时间**: 2026-11-15 11:46 CST
- **摘要**: ### What does this PR do?  Disable per-load MTP completeness validation during unquantized IPC weight transfers, since individual buckets may not contain MTP layer weights. Include the initial base-weight sync for LoRA while leaving adapter-only and quantized updates unchanged.  Use a no-op context …

### #8692 — [ci: add ROCm runner python bootstrap hook](https://github.com/verl-project/verl/pull/8692)
- **作者**: miramar-labs  **时间**: 2026-11-15 09:07 CST
- **摘要**: Adds a repository-level sitecustomize hook that primes the ROCm device inventory before the e2e containers start training.

### #8690 — [[training_utils] fix: defer RL-Insight initialization during state tracing](https://github.com/verl-project/verl/pull/8690)
- **作者**: mayunaise  **时间**: 2026-11-14 22:21 CST
- **摘要**: ### What does this PR do?  `RLInsightLogger.trace_state()` can run before the training logger supplies its configured project and experiment names. Calling `_ensure_rl_insight_init()` there initializes the SDK with defaults too early.  Remove that call so early state traces reach the SDK's pre-initi…

### #8689 — [[vllm, rollout, perf] refactor: read sampled-token log-probs from flat lists](https://github.com/verl-project/verl/pull/8689)
- **作者**: hxw12219  **时间**: 2026-11-14 15:38 CST
- **摘要**: ### What does this PR do?  Reads sampled-token log-probs from vLLM's flat lists instead of one `Logprob` dict per generated token, and makes the prompt log-prob reader work with flat lists. Outputs are unchanged. In a standalone AsyncLLM benchmark on one B300, rollout wall time fell from 6.99 s to 6…

### #8688 — [[rollout, hardware] fix: fall back to PyTorch for FP8 export without TileLang](https://github.com/verl-project/verl/pull/8688)
- **作者**: peterzhang987723  **时间**: 2026-11-14 15:36 CST
- **摘要**: ### What does this PR do?  Make `tilelang` an optional import in `verl/utils/qat/fp8.py` and provide a numerically equivalent PyTorch fallback for `fp8_quant` and `fp8_grouped_fake_quant` when `tilelang` is not installed or when running on ROCm (`torch.version.hip is not None`).  On ROCm images wher…

### #8687 — [[perf] feat: mstx profiler support schedule to actor update mini batch](https://github.com/verl-project/verl/pull/8687)
- **作者**: SoraAzzz  **时间**: 2026-11-14 14:56 CST
- **摘要**: ### What does this PR do? When Actor/Critic update has many mini-batches, full traces can be huge. This PR lets users configure schedule (skip_first / wait / warmup / active / repeat) under profiler.tool_config.npu, so only selected mini-batches are recorded. verl advances the schedule by calling pr…

### #8686 — [[rollout, vllm] feat: enable Ascend PD with MooncakeConnectorV1](https://github.com/verl-project/verl/pull/8686)
- **作者**: Mengyuyang  **时间**: 2026-11-14 14:44 CST
- **摘要**: ### What does this PR do?  Enable Ascend prefill/decode disaggregation through `MooncakeConnectorV1` on top of verl's existing single-node 1P+ND and asymmetric-TP implementation. The current base rejects NPU at launch and configures GPU connectors. This change supplies the Ascend connector/port conf…
