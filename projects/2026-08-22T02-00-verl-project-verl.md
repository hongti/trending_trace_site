# verl-project/verl — 动态追踪

> 生成时间: 2026-08-22 10:00 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📌 Issue 动态
*   **[RFC] Agent Loop 可观测性架构设计 (#7509)**：提出跨 VeRL、Uni-Agent 和 RL-Insight 三个仓库的协作方案。设计将仪表板协议保留在 RL-Insight，业务逻辑留在 Uni-Agent，而 verl 侧保持轻量级的训练器集成。
*   **[Bug] 工具 Schema 约束静默丢失 (#7505)**：指出 `OpenAIFunctionToolSchema` 存在缺陷，会静默丢弃合法的 JSON Schema 约束条件。

### 🔄 PR 动态
**🚀 新特性**
*   **配置驱动的 Checkpoint 回调 (#7513)**：新增 `trainer.checkpoint_callback_class` 配置项，允许用户通过全限定类名指定自定义的 `CheckpointCallback` 子类，由驱动器自动实例化。
*   **异步激活卸载 (#7510)**：为 veomni 模块引入异步激活 Offloading 支持，有望降低内存占用并提升训练效率。

**🛠️ 关键修复**
*   **分布式后端初始化缺陷 (#7512)**：修复 `initialize_global_process_group` 仅使用 NCCL/HCCl 后端，导致 CPU 操作（如 CPU broadcast）失败的问题。
*   **vLLM 权重同步竞态问题 (#7511)**：修复 DP>1 场景下，vLLM 权重更新暂停期间未能有效拦截 rollout 提交引发的竞态条件。
*   **vLLM CLI 参数丢失 (#7508)**：修复 `build_cli_args_from_config` 序列化时静默丢弃 `Optional[bool]` 的显式 `False` 值（如 `enable_prefix_caching=False`）的问题。
*   **NPU 设备 ID 映射错误 (#7507)**：修复当 Ray 未自动设置 `ASCEND_RT_VISIBLE_DEVICES` 时，物理 NPU ID 未正确映射到逻辑索引的问题。
*   **vLLM 多实例端口冲突 (#7504)**：修复同节点多 vLLM 实例启动时，因争用相同 torch 分布式端口导致 `EADDRINUSE` 报错的问题。
*   **FSDP 模型合并参数丢失 (#7503)**：修复 FSDP Checkpoint 合并为 HF 格式时，因张量别名导致参数被静默丢弃（表现为键重复或减少）的严重问题。

**⚡ 性能优化与清理**
*   **避免冗余 Micro-batch 打包 (#7506)**：当序列长度平衡判定仅需单个 micro-batch 时，直接返回原始 batch，避免无意义的重索引和重建开销。
*   **清理废弃配置 (#7502)**：移除已废弃的 `actor_rollout_ref.rollout.mode` 配置项。

### 📦 Release 动态
*   近期无新版本发布。

---

## 🐛 Issues

### #7509 — [[RFC] Agent Loop Observability Across VeRL, Uni-Agent and RL-Insight](https://github.com/verl-project/verl/issues/7509)
- **作者**: tardis-key  **时间**: 2026-08-21 15:06 CST
- **摘要**: ## Summary  Agent-loop observability is delivered by three coordinated repositories. The design keeps the dashboard protocol in RL-Insight, keeps trainer integration thin in verl, and keeps business instrumentation in Uni-Agent.  ## Repositories and responsibilities  | Repository | Responsibility | …

### #7505 — [[tool] bug: OpenAIFunctionToolSchema silently drops valid JSON Schema constraints](https://github.com/verl-project/verl/issues/7505)
- **作者**: shockbladenull  **时间**: 2026-08-21 10:32 CST
- **摘要**: ### System Info  - verl `main`: `040a28659773fd58eac91cc006d255826c6da03a` - Reproduced on checkout `483b8a009ba3a97563edee3a19887e4862b8094a`; its `verl/tools/schemas.py` blob (`bf1b23ab1e2817de410ec5eeab01ed769e24a012`) is identical to the blob on the `main` commit above - Python 3.11.15 - Pydanti…

## 🔀 Pull Requests

### #7513 — [[trainer, ckpt, cfg] feat: add config-driven checkpoint callback hook](https://github.com/verl-project/verl/pull/7513)
- **作者**: yueyiming2009  **时间**: 2026-08-21 21:12 CST
- **摘要**: ### What does this PR do?  Adds `trainer.checkpoint_callback_class`: a config key naming a user-defined `CheckpointCallback` subclass (fully qualified class name) that the driver instantiates and whose `on_save` hook it calls after each checkpoint save in both the classic `RayPPOTrainer` and the v1 …

### #7512 — [fix: update backend initialization in initialize_global_process_group…](https://github.com/verl-project/verl/pull/7512)
- **作者**: kahlun  **时间**: 2026-08-21 19:42 CST
- **摘要**: ### What does this PR do?  Problem  initialize_global_process_group initialized the process group using only get_nccl_backend() (e.g., nccl or hccl), which means CPU operations (e.g., broadcast for CPU tensors) fell back to the default single-backend behavior. This diverged from initialize_global_pr…

### #7511 — [[vllm, rollout] fix: gate rollout submission during weight-sync drain for DP>1](https://github.com/verl-project/verl/pull/7511)
- **作者**: EricMarcus-ai  **时间**: 2026-08-21 18:43 CST
- **摘要**: ### What does this PR do?  During a weight update, verl pauses the vLLM engines, waits for idle, swaps the weights, and resumes. The pause is vLLM's `pause_generation`, and everything turns on what that actually does: it stops the scheduler from running new requests, but it does not stop the engine …

### #7510 — [[veomni] feat:add async activation offloading support](https://github.com/verl-project/verl/pull/7510)
- **作者**: mikequan0425  **时间**: 2026-08-21 15:10 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  After https://github.com/ByteDance-Seed/VeOmni/pull/872.  The async activation offloading feature has been added, along with the co…

### #7508 — [[vllm] fix: honor explicit False on Optional[bool] engine args in CLI serialization](https://github.com/verl-project/verl/pull/7508)
- **作者**: zhtmike  **时间**: 2026-08-21 13:11 CST
- **摘要**: ### What does this PR do?  `build_cli_args_from_config` dropped every `False` boolean, so an explicit `enable_prefix_caching=False` emitted **no flag at all**. For vLLM `Optional[bool]` engine args an omitted flag means `None`, which resolves to **enabled** at engine-config time — so the trainer con…

### #7507 — [[worker, hardware] fix: map physical NPU device IDs to logical indices](https://github.com/verl-project/verl/pull/7507)
- **作者**: GJWu-zyx  **时间**: 2026-08-21 11:30 CST
- **摘要**: ### What does this PR do?  When `RAY_EXPERIMENTAL_NOSET_ASCEND_RT_VISIBLE_DEVICES` is set, Ray no longer automatically sets the `ASCEND_RT_VISIBLE_DEVICES` environment variable for each actor. In this case, the worker falls back to `ray.get_runtime_context().get_accelerator_ids()` to determine the l…

### #7506 — [[perf] avoid repacking single micro-batches](https://github.com/verl-project/verl/pull/7506)
- **作者**: XiaoBao-175  **时间**: 2026-08-21 11:22 CST
- **摘要**: ## Summary  When sequence-length balancing determines that the local batch fits in a single micro-batch, return the original batch instead of re-indexing and rebuilding every field.  This is especially useful for multimodal batches with jagged/nested tensors, where the identity path can otherwise cr…

### #7504 — [[vllm, rollout] fix: forward a race-safe port to single-node MultiprocExecutor (#6677)](https://github.com/verl-project/verl/pull/7504)
- **作者**: amanyagami  **时间**: 2026-08-21 10:28 CST
- **摘要**: ## What  #6677: multiple vLLM instances launched on the same node (common in verl's rollout setup) can select the same `torch.distributed.init_process_group` port and fail with `EADDRINUSE`.  ## Root cause (confirmed against real vllm==0.24.0 source, verl's pinned version)  For the single-node case …

### #7503 — [fix(model_merger): stop FSDP merge from silently dropping parameters via aliased tensors (#6259)](https://github.com/verl-project/verl/pull/7503)
- **作者**: amanyagami  **时间**: 2026-08-21 10:28 CST
- **摘要**: ## What  #6259 reports that merging a 64-GPU FSDP GRPO checkpoint into HF format produces `.safetensors` shards with the same key appearing to be "duplicated" (57 keys removed in shard 6/14, 41 in shard 8/14) — not reproducible at 32 GPUs.  ## Root cause  `FSDPModelMerger._merge_by_placement()`'s `R…

### #7502 — [[config] chore: remove deprecated actor_rollout_ref.rollout.mode (#4604)](https://github.com/verl-project/verl/pull/7502)
- **作者**: amanyagami  **时间**: 2026-08-21 10:28 CST
- **摘要**: ## What  Per the RFC in #4604 (labeled "good first issue" / "call for contribution" / "code quality"): `actor_rollout_ref.rollout.mode` was already dead weight before this change — `RolloutConfig.__post_init__` hard-rejected `"sync"` (raised `ValueError`) and only warned on anything but `"async"`, a…
