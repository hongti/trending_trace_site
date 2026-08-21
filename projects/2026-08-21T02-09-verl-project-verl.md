# verl-project/verl — 动态追踪

> 生成时间: 2026-08-21 10:09 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 🚨 Issue 动态
近期主要报告了训练与分布式同步相关的关键问题：
1. **异步 LoRA 权重同步失败（#7495）**：在解耦架构下，LoRA adapter 无法正常同步至 rollout 引擎，导致 rollout 一直从初始策略采样。
2. **FSDP1 梯度同步静默失效（#7493）**：在多 GPU 环境下设置 `fsdp_size=1` 时，梯度同步会被悄悄禁用，影响训练正确性。
3. **大模型训练 OOM（#7489）**：使用当前脚本训练 Qwen2.5 32B 模型时频繁遭遇显存溢出（OOM）问题。

---

### 🔀 PR 动态
PR 活动主要集中在修复上述关键 Bug、引入新算法以及完善底层硬件支持：

**✨ 新特性**
* **引入 CoDaPO 算法（#7497）**：新增来自 ICML 2026 的算法 *Confidence and Difficulty-Adaptive Policy Optimization (CoDaPO)*，该算法对 GRPO 进行了扩展，引入了置信度与难度自适应加权及 top-K 机制。

**🛠️ 核心修复**
* **修复异步 LoRA 同步问题（#7496）**：解决 #7495，确保在默认的异步 LoRA RL 路径下，LoRA adapter 能正确同步到解耦的 rollout 引擎。
* **修复 FSDP 梯度同步失效（#7494）**：解决 #7493，修正了 `fsdp_size=1` 时设备网格构建和分片策略的选择逻辑。
* **修复 AgentLoop 字段丢失（#7491）**：修复 `AgentLoopOutput.as_dict()` 在依赖默认值时意外省略 `extra_fields` 的问题。
* **修复 NPU 权重更新 HCCL 错误（#7487, #7488）**：通过回退特定提交，修复了 NPU（Ascend）上更新权重时引发的 HCCL 错误。

**🤖 CI 与测试**
* **支持 AMD 平台自动检测（#7498）**：在平台自动检测测试中加入 AMD/ROCm，支持空环境变量解析及 `VERL_PLATFORM=amd` 覆盖。
* **Ascend CI 修复（#7492, #7490）**：回退 Ascend CI 上的 Megatron 版本以修复错误，并提交了修复向量核心错误的测试 PR（仅用于触发 CI，请勿合并）。

---

### 📦 Release 动态
近期暂无版本发布信息。

---

## 🐛 Issues

### #7495 — [[BUG][LoRA][Async] Disaggregated weight sync never delivers the LoRA adapter — rollout keeps sampling from the initial policy](https://github.com/verl-project/verl/issues/7495)
- **作者**: ChaoyuWang04  **时间**: 2026-08-20 19:16 CST
- **标签**: bug
- **摘要**: ### System Info  ``` ----------Python Info---------- Version      : 3.12.14 Compiler     : Clang 22.1.3  Build        : ('main', 'Aug 14 2026 15:34:45') Arch         : ('64bit', 'ELF') ------------Pip Info----------- No corresponding pip install for current python. vllm	     : 0.12.0 sglang	     : n…

### #7493 — [[BUG][FSDP1] fsdp_size=1 on multiple GPUs silently disables gradient synchronization](https://github.com/verl-project/verl/issues/7493)
- **作者**: ChaoyuWang04  **时间**: 2026-08-20 17:22 CST
- **标签**: bug
- **摘要**: ### System Info  ``` ----------Python Info---------- Version      : 3.12.14 Compiler     : Clang 22.1.3  Build        : ('main', 'Aug 14 2026 15:34:45') Arch         : ('64bit', 'ELF') ------------Pip Info----------- No corresponding pip install for current python. vllm	     : 0.12.0 sglang	     : n…

### #7489 — [训练Qwen2.5 32B时总是内存OOM](https://github.com/verl-project/verl/issues/7489)
- **作者**: SeTsuGeka  **时间**: 2026-08-20 11:48 CST
- **摘要**: 脚本如下 adv_estimator=gae  use_kl_in_reward=False kl_coef=0.0 use_kl_loss=False kl_loss_coef=0.0  clip_ratio_low=0.2 clip_ratio_high=0.28  max_prompt_length=$((1024 * 2)) max_response_length=$((1024 * 20)) enable_overlong_buffer=True overlong_buffer_len=$((1024 * 4)) overlong_penalty_factor=1.0  loss_a…

## 🔀 Pull Requests

### #7498 — [[test] include AMD in platform auto-detection expectations](https://github.com/verl-project/verl/pull/7498)
- **作者**: tmm77  **时间**: 2026-08-21 01:52 CST
- **摘要**: ## Summary - Include AMD/ROCm in platform auto-detection tests alongside nvidia and huawei. - Empty `VERL_PLATFORM` may resolve to `amd`; assert `amd` is registered; add a `VERL_PLATFORM=amd` override test so ROCm CI matches existing coverage.  ## Test plan - [ ] `pytest tests/plugin/test_platform_a…

### #7497 — [[algo, trainer, doc] feat: add CoDaPO](https://github.com/verl-project/verl/pull/7497)
- **作者**: X1angyuLu  **时间**: 2026-08-20 21:42 CST
- **摘要**: ### What does this PR do?  Add [Confidence and Difficulty-Adaptive Policy Optimization (CoDaPO)](https://arxiv.org/abs/2606.07950) (ICML 2026) to verl. CoDaPO extends GRPO with CoDaWeighting, top-K CoDaSampling, and paired original/focused CoDaLearning updates.  The implementation registers a CoDaPO…

### #7496 — [[rollout, fully_async] fix: sync LoRA adapters to disaggregated rollout engines](https://github.com/verl-project/verl/pull/7496)
- **作者**: ChaoyuWang04  **时间**: 2026-08-20 19:18 CST
- **摘要**: ### What does this PR do?  Fixes #7495 .  This is the default path for async LoRA RL: `model.lora.merge=False` is the default and every disaggregated recipe uses a non-`naive` checkpoint-engine backend. In that configuration, every weight sync pushed the unmodified frozen base model and never transf…

### #7494 — [[fsdp] fix: fsdp_size=1 silently disables gradient synchronization](https://github.com/verl-project/verl/pull/7494)
- **作者**: ChaoyuWang04  **时间**: 2026-08-20 17:47 CST
- **标签**: wontfix
- **摘要**: ### What does this PR do?  Fixes #7493.  `create_device_mesh(world_size, fsdp_size=1)` builds a `(world_size, 1)` mesh whose shard dim is degenerate. `get_sharding_strategy` selected `HYBRID_SHARD` for it, which FSDP1 clamps to `NO_SHARD` (the shard group holds a single rank) **while still reducing …

### #7492 — [[ci] fix: revert megatron version on ascend ci](https://github.com/verl-project/verl/pull/7492)
- **作者**: lxb007981  **时间**: 2026-08-20 17:06 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do? Revert megatron version. Only affect ascend ci.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI) - `{modules}` include `fsd…

### #7491 — [[rollout] fix: preserve default AgentLoop extra fields](https://github.com/verl-project/verl/pull/7491)
- **作者**: le-czs  **时间**: 2026-08-20 16:04 CST
- **摘要**: ### What does this PR do?  Fixes #7486. `AgentLoopOutput.as_dict()` uses `model_dump(exclude_unset=True)`, which omits `extra_fields` when callers rely on the declared default. Mutating that default mapping after construction does not add the field to Pydantic's fields-set, so the existing teacher-f…

### #7490 — [[test ascend ci]Fix vector core error](https://github.com/verl-project/verl/pull/7490)
- **作者**: lxb007981  **时间**: 2026-08-20 15:50 CST
- **摘要**: ### What does this PR do?  This pr is for test purpose. It will invoke ascend workflows only. Do not merge.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by …

### #7488 — [[ckpt] fix: hccl error when update weights for npu](https://github.com/verl-project/verl/pull/7488)
- **作者**: RichardFido  **时间**: 2026-08-20 11:35 CST
- **标签**: Ascend
- **摘要**: This reverts commit 352f76f59ff014c6b423eed1a21a7881be1a0b46.  ### What does this PR do? #7205 will cause hccl error when update weights for npu  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklis…

### #7487 — [[ckpt] fix: hccl error when update weights for npu](https://github.com/verl-project/verl/pull/7487)
- **作者**: RichardFido  **时间**: 2026-08-20 11:33 CST
- **标签**: Ascend
- **摘要**: This reverts commit 352f76f59ff014c6b423eed1a21a7881be1a0b46.  ### What does this PR do? #7205 will cause hccl error when update weights for npu  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklis…
