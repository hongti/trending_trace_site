# verl-project/verl — 动态追踪

> 生成时间: 2026-10-26 13:11 CST

## AI 总结

# verl-project/verl 近期动态摘要

## 📋 Issue

1. **#8381 — vLLM 量化后端强制 DeepGEMM 导致 H100 崩溃**
   自 #8308 起，`set_default_vllm_quantization_backends` 将 `moe_backend` 强制设为 `deep_gemm`，而该 MXFP4 kernel 仅支持 SM100+，导致在 H100 上每个 rollout worker 在 `load_model` 阶段即失败。

2. **#8380 — RFC：separate_async 独立容错方案**
   提出 trainer 侧与 rollout 侧独立重启的容错（FT）方案，目标是提升 RL 训练的长期稳定性这一核心痛点。

3. **#8376 — confident-label 的 logprob 与梯度被舍入为 0**
   在 torch logprob 路径与 Triton fused linear cross-entropy 中，置信标签的 logprob 及梯度被错误地舍入为恰好 0，影响梯度计算。

## 🔀 Pull Request

**新特性**
- **#8382**：新增 Qwen3.5（含 35B-A3B MoE）在 TPU v7x 上的 GRPO recipe，含上下文并行与 QKV 融合配置。
- **#8370**：VeOmni 引擎支持 `optim.override_optimizer_config`（FSDP2 CPU offload 下 fused AdamW）及可配置的 checkpoint 加载。

**Bug 修复**
- **#8379**：修复 colocated reward-model 结果分数被记录在错误 token 上的问题（多轮对话中以 chat-template 后缀结尾时），影响 GAE / REINFORCE++ / ReMax。
- **#8378 / #8377**：分别修复 Triton fused linear cross-entropy 与 torch logprob 路径中 confident-label 的 logprob 与梯度被舍入为 0 的 Bug（对应 Issue #8376）。
- **#8375**：向 `transformers < 5.15` 回移快速多模态 special-token 检查，修复因 `_check_special_mm_tokens` 在 tensor 上执行 `list(ids).count()` 造成的严重性能退化。
- **#8374 / #8373 / #8372**：一组针对 NPU 上 FSDP CPU offload 的修复，改用阻塞式 NPU→CPU 拷贝，避免 pinned host 内存耗尽；其他设备保留非阻塞传输。
- **#8371**：使 DeepSeek-V4 VeOmni recipe 可在 AMD ROCm（MI300 / MI350 系列）开箱即用，无需额外 `export`。

## 🚀 Release

本轮数据中无 Release 动态。

---

## 🐛 Issues

### #8381 — [[rollout, vllm] `set_default_vllm_quantization_backends` forces `moe_backend=deep_gemm` for DeepSeek-V4's MXFP4 experts; vLLM's DeepGEMM MXFP4 kernel is SM100+ only, so every rollout worker fails at `load_model` on H100](https://github.com/verl-project/verl/issues/8381)
- **作者**: linmuchuiyang  **时间**: 2026-10-26 11:55 CST
- **摘要**: ## What happens  Since #8308 (2026-10-21), `verl/utils/vllm/vllm_quant_utils.py::set_default_vllm_quantization_backends` does `engine_kwargs.setdefault("linear_backend" / "moe_backend", "deep_gemm")` for every quantization config that is not NVFP4. A native DeepSeek-V4 checkpoint (Flash and Pro) car…

### #8380 — [[RFC] FT proposal for separate_async: independent trainer-side and rollout-side restarts](https://github.com/verl-project/verl/issues/8380)
- **作者**: HaoWang113  **时间**: 2026-10-26 10:54 CST
- **摘要**: # [RFC] FT proposal for separate_async: independent trainer-side and rollout-side restarts  ---  ## 1. Background  Long-run stability has always been a key requirement — and pain point — for RL training jobs:  - #6583: a 256×H200 production run died entirely after 6 hours when a single vLLM EngineCo…

### #8376 — [[Bug] Confident-label logprobs and their gradients round to exactly 0 (torch logprob path and Triton fused linear cross-entropy)](https://github.com/verl-project/verl/issues/8376)
- **作者**: catherinelee274  **时间**: 2026-10-26 07:57 CST
- **摘要**: ### System Info  - verl `main` @ 3c6d9b43b - Python 3.12.4, torch 2.9.1+cu128, Triton 3.5.1, CUDA 12.8 - NVIDIA GeForce RTX 3090, Linux  ### Information  - [x] My own modified scripts  ### Tasks  - [x] My own task or dataset (give details below)  ### Reproduction  Both log-prob paths compute `logp_t…

## 🔀 Pull Requests

### #8382 — [[hardware, recipe] feat: Qwen3.5 (incl. 35B-A3B MoE) GRPO recipe for TPU v7x, plus context parallelism and QKV fusion configs](https://github.com/verl-project/verl/pull/8382)
- **作者**: jialei777  **时间**: 2026-10-26 12:56 CST
- **摘要**: ### What does this PR do?  This PR adds a **Qwen3.5** GRPO recipe for **TPU v7x** (`examples/tpu/grpo/run_qwen3_5_torchtitan.sh`). Qwen3.5 is the hybrid GatedDeltaNet + full-attention family, dense or MoE; the recipe defaults to `Qwen/Qwen3.5-35B-A3B`.  It also adds the following to the TorchTitan T…

### #8379 — [[reward, trainer] fix: keep colocated RM outcomes on generated tokens](https://github.com/verl-project/verl/pull/8379)
- **作者**: Levius-Fubuki  **时间**: 2026-10-26 08:53 CST
- **摘要**: ### What does this PR do?  Keep colocated reward-model outcome scores on the last generated, non-padding token. A multi-turn response may end in a masked chat-template suffix; GAE, REINFORCE++ and ReMax then drop an outcome placed on that suffix. Forward the original generation mask from TransferQue…

### #8378 — [[training_utils] fix: keep confident-label logprobs and gradients in the Triton fused linear cross-entropy](https://github.com/verl-project/verl/pull/8378)
- **作者**: catherinelee274  **时间**: 2026-10-26 07:59 CST
- **摘要**: ### What does this PR do?  Fixes the Triton fused linear cross-entropy (`verl/utils/kernel/kernels.py`, used with `use_fused_kernels=True`) so a confident label keeps its logprob and gradient. Before this change the forward formed `z_t - (max + log(accu))`, where `accu` sums `exp(z - max)` over ever…

### #8377 — [[training_utils] fix: keep confident-label logprobs and gradients in the torch logprob path](https://github.com/verl-project/verl/pull/8377)
- **作者**: catherinelee274  **时间**: 2026-10-26 07:59 CST
- **摘要**: ### What does this PR do?  Fixes the torch log-prob path (`logprobs_from_logits_naive` and `logprobs_from_logits_v2`) so a confident label keeps its logprob and gradient. Before this change both rounded to exactly 0 once the label's margin over the other logits passed ~16.6 in fp32 (~5.5 for v2's bf…

### #8375 — [[perf, data] fix: backport fast multimodal special-token check for transformers < 5.15](https://github.com/verl-project/verl/pull/8375)
- **作者**: WinstonLiyt  **时间**: 2026-10-26 07:11 CST
- **摘要**: ### What does this PR do?  Fixes #8261.  With the pinned `transformers==5.12.1`, every processor call runs `ProcessorMixin._check_special_mm_tokens`, which does `list(ids).count(token_id)` on a tensor. That compares one scalar tensor at a time, so a long prompt pays ~0.5 s per call even when it has …

### #8374 — [[fsdp] fix: Use blocking NPU copies when offloading model and optimizer state](https://github.com/verl-project/verl/pull/8374)
- **作者**: coderabbitai[bot]  **时间**: 2026-10-26 03:46 CST
- **摘要**: ### What does this PR do?  Use blocking NPU-to-CPU transfers for FSDP model shards and optimizer tensors to avoid exhausting the driver's pinned host-memory pool. Preserve nonblocking transfers for other devices.  AI assistance was used to prepare this metadata. Duplicate-work verification remains p…

### #8373 — [[fsdp] fix: Use blocking CPU offload copies on NPU](https://github.com/verl-project/verl/pull/8373)
- **作者**: coderabbitai[bot]  **时间**: 2026-10-26 03:46 CST
- **摘要**: ### What does this PR do?  Use blocking copies when offloading FSDP model shards from NPU to CPU to avoid exhausting the pinned host allocator across colocated workers. Preserve nonblocking copies on other devices.  AI assistance was used to prepare this metadata. Duplicate-work checks have not been…

### #8372 — [[fsdp] fix: Avoid pinned host memory exhaustion during NPU CPU offload](https://github.com/verl-project/verl/pull/8372)
- **作者**: coderabbitai[bot]  **时间**: 2026-10-26 03:46 CST
- **摘要**: ### What does this PR do?  Use blocking CPU transfers for FSDP1 model shards and optimizer state on NPU to avoid exhausting pinned host memory across ranks. Preserve non-blocking transfers on other devices.  AI assistance was used to prepare this metadata. Duplicate-work status remains unverified; h…

### #8371 — [[rollout, recipe] fix: run the DeepSeek-V4 VeOmni recipe on AMD ROCm out of the box](https://github.com/verl-project/verl/pull/8371)
- **作者**: xiaohong42  **时间**: 2026-10-26 02:51 CST
- **摘要**: ### What does this PR do?  Makes `examples/grpo_trainer/run_deepseek_v4_veomni.sh` run unchanged on AMD Instinct MI300-series (gfx942) and MI350-series (gfx950) clusters: no `export` before `ray start`, no extra overrides. Its training parameters are untouched. Three issues, one per file:  1. **vLLM…

### #8370 — [[veomni] feat: honor optim.override_optimizer_config (fused AdamW under FSDP2 CPU offload) and make checkpoint ingestion configurable](https://github.com/verl-project/verl/pull/8370)
- **作者**: xiaohong42  **时间**: 2026-10-26 02:51 CST
- **摘要**: ### What does this PR do?  Two VeOmni-engine settings that the other engines already expose:  1. **`optim.override_optimizer_config` is ignored by the VeOmni engine** (the FSDP, Megatron and Automodel engines honor it), so it always builds VeOmni's default foreach AdamW. With `actor.veomni.enable_fs…
