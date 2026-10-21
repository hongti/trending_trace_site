# verl-project/verl — 动态追踪

> 生成时间: 2026-10-21 13:11 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📌 Issue 动态
1. **Ascend 26Q4 路线图** (#8304)：规划了下一季度对华为昇腾硬件的支持目标，包括引入 GLM5.3、kimik3、DeepSeek V4.1 等新模型，以及多项 verl 特性适配。
2. **FSDP 引擎共享前缀打包提案** (#8299)：针对 GRPO 强化学习中同一 prompt 采样多次导致的冗余前向计算问题，提出了在 FSDP 引擎中实现共享前缀打包（Shared-prefix packing）的 RFC，并为未来支持混合线性注意力模型铺路。

### 🚀 PR 动态
**重要新特性与优化：**
- **FSDP 共享前缀打包** (#8300)：落地了 Issue #8299 的提案，在 GRPO 采样时让 actor、ref 和 critic 共享同一 prompt 的前向计算，大幅减少冗余计算提升性能。
- **轮次级过程奖励** (#8305)：为 v1 训练器引入了可选的轮次级过程反馈机制，支持智能体直接输出 `turn_rewards` 或通过配置任务评分器实现。
- **Bayes@N 验证指标** (#8303)：在 V1 同步验证中新增可选的 Bayes@N 准确率摘要功能，为每个数据源提供后验均值和标准差。
- **Profiler 微批次调度支持** (#8297)：将 Profiler 功能扩展支持至微批次级别。

**修复与配置调整：**
- **PPO 裁剪范围修复** (#8302)：修复了未设置非对称边界时 `actor.clip_ratio` 失效的问题，确保 PPO 裁剪范围可控。
- **TPU GRPO 精度优化** (#8301)：在 TPU GRPO 配方中添加 `PROMOTE_F32` 标志，优化 bfloat16 的计算精度表现。
- **Ascend CI 修复** (#8298)：修复了 Ascend 工作流中 GSM8K 数据集路径错误的问题。

**硬件适配与文档：**
- **昇腾 Qwen2.5 训练方案** (#8294)：新增基于 Ascend 910A3 设备的 Qwen2.5-7B OTB 训练配方（基于 Megatron 训练和 vLLM-Ascend 生成）及实践报告。
- **文档修正** (#8296)：在 README 硬件支持列表中将 "Ascend" 规范化为 "Huawei Ascend"。

**CI 维护：**
- **测试新 Runner** (#8295)：测试带有细粒度权限的新 CI runner。

### 📦 Release 动态
- 近期无新版本发布。

---

## 🐛 Issues

### #8304 — [[roadmap][Ascend] verl ascend 26Q4 roadmap](https://github.com/verl-project/verl/issues/8304)
- **作者**: wucong25  **时间**: 2026-10-21 11:10 CST
- **标签**: Ascend
- **摘要**: ### new models ---  - [ ] GLM5.3 - [ ] kimik3 https://github.com/verl-project/verl/pull/7866 - [ ] deepseekv4.1 - [ ] DeepSeek-V4-Flash-Vision-Exp - [ ] HY4-preview  ### verl features support ---   - [ ] Implementation of the Full Vocabulary MOPD and MOPD2 - [ ] Dynamic Resource Scheduling - [ ] PD …

### #8299 — [[RFC] Shared-prefix packing for the FSDP engine (trunk + branches), with a path to hybrid linear-attention models](https://github.com/verl-project/verl/issues/8299)
- **作者**: DeeproChoudhury  **时间**: 2026-10-21 01:14 CST
- **摘要**: ## 1. Summary  In GRPO-style RL in verl every prompt is sampled `rollout.n` times, so the actor, reference and critic each run a forward pass over every (prompt, response) pair and process the same prompt `n` times per group. This RFC adds **shared-prefix packing to the FSDP engine**, where the `n` …

## 🔀 Pull Requests

### #8305 — [[trainer, reward, rollout] feat: add opt-in turn-level process rewards for v1](https://github.com/verl-project/verl/pull/8305)
- **作者**: luzy99  **时间**: 2026-10-21 11:43 CST
- **摘要**: ### What does this PR do?  Add **opt-in turn-level process feedback** to the v1 TransferQueue trainer. Agents can provide `output.extra_fields["turn_rewards"]` directly, or configure task scorers when that field is absent. Process scores shape actor advantages after the existing outcome estimator; o…

### #8303 — [[trainer, cfg, doc] feat: add Bayes@N validation accuracy summaries](https://github.com/verl-project/verl/pull/8303)
- **作者**: mohsenhariri  **时间**: 2026-10-21 05:45 CST
- **摘要**: ### What does this PR do?  Adds opt-in binary [Bayes@N](https://arxiv.org/abs/2510.04265v4) summaries to V1 synchronous validation. Each data source receives a posterior mean and posterior standard deviation for its average success probability over the fixed validation questions, using an independen…

### #8302 — [[cfg] fix: honor actor.clip_ratio when asymmetric bounds are unset](https://github.com/verl-project/verl/pull/8302)
- **作者**: Eliaaazzz  **时间**: 2026-10-21 04:02 CST
- **摘要**: ### What does this PR do?  Make `actor_rollout_ref.actor.clip_ratio` control the PPO clipping range when asymmetric bounds are unset. For example, setting only `clip_ratio=0.5` currently leaves both effective bounds at 0.2. Defaulting `clip_ratio_low` and `clip_ratio_high` to `None` enables the exis…

### #8301 — [[hardware, recipe] fix: set --xla_tpu_bf16_emission_mode=PROMOTE_F32 in the TPU GRPO recipes](https://github.com/verl-project/verl/pull/8301)
- **作者**: gutianyu-google  **时间**: 2026-10-21 01:21 CST
- **摘要**: <!-- repo: verl-project/verl | head: gutianyu-google:tpu7x-bf16-promote-f32 | base: main --> <!-- title: [hardware, recipe] fix: set --xla_tpu_bf16_emission_mode=PROMOTE_F32 in the TPU GRPO recipes --> ### What does this PR do?  Both TPU GRPO recipes (`examples/tpu/grpo/run_qwen3_0_6b_torchtitan.sh`…

### #8300 — [[fsdp, perf, trainer] feat: shared-prefix packing for the FSDP engine](https://github.com/verl-project/verl/pull/8300)
- **作者**: DeeproChoudhury  **时间**: 2026-10-21 01:14 CST
- **摘要**: ### What does this PR do?  The `rollout.n` samples of a GRPO prompt share that prompt, but the actor, ref and critic forwards compute it `n` times. With `fsdp_config.use_prefix_packing=True`, we pack adjacent samples with an identical prompt as one trunk plus one branch per sample, so we compute the…

### #8298 — [[ci] fix: change gsm8k datasets paths in Ascend jobs](https://github.com/verl-project/verl/pull/8298)
- **作者**: dodatboii  **时间**: 2026-10-20 20:40 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Fix the GSM8K source directory in the three Ascend workflows that failed during dataset preprocessing on #8251. They currently pass `${HOME}/.cache/datasets/openai/gsm8k`, which resolves to `/root/.cache/datasets/openai/gsm8k` and raises `FileNotFoundError`.  Use the propo…

### #8297 — [Feature/profiler support schedule to micro batch](https://github.com/verl-project/verl/pull/8297)
- **作者**: ZiyiXu06  **时间**: 2026-10-20 20:04 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #8296 — [[doc] fix: use Huawei Ascend in README hardware support](https://github.com/verl-project/verl/pull/8296)
- **作者**: Rain6ow  **时间**: 2026-10-20 19:34 CST
- **摘要**: Update the hardware support entry in README.md from “Ascend” to “Huawei Ascend” to explicitly identify the hardware vendor.

### #8295 — [[ci] chore: test new runner with fine-grained permissions](https://github.com/verl-project/verl/pull/8295)
- **作者**: lxb007981  **时间**: 2026-10-20 15:03 CST
- **摘要**: ### What does this PR do?  [ci] chore: test new runner with fine-grained permissions  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}`…

### #8294 — [[recipe, doc] feat: add Qwen2.5-7B OTB training on Ascend 910A3](https://github.com/verl-project/verl/pull/8294)
- **作者**: dyuan311  **时间**: 2026-10-20 14:29 CST
- **摘要**: ### What does this PR do?  Adds a Qwen2.5-7B Optimal Token Baseline (OTB) recipe for eight Ascend 910A3 devices with Megatron training and vLLM-Ascend generation, plus a practice report, training curves, and all 101 original metric records. Related task: [verl-ascend-recipe #22](https://github.com/v…
