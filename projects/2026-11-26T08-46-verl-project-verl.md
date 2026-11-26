# verl-project/verl — 动态追踪

> 生成时间: 2026-11-26 16:46 CST

## AI 总结

# verl-project/verl 仓库近期动态摘要

## Issue
本次更新范围内未包含 Issue 类活动。

## Pull Request

### 新特性（feat）

- **#8864 fused linear cross entropy 新增 cuTile 后端**：为 verl 的 fused LM head 增加可选的 CUDA Tile 后端，支持策略训练（每 token 一个 label）和蒸馏/评分中心化（多 label）场景，提升性能。
- **#8861 FSDP 引擎支持 rollout 权重量化**：实现 `set_rollout_weight_quantizer`，使 FSDP 运行时可使用量化 rollout（如 `FP8_BLOCK`）或原生量化 checkpoint。
- **#8863 Intel GPU Docker 镜像升级**：将 XPU Dockerfile 从 vLLM 0.32.0 迁移至 0.34.0，对齐核心依赖版本。
- **#8856 rollout 重启容错机制（设计评审阶段）**：实现 rollout 侧重启容错方案，包含围栏式权重同步与有界客户端重放，目前处于早期设计评审，尚不可合并。

### 修复（fix）

- **#8862 Megatron checkpoint 恢复修复**：在仅加载模型（不含 optimizer）的恢复场景下，正确重载 optimizer 的 master params。
- **#8860 Triton 融合线性交叉熵修复**：针对低共享内存 GPU 适配三阶段 `_Total_Fuse_MN` 反向路径。
- **#8859 Search-R1 精确匹配奖励修复**：修正 `normalize_answer()` 中排版标点（如弯引号）未匹配的问题，影响 NQ、TriviaQA 等多个数据源评分。
- **#8858 vLLM 合并后修复**：处理 #8780 合并后的遗留问题，包括 VeOmni CI 依赖清理、M2N 守卫、HYBRID/PD 接线、移除 ray executor、稀疏/NPU 重载等。
- **#8857 CI 清理优化**：移除独立的 `cleanup_rocm` 任务，改为作业内 `if: always()` 清理 checkpoint。
- **#8865 Ascend 文档修正**：纠正 profiler 默认值与 per-role rank 选择说明，简化 vLLM/SGLang profiling 示例，修正训练长度 key 及 AISBench 客户端用法。

## Release
本次更新范围内未发布新版本。

---

**总体评价**：本批 PR 以**性能优化与硬件适配**为主线——cuTile 后端和 FSDP 量化 rollout 增强了训练效率与灵活性；同时集中修复了 Megatron checkpoint 恢复、Triton 低显存兼容、Search-R1 奖励评分等关键问题。值得注意的是 #8856 提出的 rollout 容错机制正处于设计评审阶段，可能成为后续重要方向。

---

## 🔀 Pull Requests

### #8865 — [[doc] fix: correct profiling and evaluation examples in Ascend guides](https://github.com/verl-project/verl/pull/8865)
- **作者**: lxb007981  **时间**: 2026-11-26 15:23 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Clarify profiler defaults and per-role rank selection, and simplify the vLLM and SGLang profiling examples. Use the correct training length keys and explain the AISBench client settings.  Fix the ModelScope symlink path and document GSM8K dataset loading and RotaryMul inpu…

### #8864 — [[perf, training_utils] feat: add a cuTile backend for fused linear cross entropy](https://github.com/verl-project/verl/pull/8864)
- **作者**: Jianbing-D  **时间**: 2026-11-26 15:23 CST
- **摘要**: ### What does this PR do?  Adds an opt-in CUDA Tile (cuTile) backend for verl’s fused LM head. Policy training can request one label per token, and distillation or score centering can request multiple selected IDs per token. Both paths compute entropy over the full vocabulary and avoid storing the f…

### #8863 — [[hardware, docs] feat: move the Intel GPU (XPU) Docker image to vLLM …](https://github.com/verl-project/verl/pull/8863)
- **作者**: kahlun  **时间**: 2026-11-26 14:36 CST
- **摘要**: ### What does this PR do?  Moves `docker/xpu/Dockerfile.stable.vllm` from vLLM 0.32.0 to 0.34.0, matching the `vllm==0.34.0` core pin from 8780.  - vLLM 0.34.0+xpu and torch 2.15.0+xpu. - The Intel user-mode driver now comes from the OMIX apt repo (`--build-arg OMIX_VERSION`, default 0.4.1, as in vL…

### #8862 — [[megatron, ckpt] fix: reload optimizer master params on model-only resume](https://github.com/verl-project/verl/pull/8862)
- **作者**: Hai-Story  **时间**: 2026-11-26 14:19 CST
- **摘要**: ### What does this PR do?  Fixes #8604.  When resuming a Megatron checkpoint with `load_contents` excluding `"optimizer"` (for example `["model"]` or `["model", "extra"]`), `MegatronCheckpointManager.load_checkpoint()` loads the checkpoint weights into the BF16/FP16 model parameters without refreshi…

### #8861 — [[fsdp, rollout] feat: support the trainer-side rollout weight quantizer in the FSDP engine](https://github.com/verl-project/verl/pull/8861)
- **作者**: xiaoxlu  **时间**: 2026-11-26 14:16 CST
- **摘要**: ### What does this PR do?  Implements `set_rollout_weight_quantizer` in the FSDP engine, so FSDP runs can use a quantized rollout (`rollout.quantization.linear=FP8_BLOCK`, or a checkpoint with a native `quantization_config`).  Since #8287, `init_model` hands the actor engine a rollout weight quantiz…

### #8860 — [[training_utils] fix: adapt Triton stages to shared memory](https://github.com/verl-project/verl/pull/8860)
- **作者**: wuxibin89  **时间**: 2026-11-26 13:11 CST
- **摘要**: ### What does this PR do?  Fixes the Triton fused linear cross-entropy `_Total_Fuse_MN` backward path on GPUs with lower shared-memory limits. This is a follow-up to #8851: its three-stage configuration requires 114,720 bytes of shared memory, exceeding the 101,376-byte limit on the L20 CI runner an…

### #8859 — [[reward] fix: match typographic punctuation in Search-R1 exact match](https://github.com/verl-project/verl/pull/8859)
- **作者**: ishan-nerlekar  **时间**: 2026-11-26 13:04 CST
- **摘要**: `verl/utils/reward_score/search_r1_like_qa_em.py` is the default scorer for the `searchR1_*` data sources (NQ, TriviaQA, PopQA, HotpotQA, 2Wiki, MuSiQue, Bamboogle). Its `normalize_answer()` removes `string.punctuation`, which is **ASCII only**. Typographic punctuation, which many models write, surv…

### #8858 — [[vllm, rollout, ci] fix: post-merge fixes for #8780 (VeOmni CI pin, M2N guards, HYBRID/PD wiring, drop ray executor, sparse/NPU reload)](https://github.com/verl-project/verl/pull/8858)
- **作者**: aoshen02  **时间**: 2026-11-26 10:35 CST
- **摘要**: ### What does this PR do?  Follow-up bugfix PR addressing post-merge issues found in #8780 (`95ede340`):  1. **Remove stale `--with transformers==5.16.1` overlay in VeOmni CI/scripts**:    - Removes `--with transformers==5.16.1` from `.github/workflows/e2e_ppo_trainer_veomni_vllm.yml`, `tests/specia…

### #8857 — [[ci, hardware] fix: clean up checkpoints in-job and remove cleanup_rocm GPU job](https://github.com/verl-project/verl/pull/8857)
- **作者**: TwelveNights  **时间**: 2026-11-26 07:32 CST
- **摘要**: ### What does this PR do?  Removes the separate `cleanup_rocm` job from `.github/workflows/e2e_ppo_trainer_megatron_vllm_rocm.yml` and adds `if: always()` to the existing in-job checkpoint cleanup step in both `e2e_ppo_trainer_fsdp_vllm_rocm` and `e2e_ppo_trainer_megatron-moe-expert-parallel_rocm`. …

### #8856 — [[rollout, ckpt, trainer, vllm, cfg] feat: opt-in rollout-restart fault tolerance with fenced weight sync and bounded client replay](https://github.com/verl-project/verl/pull/8856)
- **作者**: HaoWang113  **时间**: 2026-11-25 22:35 CST
- **摘要**: ### What does this PR do?  > **Draft — published for early design review, not ready for merge.** Implements the rollout-restart fault tolerance scheme specified in #8691 (the rollout-side sub-RFC of the FT anchor #8380; design spec in #8492). **Stacked on #8855**: the first two commits are the prere…
