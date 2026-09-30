# verl-project/verl — 动态追踪

> 生成时间: 2026-09-30 13:51 CST

## AI 总结

# verl-project/verl 仓库近期动态摘要

## Issue
（无）

## Pull Request（共 10 个）

### 新特性
- **#8070**：在 LLM 服务器管理器中抽取 replica-size 计算逻辑为 `_get_rollout_replica_world_size()` 方法，支持自定义 rollout manager 复写，避免代码重复。
- **#8065**：升级 VeOmni 引擎到最新并行状态 API，恢复 SFT 引擎对比测试覆盖，并将 GPU PPO 工作流迁移至 `uv`。

### Bug 修复
- **#8068**：修复 Hydra 参数顺序及无效配置键导致 Ascend 多节点 GLM5.2 nightly CI 在训练脚本启动前即失败的问题。
- **#8064**：修复 vLLM rollout 中 FP8 异步权重同步时 `tie_word_embeddings=True` 模型存在别名导致 trainer state dict 含重复 key 的 bug，去掉冗余别名映射。
- **#8061**：修复 `pad_bshd_to_minibatch_max=True` 场景下前向 KL topk 蒸馏的张量形状不匹配问题。

### 清理与维护
- **#8069**：移除 `replica.py` 中无条件设置 `SGLANG_USE_CPU_ENGINE=1` 的残留代码及相关 import。

### CI 相关
- **#8067 / #8063 / #8066**：调整 Ascend CI runner 标签，测试多节点 CI 流程。

### 文档修复
- **#8062**：将 `NVTE_FP8_BLOCK_SCALING_FP32_SCALES=1` 标注为仅 Hopper 架构适用（Blackwell/SM100 无需此设置）。

## Release
（无）

---

## 🔀 Pull Requests

### #8070 — [[rollout] feat: expose a server manager replica-size hook](https://github.com/verl-project/verl/pull/8070)
- **作者**: NancyFyong  **时间**: 2026-09-30 13:37 CST
- **摘要**: ### Summary  Extract the existing footprint calculation into `LLMServerManager._get_rollout_replica_world_size()`. Custom rollout managers can override this method rather than copy `_initialize_llm_servers`, preserving upstream initialization and metrics behavior. The default TP/DP/PP and prefill/de…

### #8069 — [[rollout] remove leftover todo ENV, remove SGLANG_USE_CPU_ENGINE environment ](https://github.com/verl-project/verl/pull/8069)
- **作者**: kahlun  **时间**: 2026-09-30 12:37 CST
- **摘要**: ## What does this PR do?  Removes the unconditional `SGLANG_USE_CPU_ENGINE=1` set/`del` from `verl/workers/rollout/replica.py::_load_sglang()`, plus the now-unused `import os`.  ```diff  import asyncio  import logging -import os  from abc import ABC, abstractmethod   def _load_sglang(): -    os.envi…

### #8068 — [Fix hydra argument-order and invalid config key breaking Ascend multinode GLM5.2 nightly CI](https://github.com/verl-project/verl/pull/8068)
- **作者**: Copilot  **时间**: 2026-09-30 11:40 CST
- **摘要**: The "E2E Ascend RayJob multinode GRPO GLM5.2 Top16 Megatron" nightly job was failing before the training script even reached the RayJob worker setup, due to two bugs in the launch invocation.  ### Argument ordering (`unrecognized arguments`) `--config-name` was appended via the `EXTRA` array *after*…

### #8067 — [[ci] fix: change ascend ci runner](https://github.com/verl-project/verl/pull/8067)
- **作者**: yyyy2000  **时间**: 2026-09-30 10:41 CST
- **摘要**: ### What does this PR do?  change ascend ci runner  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veom…

### #8066 — [test Ascend multinode ci. ](https://github.com/verl-project/verl/pull/8066)
- **作者**: lxb007981  **时间**: 2026-09-30 09:54 CST
- **摘要**: ### What does this PR do?  Test  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, `sglang`, `vll…

### #8065 — [[veomni, ci] fix: upgrade VeOmni APIs and migrate GPU CI to uv](https://github.com/verl-project/verl/pull/8065)
- **作者**: Luosuu  **时间**: 2026-09-30 05:07 CST
- **摘要**: ### What does this PR do?  Update the VeOmni engine to its current public parallel-state API, restore VeOmni coverage in the SFT engine comparison suite, and migrate the VeOmni GPU PPO workflow to the shared UV environment. The old engine calls the removed `init_parallel_state` API and passes an ign…

### #8064 — [[rollout, vllm] fix: drop tied-embedding alias in fp8 weight sync](https://github.com/verl-project/verl/pull/8064)
- **作者**: cr-gao  **时间**: 2026-09-29 23:50 CST
- **摘要**: ### What does this PR do?  The quantized (fp8) branch of the async weight sync feeds every bucket straight into `load_quanted_weights`. For a model with `tie_word_embeddings=True` the trainer state dict carries both `model.embed_tokens.weight` and its alias `lm_head.weight`, and the two can land in …

### #8063 — [[ci] fix: change runner label for nightly_ascend.yml](https://github.com/verl-project/verl/pull/8063)
- **作者**: yyyy2000  **时间**: 2026-09-29 21:23 CST
- **摘要**: ### What does this PR do?  as title  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, `sglang`, …

### #8062 — [[doc] fix: mark NVTE_FP8_BLOCK_SCALING_FP32_SCALES=1 as Hopper-only](https://github.com/verl-project/verl/pull/8062)
- **作者**: wengeezhang  **时间**: 2026-09-29 20:40 CST
- **摘要**: ### What does this PR do?  The FP8 end-to-end section of `docs/low_precision/fp8.md` lists `NVTE_FP8_BLOCK_SCALING_FP32_SCALES=1` as a general requirement. It only applies on Hopper.  On Blackwell (SM100+), Transformer Engine has no native block-scaled GEMM. It runs the block-wise recipe through MXF…

### #8061 — [[trainer, megatron, fsdp] fix: forward-KL topk distillation shape mismatch with pad_bshd_to_minibatch_max](https://github.com/verl-project/verl/pull/8061)
- **作者**: alwaysyiyu  **时间**: 2026-09-29 19:46 CST
- **摘要**: ### What does this PR do?  When `pad_bshd_to_minibatch_max=True` (introduced in #6901), the student forward pads every micro-batch to the mini-batch's global max seqlen via `forced_max_seqlen`, but `compute_forward_kl_topk` still splits the teacher top-k tensors with `preprocess_bshd_engine` **witho…
