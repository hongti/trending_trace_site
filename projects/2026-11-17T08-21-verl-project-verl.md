# verl-project/verl — 动态追踪

> 生成时间: 2026-11-17 16:21 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🐛 Issues 动态
近期反馈的问题主要集中在 **ROCm 平台**的兼容性与稳定性上：
1. **ROCm 训练挂起** (#8712)：PPO 训练在 dataloader worker 启动时发生死锁并永久挂起，主要由 `KMP_INIT_AT_FORK=FALSE` 与 LLVM OpenMP 冲突导致。
2. **权重同步失败** (#8716)：在 ROCm 和 SGLang 环境下，进行 FP8_BLOCK rollout 权重同步时，因 TP scale 形状不匹配导致失败。
3. **CI 工作流异常** (#8717)：ROCm E2E 测试工作流出现意外取消的情况。

### 🔧 Pull Requests 动态
PR 活动包含多项重要修复、新特性以及破坏性重构：

**重要修复：**
* **修复 ROCm Dataloader 死锁** (#8713)：针对 Issue #8712，在 ROCm 的 E2E 工作流和 Dockerfile 中设置 `KMP_INIT_AT_FORK=TRUE`，解决了训练挂起的问题。
* **修复策略蒸馏长度爆炸** (#8714)：针对 OPD（On-policy distillation）中学生模型与教师模型 EOS token 不一致的问题，新增了 `distillation_loss.semantic_eos_token_ids` 选项进行修复。
* **修复 Checkpoint 配置保存** (#8710)：修复了 FSDP 和 Megatron 检查点管理器因异常静默丢弃 generation config 的问题，现在会强制保存或记录未保存日志。

**新特性与优化：**
* **NPU 增量同步** (#8718)：在检查点模块中为 NPU 添加了 delta sync 功能。
* **数据并行通信优化** (#8708)：优化 N-D 数据并行分发逻辑，可选将数据分片仅发送给模型并行的源 rank，减少冗余通信。
* **Profiler 平台钩子** (#8711)：为 torch profiler 添加了平台钩子，允许加速器平台自定义设备活动记录和额外参数。
* **Worker Hooks 辅助函数** (#8715)：在 `single_controller` 中添加了 `call_class_method`，方便 worker 方法直接挂载。

**破坏性重构:**
* **移除 DataProto 自动填充** (#8709)：`DP_COMPUTE_PROTO` 不再自动 padding，强制要求 `len(batch) % world_size == 0`，需显式调用 padding。
* **统一 AgentLoop 实现** (#8707)：将 `AgentLoopTQ` 合并至共享的 `AgentLoop` 实现中，并移除了过时的 v0 padding 逻辑。

**测试与 CI：**
* **Ascend CI 测试增强** (#8706)：在 Ascend 工作流中添加了 vLLM top-k 采样的测试用例。

### 🚀 Release 动态
近期无新版发布记录。

---

## 🐛 Issues

### #8717 — [[CI] Unexpected cancellations in ROCm E2E workflow](https://github.com/verl-project/verl/issues/8717)
- **作者**: a1021223029-lgtm  **时间**: 2026-11-17 15:12 CST
- **标签**: bug
- **摘要**: ### System Info  Repository: verl-project/verl Workflow: .github/workflows/e2e_ppo_trainer_megatron_vllm_rocm.yml Runner label: rocm-mi300 Container image: verlai/verl:rocm7.14_torch2.12_release_1112 Job timeout: 60 minutes  This report concerns the upstream GitHub Actions CI environment. I do not h…

### #8716 — [[ROCm][SGLang] FP8_BLOCK rollout weight sync fails with TP scale shape mismatch](https://github.com/verl-project/verl/issues/8716)
- **作者**: a1021223029-lgtm  **时间**: 2026-11-17 14:42 CST
- **标签**: bug
- **摘要**: ### System Info  ----------Python Info---------- Version      : 3.12.3 Compiler     : GCC 13.3.0 Build        : ('main', 'Mar 23 2026 19:04:32') Arch         : ('64bit', 'ELF') ------------Pip Info----------- Version      : 26.1.2 Directory    : /opt/venv/lib/python3.12/site-packages/pip vllm	     :…

### #8712 — [[ROCm] PPO training hangs forever at dataloader worker startup (KMP_INIT_AT_FORK=FALSE + LLVM OpenMP)](https://github.com/verl-project/verl/issues/8712)
- **作者**: PeterYang12  **时间**: 2026-11-17 11:24 CST
- **摘要**: ### System Info  ``` verl    : 0.10.0.dev, main @ 622cd6c1 image   : verlai/verl:rocm7.14_torch2.12_release_1112 (also seen on release_0724) torch   : 2.12.0+rocm7.14.0a20260608 (uses the ROCm SDK's LLVM OpenMP) ray     : 2.60.0 python  : 3.12.3 hardware: 8x AMD MI300, container pinned to 32 CPU cor…

## 🔀 Pull Requests

### #8718 — [[ckpt] feat: add delta sync in npu](https://github.com/verl-project/verl/pull/8718)
- **作者**: wucong25  **时间**: 2026-11-17 15:48 CST
- **摘要**: ### What does this PR do?  add delta sync in npu  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni…

### #8715 — [[single_controller] feat: add call_class_method helper for worker hooks](https://github.com/verl-project/verl/pull/8715)
- **作者**: vermouth1992  **时间**: 2026-11-17 13:54 CST
- **摘要**: ### What does this PR do?  Follows up on #8708 by adding a `call_class_method(method_name, worker, *args, **kwargs)` helper in `verl.single_controller.base.decorator` so worker methods can be wired directly into `@register(worker_preprocess_fn=..., worker_postprocess_fn=...)` via `functools.partial`…

### #8714 — [`[algo, fsdp, rollout, vllm, sglang, cfg, doc] feat: OPD reverse KL EOS mismatch for student vs teacher fix](https://github.com/verl-project/verl/pull/8714)
- **作者**: zzl-7  **时间**: 2026-11-17 12:47 CST
- **摘要**: ### What does this PR do?  Adds an opt-in distillation_loss.semantic_eos_token_ids option that fixes a length blow-up in on-policy distillation when the student and teacher end a turn with different tokens.  A base student (e.g. Qwen2.5-0.5B) ends with <|endoftext|> (151643), while a post-trained te…

### #8713 — [[ci, hardware] fix: set KMP_INIT_AT_FORK=TRUE on ROCm to stop dataloader fork deadlock](https://github.com/verl-project/verl/pull/8713)
- **作者**: PeterYang12  **时间**: 2026-11-17 11:36 CST
- **摘要**: ### What does this PR do?  Sets `KMP_INIT_AT_FORK=TRUE` for the ROCm e2e workflow and in `docker/rocm/Dockerfile.rocm`, to stop PPO training steps from hanging forever when the dataloader workers start. Details and stacks are in #8712.  Short version of the cause: scikit-learn and joblib (imported v…

### #8711 — [[perf, hardware] feat: platform hooks for the torch profiler's device activity and extra kwargs](https://github.com/verl-project/verl/pull/8711)
- **作者**: wenjung2007  **时间**: 2026-11-17 08:08 CST
- **摘要**: ### What does this PR do?  Lets an accelerator platform tell verl's torch profiler (`global_profiler.tool=torch`) which device activity to record and which extra `torch.profiler.profile` arguments it needs, instead of each hardware plugin patching `get_torch_profiler`.  Today `get_torch_profiler` on…

### #8710 — [[ckpt] fix: save generation configs that transformers refuses, and log when none is saved](https://github.com/verl-project/verl/pull/8710)
- **作者**: savaresejeremy  **时间**: 2026-11-17 05:38 CST
- **摘要**: ### What does this PR do?  The FSDP and Megatron checkpoint managers copy the model's generation config into the checkpoint's Hugging Face directory inside a bare `except Exception: pass` ([fsdp_checkpoint_manager.py](https://github.com/verl-project/verl/blob/909bb91387b96fd20dddc86cb9c026b536f0f05e…

### #8709 — [[BREAKING][single_controller, data, doc] refactor: remove DataProto auto padding](https://github.com/verl-project/verl/pull/8709)
- **作者**: vermouth1992  **时间**: 2026-11-16 19:47 CST
- **摘要**: ### What does this PR do?  Remove DataProto auto padding. `DP_COMPUTE_PROTO` now splits with `_split_args_kwargs_data_proto` and requires `len(batch) % world_size == 0`. Explicit `DataProto.padding`, `pad_dataproto_to_divisor`, and `unpad_dataproto` stay.  Auto padding duplicated samples so a batch …

### #8708 — [[BREAKING][single_controller, worker, trainer] feat: dispatch shards only to the model-parallel source rank](https://github.com/verl-project/verl/pull/8708)
- **作者**: vermouth1992  **时间**: 2026-11-16 19:13 CST
- **摘要**: ### What does this PR do?  N-D data-parallel dispatch currently copies each shard to every rank in the model-parallel group. This adds an optional path that sends each data-parallel shard to exactly one model-parallel source rank and leaves redistribution to a worker-side preprocess hook. The worker…

### #8707 — [[BREAKING][rollout, trainer] refactor: unify AgentLoop implementations](https://github.com/verl-project/verl/pull/8707)
- **作者**: Begunner  **时间**: 2026-11-16 17:59 CST
- **摘要**: ### What does this PR do?  Merge AgentLoopTQ into the shared AgentLoop implementation following #8625, and remove obsolete v0 padding logic.  ### Test  - CPU regression: 332 passed, 1 skipped. One unrelated Megatron test was excluded after reproducing its failure on main. - Single-A100 Qwen3-0.6B sm…

### #8706 — [[ci, vllm] test: add top-k sampling support checks to Ascend CI](https://github.com/verl-project/verl/pull/8706)
- **作者**: lxb007981  **时间**: 2026-11-16 17:07 CST
- **摘要**: ### What does this PR do?  Add the vLLM sampling test suite to the Ascend workflow.  Assert that generated completions are nonempty and contain one logprob record per token.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR titl…
