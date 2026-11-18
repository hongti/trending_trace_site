# verl-project/verl — 动态追踪

> 生成时间: 2026-11-18 15:52 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期的动态摘要：

### 📌 Issue 概览
本期共有 2 个重要 Issue，主要涉及架构优化提案与硬件兼容性崩溃：
* **#8738 [RFC] 共享 Prompt 预处理与 TransferQueue 存储**：提议在 V1 trainer 中，让同一个 prompt 的 `rollout.n` 条轨迹共享预处理过程和存储，以减少当前重复执行带来的冗余开销。
* **#8735 FSDP `batch_invariant=True` 在 A100/L40S (SM8x) 上崩溃**：报告在 SM8x 架构的 GPU 上开启 `batch_invariant` 时，首次执行 `loss.backward()` 即触发 `aten::linear_backward` CUDA 错误。

### 🔧 PR 概览
本期合并了 9 个 PR，重点在于**关键错误修复**、**Ascend NPU 兼容性改进**与**性能优化**：

**关键修复：**
* **#8736 修复 SM8x 上的 FSDP backward 崩溃**：针对 Issue #8735，通过跳过 vLLM 的 `aten::linear/matmul` 覆写，修复了 A100/L40S 上的反向传播报错。
* **#8727 修复 Qwen3.5 线性注意力循环状态泄漏**：在未安装 `flash-linear-attention` (FLA) 的环境下（如纯 PyTorch 或 Ascend NPU），使用 `use_remove_padding=True` 会导致状态泄漏，此 PR 予以修复。
* **#8729 修复 math_dapo 奖励计算中的多变量赋值丢失问题**：此前 `normalize_final_answer()` 会粗暴截取等号后的内容，导致定量推理的多变量赋值被破坏，现已修复保留。
* **#8731 修复 Qwen2-MoE MFU 估算漏算共享专家 (shared expert) 的问题**：纠正了 `qwen2_moe` 和 `qwen3_moe` 的算力利用率（FLOPS）估算逻辑。

**性能与功能改进：**
* **#8728 优化 PPO loss 计算性能**：使用静态形状的按位掩码 (`& response_mask`) 替代了动态形状的布尔索引，避免了硬件上的性能损耗。
* **#8737 增强 Profiler 权重同步捕获**：保持训练 profiler 在 step-end 权重同步阶段处于激活状态，并将 `update_weights` 标注为独立阶段。

**CI 与文档维护：**
* **#8734** 为 NPU 单元测试开启 Python fault handling (`PYTHONFAULTHANDLER=1`)，以便在解释器关闭时捕获偶发的段错误日志。
* **#8732** 添加诊断代码，用于排查 Ascend HCCL 初始化失败（错误码 7）的问题。
* **#8733** 测试 Ascend 新的 CI runner 标签。
* **#8730** 在 `RolloutTraceConfig.backend` 的文档中补充 `trackio` 选项。

### 🚀 Release 概览
* 近期无新版本发布。

---

## 🐛 Issues

### #8738 — [[RFC] Share prompt preprocessing and TransferQueue storage across the rollouts of a prompt](https://github.com/verl-project/verl/issues/8738)
- **作者**: OutstanderWang  **时间**: 2026-11-18 15:47 CST
- **摘要**: ## Summary  In the V1 trainer, the `rollout.n` trajectories of one prompt today repeat both the preprocessing and the storage of that prompt. This RFC proposes making them share it: the prompt is decoded and run through the processor once, and its dataset fields and `multi_modal_inputs` are stored o…

### #8735 — [fsdp_config.batch_invariant=True crashes at the first backward on A100 / L40S (SM8x)](https://github.com/verl-project/verl/issues/8735)
- **作者**: XLOverflow  **时间**: 2026-11-18 11:43 CST
- **摘要**: ### System Info  - verl main `b0871fd8` (code path unchanged on current main `565bc5ca`) - Image `verlai/verl:uv.cu130.dev5` + `uv sync --frozen --extra vllm --extra fsdp`: python 3.12, torch 2.13.0+cu130, vLLM 0.32.0, transformers 5.16.1 - 1× A100-80GB / L40S / H100  ### Information  - [ ] The offi…

## 🔀 Pull Requests

### #8737 — [[profiler] fix: capture weight synchronization](https://github.com/verl-project/verl/pull/8737)
- **作者**: luelueFLY  **时间**: 2026-11-18 14:22 CST
- **摘要**: Keep the training profiler active through step-end weight synchronization and annotate update_weights as a separate stage. Preserve coroutine execution across profiling contexts for NPU, torch, and Nsight backends.  Cover step/rank selection, coroutine awaits, exceptions, cancellation, checkpoint or…

### #8736 — [[fsdp, doc] fix: batch_invariant backward on SM8x (skip vLLM's aten::linear/matmul overrides)](https://github.com/verl-project/verl/pull/8736)
- **作者**: XLOverflow  **时间**: 2026-11-18 11:43 CST
- **摘要**: ### What does this PR do?  Fixes #8735. With `fsdp_config.batch_invariant=True` (#8026), the first `loss.backward()` on A100/L40S fails with `Could not run 'aten::linear_backward' ... 'CUDA' backend`. On the SM80 family vLLM's `init_batch_invariance()` also registers CUDA kernels for `aten::linear`/…

### #8734 — [[ci] chore: enable Python fault handling to diagnose intermittent NPU segfaults](https://github.com/verl-project/verl/pull/8734)
- **作者**: lxb007981  **时间**: 2026-11-18 11:22 CST
- **摘要**: ### What does this PR do?  Set PYTHONFAULTHANDLER=1 for the NPU unit test job and the Ascend transformer and engine tests. This keeps fault handling enabled during interpreter shutdown so CI logs can capture a Python traceback if the segfault recurs after pytest finishes.  ### Checklist Before Start…

### #8733 — [[ci] chore: test Ascend new ci runner label](https://github.com/verl-project/verl/pull/8733)
- **作者**: lxb007981  **时间**: 2026-11-18 11:07 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  [ci] chore: test Ascend new ci runner label  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp…

### #8732 — [[ci, fsdp, single_controller] test: correlate Ascend HCCL initialization failures](https://github.com/verl-project/verl/pull/8732)
- **作者**: Rain6ow  **时间**: 2026-11-18 10:59 CST
- **摘要**: ### Problem and purpose  Replace #8722 with a diagnostic draft that covers all three reported Ascend HCCL initialization failure sites. The same `hcclCommInitRootInfoConfig(...), error code is 7 (resource unavailable)` appears in these logs, but a shared root cause has not been established:  | Scena…

### #8731 — [fix: count the shared expert in Qwen2-MoE MFU estimation](https://github.com/verl-project/verl/pull/8731)
- **作者**: FeiyangP  **时间**: 2026-11-18 10:47 CST
- **摘要**: ### What does this PR do?  `_estimate_qwen2_moe_flops` (in `verl/utils/flops_counter.py`, used for both `qwen2_moe` and `qwen3_moe`) counts the routed experts and the router, but not the **shared expert** that Qwen2-MoE runs on every token: a gate/up/down MLP of width `shared_expert_intermediate_siz…

### #8730 — [[doc] add trackio to RolloutTraceConfig.backend docstring](https://github.com/verl-project/verl/pull/8730)
- **作者**: tx-queyu  **时间**: 2026-11-18 10:34 CST
- **摘要**: ## Summary  - Add `trackio` to the documented values of `RolloutTraceConfig.backend`. - The class docstring already mentions Trackio and `RolloutTraceConfig.init()` has a `trackio` branch, so the argument description was out of sync with the implementation.  ## Related issue  Fixes #8570  ## Validat…

### #8729 — [fix(reward): preserve multi-variable assignments in math_dapo (#8675)](https://github.com/verl-project/verl/pull/8729)
- **作者**: shotsan  **时间**: 2026-11-18 10:23 CST
- **摘要**: <!-- Describe the purpose of this PR. -->  ## Motivation  Fixes #8675.  In `verl/utils/reward_score/math_dapo.py`, `normalize_final_answer()` unconditionally applied `final_answer.split("=")[-1]`. When evaluating quantitative reasoning questions with multi-variable answers (such as `"x=1, y=3"`): 1.…

### #8728 — [[worker, perf, hardware] fix: avoid dynamic-shape boolean indexing in ppo_loss mismatch count](https://github.com/verl-project/verl/pull/8728)
- **作者**: askhat-g  **时间**: 2026-11-18 09:35 CST
- **摘要**: ### What does this PR do?  Replaces dynamic-shape boolean indexing (`[response_mask]`) with static-shape bitwise masking (`& response_mask`) when computing `actor/old_logprobs_mismatch_count` in `ppo_loss` (introduced in #8026).  It fixes the XLA re-compilation issue due to boolean indexing ([respon…

### #8727 — [fix(qwen3_5): prevent linear attention recurrent state leakage without FLA (#8629)](https://github.com/verl-project/verl/pull/8727)
- **作者**: shotsan  **时间**: 2026-11-18 07:39 CST
- **摘要**: <!-- Thank you for contributing to verl! Please read the contribution guidelines: https://verl.readthedocs.io/en/latest/community/contributing.html -->  ### What does this PR do?  Resolves #8629.  When `use_remove_padding=True` is enabled without `flash-linear-attention` (e.g., Pure PyTorch executio…
