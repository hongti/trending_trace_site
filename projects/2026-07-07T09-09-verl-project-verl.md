# verl-project/verl — 动态追踪

> 生成时间: 2026-07-07 17:09 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📌 Issue 动态
- **#6967 [fully_async] NCCL checkpoint-engine 初始化挂起（时序竞争）**
  报告了在首次权重同步前（即任何生成操作之前）发生挂起的问题。该问题可在无工具的单轮对话中复现，作者认为这与已关闭的 #5321 是不同的独立缺陷。

---

### 🔧 PR 动态
近期 PR 主要集中在 **`fully_async` 模块的健壮性修复**、**内存/性能优化**及**新模型支持**：

**1. `fully_async` 健壮性修复（作者 rawsh 集中提交）**
- **#6963**：修复缺失请求的 rollout log-probs 时抛出明确报错（fail-loud），防止批次数据静默丢失。
- **#6962**：修复合并部分 rollout 段时丢失 `extra_fields` 的问题，并加固了 `routed_experts` 的合并逻辑。
- **#6961**：修复 `FullyAsyncTrainer` 未正确读取 `rollout.checkpoint_manager_class` 配置项的问题，使其与其他 trainer 行为一致。

**2. 性能与内存修复**
- **#6958 [rollout]**：**修复 CUDA-IPC 权重传输中的显存泄漏（VRAM leak）**，改为在各次同步间复用持久化的 weight-transfer bucket，而非每次重新分配。
- **#6960 [training_utils]**：修复 fused linear-cross-entropy 反向传播问题，确保 `dlogprobs`/`dentropy` 梯度缓冲区在调用内核前转为连续内存（contiguous）。

**3. 模型与底层兼容性**
- **#6965 [perf, model]**：**新增 Qwen3.5 (`qwen3_5`) 的 MFU FLOPs 估算**，修复了此前该模型类型回退至未知估算函数导致 MFU 计算结果返回 0 的问题。
- **#6959 [megatron]**：修复 Megatron v0.12.1 兼容性崩溃问题，在导入该版本特有的私有符号前增加版本检查，防止版本不匹配时出错。

**4. 文档更新**
- **#6966 / #6968 [doc]**：新增 Atlas 950DT A5 硬件环境的安装配置指南（含软件版本、依赖及环境设置步骤，两 PR 内容相似可能为迭代修改）。

**5. 其他**
- **#6964**：描述信息不完整，暂无明确变更内容。

---

### 🚀 Release 动态
- **近期无新版本发布记录。**

---

## 🐛 Issues

### #6967 — [[fully_async] First NCCL checkpoint-engine group init hangs (timing race) — reproduces single-turn without tools](https://github.com/verl-project/verl/issues/6967)
- **作者**: chengcuiping  **时间**: 2026-07-07 16:51 CST
- **标签**: bug
- **摘要**: ### System Info   **Possibly related to the (now closed) #5321 — but our hang is at the first weight sync before any generation, and reproduces single-turn without tools, so we believe it is a distinct/lower-level problem.**  We root-caused this during a profiling effort on `recipe/fully_async_polic…

## 🔀 Pull Requests

### #6968 — [[doc] feat: add installation instructions for Atlas 950DT A5](https://github.com/verl-project/verl/pull/6968)
- **作者**: fh188  **时间**: 2026-07-07 17:02 CST
- **摘要**: ### What does this PR do?  This PR adds installation instructions for Atlas 950DT A5 to the documentation, including the required software versions, dependencies, and environment setup steps. It helps users set up the development environment for the Atlas 950DT A5 platform more easily.  ### Checklis…

### #6966 — [[doc] feat: add installation instructions for Atlas 950DT A5](https://github.com/verl-project/verl/pull/6966)
- **作者**: fh188  **时间**: 2026-07-07 15:57 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  This PR adds installation instructions for Atlas 950DT A5 to the documentation, including the required software versions, dependencies, and environment setup steps. It helps users set up the development environment for the Atlas 950DT A5 platform more easily.  ### Checklis…

### #6965 — [[perf, model] feat: add MFU flops estimation for Qwen3.5 (qwen3_5)](https://github.com/verl-project/verl/pull/6965)
- **作者**: ldemon2333  **时间**: 2026-07-07 15:36 CST
- **摘要**: ### What does this PR do?  Adds MFU FLOPs estimation for **Qwen3.5** (`model_type: "qwen3_5"`). Without it, `qwen3_5` falls through to `_estimate_unknown_flops`, which returns `0`, so `perf/mfu/*` is always zero for Qwen3.5 training runs.  Qwen3.5 differs from the models already covered by `flops_co…

### #6964 — [Zchen](https://github.com/verl-project/verl/pull/6964)
- **作者**: DoYangTan  **时间**: 2026-07-07 15:33 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #6963 — [[fully_async, trainer] fix: fail loud when requested rollout log-probs are missing](https://github.com/verl-project/verl/pull/6963)
- **作者**: rawsh  **时间**: 2026-07-07 15:12 CST
- **摘要**: ### What does this PR do?  Adds fail-loud invariants so that when rollout log-probs are **requested** (`rollout.calculate_log_probs` or `actor.use_rollout_log_probs`), a batch that silently loses `rollout_log_probs` raises instead of quietly degrading importance-correction / debug metrics to recompu…

### #6962 — [[fully_async, rollout] fix: preserve extra_fields when merging partial rollout segments](https://github.com/verl-project/verl/pull/6962)
- **作者**: rawsh  **时间**: 2026-07-07 15:12 CST
- **摘要**: ### What does this PR do?  Fixes `FullyAsyncLLMServerClient` partial-rollout merge (`verl/workers/rollout/llm_server.py`) dropping `TokenOutput.extra_fields`, and hardens the `routed_experts` merge.  **Problem / root cause.** When a request is resumed across engine versions (partial rollout), the cl…

### #6961 — [[fully_async] fix: honor rollout.checkpoint_manager_class like other trainers](https://github.com/verl-project/verl/pull/6961)
- **作者**: rawsh  **时间**: 2026-07-07 15:12 CST
- **摘要**: ### What does this PR do?  Makes `FullyAsyncTrainer` honor the existing `actor_rollout_ref.rollout.checkpoint_manager_class` config hook (`verl/workers/config/rollout.py` already declares it) instead of hardcoding `CheckpointEngineManager`.  **Problem.** The rollout config exposes `checkpoint_manage…

### #6960 — [[training_utils] fix: make fused linear-cross-entropy backward grad buffers contiguous](https://github.com/verl-project/verl/pull/6960)
- **作者**: rawsh  **时间**: 2026-07-07 15:12 CST
- **摘要**: ### What does this PR do?  Makes `LinearCrossEntropy.backward` (`verl/utils/kernel/linear_cross_entropy.py`) call `.contiguous()` on the incoming `dlogprobs` / `dentropy` grads before invoking `kernels.efficient_entropy_backward`.  **Problem / root cause.** Autograd can hand `backward` **strided vie…

### #6959 — [[megatron] fix: check versions before importing v0.12.1-only private symbols](https://github.com/verl-project/verl/pull/6959)
- **作者**: rawsh  **时间**: 2026-07-07 15:12 CST
- **摘要**: ### What does this PR do?  Reorders `apply_patch_megatron_v012_with_torch_v28_v29` (`verl/models/mcore/patch.py`) so the megatron/torch **version check runs before** importing v0.12.1-only private symbols.  **Problem / root cause.** The function imports `megatron.core.dist_checkpointing.strategies.a…

### #6958 — [[rollout] fix: reuse a persistent CUDA-IPC weight-transfer bucket across syncs](https://github.com/verl-project/verl/pull/6958)
- **作者**: rawsh  **时间**: 2026-07-07 15:12 CST
- **摘要**: ### What does this PR do?  Fixes a per-sync VRAM leak in the CUDA-IPC bucketed weight transfer path (`verl/workers/rollout/vllm_rollout/bucketed_weight_transfer.py`) by allocating and IPC-exporting the transfer bucket **once per process** and reusing it across weight syncs, instead of allocating + e…
