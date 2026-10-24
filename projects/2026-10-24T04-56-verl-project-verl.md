# verl-project/verl — 动态追踪

> 生成时间: 2026-10-24 12:56 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要。近期该仓库主要聚焦于功能扩展、算法修复以及性能与稳定性优化。

### 📦 Release
近期无新的 Release 发布。

### 🐛 Issue
近期无公开的 Issue 动态。

### 🚀 Pull Request (PR)

#### 1. 新特性
*   **MiniMax M3 模型首日支持 (#8353)**：新增 verl 原生的 MiniMax M3 文本模型，支持 FSDP2 + LoRA 强化学习，并在 FSDP 引擎上实现了 rollout 路由重放（R3）以及 NVFP4/MXFP8 激活量化模拟（QAT）和专家 LoRA。
*   **文件系统检查点引擎 (#8355)**：新增 `FileSystemCheckpointEngine`，允许通过共享文件系统在训练器和 rollout worker 之间同步 HuggingFace safetensors 格式的权重。

#### 2. 算法与逻辑修复
*   **GRPO 优势计算修复 (#8349)**：当 GRPO 组内所有响应获得相同奖励时，明确将所有 advantage 置为 0，以正确反映无偏好状态。
*   **异步 LLM 客户端预算缩减 (#8356)**：修复 `FullyAsyncLLMServerClient` 在恢复（resume）时未能根据已消耗预算正确缩减 `max_tokens`/`min_tokens` 的问题。
*   **Bypass 模式初始化时机 (#8348)**：将 bypass 模式的配置时机提前至 worker 创建之前，确保 actor 训练时能正确使用 rollout 的 log_probs。
*   **LoRA 检查点加载修复 (#8350)**：修复了当 LoRA-only 检查点缺少适配器键时，`FSDPCheckpointManager` 加载失败的问题。
*   **RL-Insight 日志元数据修复 (#8357)**：修复 `RLInsightLogger` 丢弃 `step` 元数据，并将标量名中 `/` 误替换为 `_` 的问题，保证推送导出器能正常工作。

#### 3. 环境与兼容性修复
*   **Megatron Docker 环境修复 (#8354)**：修复了 uv 训练环境中未安装 DeepEP 的问题，并恢复了 CUDA 13 镜像中缺失的 NVSHMEM 系统安装。

#### 4. 性能优化与重构
*   **vLLM LoRA IPC 传输重构 (#8352)**：将 LoRA buckets 统一暂存于单个 host block 中以提升同步性能，无行为变更。
*   **Logprobs 内存计算优化 (#8351)**：在未使用 flash-attn 交叉熵内核的回退场景下，对二维 bf16/fp16 logits 进行分块处理，优化 `logprobs_from_logits_v2` 的内存消耗和性能。

---

## 🔀 Pull Requests

### #8357 — [[training_utils] fix: preserve RL-Insight gauge scalar name and step metadata](https://github.com/verl-project/verl/pull/8357)
- **作者**: ETOgaosion  **时间**: 2026-10-24 09:47 CST
- **摘要**: ### Problem and change  `RLInsightLogger.log(data, step)` currently discards `step` and replaces `/` with `_` in every scalar name before calling `metric_gauge`. A push exporter consequently cannot recover the caller's exact metric name or associate its value with the supplied training step.  Keep t…

### #8356 — [[rollout, fully_async] fix: shrink min_tokens with the budget on FullyAsyncLLMServerClient resumes](https://github.com/verl-project/verl/pull/8356)
- **作者**: linyueqian  **时间**: 2026-10-24 01:41 CST
- **摘要**: ### What does this PR do?  `FullyAsyncLLMServerClient.generate` shrinks the length budget on every resume: after an abort it sets `max_tokens` (or `max_new_tokens`) to the original value minus the tokens already generated. A caller-set lower bound, `min_tokens` (or `min_new_tokens`), is copied as is…

### #8355 — [[ckpt, fsdp] feat: add filesystem checkpoint engine](https://github.com/verl-project/verl/pull/8355)
- **作者**: wdl339  **时间**: 2026-10-23 18:41 CST
- **摘要**: ### What does this PR do?  Add a `FileSystemCheckpointEngine` backend that synchronizes trainer weights with rollout workers through a shared filesystem.  The trainer exports HuggingFace safetensors shards to a shared mounted directory, and rollout workers load them directly. This provides a simple …

### #8354 — [[megatron, docker] fix: install DeepEP in uv training environments and restore NVSHMEM](https://github.com/verl-project/verl/pull/8354)
- **作者**: ETOgaosion  **时间**: 2026-10-23 18:34 CST
- **摘要**: Megatron MoE training can enable the DeepEP dispatcher while the uv `megatron` extra leaves `deep_ep` uninstalled. The CUDA 13 image also lacked a system NVSHMEM installation at the path embedded in the prebuilt DeepEP wheel.  Install `deep-ep==1.2.1` with the Megatron extra from the matching Torch …

### #8353 — [[model, fsdp] feat: verl-native MiniMax M3 with NVFP4/MXFP8 activation QAT, R3 and expert LoRA](https://github.com/verl-project/verl/pull/8353)
- **作者**: aoshen02  **时间**: 2026-10-23 17:23 CST
- **摘要**: ### What does this PR do?  Adds day-0 support for MiniMax M3 in verl: a verl-native MiniMax M3 text model for FSDP2 + LoRA RL, rollout routing replay (R3) on the FSDP engine, emulated activation quantization of the served NVFP4/MXFP8 GEMMs, and a MiniMax M3.1 DAPO GRPO recipe. It is not a bitwise tr…

### #8352 — [[vllm, rollout, perf] refactor: stage LoRA buckets in one host block](https://github.com/verl-project/verl/pull/8352)
- **作者**: aoshen02  **时间**: 2026-10-23 17:22 CST
- **摘要**: ### What does this PR do?  Performance only, no behaviour change.  When a LoRA adapter is synced to vLLM through the bucketed IPC transport (`update_weights_from_ipc` with `peft_config` and `base_sync_done`), every received tensor is cloned one by one on the device, cloned again in `_update_weights`…

### #8351 — [[perf] fix: chunk rows in logprobs_from_logits_v2 for 2-D bf16/fp16 logits](https://github.com/verl-project/verl/pull/8351)
- **作者**: aoshen02  **时间**: 2026-10-23 17:22 CST
- **摘要**: ### What does this PR do?  `logprobs_from_logits_v2` is the fallback used by `logprobs_from_logits` when the flash-attn cross-entropy kernel is not used (not installed, or `VERL_DISABLE_FLASH_ATTN_CE=1` for bitwise-reproducible log-probs). For 16-bit logits it loops over the first dimension. With re…

### #8350 — [[ckpt] fix: fail LoRA-only checkpoint load when adapter keys are missing](https://github.com/verl-project/verl/pull/8350)
- **作者**: aoshen02  **时间**: 2026-10-23 17:22 CST
- **摘要**: ### What does this PR do?  `FSDPCheckpointManager.load_checkpoint` loads a LoRA-only checkpoint with `strict=False` (it legitimately lacks the frozen base weights) and already raises on unexpected keys, but never looks at `missing_keys`. If the checkpoint lacks some adapter tensors (e.g. a different…

### #8349 — [[algo] fix: give uniform reward groups exactly zero grpo advantage](https://github.com/verl-project/verl/pull/8349)
- **作者**: saloni-garg  **时间**: 2026-10-23 16:55 CST
- **摘要**: ### What does this PR do?  When every response in a GRPO group gets the same fractional reward, there is no preference between the responses, so every advantage in the group should be exactly zero. Right now the float32 group statistics invent one instead: the group mean can land about an ulp away f…

### #8348 — [[trainer] fix: configure bypass mode before workers are created](https://github.com/verl-project/verl/pull/8348)
- **作者**: aoshen02  **时间**: 2026-10-23 16:16 CST
- **摘要**: ### What does this PR do?  **Problem.** `algorithm.rollout_correction.bypass_mode=true` skips the old-log-prob forward and uses `old_log_probs = rollout_log_probs`. The actor should then train with `loss_mode=bypass_mode`, but it silently keeps the default vanilla PPO loss, anchored at the rollout p…
