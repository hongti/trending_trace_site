# verl-project/verl — 动态追踪

> 生成时间: 2026-11-25 17:00 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📋 Issue (议题)
近期 Issue 主要集中在架构优化提案（RFC）、Bug 反馈及使用咨询，重点围绕 NPU/Ascend 支持、权重量化及 FSDP 训练稳定性：
* **架构与功能提案 (RFC)**：
  * **#8845**：提议为现有的 `WeightQuantizer` 增加显式的、独立于后端的权重源和精度策略（Phase 1）。
  * **#8843**：提议将 verl 中针对 NPU vLLM 的特定补丁下沉合并到 `vllm-ascend` 项目中，以保持 verl 核心代码的整洁。
  * **#8836**：针对混合模型（如 Qwen3.5, Qwen3-Next），建议在 SGLang rollout 中使用 session radix cache 来保持共享 prompt 状态。
* **Bug 反馈**：
  * **#8839**：反馈 FSDP 模式下，当 `torch.chunk` 返回的块数少于 `num_shards` 时，`TiledMLP.backward` 会丢失前几个 micro-batch 累积的参数梯度。
* **使用咨询**：
  * **#8842**：询问 VeOmni 后端是否支持跨 microbatch 的前缀复用。

### 🔨 Pull Request (拉取请求)
近期 PR 涵盖了重要的新特性引入、关键 Bug 修复以及 CI/文档维护，重要变更如下：
* **重要新特性**：
  * **引入 DPO 训练器**：**#8837** 和 **#8838** 共同为 verl 引入了离线 Direct Preference Optimization (DPO) 支持。新增了包含 7 种偏好损失函数的注册表，以及一个类似 `sft_trainer` 的单进程/GPU DPO 训练器。
  * **Megatron 权重流式传输**（**#8833**）：在开启 Megatron 参数 offload 时，rollout 同步阶段改为流式传输权重，避免一次性将所有权重加载到 GPU，大幅降低显存峰值。
  * **配置加载验证**（**#8834**）：新增 `skip.rollout_tq.load_mode=config` 策略，在写入 TransferQueue 前验证缓存的会话。
* **关键 Bug 修复**：
  * **#8840**：修复了 Issue #8839 中提到的 `TiledMLP` 丢失累积梯度的问题，改用实际 chunk 数进行最后分片的梯度回写。
  * **Ascend/NPU 相关修复**：**#8841** 修复了 Ascend 采样测试中因 NPU 在 fork 子进程中重复初始化导致的 CI 失败（改用 spawn 方式）；**#8832** 和 **#8835** 修复了 VeOmni checkpoint API 迁移带来的测试回归及版本对齐问题。
* **CI 与文档**：
  * **#8844**：在 Ascend 环境下，使用完整预训练的 Qwen3-30B 模型对 R3 模式进行对比测试。
  * **#8831**：修复并更新了 9 个 Ascend 相关的中文教程与开发指南文档。

### 🚀 Release (发布版本)
* 近期无新的 Release 版本发布。

---

## 🐛 Issues

### #8845 — [[RFC] Explicit weight-source and precision policies for existing WeightQuantizer (Phase 1)](https://github.com/verl-project/verl/issues/8845)
- **作者**: ISEEKYAN  **时间**: 2026-11-25 16:19 CST
- **摘要**: ## Feature request  Add explicit, backend-independent weight-source and precision policies to verl's existing per-tensor export -> WeightQuantizer -> checkpoint-engine pipeline.  **Phase 1 is about numerical semantics and a practical high-precision export path, not a generalized sharded transfer IR.…

### #8843 — [[RFC] Sinking NPU vLLM patches from verl into vllm-ascend](https://github.com/verl-project/verl/issues/8843)
- **作者**: floatlibai  **时间**: 2026-11-25 15:15 CST
- **摘要**: ## Summary  verl currently carries a set of NPU-specific monkey patches for vLLM (`npu_vllm_patch`). Most of them work around gaps in vllm-ascend or vLLM rather than verl-specific behavior. This RFC proposes moving the patches that belong downstream (vllm-ascend / vLLM) out of verl, and keeping only…

### #8842 — [[Question] Does the VeOmni backend support cross-microbatch prefix reuse?](https://github.com/verl-project/verl/issues/8842)
- **作者**: yue-zeng-yue  **时间**: 2026-11-25 11:27 CST
- **摘要**: I'd like to clarify one detail about shared-prefix training in the VeOmni backend. My understanding is that prefix reuse is currently limited to samples within the same microbatch and does not extend across different microbatches. Is that correct?

### #8839 — [[fsdp] TiledMLP drops accumulated gradients when torch.chunk returns fewer chunks than num_shards](https://github.com/verl-project/verl/issues/8839)
- **作者**: elipskiy  **时间**: 2026-11-25 06:28 CST
- **摘要**: ### What happens  `TiledMLP.backward` (`verl/models/transformers/tiled_mlp.py`) drops the parameter gradients that earlier micro-batches accumulated in `param.grad` whenever `torch.chunk` returns fewer chunks than `num_shards`.  `GradientAccumulator.__init__` moves any existing `param.grad` into its…

### #8836 — [[rollout, sglang] Hybrid models: hold shared prompt states with SGLang's session radix cache](https://github.com/verl-project/verl/issues/8836)
- **作者**: tianyi-zhang-02  **时间**: 2026-11-25 01:42 CST
- **摘要**: ### Feature request  For hybrid models (Qwen3.5, Qwen3-Next) on the SGLang rollout, hold each prompt's prefix state with an SGLang session (`--enable-session-radix-cache`, sglang >= 0.5.17, sgl-project/sglang#29173) until all samples of that prompt are done, then close the session.  ### Motivation  …

## 🔀 Pull Requests

### #8844 — [[ci] test: compare R3 with full pretrained Qwen3-30B on Ascend](https://github.com/verl-project/verl/pull/8844)
- **作者**: Rain6ow  **时间**: 2026-11-25 15:56 CST
- **摘要**: The isolated comparison in #8799 completed with a randomly initialized two-layer model. Both modes reported zero rewards, policy loss and gradient norm, so that run does not validate representative pretrained-model training.  Add a separate, targeted workflow that directly loads /root/.cache/modelsc…

### #8841 — [[ci, vllm] fix: use spawn for Ascend sampling tests](https://github.com/verl-project/verl/pull/8841)
- **作者**: fangweii1  **时间**: 2026-11-25 09:55 CST
- **摘要**: ### What does this PR do?  Fix the Ascend sampling/top-k CI step failing with `Cannot re-initialize NPU in forked subprocess`. Importing `extract_prompt_logprobs` during pytest collection can initialize NPU in the parent through model/FLA imports; the default `fork` then prevents EngineCore from ini…

### #8840 — [[fsdp] fix: use actual chunk count for TiledMLP last-shard gradient write-back](https://github.com/verl-project/verl/pull/8840)
- **作者**: four1er  **时间**: 2026-11-25 08:20 CST
- **摘要**: Fixes #8839  `TiledMLP.backward` dropped parameter gradients accumulated by earlier micro-batches whenever `torch.chunk` returned fewer chunks than `num_shards` (e.g. `n=9, shards=4` gives 3 chunks of 3). `GradientAccumulator` moves any existing `param.grad` into its buffer and writes it back only f…

### #8838 — [[3/n][trainer] feat: DPO trainer](https://github.com/verl-project/verl/pull/8838)
- **作者**: Indra24710  **时间**: 2026-11-25 05:46 CST
- **摘要**: ### What does this PR do?  This PR adds an offline Direct Preference Optimization (DPO) trainer, `verl/trainer/dpo_trainer.py`. It runs like `sft_trainer.py`: one process per GPU, using `TrainingWorker` with the FSDP engine. It supports LoRA, full fine-tuning and training without a reference model. …

### #8837 — [[2/n][algo, trainer] feat: DPO loss functions](https://github.com/verl-project/verl/pull/8837)
- **作者**: Indra24710  **时间**: 2026-11-25 05:46 CST
- **摘要**: ### What does this PR do?  This PR adds offline DPO loss functions to verl. It adds a registry of seven preference losses and `dpo_loss`, a loss function that `TrainingWorker.set_loss_fn` can use. Nothing calls them yet: the DPO trainer that uses them is a follow-up PR in this series. Existing losse…

### #8835 — [[veomni, ci] fix: align checkpoint API usage and score centering tests](https://github.com/verl-project/verl/pull/8835)
- **作者**: ji-huazhong  **时间**: 2026-11-24 23:42 CST
- **摘要**: ### What does this PR do?  After #8817 migrated the VeOmni engine to `veomni.models.checkpoint`, Ascend CI workflows continued installing the older VeOmni revision `10a26aca`. That revision does not provide the new checkpoint package, so importing the engine fails and training initialization subsequ…

### #8834 — [[rollout, cfg] feat: validate cached sessions in config load mode](https://github.com/verl-project/verl/pull/8834)
- **作者**: LingYi-Liang  **时间**: 2026-11-24 23:16 CST
- **摘要**: ### What does this PR do?  Add the opt-in `skip.rollout_tq.load_mode=config` policy proposed in #8777. Before writing to TransferQueue, check that the dump has one group per submitted prompt and the configured rollout sessions in every group. On a mismatch, log a warning and submit the original batc…

### #8833 — [[megatron] feat: stream offloaded weights during rollout synchronization](https://github.com/verl-project/verl/pull/8833)
- **作者**: ji-huazhong  **时间**: 2026-11-24 22:48 CST
- **摘要**: ### What does this PR do?  When Megatron parameter offload is enabled, `get_per_tensor_param()` currently reloads all local actor weights onto the GPU before exporting them to the rollout engine. In colocated training, the actor's local model weights, rollout weights, and export buffers therefore oc…

### #8832 — [[veomni, ci, ckpt] fix: complete checkpoint API migration](https://github.com/verl-project/verl/pull/8832)
- **作者**: fangweii1  **时间**: 2026-11-24 20:20 CST
- **摘要**: ### What does this PR do?  Fix the CPU test collection regression introduced by #8817. The engine now imports `veomni.models.checkpoint`, but the packed-padding and router-replay CPU tests still stub the removed `checkpoint_tensor_loading` module. Collection fails with `ModuleNotFoundError: No modul…

### #8831 — [[doc] fix：modify document by aidd](https://github.com/verl-project/verl/pull/8831)
- **作者**: zery00568  **时间**: 2026-11-24 19:31 CST
- **摘要**: ## AIDD 文档修复  本次提交包含 **9** 个文件的修改：  - `docs/ascend_tutorial/zh/README.md` - `docs/ascend_tutorial/zh/dev_guide/model_dev/transfer_to_npu_guide.md` - `docs/ascend_tutorial/zh/dev_guide/model_dev/evaluation.md` - `docs/ascend_tutorial/zh/dev_guide/model_dev/parameter_and_metrics.md` - `docs/ascend_tut…
