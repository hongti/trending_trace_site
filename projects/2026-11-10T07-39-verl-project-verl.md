# verl-project/verl — 动态追踪

> 生成时间: 2026-11-10 15:39 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📋 Issue
* **#8609 vLLM 休眠期间保留独立配置的草稿模型权重**
  作者报告了一个问题：当通过 `speculative_config` 独立配置非 MTP 草稿模型时，vLLM 的休眠与唤醒周期会导致草稿模型的权重丢失，而未被正确保留。

### 🔀 Pull Request
**重要修复：**
* **#8612 / #8613 修复 vLLM 休眠导致草稿权重丢失：** 针对 Issue #8609 提交修复。解决在 CUDA 和其他路径下，vLLM level-2 休眠会丢弃独立草稿模型权重的问题，确保唤醒后能正确恢复目标模型和草稿模型权重。
* **#8616 修复 CUDA 非法内存访问 (IMA)：** 在 vLLM 确定性测试中，将 `rollout.max_num_batched_tokens` 限制在引擎容量范围内，避免引擎初始化时发生 CUDA IMA 报错。
* **#8611 修复 Qwen-VL 视觉占位符引发的崩溃：** 禁止 Qwen-VL 在采样时生成 `<|image_pad|>` / `<|video_pad|>` 符号，避免因缺少匹配的多模态网格信息而引发 `get_rope_index` TypeError。
* **#8608 放宽 NPU 休眠级别限制：** 旧版 vllm-ascend 在 sleep level 2 下存在精度问题会导致奖励坍塌为零，现限制该问题仅在 v0.30.0 以下版本发生，允许 v0.30.0+ 使用 level 2 休眠。

**新特性与优化：**
* **#8610 蒸馏 teacher 按需服务：** 为多教师策略蒸馏 (MOPD) 引入 `on_demand` 模式，多个教师模型共享一个推理副本队列，无需再常驻 N 个 vLLM 引擎，大幅减少所需的 NPU/GPU 数量。
* **#8615 v1 PPO 训练结束回调钩子：** 为 v1 PPO 添加了运行结束时的检查点回调，允许用户在训练末期附加自定义操作（如写入运行级 `TERMINATE` 标记）而无需修改底层类。
* **#8614 TPU v7x 深度优化与 SGLang 切换：** 端到端优化了 TPU v7x 上的 Qwen3.5-35B-A3B DeepSWE 智能体 RL 配方，训练速度从约 490 秒/步大幅提升至约 320 秒/步，并支持向 SGLang TPU rollout 切换。
* **#8617 新增 TPU CI 流水线：** 为 `examples/tpu` 配方添加了基于 Cloud TPU v6e (GKE) 的持续集成测试。

### 🚀 Release
* 本次提供的数据中未包含版本发布信息。

---

## 🐛 Issues

### #8609 — [Preserve separately configured draft-model weights across vLLM sleep](https://github.com/verl-project/verl/issues/8609)
- **作者**: coderabbitai[bot]  **时间**: 2026-11-09 17:54 CST
- **摘要**: ## Summary  Preserve independently configured non-MTP draft-model weights across vLLM sleep and wake cycles.  When `engine_kwargs.vllm.speculative_config` configures a separate draft model, `vLLMHttpServer._resolve_sleep_level()` does not account for its weights. Sleep level 2 can discard those weig…

## 🔀 Pull Requests

### #8617 — [[ci, hardware] feat: add TPU CI (Cloud TPU v6e on GKE) for the examples/tpu recipes](https://github.com/verl-project/verl/pull/8617)
- **作者**: jialei777  **时间**: 2026-11-10 15:21 CST
- **摘要**: ### What does this PR do?  Adds CI for verl on **Cloud TPU** (v6e on GKE). verl's TPU support lives in [`verl-hardware-plugin`](https://github.com/verl-project/verl-hardware-plugin) and the TPU recipes in [`examples/tpu`](https://github.com/verl-project/verl/tree/main/examples/tpu); today nothing te…

### #8616 — [[vllm] fix: cap rollout max_num_batched_tokens to avoid CUDA IMA at engine init](https://github.com/verl-project/verl/pull/8616)
- **作者**: yuki1ssad  **时间**: 2026-11-10 11:19 CST
- **摘要**: ### What does this PR do?  Caps `rollout.max_num_batched_tokens` within the per-engine capacity `max_num_seqs * max_model_len` in the vLLM determinism tests, fixing a CUDA illegal memory access (IMA) that crashes both determinism gates at vLLM engine init — before any request is served.  Root cause:…

### #8615 — [[trainer, ckpt, test] feat: add run-end checkpoint callback hook for v1 PPO](https://github.com/verl-project/verl/pull/8615)
- **作者**: lingyulyu001  **时间**: 2026-11-10 06:07 CST
- **摘要**: ### What does this PR do?  Adds a **run-end checkpoint callback hook** for v1 PPO so users can attach end-of-training side effects (for example, writing a run-level `TERMINATE` marker) without subclassing the trainer.  Concretely: - Introduces `CheckpointCallback.on_train_end(...)` in `verl/trainer/…

### #8614 — [[hardware, recipe, sglang] feat: optimized Qwen3.5-35B-A3B DeepSWE agentic RL on TPU v7x (~490 -> ~320 s/step) + SGLang TPU rollout switch](https://github.com/verl-project/verl/pull/8614)
- **作者**: jialei777  **时间**: 2026-11-10 03:25 CST
- **摘要**: ### What does this PR do?  End-to-end verl changes for the **optimized Qwen3.5-35B-A3B DeepSWE (R2E-Gym) agentic RL recipe on TPU v7x**: verl + verl-hardware-plugin (TorchTitan trainer) + vLLM-TorchTPU rollout + uni-agent ReAct, sandboxes via the K8s Agent Sandbox.  Steady-state step time: **~490 s …

### #8613 — [[rollout, vllm] fix: preserve independent draft weights during sleep](https://github.com/verl-project/verl/pull/8613)
- **作者**: xie-tj  **时间**: 2026-11-09 20:13 CST
- **摘要**: ### What does this PR do?  Fixes #8609.  On the NVIDIA CUDA path, level-2 sleep discards an independently configured draft's weights. The next actor sync restores only the target weights, leaving the draft with invalid parameters.  Select level 1 when vLLM's resolved configuration contains a separat…

### #8612 — [[vllm, rollout] fix: preserve separately configured draft-model weights across vLLM sleep](https://github.com/verl-project/verl/pull/8612)
- **作者**: taruki11  **时间**: 2026-11-09 19:52 CST
- **摘要**: ### What does this PR do?  Fixes #8609.  When `engine_kwargs.vllm.speculative_config` configures a separate draft model (e.g., via `model`, `draft_model`, `draft_model_config`, or a model-backed `method` such as EAGLE/Medusa), `vLLMHttpServer._resolve_sleep_level()` previously defaulted to `VLLM_SLE…

### #8611 — [[vllm, rollout] fix: ban vision pad tokens on sampler and logit_bias to avoid get_rope_index TypeError](https://github.com/verl-project/verl/pull/8611)
- **作者**: zgh-99  **时间**: 2026-11-09 19:35 CST
- **摘要**: ### What does this PR do?  Stop Qwen-VL from sampling `<|image_pad|>` / `<|video_pad|>`. Those ids are in-vocab placeholders; if the policy emits one with no matching `image_grid_thw` / `video_grid_thw` row, HuggingFace `get_rope_index` does `next(iterator)` on an exhausted grid and raises:  ``` Typ…

### #8610 — [[distillation] feat: on-demand teacher fleet to reduce MOPD teacher NPU/GPUs needed](https://github.com/verl-project/verl/pull/8610)
- **作者**: zgh-99  **时间**: 2026-11-09 19:23 CST
- **摘要**: ### What does this PR do?  Add `distillation.teacher_serving=on_demand` for multi-teacher on-policy distillation (MOPD). N teachers no longer need N resident vLLM engines. They share a replica fleet sized by the teacher pool; each checkpoint is snapshotted once into a node-local mmap cache, and a re…

### #8608 — [[vllm, hardware] feat: allow sleep level 2 with vllm-ascend >= 0.30.0](https://github.com/verl-project/verl/pull/8608)
- **作者**: lxb007981  **时间**: 2026-11-09 17:29 CST
- **摘要**: ### What does this PR do?  Older vllm-ascend versions have an EP precision issue with sleep level 2 that can cause rewards to collapse to zero. This is fixed in v0.30.0.  Restrict the NPU sleep level 1 guard to vllm-ascend < 0.30.0, while preserving the MTP and LoRA guards.  Compare sleep levels 1 a…
