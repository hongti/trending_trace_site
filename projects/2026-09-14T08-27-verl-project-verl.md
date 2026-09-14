# verl-project/verl — 动态追踪

> 生成时间: 2026-09-14 16:27 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📋 Issue
- **#7856 [RFC] 提出 Tail-aware top-k KL 用于 on-policy distillation**
  作者提议新增一种 `forward_kl_topk_tail` 损失模式。该模式通过聚合 top-k 之外的分布质量，使得截断的 teacher-top-k forward KL 成为一个有效的 coarse-grained KL。

### 🔧 Pull Request
- **#7855 [新特性] 支持 Tail-aware top-k KL**
  实现了上述 Issue 提议的 `forward_kl_topk_tail` 损失模式，为 on-policy distillation 提供更好的截断处理，支持 fsdp、megatron 和 veomni。
- **#7860 [新特性] 新增 GLM-5.2 全异步训练示例**
  在 Ascend NPUs 上新增了 32 节点的 GLM-5.2 GRPO 全异步训练脚本，使用了 Megatron 和 vLLM-Ascend。
- **#7854 [重构] 移除原版 mBridge 并新增 Moonlight NPU 脚本**
  从 MegatronEngine 中移除了原版 mBridge 分支，简化了模型构建、权重转换等流程，并添加了 Moonlight NPU 的运行脚本。
- **#7859 [修复] 隔离共置引擎的内部端口**
  修复了同一节点上并发 vLLM 引擎在启动时可能选中相同内部 TCP 端口的问题，实现了端口隔离。
- **#7858 [修复] 修复 minimal padding 及 forward topk loss 相关问题**
  修复了在 minimal padding 下 teacher fields 的填充问题，并在 forward topk loss 中增加了对 fsdp2 的支持。
- **#7857 [文档] 修正算法文档中的配置键与默认值**
  修复了算法文档中涉及 5 个文件、4 类错误的配置项和默认值描述问题。
- **#7853 [修复] 将沙箱失败报告为运行时错误**
  修复了基于调用的沙箱包装器捕获执行错误却以退出码 0 退出的问题，现在会将其正确报告为运行时错误。

### 🚀 Release
- 近期无新版本发布。

---

## 🐛 Issues

### #7856 — [[RFC] Tail-aware top-k KL for on-policy distillation](https://github.com/verl-project/verl/issues/7856)
- **作者**: PengyuLi7  **时间**: 2026-09-13 18:49 CST
- **摘要**: ### Feature request  A new on-policy distillation loss mode, `forward_kl_topk_tail`, that makes the truncated teacher-top-k forward KL a valid coarse-grained KL by aggregating the out-of-top-k mass into a single tail bucket.  The objective keeps the existing teacher top-k payload (`[S, K]` log-probs…

## 🔀 Pull Requests

### #7860 — [[recipe] feat: add GLM-5.2 GRPO fully-async training example on Ascend NPUs](https://github.com/verl-project/verl/pull/7860)
- **作者**: lxb007981  **时间**: 2026-09-14 15:54 CST
- **摘要**: ### What does this PR do?  Add a 32-node fully-async training script for GLM5.2 using Megatron and vLLM-Ascend on DAPO-Math-17k.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (Thi…

### #7859 — [[rollout, vllm] fix: isolate internal ports across colocated engines](https://github.com/verl-project/verl/pull/7859)
- **作者**: leooop-al  **时间**: 2026-09-14 14:18 CST
- **摘要**: # [rollout, vllm] fix: isolate internal ports across colocated engines  ## What does this PR do?  Prevent concurrent vLLM engines on the same node from selecting the same internal TCP port during startup.  vLLM uses `VLLM_PORT` as the starting point for its internal port scan. During multiprocessing…

### #7858 — [[trainer] fix: pad teacher fields in minimal padding and support fsdp2 in forward topk loss](https://github.com/verl-project/verl/pull/7858)
- **作者**: frelam  **时间**: 2026-09-14 10:43 CST
- **摘要**: ### What does this PR do?  While adding OPD support to verl-omni, we encountered several issues that required changes to verl. We would like to ask whether the following two fixes could be incorporated into the verl codebase:  1. Add fsdp2 support in `compute_topk_loss` 2. Add teacher_ids/teacher_lo…

### #7857 — [[doc] fix: correct config keys and defaults in the algorithm docs](https://github.com/verl-project/verl/pull/7857)
- **作者**: JiangLLM  **时间**: 2026-09-14 06:30 CST
- **摘要**: ### What does this PR do?  The algorithm docs contain four kinds of wrong statements, spread over five files. This PR fixes all of them.   **1. A config key that does not exist?** `docs/algo/grpo.md` (twice), `docs/algo/ppo.md`, and `examples/ppo_trainer/README.md` tell the reader to set `actor_roll…

### #7855 — [[algo, fsdp, megatron, veomni] feat: add tail-aware top-k KL for on-policy distillation](https://github.com/verl-project/verl/pull/7855)
- **作者**: PengyuLi7  **时间**: 2026-09-13 18:45 CST
- **摘要**: ### What does this PR do?  Adds a `forward_kl_topk_tail` loss mode for on-policy distillation.  The existing `forward_kl_topk` objective retains only the teacher's top-k terms:  ``` L_truncated = Σ_{i∈T} p_i (log p_i − log q_i) ```  Because `T` is truncated, the retained masses `P = Σ_{i∈T} p_i` and…

### #7854 — [[megatron] refactor: remove vanilla mBridge and add Moonlight NPU script](https://github.com/verl-project/verl/pull/7854)
- **作者**: fangweii1  **时间**: 2026-09-13 17:34 CST
- **摘要**: [megatron] refactor: remove vanilla mBridge and add Moonlight NPU scripts    ### What does this PR do?    Remove the vanilla mBridge branches from MegatronEngine so model construction, weight conversion, QAT, rollout synchronization, and checkpoint handling consistently use NVIDIA   Megatron-Bridge.…

### #7853 — [[reward] fix: report call-based sandbox failures as runtime errors](https://github.com/verl-project/verl/pull/7853)
- **作者**: aprylewu  **时间**: 2026-09-13 16:50 CST
- **摘要**: ### What does this PR do?  The call-based sandbox wrapper catches execution errors but exits with code 0. If the expected output is empty or `None`, a function that raises can therefore pass and receive full reward. For example, `def solve(x): raise ValueError("failed call")` with input `1` and expe…
