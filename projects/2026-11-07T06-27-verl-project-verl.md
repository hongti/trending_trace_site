# verl-project/verl — 动态追踪

> 生成时间: 2026-11-07 14:27 CST

## AI 总结

# verl-project/verl 近期动态摘要

## Issue（3 个）

1. **#8586** — Ascend NPU quick-start 脚本无法在单卡/小规模（<8 设备）节点上运行。脚本结构上硬编码了 `n_gpus_per_node=8` 和 `ASCEND_RT_VISIBLE_DEVICES=0-7`，导致单卡 910B 用户无法使用入口。换小模型也无法解决设备数假设问题。
2. **#8580** — V1 trainer 静默忽略了 variance proxy（方差代理）指标，`_compute_metrics` 未将 `old_log_probs` 和 `sum_pi_squared` 加入 TransferQueue 请求。
3. **#8579** — `hf_processor` 用宽泛的 `except Exception` 吞掉了所有 processor 创建错误（包括自身的"Unsupported processor type"检查），仅返回 `None` 并发出警告，可能影响多模态处理。

## PR（10 个）

### 硬件适配（Ascend / ROCm）
- **#8585**：修复 vLLM-Ascend RL 模式，默认将 `rl_config.enabled` 设为 `true`，禁用 NZ weights 的 sleep/wake 和权重更新。
- **#8584**：移除 DeepSeek-V4 在 ROCm 上的 `VLLM_ROCM_DSV4_BPRESHUFFLE` workaround（上游 vLLM 已修复底层 bug）。
- **#8581**：对齐 Ascend CANN 和 Megatron 依赖版本。
- **#8574**：新增 Kimi-K3.1 在 ROCm（MI355X, gfx950）上的 true on-policy GRPO LoRA recipe，vLLM rollout log-probs 与 trainer log-probs 达到 bitwise 一致。
- **#8576**：将 A3 验证过的性能优化移植到 Qwen3.5-35B-A3B（MoE）Megatron GRPO 的 Ascend NPU 路径，包含 MTP spec decode 和 rollout 调优，并新增端到端 CI。

### Trainer 修复与优化
- **#8582**：修复 #8580，恢复 V1 trainer 中 variance proxy 指标计算。
- **#8578**（WIP）：在 GRPO rollouts 之间共享 prompt preprocessing 和 TransferQueue 存储，通过 `transfer_queue.share_prompts` 开关控制（默认关闭），减少重复计算与内存占用。
- **#8577**（WIP Draft）：规划为原生 FSDP2 后端添加 Muon 优化器支持，目前仅含实现计划。

### 模型与工具
- **#8583**：新增 VeOmni 后端对 **Qwen3.8 Flash Next**（`qwen4_exp`）的支持，包括 PLE 表并行、可选冻结 PLE embedding、actor-to-rollout 权重导出。
- **#8575**：修复 Gemma4 tool parser 无法正确解析嵌套（数组/对象）参数，以及工具名含 `-` 或 `.` 时调用被丢弃的问题。

## Release
本期无 Release。

---

## 🐛 Issues

### #8586 — [[Ascend] NPU quick-start scripts cannot run on single-card / small (<8-device) nodes](https://github.com/verl-project/verl/issues/8586)
- **作者**: WentaoXu03  **时间**: 2026-11-07 10:04 CST
- **摘要**: > TL;DR（中文）：单卡 910B 用户没有任何可用的 NPU quick-start 入口——三个脚本在结构上假设 ≥8 设备（rollout TP=2 非法、`n_gpus_per_node=8` 硬编码、`ASCEND_RT_VISIBLE_DEVICES=0-7` 硬编码）。换小模型（#6776 对 NVIDIA 侧的建议）解决不了设备数假设问题。  ### Environment  - Atlas A2 (910B), 1 node / **1 card**, CANN 8.x, vllm-ascend 0.30.x stack - verl main @ 31cfccb4  #…

### #8580 — [[trainer] V1 silently omits variance proxy metrics](https://github.com/verl-project/verl/issues/8580)
- **作者**: Huanlin233  **时间**: 2026-11-06 17:38 CST
- **标签**: bug
- **摘要**: ### System Info  verl main commit: 689fa1a2a1a35fbfee6cdd8cabdec0c15688ccaf Reproduction environment: Windows, Python 3.12.12, PyTorch 2.13.0+cpu, TensorDict 0.10.0  ### Information  - [ ] The official example scripts - [ ] My own modified scripts  ### Tasks  - [ ] An officially supported task in th…

### #8579 — [hf_processor swallows processor errors, including its own "Unsupported processor type" check](https://github.com/verl-project/verl/issues/8579)
- **作者**: Hang-get  **时间**: 2026-11-06 17:30 CST
- **摘要**: `verl.utils.tokenizer.hf_processor` wraps processor creation in `except Exception`, warns "Failed to create processor: ... This may affect multimodal processing" and returns `None` (verl/utils/tokenizer/tokenizer.py on main 689fa1a2). The in-code TODO says the try/except should be removed once trans…

## 🔀 Pull Requests

### #8585 — [[vllm, hardware] fix: enable vLLM-Ascend RL mode by default](https://github.com/verl-project/verl/pull/8585)
- **作者**: lxb007981  **时间**: 2026-11-06 22:36 CST
- **摘要**: ### What does this PR do?  Default rl_config.enabled to true when preparing vLLM engine arguments on Ascend. This disables NZ weights for sleep/wake and weight updates across actor and reward-model rollouts.  Preserve explicit user overrides and other additional configuration.  ### Checklist Before …

### #8584 — [[recipe, hardware] fix: drop the VLLM_ROCM_DSV4_BPRESHUFFLE workaround for DeepSeek-V4 on ROCm](https://github.com/verl-project/verl/pull/8584)
- **作者**: xiaohong42  **时间**: 2026-11-06 22:00 CST
- **摘要**: ### What does this PR do?  Removes the `VLLM_ROCM_DSV4_BPRESHUFFLE` workaround that verl-project/verl#8371 added for DeepSeek-V4 on ROCm: vllm-project/vllm#65897 fixes the underlying bug in vLLM instead. That PR is still a draft, so this one stays a draft until it is merged.  On ROCm with AITER, vLL…

### #8583 — [[veomni, model] feat: support Qwen3.8 Flash Next](https://github.com/verl-project/verl/pull/8583)
- **作者**: Feng0w0  **时间**: 2026-11-06 21:15 CST
- **摘要**: ### What does this PR do?  Add VeOmni backend support for Qwen3.8 Flash Next (`qwen4_exp`), including PLE table parallelism, optional frozen PLE embeddings, and actor-to-rollout weight export. Also handle autocast and multimodal position IDs needed by this integration.  ### Design & Code Changes  - …

### #8582 — [[trainer] fix: restore variance proxy metrics in V1](https://github.com/verl-project/verl/pull/8582)
- **作者**: Huanlin233  **时间**: 2026-11-06 18:09 CST
- **摘要**: ### What does this PR do?  Fixes #8580.  V1's `_compute_metrics` omits `old_log_probs` and `sum_pi_squared` from its TransferQueue request. Even when both fields are available in the queue, `compute_variance_proxy_metrics` receives an incomplete batch and returns an empty dictionary.  ### Duplicate-…

### #8581 — [[env, doc] chore: align Ascend CANN and Megatron dependencies](https://github.com/verl-project/verl/pull/8581)
- **作者**: renjunying2  **时间**: 2026-11-06 18:01 CST
- **摘要**: ### What does this PR do?  [env, doc] chore: align Ascend CANN and Megatron dependencies  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modul…

### #8578 — [WIP: [trainer, rollout] feat: share prompt preprocessing and TransferQueue…](https://github.com/verl-project/verl/pull/8578)
- **作者**: OutstanderWang  **时间**: 2026-11-06 16:43 CST
- **摘要**: … storage across GRPO rollouts  The rollout.n trajectories of a prompt in the V1 trainer now share both the work and the storage of that prompt, behind one switch `transfer_queue.share_prompts` (default True):  - Storage: the trainer puts the dataset fields of each prompt once into TransferQueue's p…

### #8577 — [[fsdp, doc] chore: plan native FSDP2 Muon support (WIP)](https://github.com/verl-project/verl/pull/8577)
- **作者**: yue-zeng-yue  **时间**: 2026-11-06 15:56 CST
- **摘要**: This draft outlines the plan for adding Muon support to verl's native FSDP2 backend.  It currently contains only the implementation plan and does not change training behavior. The implementation and validation results will be added in follow-up updates.  Refs #8549

### #8576 — [[perf, hardware, megatron, ci] feat: speed up Qwen3.5-35B Megatron GRPO on Ascend A3 with MTP spec decode and rollout tuning, plus e2e CI](https://github.com/verl-project/verl/pull/8576)
- **作者**: ruanhao566  **时间**: 2026-11-06 15:31 CST
- **摘要**: ### What does this PR do?  This PR ports A3-validated performance optimizations into the Ascend NPU path of the Qwen3.5-35B-A3B (MoE) Megatron GRPO example, and covers that path with a new PR-triggered e2e CI.  **Performance changes in `examples/grpo_trainer/run_qwen3_5_35b_megatron.sh` (all confine…

### #8575 — [[tool] fix: parse nested arguments in Gemma4 tool parser](https://github.com/verl-project/verl/pull/8575)
- **作者**: ZhuLinsen  **时间**: 2026-11-06 14:46 CST
- **摘要**: ### What does this PR do?  `Gemma4ToolParser` mangles array and object arguments, and drops calls whose tool name contains `-` or `.`. It matched arguments with a flat `key:value` regex, so a value was cut at the first `,` or `}`:  ``` call:tag{tags:[<|"|>a<|"|>,<|"|>b<|"|>],ids:[1,2,3]} before: {"t…

### #8574 — [[hardware, recipe, rollout, worker] feat: Kimi-K3.1 true on-policy GRPO on ROCm (MI355X)](https://github.com/verl-project/verl/pull/8574)
- **作者**: indianspeedster  **时间**: 2026-11-06 14:44 CST
- **摘要**: ### What does this PR do?  Adds a ROCm (MI355X, gfx950) recipe for Kimi-K3.1 LoRA GRPO where the vLLM rollout log-probs and the trainer log-probs of the same tokens are **bitwise equal** (`training/rollout_probs_diff_max` = 0.0), plus the verl changes the full model needs to sync weights at all on 2…
