# verl-project/verl — 动态追踪

> 生成时间: 2026-11-11 14:58 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
近期用户反馈了多个平台与算法层面的 Bug：
1. **ROCm 硬件兼容问题**：在执行 FP8_BLOCK rollout 权重同步时报错，提示找不到 `tilelang` 模块（#8640）。
2. **算法计算偏差**：
   - GSPO 算法的序列重要性采样 (IS) 比率计算有误，错误地包含了被 rollout correction 拒绝的 token（#8636）。
   - Clip-Cov 的 token 选择逻辑受 PPO microbatch 大小影响，导致结果不确定（#8634）。
3. **Megatron 训练状态恢复缺陷**：策略快照在保存时遗漏了持久化缓冲区，导致恢复时缺失如 `expert_bias` 等关键数据（#8632）。

---

### 🔧 PR 动态
合并了多项新特性、核心修复及重构，重点如下：

**🚀 重要新特性与重构**
- **支持 vLLM-Ascend 原生权重广播**（#8639）：新增可选的 `vllm_native_hccl` checkpoint 后端，支持在 FSDP2 训练与独立 vLLM-Ascend rollout 间直接广播全量权重，优化硬件协同。
- **QAT 配置重构（破坏性更新）**（#8638）：因 VeOmni 废弃了全局 `qat_implementation`，代码重构为按模型系列选择量化感知训练实现。

**🛠 核心 Bug 修复**
- 修复 Megatron 快照未保存持久化缓冲区的 Bug（#8633，对应 #8632）。
- 修复 Clip-Cov token 随机选择受 microbatch 影响的问题，改为在优化器 minibatch 层面进行选择（#8635，对应 #8634）。
- 在 IS 和 RS 序列指标计算中，自动跳过无有效 token 的数据行，避免计算异常（#8628）。
- 修复 TPU GRPO 镜像中 TransferQueue 依赖版本（升至 0.1.11）不匹配的问题（#8627）。

**📝 测试与文档**
- 新增 CPU 测试用例，确保所有策略损失函数在 microbatch 变动下保持正确性，并覆盖被拒绝的 token（#8637）。
- 修正多处文档字符串与函数签名不一致的问题（#8631），并澄清了 Ascend 镜像与 Qwen3-ASR 补丁的使用指南（#8630）。
- 更新 TPU GRPO 镜像中 `tpu-sync-torch` 和 `vllm-torchtpu` 的版本以修复功能缺陷（#8626）。

---

### 📦 Release 动态
近期无新版本发布。

---

## 🐛 Issues

### #8640 — [[ROCm] FP8_BLOCK rollout weight sync fails with "No module named 'tilelang'"](https://github.com/verl-project/verl/issues/8640)
- **作者**: a1021223029-lgtm  **时间**: 2026-11-11 14:00 CST
- **标签**: bug
- **摘要**: ### System Info  ----------Python Info---------- Version      : 3.12.3 Compiler     : GCC 13.3.0 Build        : ('main', 'Mar 23 2026 19:04:32') Arch         : ('64bit', 'ELF') ------------Pip Info----------- Version      : 26.1.2 Directory    : /opt/venv/lib/python3.12/site-packages/pip vllm       …

### #8636 — [[Bug] GSPO sequence IS ratio includes tokens rejected by rollout correction (IcePop)](https://github.com/verl-project/verl/issues/8636)
- **作者**: LOGO127  **时间**: 2026-11-11 11:19 CST
- **摘要**: ### System Info  Reproduced on current `main` (commit `63913d1d`) with a CPU-only tensor reproducer. Python 3.12, torch 2.15.  ### Information  - [x] The official example scripts  Any GSPO run with rollout correction enabled: `actor.policy_loss.loss_mode="gspo"` + `algorithm.rollout_correction` (`ro…

### #8634 — [[Bug] Clip-Cov token selection changes with the PPO microbatch size](https://github.com/verl-project/verl/issues/8634)
- **作者**: LOGO127  **时间**: 2026-11-11 11:15 CST
- **摘要**: ### System Info  Reproduced on current `main` (commit `63913d1d`) with a CPU-only tensor reproducer; no GPU/cluster needed. Python 3.12, torch 2.15.  ### Information  - [x] The official example scripts  Any recipe using `actor.policy_loss.loss_mode="clip_cov"` (e.g. DAPO-style runs with Clip-Cov ena…

### #8632 — [[Bug] Megatron policy snapshots do not restore persistent buffers](https://github.com/verl-project/verl/issues/8632)
- **作者**: ttqiao1617  **时间**: 2026-11-11 11:11 CST
- **摘要**: ### System Info  verl `b10a5d421a1bb3ab4067d6aa47b8e5ead1a34df3`, PyTorch 2.13.0+cu130, Megatron-Core 0.19.2, one H100, Python 3.12.  ### Information  - [x] My own modified scripts  ### Tasks  - [x] My own task or dataset: a policy snapshot regression for Decoupled PPO; no dataset is needed.  ### Re…

## 🔀 Pull Requests

### #8639 — [[ckpt, fsdp, vllm] feat: broadcast full weights directly to disaggregated vLLM-Ascend workers](https://github.com/verl-project/verl/pull/8639)
- **作者**: yeyunu  **时间**: 2026-11-11 12:17 CST
- **摘要**: ## What does this PR do?  Add an opt-in `vllm_native_hccl` checkpoint backend for disaggregated FSDP2 training and standalone vLLM-Ascend rollout. Training rank 0 uses vLLM-Ascend's native HCCL trainer to broadcast full weights directly to each inference model worker. The existing Ray manager coordi…

### #8638 — [[BREAKING][veomni, cfg] refactor: select QAT per family after VeOmni drops qat_implementation](https://github.com/verl-project/verl/pull/8638)
- **作者**: amenomori  **时间**: 2026-11-11 11:57 CST
- **摘要**: ### What does this PR do?  VeOmni's op registry now selects quantization-aware training **per family** (ByteDance-Seed/VeOmni#1414) instead of one global `qat_implementation` recipe. This carries that change into verl's VeOmni engine: the engine forwards the new per-family selectors, the DeepSeek-V4…

### #8637 — [[trainer, tests] test: minibatch-invariance and rejected-token coverage for policy losses](https://github.com/verl-project/verl/pull/8637)
- **作者**: LOGO127  **时间**: 2026-11-11 11:45 CST
- **摘要**: ### What does this PR do?  Adds a CPU test file, `tests/trainer/ppo/test_policy_loss_invariance_on_cpu.py`, that enforces two correctness invariants across all registered policy losses, following the precedent established in #7829:  1. **Minibatch invariance** — a loss evaluated on the full optimize…

### #8635 — [[algo, fsdp] fix: select Clip-Cov tokens over the optimizer minibatch](https://github.com/verl-project/verl/pull/8635)
- **作者**: LOGO127  **时间**: 2026-11-11 11:16 CST
- **摘要**: ## Body  ### What does this PR do?  Fixes #8634  `compute_policy_loss_clip_cov` computes its covariance statistics, clip quota, and random token selection on whatever tensor reaches the loss function — i.e. per **microbatch**. Splitting the same optimizer minibatch into a different number of microba…

### #8633 — [[megatron, fully_async] fix: preserve persistent buffers in policy snapshots](https://github.com/verl-project/verl/pull/8633)
- **作者**: ttqiao1617  **时间**: 2026-11-11 11:12 CST
- **摘要**: ### What does this PR do?  Fixes #8632.  Megatron's policy snapshots currently save parameters but omit persistent model buffers. If a buffer such as `expert_bias` changes during training, restoring the old snapshot can leave a mix of old parameters and newer routing state. Decoupled PPO then recomp…

### #8631 — [[doc] fix: align docstrings with actual signatures](https://github.com/verl-project/verl/pull/8631)
- **作者**: YidaWeng  **时间**: 2026-11-11 10:26 CST
- **摘要**: ## What  Fixes the docstring/signature mismatches reported in #8567, and two more instances of the same drift in `core_algos.py`.  - `compute_policy_loss_vanilla` documented a removed `rollout_log_probs` parameter; it now documents `rollout_is_weights`, which is the tensor the function actually cons…

### #8630 — [[doc] fix: clarify Ascend image and Qwen3-ASR patch guidance](https://github.com/verl-project/verl/pull/8630)
- **作者**: renjunying2  **时间**: 2026-11-11 10:06 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  [doc] fix: clarify Ascend image and Qwen3-ASR patch guidance  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{module…

### #8628 — [[trainer] fix: skip rows without valid tokens in the IS and RS sequence metrics](https://github.com/verl-project/verl/pull/8628)
- **作者**: ethannnnnn  **时间**: 2026-11-11 02:48 CST
- **摘要**: ### What does this PR do?  Follow-up to #7785, as offered in our comment there. Part of #8595 (Qwen3.5 GRPO on TPU v7x, item 1).  #7785 made the per-sequence off-policy metrics skip rows whose `response_mask` is all zero. The IS and RS sequence metrics still reduce over every row: `rollout_is_seq_*`…

### #8627 — [[hardware] fix: install TransferQueue 0.1.11 in the TPU GRPO image](https://github.com/verl-project/verl/pull/8627)
- **作者**: ethannnnnn  **时间**: 2026-11-11 01:29 CST
- **摘要**: ### What does this PR do?  Part of #8595 (item 4). `examples/tpu/grpo/Dockerfile` installs `TransferQueue==0.1.8`, while the `verl-core` extra in `pyproject.toml` requires `TransferQueue>=0.1.11`. V1 async checkpoints save TransferQueue state through its checkpoint API, which TransferQueue added in …

### #8626 — [[hardware] chore: bump tpu-sync-torch and vllm-torchtpu in the TPU GRPO image](https://github.com/verl-project/verl/pull/8626)
- **作者**: xiaoxlu  **时间**: 2026-11-11 01:29 CST
- **摘要**: ### What does this PR do?  Updates two pins in `examples/tpu/grpo/Dockerfile`:  1. **`TPU_SYNC_VERSION`**: `0.0.1.dev20261012084241` -> `0.0.1.dev20261103081657`. The newer `tpu-sync-torch` returns a future from `WeightSynchronizer.d2h()`, which `verl-hardware-plugin`'s `return_after_d2h` option (ve…
