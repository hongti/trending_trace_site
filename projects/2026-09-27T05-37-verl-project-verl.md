# verl-project/verl — 动态追踪

> 生成时间: 2026-09-27 13:37 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近的动态摘要：

### Issue 动态
* **#8027 [Feature] Make Hugging Face safetensors export configurable** (作者: lbaolin)
  * **需求背景**：当前训练内的 Hugging Face safetensors 导出路径不够灵活。
  * **期望改进**：请求允许用户自定义 `PreTrainedModel.save_pretrained` 的调用方式、artifact 布局以及 FP32 master weight 的相关处理，以提升用户体验。

### Pull Request 动态
* **#8026 feat: batch-invariant actor forward and sampler log-prob formula** (作者: cr-gao)
  * **重要变更**：为 FSDP 引擎新增了可选配置项 `batch_invariant`（默认关闭）。开启该选项后，系统会安装 vLLM 的 batch-invariant 内核覆盖（`init_batch_invariance`），确保 Actor 前向传播和采样器的 log-prob 计算具有批次不变性。
* **#8025 feat: raw log-prob mismatch metrics for rollout vs actor** (作者: cr-gao)
  * **重要变更**：在原有的 `training/rollout_probs_diff_*` 指标基础上，新增了三个原始 log-prob 不匹配监控指标（例如 `training/rollout_logprobs_mismatch_count` 等）。这些指标用于统计有效的响应 token，帮助更细粒度地对比和监控 rollout 与 actor 之间的 log-prob 差异。

### Release 动态
* 本次提供的近期动态中暂无版本发布（Release）信息。

---

## 🐛 Issues

### #8027 — [[Feature] Make Hugging Face safetensors export configurable](https://github.com/verl-project/verl/issues/8027)
- **作者**: lbaolin  **时间**: 2026-09-27 05:06 CST
- **摘要**: ### Motivation  The current in-training Hugging Face safetensors export path is not very user-friendly. Users cannot customize the `PreTrainedModel.save_pretrained` call or artifact layout, and FP32-master/BF16 mixed-precision training exports FP32 weights unless users perform a separate conversion.…

## 🔀 Pull Requests

### #8026 — [[fsdp, cfg, doc, tests] feat: batch-invariant actor forward and sampler log-prob formula](https://github.com/verl-project/verl/pull/8026)
- **作者**: cr-gao  **时间**: 2026-09-27 04:35 CST
- **摘要**: ### What does this PR do?  Adds an opt-in FSDP engine option `batch_invariant` (default off). When set, the training process installs vLLM's batch-invariant kernel overrides (`init_batch_invariance()`: matmul / softmax / mean / RMSNorm replacements plus the cuBLAS and TF32 settings) and computes log…

### #8025 — [[training_utils, trainer, tests] feat: raw log-prob mismatch metrics for rollout vs actor](https://github.com/verl-project/verl/pull/8025)
- **作者**: cr-gao  **时间**: 2026-09-27 04:11 CST
- **摘要**: ### What does this PR do?  Adds three raw log-prob metrics next to the existing `training/rollout_probs_diff_*`:  - `training/rollout_logprobs_mismatch_count`: valid response tokens whose `rollout_log_probs != old_log_probs` - `training/rollout_logprobs_diff_max` / `_mean`: |Δ| of the raw log-probs …
