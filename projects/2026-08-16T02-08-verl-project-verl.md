# verl-project/verl — 动态追踪

> 生成时间: 2026-08-16 10:08 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🛠️ Pull Requests (PR)

**✨ 新特性**
* **Rollout 负载均衡优化** (#7435)：`GlobalRequestLoadBalancer` 现在会根据副本负载重新评估粘性会话，避免长会话因前缀缓存复用而一直卡在过载的副本上。
* **检查点 Reshard 原语** (#7432)：为分阶段的 NCCL M2N Reshard 检查点路径新增了传输无关的张量布局原语。
* **SAPO 算法适配 Ascend NPU** (#7429)：新增了 SAPO（平滑优势策略优化）结合 Megatron 与华为 Ascend NPU 的可运行配方及文档。

**🐛 Bug 修复**
* **vLLM 权重同步回归修复** (#7434)：修复了 PR #7413 引入的回归问题，SGLang 特有的 `sleep_level==1` 跳过逻辑不再被错误地应用于 vLLM，确保 vLLM 在权重同步前必须恢复权重。
* **Qwen3.5 VL 统一检查点支持** (#7433)：因 Qwen3.5 的纯文本和 VL 变体共享相同的 `model_type`（不同于 Qwen3），将其加入 `_TEXT_TO_VL_FAMILY` 以修复统一检查点支持。
* **Rollout 配置缺失** (#7430)：补全了 `rollout.yaml` 中缺失的 7 个 `RolloutConfig` 键，修复了 Hydra 在 struct 模式下组合配置时的报错。
* **FSDP 温度缩放冗余计算** (#7428)：当温度系数（temperature）恰好为 1 时，跳过无意义的单位缩放计算，避免不必要的全词表显存分配。
* **FSDP2 反向传播失败** (#7427)：将融合的未绑定 LM Heads 作为根 FSDP 单元进行管理，修复了在延迟反向传播期间因生命周期无效导致的报错。

**🧹 维护与 CI**
* **CODEOWNERS 更新** (#7436)：新增 `@HollowMan6` 为 Megatron LoRA/PEFT、router-replay 等模块的代码所有者，并修复了失效路径。
* **FSDP2 测试回归** (#7431)：将稳定的 FSDP2 remove-padding 检查点测试重新集成到现有的 `model_rmpad` CI 作业中。

---

### 🐛 Issues
* 近期暂无显著的新增 Issue 动态。

---

### 🚀 Release
* 近期暂无新版本发布。

---

## 🔀 Pull Requests

### #7436 — [[misc] chore: add HollowMan6 to CODEOWNERS and repair stale paths](https://github.com/verl-project/verl/pull/7436)
- **作者**: HollowMan6  **时间**: 2026-08-16 04:18 CST
- **摘要**: ### What does this PR do?  This PR does two related things to `.github/CODEOWNERS`:  1. **Adds `@HollowMan6` as a code owner** for the Megatron LoRA/PEFT, router-replay, mcore and vLLM weight-sync areas, based on commit history in those paths. 2. **Repairs five entries that had gone stale** and were…

### #7435 — [[rollout] feat: re-evaluate sticky sessions against replica load](https://github.com/verl-project/verl/pull/7435)
- **作者**: ralovets  **时间**: 2026-08-16 01:50 CST
- **摘要**: ### What does this PR do?  `GlobalRequestLoadBalancer` pins all turns of a conversation to one replica for prefix-cache reuse, but never re-evaluates that choice. Long conversations can therefore remain concentrated on a few replicas while others go idle.  This PR adds an opt-in `affinity_break_marg…

### #7434 — [[vllm] fix: vllm always need to resume weights before weight sync](https://github.com/verl-project/verl/pull/7434)
- **作者**: HollowMan6  **时间**: 2026-08-16 01:50 CST
- **摘要**: ### What does this PR do?  https://github.com/verl-project/verl/pull/7413 introduced a regression for vLLM.  The sleep_level==1 skip is SGLang-specific and must NOT be applied to vLLM. SGLang's sleep_level=1 release() only frees kv_cache and keeps base weights mapped, so resuming weights is a no-op …

### #7433 — [fix: add Qwen3.5 VL to _TEXT_TO_VL_FAMILY for unified checkpoint support](https://github.com/verl-project/verl/pull/7433)
- **作者**: qy0720  **时间**: 2026-08-15 22:48 CST
- **摘要**: Qwen3.5 is a unified checkpoint — both its text-only and VL variants share the same model_type (qwen3_5 /qwen3_5_moe) in config.json. Unlike Qwen3 which uses distinct model types (qwen3 vs qwen3_vl), the model type alone cannot distinguish text from VL for Qwen3.5.  After PR #6804 introduced strict …

### #7432 — [[ckpt] feat: add NCCL M2N Reshard layout primitives](https://github.com/verl-project/verl/pull/7432)
- **作者**: ss16118  **时间**: 2026-08-15 21:09 CST
- **摘要**: ### What does this PR do?  Adds transport-independent tensor-layout primitives for the staged NCCL M2N Reshard checkpoint path.  veRL engine producers export rank-local parameters as `(name, tensor, ShardSpec)`, while M2N requires explicit source and destination mesh dimensions, placements, local sh…

### #7431 — [[ci] test: reintegrate stable FSDP2 rmpad test](https://github.com/verl-project/verl/pull/7431)
- **作者**: CharlesXu-HQ  **时间**: 2026-08-15 20:24 CST
- **摘要**: ### What does this PR do?  Reintegrate the stable FSDP2 remove-padding checkpoint test into the existing `model_rmpad` job.  - Run `STRATEGY=fsdp2 torchrun --nproc_per_node=8 tests/special_distributed/test_fsdp_ckpt.py` in `model_rmpad`. - Remove the dedicated `model_rmpad_fsdp2_unstable` job. - Rem…

### #7430 — [[rollout] fix: add missing RolloutConfig keys to rollout.yaml](https://github.com/verl-project/verl/pull/7430)
- **作者**: Nas01010101  **时间**: 2026-08-15 18:39 CST
- **摘要**: ### What does this PR do?  Seven `RolloutConfig` entries have no key in `verl/trainer/config/rollout/rollout.yaml`. Hydra composes the trainer config in struct mode, so a plain `actor_rollout_ref.rollout.<field>=...` override is rejected before training starts, even though every one of them is read …

### #7429 — [[algo, megatron, hardware, doc] feat: add SAPO × Megatron × Ascend NPU recipe](https://github.com/verl-project/verl/pull/7429)
- **作者**: xldeng-chn  **时间**: 2026-08-15 17:30 CST
- **摘要**: ## Proposed title  `[algo, megatron, hardware, doc] feat: add SAPO × Megatron × Ascend NPU recipe`  ## What does this PR do  Adds a runnable SAPO (Smooth Advantage Policy Optimization, [arXiv:2511.20347](https://arxiv.org/abs/2511.20347)) recipe on the Megatron backend for the Ascend 910B NPU, and s…

### #7428 — [[fsdp] fix: skip no-op unit temperature scaling](https://github.com/verl-project/verl/pull/7428)
- **作者**: Mengyuyang  **时间**: 2026-08-15 16:04 CST
- **摘要**: Preserve the original logits tensor when a host scalar temperature is exactly one, avoiding a full-vocabulary allocation while retaining the safe out-of-place path for non-unit and per-sample temperatures.  ### What does this PR do?  This PR avoids an unnecessary full-vocabulary logits allocation in…

### #7427 — [[fsdp] fix: prevent FSDP2 backward failure with fused untied LM heads](https://github.com/verl-project/verl/pull/7427)
- **作者**: Zhaoyi-Tian  **时间**: 2026-08-15 12:43 CST
- **摘要**: `[fsdp] fix: prevent FSDP2 backward failure with fused untied LM heads`  Manage fused untied LM heads as part of the root FSDP2 unit so direct weight access follows a valid FSDP lifecycle during deferred gradient synchronization.  ### What does this PR do?  I encountered this while running Qwen3.5-9…
