# verl-project/verl — 动态追踪

> 生成时间: 2026-09-28 13:45 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
共 2 个问题报告：
*   **多语言分词错误 (#8030)**：由于项目锁定了 `transformers==5.12.1`，导致 Qwen3.5 模型在处理泰语和印地语文本时出现分词错误。
*   **Megatron LM-head 挂起故障 (#8028)**：在启用 overlap 的融合 Megatron LM-head 场景下，参数 AllGather 操作会处于挂起状态，并在 H20 GPU 上运行 Qwen3.5-35B-A3B 时引发 GPU 故障。

### 🔧 Pull Request 动态
共 7 个 PR，包含 1 项新特性与多项关键修复：
*   **新特性：LoRA 内存优化 (#8032)**
    *   新增可选标志 `model.lora.resync_base`，在 rollout 休眠时释放 LoRA 基础权重，有效避免 offload（卸载）时主内存发生 OOM（内存溢出）。
*   **多模态模型修复**：
    *   **Qwen2-VL 系列训练失败修复 (#8035)**：修复新版 `transformers` 中 `model.visual()` 返回结构化输出导致的纯文本/混合模态训练失败问题。
    *   **多轮智能体多模态合并修复 (#8036)**：将已解析的媒体数据传递给 Continuous Token，修复多轮循环中的上下文合并问题。
*   **Megatron 相关修复**：
    *   **GPU 故障修复 (#8029)**：修复融合内核在读取存储前未等待实际 LM-head 参数的问题，解决上述 Issue #8028 中的 AllGather 挂起故障。
    *   **VLM 动态批处理修复 (#8034)**：限制 Megatron 动态微批中密集输入 token 的占用，解决 VLM 在打包前展开密集嵌入导致的内存问题。
    *   **MTP Loss 异常更新修复 (#8031)**：阻止 MTP（多 Token 预测）损失在特定 megatron-core 版本中意外更新 LM head。
*   **硬件适配修复**：
    *   **NPU 性能衰退规避方案 (#8033)**：针对上游 transformers 代码改动引发的 NPU 性能回退，为 Qwen3-Next 添加了可选的状态分配临时规避方案。

### 🚀 Release 动态
*   近期无新版本发布。

---

## 🐛 Issues

### #8030 — [[tokenizer] Qwen3.5+ Thai/Hindi text mis-tokenized under pinned transformers==5.12.1 (transformers#49066)](https://github.com/verl-project/verl/issues/8030)
- **作者**: dengoswei  **时间**: 2026-09-28 06:29 CST
- **摘要**: ### System Info  - verl `main` @ `6093e007cc` — `pyproject.toml` pins `transformers==5.12.1` (and `sglang==0.5.20`, which itself hard-pins `transformers==5.12.1`), from #7973. - Repro below run on CPU with `transformers==5.8.1` / `tokenizers` from the same env; the upstream report (huggingface/trans…

### #8028 — [[Bug] Fused Megatron LM-head leaves parameter AllGather pending with overlap enabled](https://github.com/verl-project/verl/issues/8028)
- **作者**: ewan0x79  **时间**: 2026-09-27 22:09 CST
- **摘要**: ## System Info  | Evidence | Environment | | --- | --- | | Observed GPU failure | A downstream VERL SFT integration using vendored Megatron Core 0.16.2; Qwen3.5-35B-A3B; BF16; 2 × 8 NVIDIA H20 GPUs; TP1 / PP1 / CP4 / EP8; packed inputs | | Upstream code examined | VERL `6093e007cc341973c9d9a6fb3867a…

## 🔀 Pull Requests

### #8036 — [[rollout] fix: pass resolved media to Continuous Token merges](https://github.com/verl-project/verl/pull/8036)
- **作者**: zackcxb  **时间**: 2026-09-28 12:16 CST
- **摘要**: ### What does this PR do?  Fixes Continuous Token multimodal context merges for multi-turn agent loops. The gateway/dataset layer already resolves image/video/audio URLs into processor-ready media, but incremental CT rendering previously re-extracted media from URL-bearing messages. This patch threa…

### #8035 — [[model] fix: unpack visual output in Qwen2-VL and GLM4V dummy forward paths](https://github.com/verl-project/verl/pull/8035)
- **作者**: chenyingshu  **时间**: 2026-09-28 11:47 CST
- **摘要**: ### What does this PR do?  - Fix text-only (and mixed-modality) training failures for **Qwen2-VL / Qwen2.5-VL ** when using recent `transformers`, where `model.visual()` returns a structured output (e.g. `BaseModelOutputWithPooling`) instead of a raw tensor. - Route **dummy** and real visual forward…

### #8034 — [[megatron, training_utils] fix: account for padded tokens in dynamic batching for VLM](https://github.com/verl-project/verl/pull/8034)
- **作者**: qiangyupei  **时间**: 2026-09-28 10:36 CST
- **摘要**: ## Summary  Bound the dense input-token footprint of Megatron dynamic micro-batches, in addition to the existing effective-token budget. VLMs can materialize dense embeddings before packing, so `use_remove_padding=True` does not eliminate this peak.  For example, two sequences of 50,278 and 78,701 t…

### #8033 — [[model, hardware] fix: add opt-in Qwen3-Next NPU state allocation workaround](https://github.com/verl-project/verl/pull/8033)
- **作者**: lxb007981  **时间**: 2026-09-28 09:56 CST
- **摘要**: ### What does this PR do?  Transformers PR #45665 removed a blocking CPU-to-device state copy. Removing that implicit synchronization exposed an NPU performance regression, with allocator retries and increased actor time.  Restore the wait before initial recurrent state allocation in the native gate…

### #8032 — [[rollout] feat: release LoRA base weights on sleep via lora.resync_base](https://github.com/verl-project/verl/pull/8032)
- **作者**: HollowMan6  **时间**: 2026-09-28 06:52 CST
- **摘要**: ### What does this PR do?  To avoid main memory OOM when offload, add an opt-in `model.lora.resync_base` flag for LoRA adapter mode (`model.lora.merge=False`). When it is set, the rollout releases the base weights on sleep, and the trainer re-syncs the full base weights before the adapter on every w…

### #8031 — [[megatron] fix: stop MTP loss from updating the LM head](https://github.com/verl-project/verl/pull/8031)
- **作者**: dengoswei  **时间**: 2026-09-28 06:32 CST
- **摘要**: ### What does this PR do?  With MTP training enabled on megatron-core versions that provide `process_mtp_loss` (including the pinned `core_v0.18.0`), the MTP loss silently updates the LM head, and also the input embedding when `share_embeddings_and_output_weights=True`. This PR detaches the output-l…

### #8029 — [[megatron] fix: publish output weights before fused kernels](https://github.com/verl-project/verl/pull/8029)
- **作者**: ewan0x79  **时间**: 2026-09-27 22:18 CST
- **摘要**: ### What does this PR do?  Fixes #8028  Wait for the actual LM-head parameter before fused output kernels read its storage. Fused consumers bypass `output_layer.forward`, so they also bypass the DDP pre-hook that normally completes parameter AllGather. In our local VERL training stack this left an o…
