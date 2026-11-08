# verl-project/verl — 动态追踪

> 生成时间: 2026-11-08 14:37 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🛠️ Pull Requests (合并请求)
近期 PR 主要集中在**引入新训练配方、扩展硬件/引擎兼容性以及增强系统鲁棒性**。

**新特性与功能**
*   **多教师策略蒸馏 (MOPD)**：新增了在 Megatron 后端 (EP4) 和全异步 rollout 下，针对 Qwen3.6-35B-A3B 模型的多教师策略蒸馏训练配方 (#8587)。
*   **TPU 支持 SGLang**：TPU 环境的 GRPO 示例现在支持将 rollout 引擎从默认的 vLLM 切换为 `sglang` (#8589)。
*   **Score Centering 传递**：当开启 score centering (`rollout.topk_log_probs > 0`) 时，vLLM 服务器会将每个生成 token 的 top-k 采样 ID、对数概率等信息写入 TransferQueue，供后续算法使用 (#8592)。

**重要修复与优化**
*   **Router Replay 稳定性**：强化了 Megatron router replay (R2/R3) 的启动逻辑（不支持配置快速失败、防止 replay 状态跨步骤泄漏），并将缓存类型从 int64 降至 int16 以节省资源。同时修复了相关示例脚本以适配统一配置，清理了无用字段 (#8594, #8593)。
*   **LoRA 训练修复**：修复了使用 LoRA actor (`ref_in_actor`) 时，V1 trainer 在参考对数概率计算阶段 `no_lora_adapter` 作用域溢出的问题 (#8591)。
*   **数据处理报错修复**：修复了 `hf_processor` 吞没所有异常的问题（此前加载错误和类型检查错误仅被转为警告），现在会正确抛出错误 (#8590)。
*   **昇腾 硬件适配**：修复了 vLLM-Ascend 0.30.x 版本中的权重同步失败问题，并解决了单卡节点上运行快速入门/PPO 脚本的阻塞问题 (#8588)。

### 📝 Issues (议题)
本期无独立的新增 Issue 记录，近期活动主要集中在通过上述 PR 修复已知问题（如 #8579 处理器报错吞没问题、#8586 昇腾环境兼容问题等）。

### 🚀 Release (版本发布)
近期无新的版本发布。

---

## 🔀 Pull Requests

### #8594 — [[megatron] fix: harden router replay startup, state cleanup and int16…](https://github.com/verl-project/verl/pull/8594)
- **作者**: ys2025-AI  **时间**: 2026-11-08 13:52 CST
- **摘要**: ### What does this PR do?  Hardens Megatron router replay (R2/R3) so unsupported setups fail fast and replay state cannot leak across steps, and reduces the replay route cache from int64 to int16.  - Startup guards: raise a clear error when Megatron-Core MoE helpers failed to import; reject `moe_per…

### #8593 — [[router_replay] fix: use top-level model_engine and drop dead replay …](https://github.com/verl-project/verl/pull/8593)
- **作者**: ys2025-AI  **时间**: 2026-11-08 13:50 CST
- **摘要**: ### What does this PR do?  Repairs `examples/router_replay/run_qwen3_30b_a3b_megatron.sh` for the unified config layout and removes dead router-replay config fields.  - The example still overrode `actor_rollout_ref.actor.model_engine=megatron`, which no longer exists. On current `main` Hydra fails w…

### #8592 — [[rollout, vllm, trainer, worker, algo] feat: write score-centering top-k heads from the vLLM server into TransferQueue](https://github.com/verl-project/verl/pull/8592)
- **作者**: cr-gao  **时间**: 2026-11-08 12:33 CST
- **摘要**: ### What does this PR do?  With `rollout.topk_log_probs > 0` (score centering, #8010) the vLLM server returns, per generated token, `k` sampler top-k ids, `k` log-probs and a support-coverage bit. Until now they travelled vLLM server → agent loop worker (Ray RPC, pickled numpy) → TransferQueue, and …

### #8591 — [[trainer] fix: scope no_lora_adapter to the V1 ref log-prob pass](https://github.com/verl-project/verl/pull/8591)
- **作者**: Hang-get  **时间**: 2026-11-07 17:45 CST
- **摘要**: ### What does this PR do?  With a LoRA actor (`ref_in_actor`), the V1 trainer runs the reference log-prob pass on the actor with its adapter disabled, by putting `no_lora_adapter=True` into `batch.extra_info`. Every stage of `_step_once` updates that same dict, so the flag was still set when `_compu…

### #8590 — [[data] fix: stop hf_processor from swallowing processor errors](https://github.com/verl-project/verl/pull/8590)
- **作者**: Hang-get  **时间**: 2026-11-07 17:19 CST
- **摘要**: ### What does this PR do?  Fixes #8579. `hf_processor` caught every exception, so processor load errors, and even its own "Unsupported processor type" check, became a warning plus `processor=None`. A multimodal run then failed later in `RLHFDataset` ("processor is needed to process multimodal data")…

### #8589 — [[sglang, rollout, hardware, doc] feat: support ROLLOUT_NAME=sglang in TPU GRPO examples](https://github.com/verl-project/verl/pull/8589)
- **作者**: jialei777  **时间**: 2026-11-07 16:13 CST
- **摘要**: ### What does this PR do?  Allow switching the rollout engine between `vllm` (default) and `sglang` (`ROLLOUT_NAME=sglang`, backed by [`sglang-torchtpu`](https://github.com/sgl-project/sglang-torchtpu)) in the Cloud TPU GRPO example recipes (`examples/tpu/grpo/`), paired with TPU SGLang rollout + `t…

### #8588 — [[hardware, ci, doc] fix: unblock Ascend quick-start/PPO scripts on vLLM-Ascend 0.30.x and single-card nodes](https://github.com/verl-project/verl/pull/8588)
- **作者**: WentaoXu03  **时间**: 2026-11-07 15:47 CST
- **摘要**: ### What does this PR do?  Fixes #8586. Part of #8304. Two problems, one PR:  1. **vLLM-Ascend 0.30.x weight-sync failure (all node sizes).** Since the stack upgrade in #8525, vLLM-Ascend hard-fails RL weight sync when FRACTAL_NZ weights are enabled (`vllm_ascend/worker/worker.py:282`, `wake_up`): `…

### #8587 — [[rollout, vllm, recipe] feat: Qwen3.6-35B-A3B multi-teacher OPD on Megatron EP4 with fully async rollout](https://github.com/verl-project/verl/pull/8587)
- **作者**: aoshen02  **时间**: 2026-11-07 15:08 CST
- **摘要**: ### What does this PR do?  Adds a multi-teacher on-policy distillation (MOPD) recipe for Qwen3.6-35B-A3B on two reasoning_gym tasks, trained with the Megatron backend (EP4) on the fully async trainer (`verl.experimental.fully_async_policy`), with MTP speculative decoding on the vLLM rollout. It also…
