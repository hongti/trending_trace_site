# verl-project/verl — 动态追踪

> 生成时间: 2026-10-10 14:15 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📦 Release
本期无版本发布动态。

### 🐛 Issue
* **#8178 Qwen3.5 Ulysses 序列并行梯度不匹配问题**
  作者报告了在 Qwen3.5-9B（混合全注意力 + Gated DeltaNet）上使用 Ulysses 序列并行时存在的问题：虽然前向传播结果与数据并行一致，但参数梯度却无法匹配（涉及 GDN cp_context 反向传播与 engine lm_head 梯度计算）。

### 🔀 Pull Request (PR)

**✨ 新特性与功能改进**
* **#8179 [rollout, vllm] 多轮对话会话绑定 vLLM DP rank**：在 vLLM rollout 副本的数据并行大小 > 1 时，将每个多轮对话会话固定到一个 DP 引擎上，以更好地利用各引擎独立的 prefix cache，提升效率。
* **#8180 [fsdp] 支持保存合并后的 LoRA HF 模型**：跟进之前的 PR，新增支持保存合并后的 LoRA hf_model。

**🛠️ Bug 修复与兼容性**
* **#8184 [ci, fsdp] 修复 FSDP1 LoRA 兼容性**：将 NPU 环境的 `peft` 版本固定为 0.20.0。因为 PEFT 0.21.0 对 state-dict 处理进行了重构，破坏了 FSDP1 的 LoRA 兼容性。
* **#8182 & #8175 [reward] 修复 prime_code 评分逻辑**：修复了 `prime_code` 对 APPS/TACO 调用型任务（call-based）评分全为 0 的 Bug。此前即使答案正确也会得 0 分且无法获得部分得分，现已修复测试用例格式解析逻辑。
* **#8177 [fsdp] 修复 FSDP2 异步训练分片保存**：修复了在完全异步训练器中开启 `param_offload=True` 时，`fsdp2_sharded_save_to_cpu` 的逻辑错误（此时本地分片已在 CPU 上，`.detach().cpu()` 返回的 tensor 处理有误）。
* **#8176 [algo] 修复 bypass 模式 Loss 归一化**：修复了在 bypass 模式下，未根据拒绝采样后保留的 token 数量对 token-mean loss 进行正确归一化的问题。
* **#8173 [perf] 修复 Triton 融合内核性能问题**：移除了对内部单位步长误标注为 `tl.int64` 的行为，使 Triton 编译器能够重新识别连续内存地址，恢复性能。

**⚙️ 其他更新**
* **#8183 [perf] 修复 Qwen3-VL FLOPS 估算**：改为直接从模型 config 中读取 `head_dim`，而不是通过 `hidden_size // num_attention_heads` 推导，提高估算准确性。
* **#8181 [data] 更新 TQ 版本**：将 TQ 版本更新至 v0.1.11，并同步对齐了相关的配置 yaml 文件。

---

## 🐛 Issues

### #8178 — [[fsdp, model] Qwen3.5 Ulysses SP: forward matches DP but parameter gradients don't (GDN cp_context backward + engine lm_head grad)](https://github.com/verl-project/verl/issues/8178)
- **作者**: MaCoredroid  **时间**: 2026-10-10 08:55 CST
- **摘要**: ### Summary  With Ulysses sequence parallelism on **Qwen3.5-9B** (hybrid full attention + Gated DeltaNet), the **forward pass matches data parallelism but the parameter gradients do not**. Across two independent checks, SP-vs-DP gradient cosine is 0.85–0.96, while the DP-vs-DP noise floor is ≥ 0.999…

## 🔀 Pull Requests

### #8184 — [[ci, fsdp] fix: pin PEFT to 0.20.0 for FSDP1 LoRA compatibility](https://github.com/verl-project/verl/pull/8184)
- **作者**: MrJVium  **时间**: 2026-10-10 14:07 CST
- **摘要**: ### What does this PR do?  Pin `peft` to `0.20.0` in `requirements-npu.txt` to restore FSDP1 LoRA compatibility.  PEFT 0.21.0 introduced a state-dict handling refactor in [huggingface/peft#3490](https://github.com/huggingface/peft/pull/3490). For nested FSDP1 models, PEFT may fail to match adapter m…

### #8183 — [[perf] fix: read head_dim from config for Qwen3-VL FLOPS estimation](https://github.com/verl-project/verl/pull/8183)
- **作者**: kkyyxhll  **时间**: 2026-10-10 13:25 CST
- **摘要**: ### What does this PR do?  Fixes `_estimate_qwen3_vl_flops` in `verl/utils/flops_counter.py` to read `head_dim` from the model config instead of deriving it as `hidden_size // num_attention_heads`, which underestimates the FLOPS (and therefore underreports MFU) for Qwen3-VL models whose `num_attenti…

### #8182 — [[reward] fix: prime_code scores every call-based APPS/TACO task 0](https://github.com/verl-project/verl/pull/8182)
- **作者**: zhongerdeajie  **时间**: 2026-10-10 11:55 CST
- **摘要**: ### What does this PR do?  Fixes #8174. `prime_code` graded every call-based (function-call) APPS/TACO sample **0, even for a correct solution**, and additionally gave 0 instead of partial credit under `continuous=True`. Two defects, two commits:  1. **`testing_util.py` — parse layout (commit 1).** …

### #8181 — [[data] fix: Update TQ version and align config yaml](https://github.com/verl-project/verl/pull/8181)
- **作者**: 0oshowero0  **时间**: 2026-10-10 11:27 CST
- **摘要**: ### What does this PR do?  - Update TQ version to v0.1.11 - Align TQ related config yaml  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modul…

### #8180 — [[fsdp] fix: support save merged lora hf_model](https://github.com/verl-project/verl/pull/8180)
- **作者**: wuxibin89  **时间**: 2026-10-10 11:18 CST
- **摘要**: ### What does this PR do?  Follow up https://github.com/verl-project/verl/pull/8171, support save merged lora hf_model

### #8179 — [[rollout, vllm] feat: pin each multi-turn session to one vLLM DP rank](https://github.com/verl-project/verl/pull/8179)
- **作者**: aoshen02  **时间**: 2026-10-10 10:54 CST
- **摘要**: ### What does this PR do?  With a vLLM rollout replica that has `data_parallel_size > 1` (e.g. DP + EP inside one server), each DP engine has its own prefix cache. `LLMServerClient` sends every turn under a fresh uuid and no DP rank, so vLLM's internal DP load balancer picks the least-loaded engine …

### #8177 — [[fsdp] fix: copy offloaded shards in fsdp2_sharded_save_to_cpu](https://github.com/verl-project/verl/pull/8177)
- **作者**: ZhiliangWu  **时间**: 2026-10-10 04:58 CST
- **摘要**: ### What does this PR do?  Fix `fsdp2_sharded_save_to_cpu` when `param_offload=True` in the fully async trainer.  With offload on, the local shards are already on CPU, so `.detach().cpu()` returns a tensor that shares memory with the live shard, not a copy. In `FullyAsyncTrainer._compute_old_log_pro…

### #8176 — [[algo] fix: normalize bypass-mode token-mean loss by tokens kept after rejection sampling](https://github.com/verl-project/verl/pull/8176)
- **作者**: Alexander230  **时间**: 2026-10-10 04:19 CST
- **摘要**: ### What does this PR do?  Fixes #4852.  In bypass mode, rejection sampling (`rollout_rs`) masks tokens inside the policy loss, per micro-batch, using the current log probs. By then the engine has already counted the global batch's tokens (`batch_num_tokens`) from the unrejected `loss_mask`, and `pp…

### #8175 — [[reward] fix: read APPS/TACO call-based tests in prime_code](https://github.com/verl-project/verl/pull/8175)
- **作者**: prabhu-gopal  **时间**: 2026-10-10 01:38 CST
- **摘要**: ### What does this PR do?  Fixes #8174. prime_code's call-based path only parsed LiveCodeBench-style tests (each input a string of JSON lines). APPS and TACO publish call-based tests with each input as a list of arguments, so every such task crashed before running and scored 0, even for a correct so…

### #8173 — [[perf, training_utils] fix: don't annotate unit strides as tl.int64 in fused kernels](https://github.com/verl-project/verl/pull/8173)
- **作者**: he-weiwen  **时间**: 2026-10-09 23:39 CST
- **摘要**: ### <p>Slop free TLDR:</p> <p>Annotating the inner strides in with <code>tl.int64</code> makes the triton compiler unable to recognize if the stride is equal to 1. This info is needed to decide if the addresses are contiguous, so the current code causes failures in vectorization/coalescing memory ac…
