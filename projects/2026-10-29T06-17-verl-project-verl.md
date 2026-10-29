# verl-project/verl — 动态追踪

> 生成时间: 2026-10-29 14:17 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 🐛 Issue 摘要
- **#8433 SFT 训练器断点恢复时未能保持全局 CPU RNG 状态**
  作者指出 `SFTTrainer` 在构建 `StatefulDataLoader` 时未传入 `generator`，导致每次迭代构造时都会从全局 CPU RNG 中提取 `base_seed`。这使得 SFT 训练在恢复时无法正确还原随机数状态，可能影响训练的可重复性。

### 🔧 Pull Request 摘要

**核心修复**
- **#8434 修复 SFT 数据加载器的随机种子问题**：对应 Issue #8433，通过为训练/验证的 `StatefulDataLoader` 显式提供生成器并设置种子，确保断点恢复后能保持之前恢复的 CPU RNG 状态。
- **#8436 修复 Agent 循环中重复惩罚失效问题**：修复了 `AgentLoopWorker` 及 V1 版本的 `AgentLoopWorkerTQ` 强制将请求级别的 `repetition_penalty` 覆盖为 `1.0` 的行为，现在能正确应用配置的重复惩罚参数。
- **#8435 优化 SGLang 权重同步时的显存管理**：减少了 SGLang 分桶权重同步期间张量生命周期的重叠，当未融合的视图溢出当前桶时，会在该桶消费后进行压缩，有助于降低显存峰值占用。

**重构与改进**
- **#8438 & #8439 现代化 Megatron 集成**：作为 RFC #8324 路线图的一部分（1/5 和 2/5），集中处理了 Megatron 的兼容性层，并改用原生输出回调，提升代码可维护性。
- **#8437 测试硬件解耦**：对测试套件进行第二轮清理，移除硬编码的 `"cuda"` 字符串，使测试逻辑与具体硬件设备无关。
- **#8432 更新 ROCm Docker 环境**：更新 ROCm Dockerfile 以支持最新的 Primus v26.8 训练镜像，并同步刷新了相关文档。

**依赖更新 (Dependabot)**
- **#8429** 将 `torch` 从 2.13.0 升级至 2.14.1
- **#8430** 将 `flash-attn` 从 2.8.3 升级至 2.8.3.post2
- **#8431** 将 `spmd-types` 从 0.2.3 升级至 0.3.1

### 🚀 Release 摘要
- 本阶段无新版本发布。

---

## 🐛 Issues

### #8433 — [[trainer] SFT resume does not keep the restored global CPU RNG: StatefulDataLoader is built without a generator](https://github.com/verl-project/verl/issues/8433)
- **作者**: JonnesLin  **时间**: 2026-10-29 07:35 CST
- **摘要**: ### System Info  - verl `main` @ 2ab3529 (2026-10-27) - torch 2.15.0, torchdata 0.11.0 (also reproduced the same mechanism on torch 2.10 / torchdata 0.11)  ### Information  - [x] The official example scripts  ### Tasks  - [x] An officially supported task in the `examples` folder  ### Reproduction  `…

## 🔀 Pull Requests

### #8439 — [[megatron, model] refactor: modernize Megatron integration - use native output callbacks [2/5]](https://github.com/verl-project/verl/pull/8439)
- **作者**: ISEEKYAN  **时间**: 2026-10-29 11:02 CST
- **摘要**: ### What does this PR do?  Roadmap item 2 of [RFC #8324](https://github.com/verl-project/verl/issues/8324), split from [integration preview #25](https://github.com/ISEEKYAN/verl/pull/25), **depends on [upstream #8438](https://github.com/verl-project/verl/pull/8438)**. This upstream draft targets `ma…

### #8438 — [[megatron] refactor: modernize Megatron integration - centralize compatibility [1/5]](https://github.com/verl-project/verl/pull/8438)
- **作者**: ISEEKYAN  **时间**: 2026-10-29 11:02 CST
- **摘要**: ### What does this PR do?  Roadmap item 1 of [RFC #8324](https://github.com/verl-project/verl/issues/8324), split from [integration preview #25](https://github.com/ISEEKYAN/verl/pull/25). Centralize bootstrap, engine, construction, per-model MTP and forward compatibility selection in `compat.py`. Ea…

### #8437 — [[tests,hardware] pytest changes for device-agnostic case ](https://github.com/verl-project/verl/pull/8437)
- **作者**: kahlun  **时间**: 2026-10-29 09:26 CST
- **摘要**: ## What does this PR do?  Follow-up to the earlier cuda/nccl test device-agnostic cleanup. A second pass over `tests/` found several files that were still hardcoding `"cuda"` or asserting against `torch.cuda` directly instead of going through `verl.utils.device`'s platform helpers (`get_device_name(…

### #8436 — [[rollout] fix: honor repetition penalty in agent loops](https://github.com/verl-project/verl/pull/8436)
- **作者**: taking-lying-flat  **时间**: 2026-10-29 09:21 CST
- **摘要**: ### What does this PR do?  Stops both AgentLoop implementations from overriding the configured rollout `repetition_penalty` with a request-level `1.0`.  `AgentLoopWorker` and the V1 `AgentLoopWorkerTQ` currently include `repetition_penalty=1.0` in every request. The vLLM async server intentionally u…

### #8435 — [[rollout, sglang] fix: reduce tensor lifetime overlap during bucketed weight sync](https://github.com/verl-project/verl/pull/8435)
- **作者**: Levius-Fubuki  **时间**: 2026-10-29 09:02 CST
- **摘要**: ### What does this PR do?  Reduce overlapping tensor lifetimes during SGLang bucketed weight sync. A non-fused view that overflows the current bucket is now compacted after that bucket has been consumed. The generator drops references to flushed entries, and the adapter drops `params_batch` after th…

### #8434 — [[trainer] fix: seed SFT dataloader generators so resume keeps the restored CPU RNG](https://github.com/verl-project/verl/pull/8434)
- **作者**: JonnesLin  **时间**: 2026-10-29 07:35 CST
- **摘要**: ### What does this PR do?  Fixes #8433.  `SFTTrainer` built its train/val `StatefulDataLoader`s without a `generator`, so every iterator construction drew a `base_seed` from the global CPU RNG. On resume that draw happens right after the checkpoint manager restores `torch.get_rng_state()`, so the re…

### #8432 — [[hardware, doc] feat: update the ROCm Dockerfile for Primus v26.8](https://github.com/verl-project/verl/pull/8432)
- **作者**: tmm77  **时间**: 2026-10-29 06:25 CST
- **摘要**: ### What does this PR do?  Updates [`docker/rocm/Dockerfile.rocm`](docker/rocm/Dockerfile.rocm) for the current Primus ROCm training image and refreshes [`docker/rocm/README.md`](docker/rocm/README.md) so the documented build matches that recipe. This follows the ROCm Dockerfile/README added in #661…

### #8431 — [build(deps-dev): bump spmd-types from 0.2.3 to 0.3.1](https://github.com/verl-project/verl/pull/8431)
- **作者**: dependabot[bot]  **时间**: 2026-10-29 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [spmd-types](https://github.com/meta-pytorch/spmd_types) from 0.2.3 to 0.3.1. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/meta-pytorch/spmd_types/releases">spmd-types's releases</a>.</em></p> <blockquote> <h2>v0.3.1</h2> <p><strong>Full Changelog<…

### #8430 — [build(deps-dev): bump flash-attn from 2.8.3 to 2.8.3.post2](https://github.com/verl-project/verl/pull/8430)
- **作者**: dependabot[bot]  **时间**: 2026-10-29 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [flash-attn](https://github.com/Dao-AILab/flash-attention) from 2.8.3 to 2.8.3.post2. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/Dao-AILab/flash-attention/releases">flash-attn's releases</a>.</em></p> <blockquote> <h2>v2.8.3.post2</h2> <p>Mainten…

### #8429 — [build(deps-dev): bump torch from 2.13.0 to 2.14.1](https://github.com/verl-project/verl/pull/8429)
- **作者**: dependabot[bot]  **时间**: 2026-10-29 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [torch](https://github.com/pytorch/pytorch) from 2.13.0 to 2.14.1. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/pytorch/pytorch/releases">torch's releases</a>.</em></p> <blockquote> <h2>PyTorch 2.14.1 Release</h2> <p>This release is meant to fix th…
