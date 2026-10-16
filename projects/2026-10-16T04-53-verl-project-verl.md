# verl-project/verl — 动态追踪

> 生成时间: 2026-10-16 12:53 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📋 Issue 动态
*   **新特性探讨 (RFC)**：提议在训练正式开始前（即 trainer 和推理引擎均加载完权重后），增加一次可选的权重一致性校验机制，以防权重更新出错（#8252）。
*   **问题排查**：针对异步多轮工具调用强化学习（基于 GRPO 和 GSPO，verl v0.9.1）中出现的“Actor 熵先暴增后崩溃”的不稳定现象展开调查与讨论（#8250）。

### 🔧 Pull Request 动态
*   **重要新特性与模型支持**
    *   **Muse Spark 模型支持**：在 FSDP2 引擎上实现了对 **Muse Spark 816B-A42B (NVFP4 MoE)** 模型的 GRPO + LoRA 训练支持，并适配了 vLLM rollout 的路由重放（#8257）。
    *   **视觉塔冻结恢复**：恢复了 0.8 版本后丢失的 `freeze_vision_tower` 功能，并在视觉塔冻结时跳过虚拟前向传播以提升效率（#8249）。
*   **硬件适配与性能优化（TPU / NPU）**
    *   **TPU 提速**：通过优化 rollout/training 的重叠设置，将 Qwen3-4B TPU GRPO 示例的运行速度提升了 **14%**（#8258）；同时将该示例迁移至 `torchtitan` 并更新了依赖版本（#8259）。
    *   **NPU 支持**：稳定了 FSDP-Turbo 训练流程，修复了 fused MoE 权重转置问题，从而支持 **DeepSeek V4-flash** 在 NPU 上的训练（#8251）。
*   **兼容性与 Bug 修复**
    *   **Transformers 5 适配**：修复了 Qwen3.5 模型在 `transformers >= 5.13` 上的 decoder layer 前向传播问题（#8255），并支持了 Qwen3.5 的动态缓存（#8256）。
    *   **蒸馏温度修复**：修复了 on-policy 蒸馏中，计算 vLLM teacher prompt logprobs 时未正确应用 teacher 温度（此前硬编码为 T=1）的问题（#8253）。
*   **文档完善**
    *   修复了 TPU GRPO README 中缺失的环境配置命令与计时说明（#8254）。
    *   修复了文档 `rollout_corr_math` 中数学公式推导部分无法正常渲染的问题（#8248）。

### 🚀 Release 动态
*   近期无新版本发布。

---

## 🐛 Issues

### #8252 — [[RFC] Add a weight checker for weight update](https://github.com/verl-project/verl/issues/8252)
- **作者**: shiyuan680  **时间**: 2026-10-15 15:44 CST
- **摘要**: ### Feature request  Can we add an optional weight check that runs **once, before training starts**, at the point where the trainer and the inference engines have **each finished loading their weights** from the same checkpoint? Let the engine snapshot its current weights, corrupt them, have the tra…

### #8250 — [[Investigation] Actor entropy explosion followed by collapse in async multi-turn tool-use RL (GRPO and GSPO, verl v0.9.1)](https://github.com/verl-project/verl/issues/8250)
- **作者**: Maclaurin1  **时间**: 2026-10-15 13:33 CST
- **标签**: bug
- **摘要**: ## Summary  We are training a multi-turn, tool-using financial agent with separate asynchronous vLLM rollout and FSDP2 actor training. We have observed unstable actor entropy under more than one policy-loss configuration. The most severe runs show a sharp entropy increase, reward collapse, malformed…

## 🔀 Pull Requests

### #8259 — [[hardware, perf, doc] feat: move the TPU GRPO image to torchtitan ea7fec0f (jax 0.11.1, libtpu 0.0.50) and speed up the Qwen3-4B example](https://github.com/verl-project/verl/pull/8259)
- **作者**: jialei777  **时间**: 2026-10-16 09:48 CST
- **摘要**: ### What does this PR do?  > Moves the TPU GRPO example (`examples/tpu/grpo`) to `google-pytorch/torchtitan` `ea7fec0f` with that commit's TPU package versions, and points the README at the verl-hardware-plugin commit that supports it (`f5876f75`, the merge of verl-project/verl-hardware-plugin#60: a…

### #8258 — [[hardware, perf, doc] feat: speed up the Qwen3-4B TPU GRPO example by 14% with three overlap settings](https://github.com/verl-project/verl/pull/8258)
- **作者**: jialei777  **时间**: 2026-10-16 04:45 CST
- **摘要**: ### What does this PR do?  > This PR tunes the rollout/training overlap of the Qwen3-4B TPU GRPO example ([`examples/tpu/grpo/run_qwen3_4b_torchtitan.sh`](https://github.com/jialei777/verl-upstream/blob/perf/tpu-grpo-4b-overlap-config/examples/tpu/grpo/run_qwen3_4b_torchtitan.sh)). It changes the de…

### #8257 — [[model, fsdp, rollout, vllm] feat: Muse Spark NVFP4 GRPO with LoRA on FSDP2](https://github.com/verl-project/verl/pull/8257)
- **作者**: andakai  **时间**: 2026-10-16 01:22 CST
- **摘要**: ### What does this PR do?  Adds RL support for **Muse Spark 816B-A42B** (NVFP4 MoE) on the FSDP2 engine: GRPO with LoRA against a vLLM rollout that serves the NVFP4 checkpoint, rollout routing replay (R3), and adapter-only weight sync. verl has no Muse Spark model, so the PR adds a verl-native text …

### #8256 — [[model] fix: support Transformers 5 Qwen3.5 dynamic cache](https://github.com/verl-project/verl/pull/8256)
- **作者**: LZC-BELIEVER  **时间**: 2026-10-15 23:31 CST
- **摘要**: # PR Draft: Qwen3.5 Transformers 5 DynamicCache  Status: prepared for human review; the clean branch is pushed to the personal fork, but no upstream PR has been created. The branch is rebased onto the latest canonical `main` (`5b550b0c3dad8c7bb7aab834a3fe195f8cdfc517`). PR #7471 is still open, so it…

### #8255 — [[model, fsdp] fix: Qwen3.5 decoder layer forward on transformers >= 5.13](https://github.com/verl-project/verl/pull/8255)
- **作者**: kahlun  **时间**: 2026-10-15 18:32 CST
- **摘要**: ## Motivation The reason for this PR is to resolve a future or the needs of using non pinned transformers. from current pinned 5.5.3,!=5.6.0,<5.13  ## What is wrong  `qwen3_5_decoder_layer_forward` (the patched `Qwen3_5DecoderLayer.forward`) reads `self.layer_type`. transformers 5.13.0 renamed that …

### #8254 — [[hardware, doc] fix: TPU GRPO README missing setup commands and measured timings](https://github.com/verl-project/verl/pull/8254)
- **作者**: gutianyu-google  **时间**: 2026-10-15 16:59 CST
- **摘要**: Following examples/tpu/grpo/README.md on a fresh project (verl f724e7f2, verl-hardware-plugin e5ab532, 2 x v6e-4 spot): the Artifact Registry repository and the GCS bucket are never created, and the bucket's region is not mentioned. With the bucket in another region the Qwen3-4B run did not reach a …

### #8253 — [[vllm, rollout] fix: honor teacher temperature in on-policy distillation](https://github.com/verl-project/verl/pull/8253)
- **作者**: yph22  **时间**: 2026-10-15 16:46 CST
- **摘要**: ### What does this PR do?  Honor `inference.temperature` when computing vLLM teacher prompt logprobs for on-policy distillation.  Previously, teacher prompt logprobs were computed at T=1 even when the configured temperature differed. This could produce a temperature mismatch between teacher targets …

### #8251 — [[fsdp, vllm] fix: stabilize FSDP-Turbo training and fix the transposition of fused moe weights to support deepseek-v4-flash training on NPU](https://github.com/verl-project/verl/pull/8251)
- **作者**: zhouhengan1211  **时间**: 2026-10-15 14:41 CST
- **摘要**: ### What does this PR do?  This PR fixes several issues observed when running DeepSeek V4 with FSDP-Turbo and vLLM-Ascend on NPU: - Fix legacy `FusedMoE` weight-layout detection for `w1` and `w3`.   - To adapt to operators such as Grouped MatMul, the Ascend backend merges the internal weights w1 and…

### #8249 — [[model, fsdp, cfg] feat: restore freeze_vision_tower and skip the dummy visual forward for frozen vision towers](https://github.com/verl-project/verl/pull/8249)
- **作者**: richardChenzhihui  **时间**: 2026-10-15 13:20 CST
- **摘要**: ### What does this PR do?  `freeze_vision_tower` is present in `ActorConfig`, the generated trainer YAMLs and the Ascend docs table, but no code consumes it: the implementation was dropped in the 0.8 engine refactor. Pre-0.8 `fsdp_workers.py` froze the tower (`get_vl_model_vision_tower()` + `require…

### #8248 — [[doc] fix: render display math in rollout_corr_math §3.2.2 derivation](https://github.com/verl-project/verl/pull/8248)
- **作者**: tguens  **时间**: 2026-10-15 12:22 CST
- **摘要**: ### What does this PR do?  Fixes the rendering of the "Formal derivation" in §3.2.2 *Policy Gradient Loss (with IS/RS Correction)* of [`docs/algo/rollout_corr_math.md`](https://verl.readthedocs.io/en/latest/algo/rollout_corr_math.html#policy-gradient-loss-with-is-rs-correction).  In steps 2–4, each …
