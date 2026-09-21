# verl-project/verl — 动态追踪

> 生成时间: 2026-09-21 13:19 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🚀 Release
**v0.9.1 版本发布** (2026-09-20)
*   **版本亮点**：统一 V1 训练器引入了 `separate_async` 优化。现在它可以将空闲的训练器 GPU 借出用于生成任务，即在某一步的 prompts 提交后，闲置的 GPU 资源可以被重新利用，从而提升整体资源利用率。

### 🔧 Pull Requests (PR)
近期 PR 主要集中在新硬件/算法支持、核心 Bug 修复以及 CI/CD 稳定性提升。

**1. 新特性**
*   **#7971 原生 NVFP4 训练与 Rollout**：新增在 NVIDIA Blackwell GPU 上的端到端原生 NVFP4 W4A4 训练和 rollout 支持（结合 Megatron/Transformer Engine 和 vLLM，已通过 Qwen3-30B GRPO 验证），并支持 live refit。

**2. 重要 Bug 修复**
*   **#7970 KL 惩罚计算修复**：在计算和上报 KL 惩罚时，排除被中止的 rollout，避免对结果产生污染。
*   **#7969 FSDP 分离 Critic 构建修复**：修复了分离训练器中 critic 的构建逻辑，使其兼容 FSDP 和 FSDP2（从 `critic.engine` 而非缺失的 `critic.model.fsdp_config` 读取配置）。
*   **#7965 GDPO 权重校验修复**：为 `gdpo_reward_weights` 和 `gdpo_reward_keys` 增加长度匹配校验，防止因位置对应不上导致的潜在错误。
*   **#7964 RLOO 优势数据类型修复**：修复了 `torch.bincount` 强制将数据类型提升为 float64 的问题，保持 `rloo_vectorized` 优势计算与原有的 reward 数据类型一致。
*   **#7967 / #7966 CI 工作流修复**：分别通过锁定 `transformers==5.5.4` 和升级 `nvidia-resiliency-ext`，修复了 `e2e_sft_llm_ascend` 和 `vllm_ascend` 在主干分支上的 CI 失败问题。

**3. 测试与维护**
*   **#7972**：在 CI 中使用持久化缓存。
*   **#7968**：为被多处调用的 `net_utils.py`（用于为 sglang/vllm rollout 服务器保留端口）补充测试覆盖率。
*   **#7963**：将 `verl-wheelhouse` 路径更改为 `/cu130/torch2.11/simple`，以支持基于多个 torch 版本的预构建。

### ⚠️ Issue
*   近期无新增 Issue 动态。

---

## 🔀 Pull Requests

### #7972 — [[uv, ci] fix: use persistent cache in CI](https://github.com/verl-project/verl/pull/7972)
- **作者**: ETOgaosion  **时间**: 2026-09-21 12:25 CST
- **摘要**: ### What does this PR do?  As title  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, `sglang`, …

### #7971 — [[megatron, vllm, env] feat: add native NVFP4 training and rollout with live refit](https://github.com/verl-project/verl/pull/7971)
- **作者**: zhangyimi  **时间**: 2026-09-21 12:06 CST
- **摘要**: ### What does this PR do?  Add end-to-end native NVFP4 W4A4 training and rollout on NVIDIA Blackwell GPUs, demonstrated with Qwen3-30B-A3B-Base GRPO using Megatron/Transformer Engine and vLLM.  This contributes to Sophia's [low-precision roadmap #5972](https://github.com/verl-project/verl/issues/597…

### #7970 — [[algo, trainer] fix: exclude aborted rollouts from the reported KL penalty](https://github.com/verl-project/verl/pull/7970)
- **作者**: linhongyu510  **时间**: 2026-09-21 10:36 CST
- **摘要**: ### What does this PR do?  `apply_kl_penalty` averages the per-sequence KL over **every** row of the batch:  ```python current_kl = masked_mean(kld, mask=response_mask, axis=-1) current_kl = torch.mean(current_kl, dim=0).item() ```  Aborted rollouts have an all-zero response mask. `masked_mean` divi…

### #7969 — [[trainer, fsdp] fix: build separation critics with FSDP and FSDP2](https://github.com/verl-project/verl/pull/7969)
- **作者**: RainieLLM  **时间**: 2026-09-21 10:21 CST
- **摘要**: ### What does this PR do?  Fix critic construction in the separation trainer: accept `fsdp2` and read the engine from `critic.engine` instead of the missing `critic.model.fsdp_config`.  Related to #7822. #7465 handles batching and includes the same engine fix, but leaves FSDP2 support out of scope. …

### #7968 — [[utils] test: add coverage for net_utils](https://github.com/verl-project/verl/pull/7968)
- **作者**: Redemption-ZTX  **时间**: 2026-09-21 03:55 CST
- **摘要**: Test coverage only — no behaviour change.  ## What  `verl/utils/net_utils.py` currently has no tests at all, despite being used by 13 call sites to reserve ports for the rollout servers (sglang / vllm, plus their PD replicas and async servers) and for the checkpoint engines (nccl, nixl, mooncake, ki…

### #7967 — [[ci] fix: pin transformers==5.5.4 in e2e_sft_llm_ascend workflow](https://github.com/verl-project/verl/pull/7967)
- **作者**: aass-79  **时间**: 2026-09-21 03:30 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Fixes the `e2e_sft_llm_ascend` CI job, currently failing on `main` (e.g. https://github.com/verl-project/verl/actions/runs/35522678515/job/106109358443) and on open PRs such as #7928.  The Ascend CI image recently started shipping `transformers 5.10.4`, whose `create_causa…

### #7966 — [[ci] fix: upgrade nvidia-resiliency-ext in vllm_ascend workflow](https://github.com/verl-project/verl/pull/7966)
- **作者**: aass-79  **时间**: 2026-09-21 03:29 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Fixes the `vllm_ascend` CI job, currently failing on `main` (e.g. https://github.com/verl-project/verl/actions/runs/35522678520/job/106109358524) and on open PRs such as #7928.  The Ascend CI image (`verl:latest-vllm-a3-ubuntu`) ships `nvidia-resiliency-ext 0.4.1`, whose `…

### #7965 — [[algo] fix: validate gdpo_reward_weights against gdpo_reward_keys](https://github.com/verl-project/verl/pull/7965)
- **作者**: linhongyu510  **时间**: 2026-09-21 02:02 CST
- **摘要**: ### What does this PR do?  GDPO pairs `algorithm.gdpo_reward_weights` with `algorithm.gdpo_reward_keys` **by position**, but nothing checks that the two lists are the same length. `AlgoConfig` has no `__post_init__`, and the trainer's only other use of `gdpo_reward_keys` is emitting per-component me…

### #7964 — [[algo] fix: keep rloo_vectorized advantages in the reward dtype](https://github.com/verl-project/verl/pull/7964)
- **作者**: linhongyu510  **时间**: 2026-09-21 01:39 CST
- **摘要**: ### What does this PR do?  `torch.bincount(weights=...)` accumulates in **float64** regardless of the weights' dtype, so the group sums in `compute_rloo_vectorized_outcome_advantage` promote the whole expression. A bf16 or fp16 reward tensor comes back as float64 advantages, while `compute_rloo_outc…

### #7963 — [[misc] chore: change verl-wheelhouse to /cu130/torch2.11/simple](https://github.com/verl-project/verl/pull/7963)
- **作者**: wuxibin89  **时间**: 2026-09-20 23:22 CST
- **摘要**: ### What does this PR do?  Change verl-wheelhouse to `/cu130/torch2.11/simple` to support prebuilt from multiple torch versions.

## 🚀 Releases

### [v0.9.1](https://github.com/verl-project/verl/releases/tag/v0.9.1)
- **作者**: wuxibin89  **时间**: 2026-09-20 15:24 CST
- **摘要**: ## Highlights  ### Trainer  #### Unified V1 trainer - `separate_async` can now lend idle trainer GPUs to generation: once a step's prompts are submitted, the trainer keeps its replicas in rollout mode until the replay buffer holds enough sampleable groups, driven by an adaptive starvation threshold.…
