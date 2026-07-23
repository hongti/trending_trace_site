# verl-project/verl — 动态追踪

> 生成时间: 2026-07-23 09:05 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文简洁摘要：

### 🚀 Release（版本发布）
本期无新的 Release 版本发布。

### 🐛 Issue（议题）
本期无新增的 Issue 讨论。

### 🔀 Pull Request（拉取请求）
本期 PR 活动丰富，主要集中在**引入新优化器、推理调度优化及多个关键 Bug 修复**：

**1. 重要新特性**
*   **Megatron 后端支持 Muon 优化器 (#7119, #7120)：** 两项 PR 协作，为 verl 的原生 Megatron 后端引入了 Muon 优化器支持（接入 Megatron-Core 的 TensorParallelMuon），打破了此前仅支持 Adam/SGD 的限制。
*   **新增 KV-cache 感知的请求负载均衡器 (#7115)：** 为 vLLM 的 rollout 服务器新增了基于 KV-cache 感知的路由选项（从 `uni-agent` 迁移而来），旨在优化推理阶段的请求调度效率。

**2. 关键修复**
*   **修复 FSDP 下 PEFT/LoRA 模型检查点保存错误 (#7117)：** 修正了在使用 FSDP 保存检查点时，错误地保存了 PeftModel 包装器代码而非基座模型代码的问题，尤其修复了对 `trust_remote_code` 模型的支持。
*   **修复 Megatron 保存 HF 格式检查点的 strict 参数问题 (#7114)：** 修正了 Megatron 通过桥接保存为 HuggingFace 格式时，未正确应用配置中的 `strict=False` 选项的问题。
*   **修复 SGLang delta 更新作用域泄漏 (#7113)：** 修正了 SGLang 加载器 `delta_sharded` 在处理稀疏更新时，临时 Patch `Tensor.copy_` 作用域为进程全局从而影响其他线程的严重问题，现已正确限定其作用域。

**3. 依赖更新与文档/杂项**
*   **放宽 transformers 依赖版本 (#7118)：** 将 `transformers` 的版本要求从 `<5.12.0` 更新至 `<5.15.0`，以支持最新版本。
*   **文档与 CI 维护 (#7116, #7112, #7111)：** 在 PR 提交规范中增加了避免 Markdown 硬换行的警告；统一将文档中的 Ascend NPU 产品命名从 A5/Atlas 950DT A5 更新为 Ascend 950；并暂时禁用了 Ascend Qwen3.5 35B Megatron vLLM 的 nightly CI 任务。

---

## 🔀 Pull Requests

### #7120 — [[megatron] feat: add Muon optimizer support (expose Megatron-Core TensorParallelMuon)](https://github.com/verl-project/verl/pull/7120)
- **作者**: ISEEKYAN  **时间**: 2026-07-23 03:04 CST
- **摘要**: Add Muon optimizer support to verl native Megatron backend.  verl's native Megatron backend previously had no Muon support (AdamW-type optimizers only). Megatron-Core ships TensorParallelMuon (via the emerging_optimizers package), but verl did not expose it. This PR wires it through so verl users on…

### #7119 — [[megatron] feat: wire up Muon optimizer (config + adapter passthrough)](https://github.com/verl-project/verl/pull/7119)
- **作者**: ISEEKYAN  **时间**: 2026-07-23 02:14 CST
- **摘要**: ### What & why  verl's Megatron backend had **no way to select the Muon optimizer** — the Megatron optimizer path only ever built Adam/SGD. Megatron-Core already ships a tensor-parallel-aware Muon (`TensorParallelMuon`, via the `emerging_optimizers` package) that `megatron.core.optimizer.get_megatro…

### #7118 — [build(deps): update transformers requirement from !=5.6.0,<5.12.0 to !=5.6.0,<5.15.0](https://github.com/verl-project/verl/pull/7118)
- **作者**: dependabot[bot]  **时间**: 2026-07-23 01:33 CST
- **标签**: dependencies, python
- **摘要**: Updates the requirements on [transformers](https://github.com/huggingface/transformers) to permit the latest version. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/transformers/releases">transformers's releases</a>.</em></p> <blockquote> <h2>P…

### #7117 — [[ckpt] fix: save base model's code, not the PeftModel wrapper's, in FSDP checkpoints](https://github.com/verl-project/verl/pull/7117)
- **作者**: ZhiliangWu  **时间**: 2026-07-22 22:26 CST
- **摘要**: ### What does this PR do?  Fix LoRA/PEFT checkpoint saving for `trust_remote_code` models. In `FSDPCheckpointManager.save_checkpoint`, `custom_object_save` copies the source of `type(obj).__module__` to make the checkpoint self-contained, so it must be given the base model. On a PEFT run it was pass…

### #7116 — [[doc] chore: warn against hard-wrapping PR body markdown in pr skill](https://github.com/verl-project/verl/pull/7116)
- **作者**: zhangxin81  **时间**: 2026-07-22 17:35 CST
- **摘要**: ### What does this PR do?  The `pr` skill (`.agent/skills/pr/SKILL.md`, also surfaced via the `.claude` and `.codex` symlinks) tells an agent how to create or update a PR, but it never said how to format the PR body. As a result, bodies were sometimes hand-wrapped at a fixed column width. GitHub ren…

### #7115 — [[rollout, vllm] feat: KV-cache-aware request load balancer](https://github.com/verl-project/verl/pull/7115)
- **作者**: touch869  **时间**: 2026-07-22 17:29 CST
- **摘要**: ### What does this PR do?  Add a **KV-cache-aware request load balancer** as a new routing option for verl's rollout servers, migrated from the standalone `uni-agent` LLM router. The new balancer routes each request by combining prefix-cache hit rates (GPU/CPU/SSD tiers) with live load metrics (KV-c…

### #7114 — [[ckpt, megatron] fix: megatron save checkpoints with strict false](https://github.com/verl-project/verl/pull/7114)
- **作者**: RichardFido  **时间**: 2026-07-22 17:14 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?   What   When saving model weights as HuggingFace format through the Megatron checkpoint manager (_save_model_as_hf_via_bridge), the strict option from checkpoint_config was previously ignored — the   call always used the bridge's default behavior. This commit reads checkpo…

### #7113 — [[rollout, ckpt] fix: scope SGLang delta masked copies](https://github.com/verl-project/verl/pull/7113)
- **作者**: zhangxin81  **时间**: 2026-07-22 17:13 CST
- **摘要**: ### What does this PR do?  The `delta_sharded` SGLang loader applies sparse updates by temporarily patching `Tensor.copy_`: NaN positions mean "keep the current model value."  The patch is process-global while `model.load_weights()` runs. Previously, any same-shaped floating-point copy in that call …

### #7112 — [[doc] chore: update Ascend 950 references](https://github.com/verl-project/verl/pull/7112)
- **作者**: wangdongleix  **时间**: 2026-07-22 13:03 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Update Ascend product naming in NPU and checkpoint-engine documentation from A5/Atlas 950DT A5 to Ascend 950 terminology.  ### Checklist Before Starting  - [ ] Search for similar PRs. Query: https://github.com/verl-project/verl/pulls?q=is%3Apr+is%3Aopen+%22Ascend+950%22 - …

### #7111 — [[doc] chore: update Ascend 950 references and disable nightly test](https://github.com/verl-project/verl/pull/7111)
- **作者**: wangdongleix  **时间**: 2026-07-22 12:58 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Update Ascend 950 product naming across related documentation and disable the Ascend Qwen3.5 35B Megatron vLLM nightly CI job.  ### Checklist Before Starting  - [ ] Search for similar PRs. Query: https://github.com/verl-project/verl/pulls?q=is%3Apr+is%3Aopen+Ascend+950+nig…
