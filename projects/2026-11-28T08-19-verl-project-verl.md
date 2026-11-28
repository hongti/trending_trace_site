# verl-project/verl — 动态追踪

> 生成时间: 2026-11-28 16:19 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 🐛 Issue 摘要
近期主要报告了 2 个 Bug：
1. **SGLang 死锁问题 (#8893)**：`SGLangHttpServer.abort_all_requests` 在遇到 `asyncio.CancelledError` 时未能恢复 `_generation_allowed` 状态，导致 `generate()` 发生永久死锁。
2. **V1 Trainer 无参考策略崩溃 (#8891)**：在 verl 0.10.0.dev 中，V1 trainer 的 `_compute_advantage()` 强制请求 `ref_log_prob`，导致所有不使用参考策略（reference policy）的配方在训练第一步就发生崩溃。

---

### 🔧 Pull Request 摘要
近期提交的 PR 主要集中在修复核心训练逻辑的 Bug、提升多硬件兼容性及性能优化：

**重要修复**
- **#8894**：修复上述 Issue #8891，使 V1 trainer 仅在使用参考策略时才请求 `ref_log_prob`。
- **#8895**：修复未触发 hook（如 MoE 专家未被前序批次使用）时，能够原子性地跳过整个更新步骤的问题。
- **#8889**：修复 Rollout 缓存轨迹重放时，硬编码 `index=0` 导致丢失原始 session/index 标识的问题。
- **#8884**：修复 V1 rollout 中 MoE 负载指标与实际响应序列不对齐的问题（原代码错误地截取了相同的后缀）。
- **#8886**：修复数学奖励评估 (`math_reward.py`) 中的多个边缘情况，包括处理 `\fbox`、混合 `\boxed` 间距及多单位情况，减少误报和崩溃。
- **#8885**：修复 `seqlen_balancing.py` 中处理非连续 UID、零 token 掩码及长度不均时引发的崩溃。

**新特性与性能优化**
- **#8892**：为 TPU 静态形状微批次引入全局 FFD 序列打包计划（通过 `VERL_TPU_GLOBAL_PACK` 开启），提升 TPU 引擎效率。
- **#8890**：新增针对 Ascend A3 的 CANN 9.1.0 SGLang 镜像，集成 Python 3.12、SGLang v0.5.20 及 torch-npu。
- **#8888**：重构融合线性交叉熵模块，跨 CUDA 与 Ascend 平台提供统一的 `triton`、`tilelang` 和 `liger` 后端，并针对 SM90 Hopper 等架构进行深度优化。
- **#8887**：vLLM 在权重同步期间预分配共享内存（SHM）页面以提升性能，并修复了 Ascend 设备清理时触发 `npu-smi` 查询导致的潜在问题。

---

### 🚀 Release 摘要
近期无新的版本发布。

---

## 🐛 Issues

### #8893 — [[Bug] SGLangHttpServer.abort_all_requests leaves _generation_allowed permanently cleared on asyncio.CancelledError, deadlocking generate()](https://github.com/verl-project/verl/issues/8893)
- **作者**: SendoRay  **时间**: 2026-11-28 14:21 CST
- **摘要**: ### Summary In `verl/workers/rollout/sglang_rollout/async_sglang_server.py:760-776` (`SGLangHttpServer.abort_all_requests`, commit `22a8903ae6e661adc19ea9f8af2b9397eb5773c9`), `_generation_allowed` is cleared before awaiting `pause_generation(...)` and restored only under `except Exception:`:  ```py…

### #8891 — [[Bug] V1 trainer: _compute_advantage() always requests ref_log_prob, so every no-reference-policy recipe (default config, all OPD examples) crashes on step 1](https://github.com/verl-project/verl/issues/8891)
- **作者**: cyin-git  **时间**: 2026-11-28 11:27 CST
- **标签**: bug
- **摘要**: ### System Info  ### Summary  In verl 0.10.0.dev, `verl/trainer/main_ppo.py` always starts the V1 trainer (`main_ppo.py:203` -> `TaskRunnerV1`), and the V1 trainer cannot run **any** configuration with `use_reference_policy == False`. `_compute_advantage()` hard-codes `"ref_log_prob"` into the Trans…

## 🔀 Pull Requests

### #8895 — [factory: resolve issue #144](https://github.com/verl-project/verl/pull/8895)
- **作者**: 995114879  **时间**: 2026-11-28 15:30 CST
- **摘要**: ## 摘要 never fired a hook (e.g. accumulated in earlier micro-batches, as with an MoE expert unused by the last batch) is detected before any stepping, so the whole update is skipped atomically.    - `_reduce_partial_norm()` now adds the norm of those leftover accumulated gradients, so the fused grad …

### #8894 — [[trainer] fix: request ref_log_prob in _compute_advantage only when a reference policy is used](https://github.com/verl-project/verl/pull/8894)
- **作者**: cyin-git  **时间**: 2026-11-28 15:14 CST
- **摘要**: ### What does this PR do?  Fixes the V1 trainer crash on every configuration that has no reference policy, i.e. `algorithm.use_kl_in_reward=False` together with `actor_rollout_ref.actor.use_kl_loss=False`, which is also the default.  `PPOTrainer._compute_advantage()` selects `ref_log_prob` from Tran…

### #8892 — [[trainer] feat: global FFD sequence packing plan for static-shape (TPU) micro-batches (VERL_TPU_GLOBAL_PACK)](https://github.com/verl-project/verl/pull/8892)
- **作者**: jialei777  **时间**: 2026-11-28 14:00 CST
- **摘要**: ### What does this PR do?  Adds an opt-in, controller-side **global sequence-packing plan** for static-shape engines (the TPU TorchTitan engine with `VERL_TPU_BIN_SLOT_LEN`), enabled by `VERL_TPU_GLOBAL_PACK=1`. It changes only *which* sequences each DP rank receives in `_balance_batch`; dispatch, t…

### #8890 — [[ci, sglang] feat: add CANN 9.1.0 A3 SGLang image](https://github.com/verl-project/verl/pull/8890)
- **作者**: fangweii1  **时间**: 2026-11-28 10:45 CST
- **摘要**: ### What does this PR do?  Add an Ascend A3 image for CANN 9.1.0, Python 3.12, SGLang v0.5.20 and torch-npu 2.10.0.post6, with the Megatron/MindSpeed training components.  Extend `docker-build-ascend-sglang-a3.yml` using the two-stage pattern from `docker-build-ascend-a3.yml`: build on native amd64 …

### #8889 — [[rollout] fix: replay cached trajectory rows with original session/index identity](https://github.com/verl-project/verl/pull/8889)
- **作者**: orange-hlady  **时间**: 2026-11-28 00:59 CST
- **摘要**: ### What does this PR do?  Fixes items 1 and 3 in #8777.  `RolloutTqSkip.load_dump_data` rebuilt trajectory keys with a hardcoded `index=0` suffix and sized the replay with `for session_id in range(n)`:  - rows whose key index was > 0 were dropped; - rows were relabelled across sessions, so the key'…

### #8888 — [[perf, hardware, training_utils] feat: optimize CUDA/Ascend fused linear cross entropy with Triton and TileLang](https://github.com/verl-project/verl/pull/8888)
- **作者**: wuxibin89  **时间**: 2026-11-28 00:20 CST
- **摘要**: ### What does this PR do?  Modularizes `verl/utils/kernel/linear_cross_entropy` into unified `triton`, `tilelang`, and `liger` backends across CUDA GPUs and Ascend NPUs, optimizes the CUDA (`SM90` Hopper / `SM100` Blackwell) and Ascend (`910` / `950`) fused linear cross-entropy kernels for both sing…

### #8887 — [[vllm, perf] feat: preallocate SHM payload pages during weight sync](https://github.com/verl-project/verl/pull/8887)
- **作者**: WanhaoZhang  **时间**: 2026-11-27 23:43 CST
- **摘要**: ### What does this PR do?  When the vLLM rollout adapter selects shared memory, both endpoints still query device IPC support during cleanup. On Ascend that query invokes `npu-smi`. In one upstream replay its output failed to decode, so an otherwise completed SHM transfer raised during cleanup. This…

### #8886 — [[reward] fix: handle \fbox, mixed \boxed spacing, and multiple units in math_reward](https://github.com/verl-project/verl/pull/8886)
- **作者**: peterzhang987723  **时间**: 2026-11-27 23:03 CST
- **摘要**: ## Summary Fixes several corner-case crashes and false negatives in `verl/utils/reward_score/math_reward.py`:  1. **`last_boxed_only_string`**: Take `max(rfind("\\boxed"), rfind("\\fbox"))` instead of letting an earlier spaced `\\boxed ` shadow a trailing `\\boxed{...}` or `\\fbox{...}`. 2. **`remov…

### #8885 — [[utils] fix: handle non-contiguous UIDs, zero-token masks, and uneven lengths in seqlen_balancing](https://github.com/verl-project/verl/pull/8885)
- **作者**: peterzhang987723  **时间**: 2026-11-27 23:03 CST
- **摘要**: ## Summary Three surgical edge-case fixes in `verl/utils/seqlen_balancing.py` discovered during static + property-based auditing:  1. **Zero-token mask crash in `rearrange_micro_batches`**: Clamp `num_micro_batches = max(1, min(num_groups, ceildiv(total_seqlen, max_token_len)))` so all-zero attentio…

### #8884 — [[trainer] fix: align V1 rollout MoE load metrics to responses](https://github.com/verl-project/verl/pull/8884)
- **作者**: WanhaoZhang  **时间**: 2026-11-27 19:15 CST
- **摘要**: ### What does this PR do?  V1 reads rollout trajectories from TransferQueue and right-pads the full sequences. The MoE metrics code takes the same suffix from every row. With prompts of 4 and 12 tokens and responses of 4 tokens each, it reads positions 12–14 for both rows. The shorter row's routed r…
