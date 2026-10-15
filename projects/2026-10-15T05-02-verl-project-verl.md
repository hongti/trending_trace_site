# verl-project/verl — 动态追踪

> 生成时间: 2026-10-15 13:02 CST

## AI 总结

# verl-project/verl 近期动态摘要

## Issue
本次未提供相关 Issue 记录。

## Pull Request

### 🌟 重要新特性
- **#8246**：在 **FSDP2 引擎**上实现 **Step-5-Preview**（FP8 块量化 MoE + 稀疏 DSA 注意力）的 RL 支持，覆盖 GRPO + LoRA + vLLM rollout，属于重要的模型与训练能力扩展。

### 🔧 硬件适配与修复
- **#8243**：更新 TPU GRPO 示例，适配 `verl-hardware-plugin` 主分支（Ray 2.59 镜像、pip 安装插件、移除 NOSET）。
- **#8239**：修复 Ray `< 2.59.0` 下 TPU GRPO 示例（`TPU_VISIBLE_CHIPS`），并验证 Qwen3-4B 完整 100 步训练端到端稳定运行，解决 HBM 碎片/异常问题。

### 📝 文档修复
- **#8248**：修复 `rollout_corr_math.md` §3.2.2 中公式渲染问题（IS/RS 校正策略梯度损失的形式化推导）。
- **#8247**：在 Ascend GRPO 模型支持表中补充 **GLM-5.2**。

### 📦 依赖升级（Dependabot）
| PR | 依赖 | 升级范围 |
|----|------|----------|
| #8245 | tokenizers | 0.21 → 0.23.3 |
| #8244 | peft | ≥0.15.2 → ≥0.21.2 |
| #8242 | trl | 0.27.0 → 1.14.2 |
| #8241 | av | ≥10.0,<19.0 → ≥17.1.0,<19.0 |
| #8240 | transformers | 5.12.1 → 5.19.0 |

## Release
本次未提供相关 Release 记录。

---
**总体评价**：本期最值得关注的是 **Step-5-Preview FP8 MoE 模型在 FSDP2 上的 GRPO+LoRA 训练支持**（#8246），以及 TPU 平台的多项稳定性修复。此外有较大规模的依赖版本升级，其中 trl 从 0.27 跨大版本升级到 1.14，transformers 与 peft 均有显著更新，建议关注兼容性。

---

## 🔀 Pull Requests

### #8248 — [[doc] fix: render display math in rollout_corr_math §3.2.2 derivation](https://github.com/verl-project/verl/pull/8248)
- **作者**: tguens  **时间**: 2026-10-15 12:22 CST
- **摘要**: ### What does this PR do?  Fixes the rendering of the "Formal derivation" in §3.2.2 *Policy Gradient Loss (with IS/RS Correction)* of [`docs/algo/rollout_corr_math.md`](https://verl.readthedocs.io/en/latest/algo/rollout_corr_math.html#policy-gradient-loss-with-is-rs-correction).  In steps 2–4, each …

### #8247 — [[doc] chore: add GLM-5.2 to Ascend GRPO model support table](https://github.com/verl-project/verl/pull/8247)
- **作者**: lxb007981  **时间**: 2026-10-15 12:00 CST
- **摘要**: ### What does this PR do?  Add GLM-5.2 to Ascend GRPO model support table  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `f…

### #8246 — [[model, fsdp, rollout, vllm] feat: Step-5-Preview FP8 GRPO with LoRA on FSDP2](https://github.com/verl-project/verl/pull/8246)
- **作者**: aoshen02  **时间**: 2026-10-15 10:33 CST
- **摘要**: ### What does this PR do?  Adds RL support for **Step-5-Preview** (FP8 block-quantized MoE with sparse DSA attention) on the FSDP2 engine. It covers GRPO with LoRA against a vLLM rollout, rollout routing replay (R3), and adapter-only weight sync.  verl has no Step5 model, so the PR adds a verl-nativ…

### #8245 — [build(deps): bump tokenizers from 0.21 to 0.23.3](https://github.com/verl-project/verl/pull/8245)
- **作者**: dependabot[bot]  **时间**: 2026-10-15 05:38 CST
- **标签**: dependencies, python
- **摘要**: Bumps [tokenizers](https://github.com/huggingface/tokenizers) from 0.21 to 0.23.3. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/tokenizers/releases">tokenizers's releases</a>.</em></p> <blockquote> <h2>Release v0.23.3 (python only)</h2> <h1>R…

### #8244 — [build(deps): update peft requirement from >=0.15.2 to >=0.21.2](https://github.com/verl-project/verl/pull/8244)
- **作者**: dependabot[bot]  **时间**: 2026-10-15 05:17 CST
- **标签**: dependencies, python
- **摘要**: Updates the requirements on [peft](https://github.com/huggingface/peft) to permit the latest version. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/peft/releases">peft's releases</a>.</em></p> <blockquote> <h2>v0.21.2</h2> <p>This is a PEFT re…

### #8243 — [[hardware, doc] fix: update the TPU GRPO example for verl-hardware-plugin main (Ray 2.59 image, pip-installed plugin, no NOSET)](https://github.com/verl-project/verl/pull/8243)
- **作者**: xiaoxlu  **时间**: 2026-10-15 05:14 CST
- **摘要**: ### What does this PR do?  Makes `examples/tpu/grpo` (scripts, README and image) work with `verl-hardware-plugin` `main`, and prepares it for the plugin loading its vLLM patches through an entry point. Only `examples/tpu/grpo/` changes.  1. **Drop `RAY_EXPERIMENTAL_NOSET_TPU_VISIBLE_CHIPS`** from bo…

### #8242 — [build(deps-dev): bump trl from 0.27.0 to 1.14.2](https://github.com/verl-project/verl/pull/8242)
- **作者**: dependabot[bot]  **时间**: 2026-10-15 05:02 CST
- **标签**: dependencies, python
- **摘要**: Bumps [trl](https://github.com/huggingface/trl) from 0.27.0 to 1.14.2. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/trl/releases">trl's releases</a>.</em></p> <blockquote> <h2>v1.14.2</h2> <p>Patch release fixing two cases of silently wrong t…

### #8241 — [build(deps): update av requirement from <19.0,>=10.0 to >=17.1.0,<19.0](https://github.com/verl-project/verl/pull/8241)
- **作者**: dependabot[bot]  **时间**: 2026-10-15 05:02 CST
- **标签**: dependencies, python
- **摘要**: Updates the requirements on [av](https://github.com/PyAV-Org/PyAV) to permit the latest version. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/PyAV-Org/PyAV/releases">av's releases</a>.</em></p> <blockquote> <h2>v17.1.0</h2> <h3>Breaking</h3> <ul> <li>Rem…

### #8240 — [build(deps): bump transformers from 5.12.1 to 5.19.0](https://github.com/verl-project/verl/pull/8240)
- **作者**: dependabot[bot]  **时间**: 2026-10-15 05:01 CST
- **标签**: dependencies, python
- **摘要**: Bumps [transformers](https://github.com/huggingface/transformers) from 5.12.1 to 5.19.0. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/transformers/releases">transformers's releases</a>.</em></p> <blockquote> <h1>Release v5.19.0</h1> <h2>New M…

### #8239 — [[hardware, doc] fix: update TPU GRPO examples for Ray TPU_VISIBLE_CHIPS and Qwen3-4B 100-step stability](https://github.com/verl-project/verl/pull/8239)
- **作者**: jialei777  **时间**: 2026-10-15 03:25 CST
- **摘要**: ### What does this PR do?  Fixes the TPU GRPO examples (`examples/tpu/grpo`) on Ray `< 2.59.0` and ensures the full 100-step Qwen3-4B GRPO training run completes end-to-end without HBM fragmentation/exhaustion:  1. **Remove `RAY_EXPERIMENTAL_NOSET_TPU_VISIBLE_CHIPS=1` from TPU GRPO scripts and `READ…
