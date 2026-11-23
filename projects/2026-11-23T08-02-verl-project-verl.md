# verl-project/verl — 动态追踪

> 生成时间: 2026-11-23 16:02 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📋 Issue 摘要
- **预计算 flash-attention 参数的性能权衡 (#8797)**
  - **内容**：针对 Ascend NPU 上的 varlen flash-attention 路径，提出预计算 `cu_seq_lens` 参数以消除每层的主机-设备同步。
  - **结果**：在隔离测试中获得了 150 倍的巨大性能提升，但在 `FSDP1 + param_offload` 模式下的更新阶段引发了 62% 的性能回退。这表明在特定分布式训练配置下存在性能瓶颈，需要进一步优化。

### 🔧 Pull Request 摘要

**✨ 新特性与重要变更**
- **引入单基础模型多 LoRA 常驻训练 (#8805)**：实现了在单个共享基础模型上同时常驻多个 LoRA 适配器的基础功能（Phase 1），为多 LoRA 训练打下基础。
- **合并 PEAR 引擎补丁 (#8796)**：将 PEAR（用于具身智能机器人训练的后训练引擎）的 verl 补丁系列应用至当前主干分支。

**🚀 性能优化**
- **Megatron THD 打包向量化 (#8801)**：当 `cp_size == 1` 时，通过向量化操作优化将 NestedTensor 打包为扁平 THD 缓冲区的过程，消除了原有的 Python 循环，提升了处理效率。

**🐛 Bug 修复**
- **修复 Rollout 服务启动阻塞 (#8806)**：修复了 Uvicorn 在加载配置时异常退出，导致 rollout helper 无限期等待端口就绪事件的问题。
- **修复 `prime_code` 评测安全隐患 (#8800)**：将生成的代码解决方案放入独立进程中运行，防止其读取并作弊获取预期测试输出。
- **修复 Profiler 配置校验 (#8802)**：当触发了全局性能分析（`global_profiler.steps`）但没有角色启用对应开关时，现在会发出警告提示。

**📦 依赖与 CI 维护**
- **升级 Megatron 技术栈 (#8798)**：将 CUDA Megatron 环境升级至 Megatron-Core 0.20.1 和 Megatron-Bridge 0.7.1。
- **更新 TPU GRPO 镜像 (#8803)**：将镜像中的 vLLM 升级至 0.33.0，并同步更新 torchtitan 和 TransferQueue。
- **Ascend CI 修复 (#8804, #8799)**：移除了已废弃的集群内 PyPI 镜像配置；将 Qwen3-30B 的对比测试隔离到独立工作流中，以避免受其他测试的 HCCL 资源错误影响。

### 🚀 Release 摘要
- 本期无新版本发布。

---

## 🐛 Issues

### #8797 — [[perf] Precomputing flash-attention cu_seq_lens kwargs: 150x isolated win but +62% update-phase regression under FSDP1 + param_offload](https://github.com/verl-project/verl/issues/8797)
- **作者**: dahai80  **时间**: 2026-11-22 22:26 CST
- **摘要**: ## Summary  We tested eliminating the per-layer host-device syncs in the varlen flash-attention path (the `prepare_fa_kwargs_from_position_ids` / `npu_flash_attn_varlen_func` route on Ascend NPU, transformers 5.10.4, `use_remove_padding=True`, Ulysses SP=4). The change precomputes `cu_seq_lens_q/k` …

## 🔀 Pull Requests

### #8806 — [[rollout] fix: propagate HTTP server failures before port discovery](https://github.com/verl-project/verl/pull/8806)
- **作者**: doublestrips  **时间**: 2026-11-23 15:33 CST
- **摘要**: ### What does this PR do?  If Uvicorn exits while loading the configuration, the rollout helper can wait indefinitely for the port-ready event. A rejected ASGI lifespan startup can also raise an AttributeError while trying to read sockets that were never initialized.  Wake the port waiter when the s…

### #8805 — [[fsdp, ckpt, worker] feat: phase 1 resident multi-LoRA adapters on one base model](https://github.com/verl-project/verl/pull/8805)
- **作者**: arun-elr  **时间**: 2026-11-23 15:18 CST
- **摘要**: ### What does this PR do?  Implements the Phase 1 foundation for resident multi-LoRA training on a single shared base model (refs #7909, follow-up to #4421). Multiple adapters can live on one base model simultaneously with per-adapter FSDP2 sharding, isolated optimizer/scheduler/scaler updates, adap…

### #8804 — [[ci] fix: remove retired in-cluster pypi mirror from Ascend workflows](https://github.com/verl-project/verl/pull/8804)
- **作者**: ccijunk  **时间**: 2026-11-23 15:11 CST
- **摘要**: ### What does this PR do?  The self-hosted Ascend CI runners no longer use the in-cluster nginx pypi/apt mirror (`cache-service.nginx-pypi-cache.svc.cluster.local`). Public indexes are now reached through the cluster-level squid caching proxy, so the hard-coded internal-mirror env vars are dead conf…

### #8803 — [[hardware, docker, vllm] chore: update vllm-torchtpu (vLLM 0.33.0), torchtitan, and TransferQueue 0.1.12 in the TPU GRPO image](https://github.com/verl-project/verl/pull/8803)
- **作者**: jialei777  **时间**: 2026-11-23 14:34 CST
- **摘要**: ### What does this PR do?  Updates `examples/tpu/grpo/Dockerfile` and `examples/tpu/grpo/README.md` so the TPU GRPO image matches current `verl` `main`, `verl-hardware-plugin` `main`, and the `deepswe_tpu` Qwen3.5-35B / Qwen3.5-397B FP8 rollout recipes (`verl-project/verl-recipe#174`, `#177`):  - **…

### #8802 — [[cfg, perf] fix: warn when profiling is triggered but no role enables it](https://github.com/verl-project/verl/pull/8802)
- **作者**: tx-queyu  **时间**: 2026-11-23 14:29 CST
- **摘要**: ### What this fixes  Closes #8789.  `global_profiler.steps` and each role's `profiler.enable` are two halves of one switch, and nothing checks them against each other:  - `global_profiler.steps` decides on **which steps** the trainer calls `start_profile` / `stop_profile` (`trainer_base.py::_start_p…

### #8801 — [[megatron, perf] perf: vectorize THD packing when cp_size == 1](https://github.com/verl-project/verl/pull/8801)
- **作者**: helen88  **时间**: 2026-11-23 13:44 CST
- **摘要**: ### What does this PR do?  `preprocess_thd_engine` packs a jagged `NestedTensor` into a flat THD buffer with one Python loop iteration per sequence. When `cp_size == 1` the nested tensor already stores its rows in THD order, so the `input_ids[i]` inside that loop is a round trip through `NestedTenso…

### #8800 — [[reward] fix: run prime_code solutions in a separate process so they cannot read expected outputs](https://github.com/verl-project/verl/pull/8800)
- **作者**: haehoonseo  **时间**: 2026-11-23 10:42 CST
- **摘要**: ### What does this PR do?  `prime_code.testing_util.run_test` execs the generated solution in the same process, and on the same call stack, as the code that holds the test cases. The solution can therefore walk `sys._getframe()` up to `run_test` and read `in_outs["outputs"][index]`, the expected out…

### #8799 — [[ci] test: isolate Ascend Qwen3-30B R3 comparison](https://github.com/verl-project/verl/pull/8799)
- **作者**: Rain6ow  **时间**: 2026-11-23 09:39 CST
- **摘要**: Qwen3-30B in #8779 failed with HCCL error 7 (resource unavailable), and the preceding PPO/FSDP test in the same job failed with the same error. Add a dedicated workflow that runs only the reduced Qwen3-30B MindSpeed baseline/R3 comparison in its own job, without any preceding training cases. This is…

### #8798 — [[megatron, env] chore: upgrade Megatron-Core to 0.20.1 and Bridge to 0.7.1](https://github.com/verl-project/verl/pull/8798)
- **作者**: amenomori  **时间**: 2026-11-23 09:39 CST
- **摘要**: ### What does this PR do?  Upgrade the CUDA Megatron stack to the released Megatron-Core 0.20.1 and Megatron-Bridge 0.7.1. Core stays pinned to its release tag. The upstream Bridge wheel is available on PyPI for both supported Linux architectures. The rollout stack keeps its existing Transformers 5.…

### #8796 — [[trainer, rollout, fsdp, model, ckpt] feat: PEAR's verl overlay on current main](https://github.com/verl-project/verl/pull/8796)
- **作者**: gemcollector  **时间**: 2026-11-22 18:26 CST
- **摘要**: ### What does this PR do?  Applies the changes to verl that [PEAR](https://github.com/mit-han-lab/pear), a Post-training Engine for Agentic Robotics (Apache-2.0), carries as a patch series, its verl overlay, to current `main` (453954b3).  PEAR trains reasoning models to use robots with reinforcement…
