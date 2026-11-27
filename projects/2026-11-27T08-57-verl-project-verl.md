# verl-project/verl — 动态追踪

> 生成时间: 2026-11-27 16:57 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要。本次提供的数据仅包含 Pull Request 活动，无 Issue 和 Release 相关记录。

### 🛠️ Pull Request (PR)

**重要新特性**
- **支持 AMD 显卡 MXFP8 强化学习**：在 AMD MI350 / MI355 (gfx950) 硬件上引入了 MXFP8 RL 训练配方（包含基于 Transformer Engine 的训练与 MXFP8 推理），扩展了硬件兼容性。(#8880)

**重要修复与优化**
- **优势计算 Bug 修复**：修复了 V1 多轨迹和 group-outcome 优势计算路径中的相关问题，扩展了多轨迹会话缩减至 group-outcome 估计器，并转发了 `norm_adv_by_std_in_grpo`。(#8882)
- **CPU 分布式初始化修复**：修复了纯 CPU Worker 因默认后端字符串 (`cpu:gloo,cpu:nccl`) 被判定为重复设备类型而导致启动失败的问题，改用 `gloo` 后端。(#8881)
- **FSDP 内存优化**：修复了 FSDP 在进行 log-prob 温度缩放时因大词表产生额外缓冲区而导致的显存峰值激增问题。(#8879)
- **数值缺陷修复**：修复了熵估计器 `_ExpectedLogit` 使用 bf16 输出矩阵乘法导致的数值精度缺陷。(#8878)
- **SGLang 兼容与 EOS 处理修复**：
  - 修复了在策略蒸馏 (OPD) 中，当 teacher 运行在 SGLang 时因携带 vLLM 专属的 `detokenize: False` 参数而导致的崩溃。(#8876)
  - 修复了 `SGLangHttpServer` 相比 vLLM 缺失默认 EOS `stop_token_ids` 和 `ignore_eos` 传参的问题。(#8875)
- **测试构建修复**：修复了 #8861 引入的问题，确保在 LoRA staging 导出测试中正确构建真实的 `FSDPEngine`。(#8877)

**文档改进**
- 对多份文档（feature / contrib / README / 性能 / 精度）进行了格式与术语规范清理，修复了中英文混排的多余空格、统一标点符号全半角，并修正了专有名词大小写（如 `vllm` 改为 `vLLM`）。(#8873, #8874)

### 📋 Issue
本次动态数据中未提供相关 Issue 记录。

### 🚀 Release
本次动态数据中未提供新的 Release 记录。

---

## 🔀 Pull Requests

### #8882 — [[trainer, algo] fix: extend multi-trajectory session reduction to group-outcome estimators and forward norm_adv_by_std_in_grpo](https://github.com/verl-project/verl/pull/8882)
- **作者**: ca33-zx  **时间**: 2026-11-27 13:53 CST
- **摘要**: ### What does this PR do?  Fixes #8741 by addressing two closely related advantage-computation bugs in V1 multi-trajectory and group-outcome advantage paths:  1. **Extend multi-trajectory session reduction beyond `GRPO` (`verl/trainer/ppo/v1/utils.py`)**:    - Previously, `compute_advantage_for_mult…

### #8881 — [fix(utils): use gloo for CPU distributed initialization](https://github.com/verl-project/verl/pull/8881)
- **作者**: LittleChenLiya  **时间**: 2026-11-27 12:25 CST
- **摘要**: ## Why  Closes #6237. When `get_device_name()` returns `cpu`, the default backend string becomes `cpu:gloo,cpu:nccl`, which PyTorch rejects as a duplicate device type. CPU-only workers therefore fail before training starts.  ## What  - Add a shared default-backend helper that returns `gloo` for CPU.…

### #8880 — [[megatron, hardware, deployment, recipe, doc] feat: MXFP8 RL on AMD MI350 / MI355 (gfx950)](https://github.com/verl-project/verl/pull/8880)
- **作者**: Vivicai1005  **时间**: 2026-11-27 11:21 CST
- **摘要**: ### What does this PR do?  Runs the [MXFP8 RL](https://github.com/verl-project/verl/blob/main/docs/low_precision/mxfp8.md) recipe (Transformer Engine's MXFP8 training recipe with an MXFP8 rollout) on AMD Instinct MI350 / MI355 (gfx950), with the same per-layer precision as on Blackwell: MXFP8 actor …

### #8879 — [[fsdp] fix: reduce peak memory during log-prob temperature scaling](https://github.com/verl-project/verl/pull/8879)
- **作者**: blessedcursed  **时间**: 2026-11-27 03:22 CST
- **摘要**: ### What does this PR do?  The dense FSDP training path divides the full logits tensor by temperature before computing token log-probabilities. With a large vocabulary, the extra buffer can take several GiB.  Add an opt-in chunk size for single-sequence micro-batches on the eager path. Each block is…

### #8878 — [factory: resolve issue #97](https://github.com/verl-project/verl/pull/8878)
- **作者**: 995114879  **时间**: 2026-11-27 01:30 CST
- **摘要**: ## 摘要 All work is committed with DCO sign-off and the tree is clean. Here is the final summary.  ## 本轮（断点续跑）完成的工作  核对现状后发现前 5 个提交已完成大部分实现，但**存在一个会导致验收失败的数值缺陷**：熵估计器 `_ExpectedLogit` 用 bf16 输出矩阵乘（`hidden @ weight.t()`）计算 logits，而现有 Triton 内核是 `tl.dot(..., acc=fp32)`——logits 以 **fp32 累加**。在 vocab=1519…

### #8877 — [  [fsdp] fix: build a real FSDPEngine in the LoRA staging export tests](https://github.com/verl-project/verl/pull/8877)
- **作者**: xiaoxlu  **时间**: 2026-11-27 01:13 CST
- **摘要**: ### What does this PR do?  Follow-up to #8861, which broke two CPU tests on `main`.  #8861 moved the body of `FSDPEngine.get_per_tensor_param` into `_get_per_tensor_param`, so the public method now calls a helper on `self`. Both tests in `tests/workers/test_fsdp_lora_base_export_staging_on_cpu.py` p…

### #8876 — [[rollout, sglang] fix: drop vLLM-only detokenize from teacher sampling params](https://github.com/verl-project/verl/pull/8876)
- **作者**: DeeproChoudhury  **时间**: 2026-11-26 22:37 CST
- **摘要**: ### What does this PR do?  Fix a crash in on-policy distillation (OPD) when the teacher runs on SGLang.  - #7648 added `"detokenize": False` to the teacher sampling params in `teacher_manager.py`. - These params go to every teacher backend, but `detokenize` is a vLLM-only field. - SGLang 0.5.20 reje…

### #8875 — [[rollout, sglang] fix: propagate default EOS stop_token_ids and ignore_eos in SGLangHttpServer](https://github.com/verl-project/verl/pull/8875)
- **作者**: HOWILLMAKEIT  **时间**: 2026-11-26 21:46 CST
- **摘要**: ### What does this PR do?  `SGLangHttpServer` has two EOS-handling gaps compared to `vLLMHttpServer`:  1. **Missing default EOS stop tokens under `skip_tokenizer_init=True`:** `RolloutConfig.skip_tokenizer_init` defaults to `True` ([rollout.py](https://github.com/verl-project/verl/blob/2d0283f89d7e3…

### #8874 — [[doc] fix: spacing, punctuation and anchors in feature/contrib/README docs](https://github.com/verl-project/verl/pull/8874)
- **作者**: zery00568  **时间**: 2026-11-26 18:49 CST
- **摘要**: ### What does this PR do?  -  Remove redundant spaces between Chinese and English; convert half-width punctuation to full-width (e.g., `调用链:` -> `调用链：`). - **Terminology**: `vllm` -> `vLLM`, `sglang` -> `SGLang` (prose only). - **Markdown structure**: blank lines around headings/tables/code blocks, …

### #8873 — [[doc] fix: spacing, punctuation and terminology in performance/precision docs](https://github.com/verl-project/verl/pull/8873)
- **作者**: zery00568  **时间**: 2026-11-26 18:49 CST
- **摘要**: ### What does this PR do?  Clean up spacing, punctuation and terminology issues in the performance/precision docs.  - Remove redundant spaces between Chinese and English; convert half-width punctuation to full-width and add missing periods. - **Terminology**: `Swiglu` -> `SwiGLU`, `mte2` -> `MTE2`, …
