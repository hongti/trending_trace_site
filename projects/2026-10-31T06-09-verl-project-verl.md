# verl-project/verl — 动态追踪

> 生成时间: 2026-10-31 14:09 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📦 Release（发布）
本次更新周期内无新的版本发布。

### 🐛 Issue（议题）
*   **#8462 Continuous Token 合并未对照聊天模板进行检查问题**
    *   **作者**: khazic
    *   **摘要**: 指出在多轮 rollout 训练中，Continuous Token（#6719）直接保留生成的 token id 并以增量形式追加新的对话轮次，但这一合并过程未与聊天模板进行一致性校验。此外，`tokenization_sanity_check_mode` 自 #4422 合并以来一直处于无效状态。

### 🔧 Pull Request（合并请求）

**🚀 重要新特性与功能增强**
*   **#8469 TPU v7x 上的 Qwen3.5-397B-A17B 全参数 GRPO 配方**
    *   **作者**: jialei777
    *   **亮点**: 新增了对超大规模 **Qwen3.5-397B-A17B** MoE 模型全参数 GRPO 训练的支持。包含 8K 思考上下文（512 + 7680）、多副本 vLLM rollout 以及多 TB 级 GCS 检查点功能。
*   **#8466 / #8465 解耦 GRPO 的零陈旧异步分组交付**
    *   **作者**: PengyuLi7
    *   **亮点**: 引入 `trainer.v1.trainer_mode=group_delivery` 模式，支持在**物理隔离的 GPU 池**上分别运行 rollout 和 Megatron 训练，实现零陈旧度的异步交付。
*   **#8463 Continuous Token 合并的聊天模板校验**
    *   **作者**: khazic
    *   **亮点**: 修复了上述 Issue #8462 提到的问题，在 Continuous Token 合并时增加针对聊天模板的边界检查，防止错误的 naïve merge 破坏模型输出结构。
*   **#8464 FSDP 将打包序列边界传递给 Flash Attention**
    *   **作者**: kjgggggg
    *   **亮点**: 在 `use_remove_padding` 模式下，将打包序列的边界作为 `varlen kwargs` 传递给 Flash Attention，优化执行效率。

**⚡ 性能优化**
*   **#8459 蒸馏教师并发初始化**
    *   **作者**: zgh-99
    *   **亮点**: 改变多教师蒸馏模型按序初始化的策略为并发启动，大幅缩减了多教师配方的启动时间。

**🛠️ Bug 修复**
*   **#8468 失败验证结算与远程生成取消** (zzzzzzxh)：修复中断验证时的结算问题，确保分母完整，并取消远程生成任务。
*   **#8467 保留 NVFP4 激活缩放因子** (cyanseek)：修复通过 Bridge 导出 Megatron QAT 权重时 NVFP4 激活缩放因子丢失的问题。
*   **#8461 TransferQueue 字段请求修复** (renjunying2)：修复在未启用 critic 时仍错误请求 `values` 字段导致的 GRPO 训练失败问题。

**🧹 工程与维护**
*   **#8460 CI 优化** (lxb007981)：移除了 CI 文件缓存并进行精简。

---

## 🐛 Issues

### #8462 — [[rollout] Continuous Token merges are not checked against the chat template during training; tokenization_sanity_check_mode has been a no-op since #4422](https://github.com/verl-project/verl/issues/8462)
- **作者**: khazic  **时间**: 2026-10-30 17:41 CST
- **摘要**: ## Summary  Continuous Token (#6719) builds multi-turn rollouts token-in, token-out: generated token ids are kept as-is and every tool/user turn is appended as a token delta. Two consistency properties are involved:  - **A. Rollout vs. training.** The trainer sees exactly the token ids the inference…

## 🔀 Pull Requests

### #8469 — [[hardware, recipe, trainer] feat: Qwen3.5-397B-A17B full-parameter GRPO recipe on TPU v7x (8K context, multi-replica rollout, GCS checkpoints)](https://github.com/verl-project/verl/pull/8469)
- **作者**: jialei777  **时间**: 2026-10-31 13:20 CST
- **摘要**: ### What does this PR do?  Adds full-parameter **Qwen3.5-397B-A17B** MoE GRPO support (including **8K thinking context** `512 + 7,680 = 8,192`, multi-replica vLLM rollout, and multi-terabyte GCS checkpoint streaming) to `examples/tpu/grpo/run_qwen3_5_torchtitan.sh`, paired with [verl-project/verl-ha…

### #8468 — [[trainer, rollout] fix: settle failed validation and cancel remote generation](https://github.com/verl-project/verl/pull/8468)
- **作者**: zzzzzzzxh  **时间**: 2026-10-31 10:52 CST
- **摘要**: ### What does this PR do?  This is the companion dependency for [uni-agent #294](https://github.com/verl-project/uni-agent/pull/294). It makes interrupted validation settle with complete denominators: the V1 trainer invokes optional lifecycle hooks, skips empty TQ batches when all observations fail,…

### #8467 — [[megatron] fix: preserve NVFP4 activation scales across pipeline export](https://github.com/verl-project/verl/pull/8467)
- **作者**: cyanseek  **时间**: 2026-10-31 02:19 CST
- **摘要**: ### What does this PR do?  Preserve observed NVFP4 activation scales when exporting Megatron QAT weights through Bridge.  The collector uses the first numeric module-path component as a layer ID, although local layer indices restart on each pipeline stage and virtual chunk. It also collapses Sequent…

### #8466 — [[megatron, trainer, worker, env] feat: zero-staleness async group delivery for disaggregated GRPO](https://github.com/verl-project/verl/pull/8466)
- **作者**: PengyuLi7  **时间**: 2026-10-31 00:07 CST
- **摘要**: ### What does this PR do?  `trainer.v1.trainer_mode=group_delivery` runs rollout and Megatron training on **disaggregated** GPU pools (`actor_rollout_ref.hybrid_engine=False`), which is where the long tail of agentic rollouts otherwise leaves the training pool idle. This PR turns that topology into …

### #8465 — [[megatron, trainer, worker, env] feat: zero-staleness async group delivery for disaggregated GRPO](https://github.com/verl-project/verl/pull/8465)
- **作者**: PengyuLi7  **时间**: 2026-10-30 23:57 CST
- **摘要**: ### What does this PR do?  `trainer.v1.trainer_mode=group_delivery` runs rollout and Megatron training on **disaggregated** GPU pools (`actor_rollout_ref.hybrid_engine=False`), which is where the long tail of agentic rollouts otherwise leaves the training pool idle. This PR turns that topology into …

### #8464 — [[fsdp, perf] feat: pass packed-sequence boundaries to flash attention as varlen kwargs](https://github.com/verl-project/verl/pull/8464)
- **作者**: kjgggggg  **时间**: 2026-10-30 18:38 CST
- **摘要**: ### What does this PR do?  With `use_remove_padding`, the FSDP engine packs a micro-batch into one `(1, total_nnz)` row and passes only `position_ids` to the model. transformers then re-derives `cu_seqlens` / `max_seqlen` from `position_ids` inside **every attention layer** (`nonzero`, `.item()`, a …

### #8463 — [[rollout] feat: check Continuous Token merges against the chat template](https://github.com/verl-project/verl/pull/8463)
- **作者**: khazic  **时间**: 2026-10-30 18:11 CST
- **摘要**: ### What does this PR do?  Closes #8462.  #6719 identified the boundary problem this PR checks for. Its pitfall 3 gives two counterexamples where a naive merge goes wrong: Qwen3 stops at `<|im_end|>` while the template continues with `<|im_end|>\n`, and GLM-4.7's `<|observation|>` is both a stop tok…

### #8461 — [[trainer] fix: request values from TransferQueue only when critic is enabled](https://github.com/verl-project/verl/pull/8461)
- **作者**: renjunying2  **时间**: 2026-10-30 16:26 CST
- **摘要**: ### What does this PR do?  Fix GRPO training failures with newer TransferQueue versions that reject requests for missing fields. When the critic is disabled, `values` is not produced but was still requested during advantage and metrics computation.  ### Checklist Before Starting  - [ ] Search for si…

### #8460 — [[ci] chore: drop file cache and compact](https://github.com/verl-project/verl/pull/8460)
- **作者**: lxb007981  **时间**: 2026-10-30 15:55 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #8459 — [[distillation] perf: initialize teachers concurrently to cut startup time](https://github.com/verl-project/verl/pull/8459)
- **作者**: zgh-99  **时间**: 2026-10-30 14:49 CST
- **摘要**: ### What does this PR do?  Starts all distillation teachers at once instead of one after another, which is what dominates startup when the recipe has many teachers.  `MultiTeacherModelManager` used to build one `TeacherModelManager` per teacher in a plain `for` loop, and each constructor blocked unt…
