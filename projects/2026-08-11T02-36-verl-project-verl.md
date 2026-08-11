# verl-project/verl — 动态追踪

> 生成时间: 2026-08-11 10:36 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📌 Issue
本期无 Issue 动态。

### 🚀 Release
本期无 Release 动态。

### 🛠️ Pull Request
近期的 PR 主要集中在**核心训练与推理引擎的 Bug 修复**、**新特性增强**以及**NPU 适配与 CI 维护**上：

**1. 核心修复与稳定性提升**
*   **Megatron 修复**：
    *   修复了启用 CPU-offload 优化器恢复训练时的崩溃问题，现在当 `pin_memory()` 失败时会自动回退到非锁定内存拷贝 (#7342)。
    *   修复了 Qwen 3 / 3.5 VL 的路由重放 Bug，解决了 decoder 被重复初始化的问题 (#7340)。
*   **vLLM 修复**：修复了 `BucketedWeightReceiver` 回调失败时导致 ZMQ 无限阻塞的问题，现在会正确传播错误 (#7344)。
*   **Rollout 修复**：修复了 agent-loop/reward-loop workers 被错误调度到全局所有节点的问题，现在会严格限制在当前作业的节点组内 (#7343)。
*   **LoRA/SGLang 修复**：修复了在 LoRA 适配器模式 + `sleep_level=1` 时，权重同步引发的 `KeyError` 崩溃问题 (#7339)。

**2. 新特性**
*   **Model Merger 增强**：为 `model_merger merge` 命令新增了 `--fuse-lora` 标志，支持将 LoRA 权重直接融合进基础模型中，而不再仅是分开保存 (#7341)。

**3. 文档与 CI/CD**
*   **文档**：补充了 MTP 所需的 Megatron commit 说明（关于 `recompute_granularity=full` 的依赖）(#7346)。
*   **NPU 适配与 CI**：修复了 NPU 夜间 CI 测试，更新了 Triton 版本至 5.3.2 (#7345)；更新了 NPU Docker 镜像的 CANN 版本 (#7338)；为 NPU 夜间 CI 添加了三个新的基线测试 (#7337)。

---

## 🔀 Pull Requests

### #7346 — [[doc] fix: note the megatron commit MTP needs for recompute_granularity=full](https://github.com/verl-project/verl/pull/7346)
- **作者**: gaohongkui  **时间**: 2026-08-11 10:10 CST
- **摘要**: ### What does this PR do?  Follow-up to #7326, where the review outcome was that verl should document the required Megatron fix rather than carry a compatibility shim. This does that.  `docs/advance/mtp.md` pins megatron dev at [`23e092f41`](https://github.com/NVIDIA/Megatron-LM/tree/23e092f41ec8bc6…

### #7345 — [[ci] chore: Fix npu nightly ci](https://github.com/verl-project/verl/pull/7345)
- **作者**: LeoYao123  **时间**: 2026-08-11 10:01 CST
- **摘要**: ### What does this PR do? 1.fix npu nightly ci 2.update triton==5.3.2  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query l…

### #7344 — [[vllm] fix: propagate bucketed receiver failures](https://github.com/verl-project/verl/pull/7344)
- **作者**: nataliekung  **时间**: 2026-08-11 09:16 CST
- **摘要**: ### What does this PR do?  Propagates a `BucketedWeightReceiver` callback failure through the existing ZMQ reply so the waiting `BucketedWeightSender` raises instead of blocking forever in `socket.recv()`.  This is orthogonal to and based on the receiver callback shape established by #7327. That mer…

### #7343 — [[rollout] fix: keep agent-loop/reward-loop workers on the job's node group](https://github.com/verl-project/verl/pull/7343)
- **作者**: nataliekung  **时间**: 2026-08-11 06:46 CST
- **摘要**: ## Problem  `AgentLoopManager._init_agent_loop_workers` and `RewardLoopManager._init_reward_loop_workers` schedule their workers by round-robin over **every** alive node returned by `ray.nodes()` that has CPU, using a `soft=True` `NodeAffinitySchedulingStrategy`:  ```python node_ids = [node["NodeID"…

### #7342 — [fix(megatron): fall back to non-pinned CPU copy when pin_memory() fails on resume](https://github.com/verl-project/verl/pull/7342)
- **作者**: shotsan  **时间**: 2026-08-11 00:48 CST
- **摘要**: ## Problem  When resuming Megatron training with CPU-offloaded optimizer enabled, the process crashes in `offload_megatron_model_to_cpu` with:  ``` torch.AcceleratorError: CUDA error: invalid argument (cudaErrorInvalidValue) ```  at the `pin_memory()` call. Training from scratch works fine; only res…

### #7341 — [fix(model_merger): add --fuse-lora flag to merge LoRA weights into base model](https://github.com/verl-project/verl/pull/7341)
- **作者**: shotsan  **时间**: 2026-08-11 00:43 CST
- **摘要**: ## Problem  When using `model_merger merge` on a LoRA-trained checkpoint, the output directory contains the **unmodified base model weights** plus a separate `lora_adapter/` directory. The base model files are byte-identical to the original pre-trained model — the LoRA weights are never actually mer…

### #7340 — [[megatron] fix: bugfix qwen 3 qwen 3.5 router replay](https://github.com/verl-project/verl/pull/7340)
- **作者**: EricMarcus-ai  **时间**: 2026-08-11 00:38 CST
- **摘要**: ### What does this PR do?  Fixes a router replay bug for Qwen 3 VL / Qwen 3.5.   `Qwen3VLGPTModel.__init__` in megatron bridge builds its decoder twice: `super().__init__()` runs `GPTModel.__init__` ([here](https://github.com/NVIDIA-NeMo/Megatron-Bridge/blob/d17190fa4d2d023b9b2531c8d449ddb21c13b908/…

### #7339 — [fix: skip weight resume when sleep_level=1 in LoRA adapter mode (#7289)](https://github.com/verl-project/verl/pull/7339)
- **作者**: shotsan  **时间**: 2026-08-11 00:34 CST
- **摘要**: ## What this fixes  When using LoRA as a hot-swappable adapter (`model.lora.merge=False`) with colocated SGLang rollout and `free_cache_engine=True`, the second weight sync crashes with:  ``` KeyError: 'weights'   ... sglang/srt/managers/scheduler_update_weights_mixin.py, in resume_memory_occupation…

### #7338 — [[ci] chore: Update npu docker image cann version](https://github.com/verl-project/verl/pull/7338)
- **作者**: LeoYao123  **时间**: 2026-08-10 19:32 CST
- **摘要**: ### What does this PR do? Update npu docker image cann version  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `mega…

### #7337 — [[ci] chore: add three baselines for npu's nightly ci](https://github.com/verl-project/verl/pull/7337)
- **作者**: daikang6  **时间**: 2026-08-10 15:39 CST
- **摘要**: ### What does this PR do?  > add three baselines for npu's nightly ci  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`…
