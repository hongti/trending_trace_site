# verl-project/verl — 动态追踪

> 生成时间: 2026-09-23 13:03 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📝 Issue (问题讨论)
本期共记录 3 个 Issue，主要涉及新功能设计、性能分析需求及高并发下的稳定性问题：
* **架构设计 #7998**：提出了兼容单智能体的在线蒸馏（OPD）设计方案，探讨如何通过双路径计算 teacher logprobs。
* **功能请求 #7995**：希望在 NPU 性能分析阶段，支持基于 micro batch 维度采集数据以更新 actor，而非仅限于 train batch size。
* **Bug 反馈 #7990**：指出 `colocate_async` 在高并发请求负载下进行权重同步时可能发生死锁。

### 🔀 Pull Request (代码合并)
本期共有 9 个 PR，涵盖多项重要修复与新特性，主要集中在 Rollout 调度、Checkpoint 存储及模型兼容性方面：
* **Rollout 与 vLLM 优化**：
  * **修复死锁 #7999**：针对 Issue #7990，为 rollout server 新增 `ray_actor_max_concurrency` 配置，解决高并发下的死锁问题。
  * **请求与缓存管理 #7994**：新增 `abort_requests()` 功能，允许在不暂停服务或清空缓存的情况下取消 vLLM 进行中的请求；同时暴露 KV-cache 快照以供读取。
  * **资源池管理 #7993**：`RolloutReplica.init_standalone` 现支持接入外部已有的 `RayResourcePool`。
  * **权重同步修复 #7987**：修复标准 vLLM 权重同步问题，使其在 vLLM 自身的逐层重载生命周期内运行，移除对 MoE kernel 的依赖。
* **Checkpoint 与存储优化**：
  * **分片大小配置 #7997**：支持通过 `hf_save_pretrained_kwargs` 配置 Hugging Face 模型导出的分片大小（涉及 FSDP 和 VeOmni）。
  * **Megatron 写入优化 #7992**：设置本地目录时，Megatron HF safetensors 的写入将先暂存至本地，提升写入稳定性。
  * **清理序列化修复 #7989**：修复分布式 Megatron checkpoint 在共享文件系统上执行保留策略（retention cleanup）时可能失败的问题，增加了清理操作的序列化。
* **模型与数据处理修复**：
  * **Qwen VL 媒体处理 #7996**：修复 Qwen VL 媒体数据被重复 resize 的问题。
  * **Logit 软限幅修复 #7991**：修复模型（如 Gemma4）启用 `final_logit_softcapping` 时，通用融合前向传播报错的问题，现会拒绝不兼容的配置。
* **性能分析 #7988**：为硬件平台添加了可选的 `torch.profiler` 活动钩子。

### 🚀 Release (版本发布)
* 本期动态中未包含版本发布信息。

---

## 🐛 Issues

### #7998 — [[RFC] On-policy distillation (OPD) design compatible with uni-agent](https://github.com/verl-project/verl/issues/7998)
- **作者**: wangtiance  **时间**: 2026-09-23 09:51 CST
- **摘要**: # On-Policy Distillation: Dual Teacher-Logprob Computation Paths  ## Overview  OPD requires per-token teacher logprobs for every trained trajectory. Because trajectories can originate from two different generation paths (verl's native `AgentLoopWorker`, or an external agent framework such as [uni-ag…

### #7995 — [[feature request] Support implementing update actor to collect data based on micro batch on npu.profile](https://github.com/verl-project/verl/issues/7995)
- **作者**: mikequan0425  **时间**: 2026-09-22 21:13 CST
- **摘要**: ### Feature request  Currently, when using npu.profile for profiling during the update actor phase, collection can only be performed at the train batch size dimension. Previously, @mengchengTang has implemented mini batch-based collection in https://github.com/verl-project/verl/pull/7105, and it is …

### #7990 — [[Bug] colocate_async can deadlock during weight synchronization under high outstanding-request load](https://github.com/verl-project/verl/issues/7990)
- **作者**: janbernloehr  **时间**: 2026-09-22 17:37 CST
- **摘要**: ### System Info  Environment captured from our containerized CI runs (not a fresh `scripts/diagnose.py` report):  - verl `0.10.0.dev0+git68c9ac3` for the first successful patch validation; `0.10.0.dev0+git10db40d` for the repeat validation. - vLLM `0.24.0`, Ray `2.57.0`, PyTorch `2.11.0+cu130`. - NV…

## 🔀 Pull Requests

### #7999 — [[rollout] feat: add ray_actor_max_concurrency for server replica](https://github.com/verl-project/verl/pull/7999)
- **作者**: wuxibin89  **时间**: 2026-09-23 11:27 CST
- **摘要**: ### What does this PR do?  Fix https://github.com/verl-project/verl/issues/7990, add `ray_actor_max_concurrency` for  rollout server actor. Raise it above the peak number of in-flight generate requests per replica, otherwise control calls (wake_up, weight sync) can be starved.

### #7997 — [[fsdp, veomni, ckpt] feat: configure Hugging Face export shard size](https://github.com/verl-project/verl/pull/7997)
- **作者**: lbaolin  **时间**: 2026-09-23 08:19 CST
- **摘要**: ### What does this PR do?  Expose `hf_save_pretrained_kwargs` for Hugging Face model exports written by `FSDPCheckpointManager`. FSDP and VeOmni actor configs plus SFT predeclare `max_shard_size: null`, so the common shard-size override works without `+`; the null value is omitted and preserves the …

### #7996 — [[data, rollout] fix: avoid duplicate Qwen VL media resize](https://github.com/verl-project/verl/pull/7996)
- **作者**: Tohrusky  **时间**: 2026-09-22 21:59 CST
- **摘要**: ### What does this PR do?  Qwen VL media is already decoded, sampled, and smart-resized by `qwen_vl_utils` before it reaches the Hugging Face processor or vLLM. This change defaults `mm_processor_kwargs.do_resize` to `False` for Qwen2-VL, Qwen2.5-VL, and Qwen3-VL processors, including Qwen3.5 checkp…

### #7994 — [[rollout, vllm] feat: abort in-flight requests without pausing, and expose KV snapshot](https://github.com/verl-project/verl/pull/7994)
- **作者**: tongyx361  **时间**: 2026-09-22 19:31 CST
- **摘要**: ### What does this PR do?  Add `abort_requests()` so a controller can cancel in-flight vLLM requests without closing admission or clearing caches, and `snapshot()` so it can read KV-cache usage and scheduler queue depths from the engine Prometheus gauges. `vLLMReplica.abort_all_requests` now forward…

### #7993 — [[rollout] feat: accept an existing resource pool in init_standalone](https://github.com/verl-project/verl/pull/7993)
- **作者**: tongyx361  **时间**: 2026-09-22 19:31 CST
- **摘要**: ### What does this PR do?  Let `RolloutReplica.init_standalone` attach a caller-owned `RayResourcePool`. Omitting the argument still creates a per-replica pool, which is the previous behavior.  This is one independent slice of #7570. That combined PR is being closed in favor of this change plus the …

### #7992 — [[ckpt] feat: stage Megatron HF safetensors writes when a local dir is set](https://github.com/verl-project/verl/pull/7992)
- **作者**: tongyx361  **时间**: 2026-09-22 19:31 CST
- **摘要**: ### What does this PR do?  Megatron HF safetensors saves write each shard directly on the destination filesystem. safetensors >= 0.8 `serialize_file` ([safetensors#764](https://github.com/safetensors/safetensors/pull/764)) preallocates with `File::set_len` and then `rename`/`chmod`s a sibling temp f…

### #7991 — [[model] fix: reject logit softcapping in generic fused forward](https://github.com/verl-project/verl/pull/7991)
- **作者**: hscspring  **时间**: 2026-09-22 18:13 CST
- **摘要**: ### What does this PR do?  Reject the generic fused forward when the model's text configuration enables `final_logit_softcapping`. For example, Gemma4 uses `30 * tanh(logits / 30)`, but `dense_common` bypasses the original LM-head forward and computes log-probabilities and entropy without that trans…

### #7989 — [[ckpt, megatron] fix: serialize shared checkpoint retention cleanup](https://github.com/verl-project/verl/pull/7989)
- **作者**: FZYsheep  **时间**: 2026-09-22 17:36 CST
- **摘要**: ### What does this PR do?  Fix distributed Megatron checkpoint saves that can fail while enforcing retention on a shared filesystem.  In an actual multi-rank run, checkpoint saving failed through `save_checkpoint()` → `ensure_checkpoint_capacity()` → `remove_previous_save_local_path()` → `shutil.rmt…

### #7988 — [[hardware, profile] add optional torch.profiler activity hook to Plat…](https://github.com/verl-project/verl/pull/7988)
- **作者**: kahlun  **时间**: 2026-09-22 17:35 CST
- **摘要**: to do <pytorch profiler, addon on top of itt_profiler / range_push/range_pop >

### #7987 — [[rollout, vllm] fix: run the standard weight sync through vLLM's layerwise reload lifecycle](https://github.com/verl-project/verl/pull/7987)
- **作者**: wengeezhang  **时间**: 2026-09-22 17:24 CST
- **摘要**: ### What does this PR do?  Fixes #7978: the **standard (non-quantized) vLLM weight sync now runs inside vLLM's own layerwise reload lifecycle**, so it no longer depends on the MoE kernel keeping the expert weights in checkpoint layout.  **Root cause (verified against vLLM 0.24.0 and 0.29.0 source).*…
