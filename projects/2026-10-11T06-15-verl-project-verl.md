# verl-project/verl — 动态追踪

> 生成时间: 2026-10-11 14:15 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### Issue (问题)
本期未收录相关 Issue 动态。

### Pull Request (合并请求)
近期共有 9 个 PR，主要涉及算法增强、Bug 修复以及针对 Ascend (NPU) 硬件的深度适配：

**新特性与功能增强**
*   **截断采样下的分数中心化** (#8196)：支持在截断采样（`top_p < 1` 或 `top_k > 0`）场景下进行分数中心化处理。
*   **FSDP 支持 R3 路由重放** (#8194)：将 R3（rollout router replay）机制接入原生 FSDP 引擎，使其在 MoE 路由重放方面与 Megatron 和 veomni 引擎对齐。

**核心 Bug 修复**
*   **FSDP 精度缓冲区修复** (#8189)：修复了 FSDP 引擎在将模型转换为 `model_dtype` 时，误将 FP32 缓冲区一并转换导致损坏的问题。
*   **Megatron 反向传播修复** (#8195)：修复了使用 Megatron 引擎且启用分块词表并行熵时，在反向传播阶段因计算图变量缺失导致的崩溃问题。
*   **分词器一致性修复** (#8193)：修复了 `AutoTokenizer` 重建预分词器导致与 `tokenizer.json` 行为不一致的问题，确保按文件原样加载。

**NPU (Ascend) 硬件适配与修复**
*   **算子分发修复** (#8192)：修复了在 Ascend 环境下，CPU logits 被错误分发给 NPU 专用算子 `npu_cross_entropy_loss` 导致报错的问题。
*   **性能分析器异常处理** (#8191)：修复了 NPUProfiler 在捕获到异常时跳过清理步骤的问题，避免错误保留运行状态。
*   **Qwen3-VL-MoE 检查点加载** (#8190)：修复了应用 NPU 补丁后，无法在 Ascend 上加载 Transformers 5 原生 Qwen3-VL-MoE 检查点的问题。
*   **FSDP2 CPU 卸载修复** (#8187)：修复了在参数分片不均时，将 FSDP2 模型从 NPU 卸载到 CPU 导致参数损坏的问题。

**CI / 基础设施**
*   **升级 Ascend 测试环境** (#8188)：将部分 Ascend FSDP2 作业迁移至 16 卡 A3 800I 运行器，并将共享内存提升至 160 GiB，以支持 Qwen3-VL 等大模型的快速测试。

### Release (版本发布)
本期暂无新版本发布。

---

## 🔀 Pull Requests

### #8196 — [[algo, cfg, doc] feat: score centering over the truncated sampling support](https://github.com/verl-project/verl/pull/8196)
- **作者**: cr-gao  **时间**: 2026-10-11 11:30 CST
- **摘要**: ### What does this PR do?  Score centering (#8010) under truncated sampling (`top_p < 1` / `top_k > 0`). The head stored by #8010 already holds the sampler's whole support when `0 < top_k <= topk_log_probs` (vLLM `processed_logprobs` fills the truncated tokens with `-inf`), so the new `score_centeri…

### #8195 — [[megatron] fix: backward through chunked vocab-parallel entropy](https://github.com/verl-project/verl/pull/8195)
- **作者**: aoshen02  **时间**: 2026-10-11 10:41 CST
- **摘要**: ### What does this PR do?  With the Megatron engine, `entropy_from_logits_with_chunking=True` fails in backward whenever the entropy is part of the graph:  ``` RuntimeError: one of the variables needed for gradient computation has been modified by an inplace operation: [torch.cuda.FloatTensor [1, 20…

### #8194 — [[fsdp] feat: support R3 router replay](https://github.com/verl-project/verl/pull/8194)
- **作者**: aoshen02  **时间**: 2026-10-11 10:38 CST
- **摘要**: ### What does this PR do?  Wires R3 (rollout router replay) into the plain FSDP engine. The megatron and veomni engines already replay the rollout's MoE routing; the FSDP engine dropped `routed_experts`.  With `actor.fsdp_config.router_replay.mode=R3` and `rollout.enable_rollout_routing_replay=True`…

### #8193 — [[utils] fix: honor tokenizer.json when AutoTokenizer rebuilds the pre-tokenizer](https://github.com/verl-project/verl/pull/8193)
- **作者**: aoshen02  **时间**: 2026-10-11 09:25 CST
- **摘要**: ### What does this PR do?  `hf_tokenizer` now checks that the tokenizer returned by `AutoTokenizer` encodes like the `tokenizer.json` it was loaded from and, on mismatch, loads the file as stored (`PreTrainedTokenizerFast`).  transformers 5 rebuilds some tokenizer classes (e.g. `LlamaTokenizer`) fro…

### #8192 — [[training_utils, hardware] fix: dispatch log-prob kernels by tensor device](https://github.com/verl-project/verl/pull/8192)
- **作者**: liuhao-labs  **时间**: 2026-10-11 01:54 CST
- **摘要**: ### What does this PR do?  On an Ascend environment with `torch_npu.npu_cross_entropy_loss` installed, `logprobs_from_logits()` dispatches CPU logits to that NPU-only operator and raises `NotImplementedError`. Installing Flash-Attention causes the equivalent problem because its availability takes pr…

### #8191 — [[perf, hardware] fix: flush NPU profiler scopes on exceptions](https://github.com/verl-project/verl/pull/8191)
- **作者**: liuhao-labs  **时间**: 2026-10-11 01:35 CST
- **摘要**: ### What does this PR do?  When an Ascend training stage raises inside `NPUProfiler.annotate()`, the wrapper skips `mark_end_range()`, `step()` and `stop()`. The failure's traceback can retain the running profiler: its framework operation trace has not been flushed when the exception reaches the cal…

### #8190 — [[model, hardware] fix: preserve Transformers 5 Qwen3-VL-MoE checkpoint layout on NPU](https://github.com/verl-project/verl/pull/8190)
- **作者**: liuhao-labs  **时间**: 2026-10-11 01:13 CST
- **摘要**: ### What does this PR do?  Loading a native Transformers 5 Qwen3-VL-MoE checkpoint on Ascend fails after verl applies its NPU patches. The patch replaces the entire sparse-MoE class with the Transformers 4 implementation, reversing the last two dimensions of the expert parameters. A small real HF ch…

### #8189 — [[fsdp] fix: preserve FP32 buffers when casting the model to model_dtype](https://github.com/verl-project/verl/pull/8189)
- **作者**: li-lizhe  **时间**: 2026-10-10 23:12 CST
- **摘要**: ## What does this PR do?  Fixes #8154.  `FSDPEngine._build_module()` ends with a blanket `module.to(torch_dtype)`. `Tensor.to(dtype)` converts registered buffers as well as parameters, so floating-point buffers that the model implementation builds in FP32 are rounded to the model dtype. `RotaryEmbed…

### #8188 — [[ci] chore: Move selected Ascend FSDP2 jobs to 16-NPU runners](https://github.com/verl-project/verl/pull/8188)
- **作者**: lxb007981  **时间**: 2026-10-10 18:17 CST
- **摘要**: ### What does this PR do?  Use 16-device A3 800I runners and 160 GiB shared memory for the Qwen3-VL, Qwen3-0.6B quick-start, and Qwen3-8B GSPO jobs.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}:…

### #8187 — [[fsdp] fix: prevent NPU parameter corruption during FSDP2 CPU offload](https://github.com/verl-project/verl/pull/8187)
- **作者**: SelfDLer  **时间**: 2026-10-10 18:05 CST
- **摘要**: ### What does this PR do?  Prevent parameter corruption when offloading an FSDP2 model from NPU to CPU with uneven parameter shards.  With `model.to("cpu", non_blocking=True)`, the NPU-to-CPU transfer can still be in progress when FSDP2 repads the local shard. This repadding happens inside `FSDPModu…
