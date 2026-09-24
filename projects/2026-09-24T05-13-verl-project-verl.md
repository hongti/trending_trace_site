# verl-project/verl — 动态追踪

> 生成时间: 2026-09-24 13:13 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📋 Issue（议题）
*   **#8009 [RFC] bypass 模式下的评分中心化支持**
    *   作者提出在 bypass 模式下引入加性（additive）的 rollout-training 不匹配纠正机制（即“评分中心化”），以替代目前仅依赖乘性重加权或掩码的方法。配套的 PR #8010 已在准备中。
*   **#8005 询问是否支持 DeepSeek-V4.1**
    *   社区用户询问 veRL 未来是否计划支持 DeepSeek-V4.1 的训练与 rollout，以及预期的时间线。

### 🔧 Pull Request（拉取请求）
**新特性与算法增强：**
*   **#8010 bypass-mode REINFORCE 的评分中心化**：实现了 Issue #8009 的提案，为 bypass 模式下的 REINFORCE 损失函数引入评分中心化机制。
*   **#8004 支持 FP32 lm_head 投影**：为 FSDP actor/reference 模型和 vLLM rollout 引擎新增可选的 FP32 `lm_head` 投影功能（可通过配置 `lm_head_dtype: float32` 开启），有助于提升计算精度。
*   **#8002 新增 NPU 上的 Uni-Agent Docker 镜像**：为 Ascend NPU 环境下的 Uni-Agent 提供了开箱即用的 Docker 镜像。

**Bug 修复与优化：**
*   **#8003 修复 Megatron SFT 异步检查点报错**：修复了在启用 `checkpoint.async_save=true` 时，因缺失 `async_calls` 模块导致的错误，确保 SFT 退出前能正常排空异步检查点。
*   **#8001 修复 Ascend NPU 上的 Triton 内核回退问题**：阻止 HF/FSDP 融合输出头在 Ascend NPU 上错误调用 Triton 实现，使其安全回退到 Torch 融合内核。
*   **#8006 修复 Qwen3-8B NPU 示例中的注意力问题**：通过设置 paged attention 形状列表，修复了 vllm-ascend 中的 FIA decode mask 回退问题。

**测试与依赖更新：**
*   **#8007** 为 Ascend 相关文档添加了 doctests 测试。
*   **#8008** 通过 Dependabot 将 `peft` 依赖要求从 >=0.15.2 升级至 >=0.21.0。

### 🚀 Release（版本发布）
*   本期动态中 **无新的版本发布**。

---

## 🐛 Issues

### #8009 — [[RFC] Support score centering: additive rollout-training mismatch correction in bypass mode](https://github.com/verl-project/verl/issues/8009)
- **作者**: cr-gao  **时间**: 2026-09-24 05:59 CST
- **摘要**: **PR #8010 in preparation.**  ### Motivation  Every rollout correction in verl today is multiplicative: TIS / MIS / IcePop / KPop / TOPR / CPPO all reweight or mask the sampled token by a ratio of trainer and sampler probabilities. [Score Centering Stabilizes Off-policy Reinforcement Learning](https…

### #8005 — [Is there a plan to support DeepSeek-V4.1 in veRL?](https://github.com/verl-project/verl/issues/8005)
- **作者**: franklwy  **时间**: 2026-09-23 17:37 CST
- **摘要**: ### Feature request  Could the maintainers share whether veRL plans to support DeepSeek-V4.1? In particular, is support for training and rollout being considered, and is there an expected timeline?  ### Motivation  Knowing the support plan would help users plan their experiments and decide which bac…

## 🔀 Pull Requests

### #8010 — [[rollout, vllm, algo, fsdp, trainer, doc] feat: score centering for bypass-mode REINFORCE](https://github.com/verl-project/verl/pull/8010)
- **作者**: cr-gao  **时间**: 2026-09-24 07:14 CST
- **摘要**: ### What does this PR do?  Adds score centering ([Marek & Ryabinin, 2026](https://arxiv.org/abs/2609.20807)) to the bypass-mode REINFORCE loss. Implements RFC #8009.  Under training-inference mismatch the policy gradient carries a drift term `E_q[R] * E_q[∇log p]` that distills the trainer toward th…

### #8008 — [build(deps): update peft requirement from >=0.15.2 to >=0.21.0](https://github.com/verl-project/verl/pull/8008)
- **作者**: dependabot[bot]  **时间**: 2026-09-24 01:33 CST
- **标签**: dependencies, python
- **摘要**: Updates the requirements on [peft](https://github.com/huggingface/peft) to permit the latest version. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/peft/releases">peft's releases</a>.</em></p> <blockquote> <h2>v0.21.0</h2> <h1>Highlights</h1> …

### #8007 — [test: add Ascend documentation doctests](https://github.com/verl-project/verl/pull/8007)
- **作者**: likaizheng666-sketch  **时间**: 2026-09-23 19:20 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do? add Ascend documentation doctests > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: .…

### #8006 — [[recipe] fix: use paged attention in Qwen3-8B NPU example](https://github.com/verl-project/verl/pull/8006)
- **作者**: lxb007981  **时间**: 2026-09-23 17:48 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Set pa_shape_list to avoid the FIA decode mask regression in vllm-ascend (vllm-project/vllm-ascend#16889).  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}`…

### #8004 — [[fsdp, vllm] feat: support FP32 lm_head projection](https://github.com/verl-project/verl/pull/8004)
- **作者**: andakai  **时间**: 2026-09-23 17:30 CST
- **摘要**: ## Summary  This PR adds an opt-in FP32 `lm_head` projection shared by FSDP actor/reference models and the vLLM rollout engine:  ```yaml actor_rollout_ref:   model:     lm_head_dtype: float32 ```  The projection uses BF16 hidden states and weights while retaining its output and probability computati…

### #8003 — [[ckpt, sft, megatron] fix: drain async checkpoints before SFT exits](https://github.com/verl-project/verl/pull/8003)
- **作者**: ewan0x79  **时间**: 2026-09-23 15:51 CST
- **摘要**: ### What does this PR do?  Fixes #8000. With Megatron SFT and `checkpoint.async_save=true`, the manager imports `strategies.base.async_calls`, which is absent from the pinned Megatron-Core `core_v0.18.0`; the first save cannot schedule its async requests. After scheduling is corrected, both SFT loop…

### #8002 — [[env] feat: Add a docker image for Uni-Agent on NPU](https://github.com/verl-project/verl/pull/8002)
- **作者**: zhouhengan1211  **时间**: 2026-09-23 15:19 CST
- **摘要**: ### What does this PR do?  - Creating an docker image provided for the Uni-Agent  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least …

### #8001 — [[model, doc, test] fix: fall back to torch fused kernels on Ascend](https://github.com/verl-project/verl/pull/8001)
- **作者**: LZC-BELIEVER  **时间**: 2026-09-23 15:02 CST
- **摘要**: # [model, doc, test] fix: fall back to torch fused kernels on Ascend  ## What does this PR do?  This PR prevents the HF/FSDP fused output-head path from selecting the Triton implementation on Ascend NPU. When `actor_rollout_ref.model.fused_kernel_options.impl_backend` is explicitly set to `triton` a…
