# verl-project/verl — 动态追踪

> 生成时间: 2026-11-20 15:40 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🐛 Issues 归纳
近期 Issue 主要集中在 v0.9.1 版本的安装与运行时 Bug 反馈，以及架构重构讨论：
* **安装与环境问题**：
  * #8769: v0.9.1 无法按官方文档成功安装，`uv.lock` 中锁定的预构建 wheel URL（flash-attn, transformer-engine 等）均返回 404。
  * #8770: 官方预构建 Docker 镜像 `verlai/verl:vllm024.dev2` 中，`import megatron.bridge` 失败，提示找不到 `modelopt` 模块。
* **训练与稳定性 Bug**：
  * #8768: 在 8x B200 上运行 Qwen2-7B GSM8K GRPO recipe 时，`TransferQueue` 因 30 秒 ACK 超时丢弃样本，导致后续 `compute_log_prob` 报错 `KeyError: 'loss_mask'`。
* **架构重构提案 (RFC)**：
  * #8777: 提议重构 `SkipManager.rollout_tq`，以优化缓存 V1 TransferQueue 批次的重放机制。

### 🚀 Pull Requests 归纳
近期 PR 主要围绕**性能优化（减少同步阻塞）**、**算法数值稳定性修复**以及**硬件/CI 适配**展开：

* **性能优化（减少 Host-Device 同步）**：
  * #8776 & #8767: 优化 PPO loss 计算路径，移除每个 micro-batch 强制进行的 host 同步（`.item()` 调用）。改为在设备上保留标量指标并延迟批量读取，大幅提升后端（尤其是延迟执行后端）的执行效率。
* **算法与训练修复**：
  * #8774: 修复 rollout 中 Importance Sampling (IS) 标准差的计算问题，通过调整总体方差的计算方式解决 float32 精度下的数值抵消问题。
  * #8773: 修复 advantage 白化逻辑，将其改为在每个 mini-batch chunk 内单独进行，而非全量采样批次，修复了训练过程中的逻辑不一致问题。
* **硬件与设备支持**：
  * #8772: 补充了 B300 和 GB300 芯片的峰值 FLOPS 配置，修复了在此类硬件上 MFU（模型利用率）指标始终为 0 的问题。
  * #8771: 为 Qwen3-30B-A3B TPU GRPO 训练脚本新增可选的 `graph_trainer` 模式，通过图模式加速 actor 更新。
* **CI 测试与依赖维护**：
  * #8762: 将 TransferQueue 依赖从 0.1.11 升级至 0.1.12。
  * #8764 / #8763: 在 Ascend NPU 的 Qwen E2E 测试用例中启用 R3 并对比其性能表现。
  * #8775: 修复 NPU 单元测试中的 HCCL 端口绑定冲突问题，改为自动分配端口。

### 📦 Release
* 本次提供的数据中未包含 Release 信息。

---

## 🐛 Issues

### #8777 — [[RFC] Skipmanager Refactoring](https://github.com/verl-project/verl/issues/8777)
- **作者**: mikequan0425  **时间**: 2026-11-20 15:29 CST
- **摘要**: # Summary  `SkipManager.rollout_tq` currently replays a cached V1 TransferQueue batch by forcing each prompt to exactly `rollout.n` rows and rewriting every trajectory key as `{new_uid}_{session_id}_0`. This is incorrect for agentic RL, where one session may emit multiple `AgentLoopOutput` rows with…

### #8770 — [[Bug] verlai/verl:vllm024.dev2 (Dockerfile.stable.vllm): `import megatron.bridge` fails with No module named 'modelopt'](https://github.com/verl-project/verl/issues/8770)
- **作者**: AmineDiro  **时间**: 2026-11-20 06:27 CST
- **摘要**: ## Summary  In verl's prebuilt image `verlai/verl:vllm024.dev2` (the tag `docker/Dockerfile.stable.vllm` targets, and the image the install docs point to under "Install from docker image"), `import megatron.bridge` fails with `No module named 'modelopt'`. The Dockerfile installs Megatron-Bridge with…

### #8769 — [[Bug] v0.9.1 can't be installed as documented: uv.lock wheel URLs (flash-attn, transformer-engine, apex, megatron-bridge) return 404](https://github.com/verl-project/verl/issues/8769)
- **作者**: AmineDiro  **时间**: 2026-11-20 06:27 CST
- **摘要**: ## Summary  Installing verl v0.9.1 the documented way (`uv sync --frozen --all-packages --extra vllm --extra fsdp`, `docs/start/install.rst`) fails today: the v0.9.1 `uv.lock` pins four prebuilt wheels by URL under verl-wheelhouse release tags that no longer exist. The same files are still there und…

### #8768 — [[Bug] TransferQueue drops samples after a 30 s ACK timeout, then compute_log_prob fails with KeyError: 'loss_mask' (Qwen2-7B GSM8K GRPO recipe)](https://github.com/verl-project/verl/issues/8768)
- **作者**: AmineDiro  **时间**: 2026-11-20 06:27 CST
- **摘要**: ## Summary  On verl v0.9.1, verl's own Qwen2-7B-Instruct GRPO GSM8K recipe (the `qwen2-7b-fsdp2.log` baseline in `docs/algo/baseline.md`) crashed twice on 8x B200, both times on the first training step after a validation pass. Right before the crash, the agent-loop workers log `Timeout waiting for d…

## 🔀 Pull Requests

### #8776 — [[trainer, worker] feat: keep per-micro-batch scalar metrics on device, read them back lazily in one batch](https://github.com/verl-project/verl/pull/8776)
- **作者**: jialei777  **时间**: 2026-11-20 13:38 CST
- **摘要**: ### What does this PR do?  Removes the per-micro-batch host syncs that verl's PPO loss path forces between forward and backward.  `compute_policy_loss_*` call `.item()` on `pg_clipfrac` / `ppo_kl` / `pg_clipfrac_lower`, and `Metric.append` calls `.item()` on every scalar tensor metric (`actor/pg_los…

### #8775 — [[ci] fix: use automatic HCCL ports in NPU unit tests](https://github.com/verl-project/verl/pull/8775)
- **作者**: lxb007981  **时间**: 2026-11-20 10:55 CST
- **摘要**: ### What does this PR do? Use automatic HCCL ports in NPU unit tests  Overlapping process groups on the same NPU can both try to bind port 16666, causing HCCL initialization to fail. Set the host and NPU socket port ranges to auto at job scope so standalone torchrun tests inherit both settings.  ###…

### #8774 — [[trainer] fix: stabilize rollout IS standard deviation](https://github.com/verl-project/verl/pull/8774)
- **作者**: wuxibin89  **时间**: 2026-11-20 10:13 CST
- **摘要**: ### What does this PR do?  Population variance, centered before squaring. E[x^2] - (E[x])^2 cancels in float32 when sequence-level weights are constant across a sequence, so a one-row batch and the same row padded with empty rows disagree by ~1e-4.

### #8773 — [[trainer, worker, algo] fix: whiten advantages per mini-batch chunk in TrainingWorker](https://github.com/verl-project/verl/pull/8773)
- **作者**: Doublrest  **时间**: 2026-11-20 09:18 CST
- **摘要**: ### What does this PR do?  Fixes #8766.  In V1 trainers (`sync` and `colocate_async`), `_compute_advantage()` whitens advantages once over the full sampled batch, whereas `TrainingWorker.train_mini_batch()` splits the batch into `ppo_mini_batch_size` optimizer chunks without re-normalizing per chunk…

### #8772 — [[hardware] fix: add B300 and GB300 peak FLOPS](https://github.com/verl-project/verl/pull/8772)
- **作者**: hxw12219  **时间**: 2026-11-20 08:06 CST
- **摘要**: ### What does this PR do?  B300 and GB300 are missing from `_DEVICE_FLOPS`, so `get_device_flops()` returns `inf` on them and `perf/mfu/actor` and `perf/mfu/actor_infer` are always 0. This PR adds their peak FLOPS.  ### Checklist Before Starting  - [x ] Search for similar PRs. Paste at least one que…

### #8771 — [[hardware, recipe] feat: opt-in graph-mode actor update for the Qwen3-30B-A3B TPU recipe](https://github.com/verl-project/verl/pull/8771)
- **作者**: wenjung2007  **时间**: 2026-11-20 07:46 CST
- **摘要**: ### What does this PR do?  Adds an opt-in `TRAINER_MODE=graph_trainer` switch to the Qwen3-30B-A3B TPU GRPO recipe (`examples/tpu/grpo/run_qwen3_30b_a3b_torchtitan.sh`). It runs the actor update through TorchTitan's `TpuGraphTrainer`: forward, fused log-prob, PPO loss and backward of a micro-batch a…

### #8767 — [[trainer, training_utils, perf] feat: deferred host scalars for the policy loss metrics](https://github.com/verl-project/verl/pull/8767)
- **作者**: ethannnnnn  **时间**: 2026-11-20 00:16 CST
- **摘要**: ### What does this PR do?  This is item 3 of #8595.  On a lazily executing backend (torch_tpu's DEFER_AND_FUSE eager mode is one), every `Tensor.item()` or `if tensor:` makes the host wait for the device.  The bypass-mode `ppo_loss` with token-level rollout IS reads 36 device values on the host per …

### #8764 — [[ci] test: compare R3 performance in Ascend Qwen E2E cases](https://github.com/verl-project/verl/pull/8764)
- **作者**: Rain6ow  **时间**: 2026-11-19 20:30 CST
- **摘要**: The existing Qwen3.5-35B Megatron and Qwen3-30B MindSpeed Ascend E2E cases run without rollout router replay and stop after one training step. This draft enables R3 in both launchers and uses the existing CI jobs to compare six training steps with replay disabled against six steps with R3 enabled.  …

### #8763 — [[ci] test: compare R3 performance in Ascend Qwen E2E cases](https://github.com/verl-project/verl/pull/8763)
- **作者**: Rain6ow  **时间**: 2026-11-19 20:14 CST
- **摘要**: The existing Qwen3.5-35B Megatron and Qwen3-30B MindSpeed Ascend E2E cases run without rollout router replay and stop after one training step. This draft enables R3 in both launchers and uses the existing CI jobs to compare six training steps with replay disabled against six steps with R3 enabled.  …

### #8762 — [[ci, docker, deps] chore: bump TransferQueue to 0.1.12](https://github.com/verl-project/verl/pull/8762)
- **作者**: OutstanderWang  **时间**: 2026-11-19 20:08 CST
- **摘要**: Bumps TransferQueue from `0.1.11` to `0.1.12`. The scope follows the `0.1.11` bump in #8181: `requirements.txt`, `requirements-npu.txt`, `pyproject.toml`, `uv.lock`, the install hint the mock client raises, the e2e workflows that install TransferQueue explicitly, and the two ROCm images. Also `e2e_p…
