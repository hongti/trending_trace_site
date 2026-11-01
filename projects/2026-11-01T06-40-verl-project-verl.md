# verl-project/verl — 动态追踪

> 生成时间: 2026-11-01 14:40 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 🐛 Issue 摘要
1. **ToolAgentLoop 空响应异常 (#8478)**：指出在首轮生成未产生任何 token 时，`ToolAgentLoop` 会错误地将 Prompt 直接作为 Response 返回。
2. **Megatron 纯推理 MTP 内核限制疑问 (#8475)**：对纯推理 MTP 场景下被强制禁用 fused `linear_cross_entropy` 内核的设计提出疑问，并建议支持两者兼容运行。

### 🔧 Pull Request 摘要
**Bug 修复**
* **#8479**：修复上述 Issue #8478，确保在首轮无 token 生成时，不再将 Prompt 误作为最终响应。
* **#8476**：放宽 Megatron fused-kernel 限制，允许纯推理 MTP 与 fused `linear_cross_entropy` 在 THD 路径上同时启用。
* **#8474**：修复 v1 trainer 结束时的两个问题：确保在训练最后一步能正常触发 `on_train_end()` 回调，并正确停止 train dataloader workers。
* **#8477**：修复 Ascend 平台 fused linear cross-entropy 的 entropy 计算问题，并在单独提交中减少了混合 CE/entropy 反向传播的 workspace 占用。
* **#8471**：修复 VeOmni R3 路由重放问题，通过显式 mask 确保仅重放实际携带 rollout routing 的 token 行。

**性能优化**
* **#8473**：优化 MXFP4 / MXFP8 MoE rollouts 导出过程，将每层专家的压缩改为每次调用仅一次投影，使训练侧 rollout 导出速度提升约 3 倍，且输出结果位级一致。

**CI 与基础设施**
* **#8472**：优化 Ascend Docker 构建 CI，将触发条件从 push 改为 pull_request，PR 中仅进行缓存构建，跳过登录、推送等流程以提升效率。
* **#8470**：启用 Ascend A5 Docker 镜像的每日定时构建（UTC 19:00）。

### 🚀 Release 摘要
* 近期暂无正式版本发布。

---

## 🐛 Issues

### #8478 — [[rollout] ToolAgentLoop returns the prompt as the response when the first turn generates no tokens](https://github.com/verl-project/verl/issues/8478)
- **作者**: arun-elr  **时间**: 2026-11-01 09:18 CST
- **摘要**: ### System Info  verl main @ 8df5baee, CPU repro (no GPU needed), Python 3.12.  ### Information  - [ ] The official example scripts - [x] My own modified scripts  ### Tasks  - [ ] An officially supported task in the `examples` folder (such as GLUE/SQuAD, ...) - [x] My own task or dataset (give detai…

### #8475 — [[megatron] Why does inference-only MTP disable fused linear_cross_entropy?](https://github.com/verl-project/verl/issues/8475)
- **作者**: RitianD  **时间**: 2026-10-31 19:52 CST
- **摘要**: ### Feature request  Why does `_maybe_enable_fused_kernels` refuse fused log-prob kernels whenever `mtp.enable` is set, including inference-only MTP?  Please consider allowing this combination:  - `actor_rollout_ref.model.mtp.enable=True` - `actor_rollout_ref.model.mtp.enable_rollout=True` - `actor_…

## 🔀 Pull Requests

### #8479 — [[rollout] fix: keep the prompt out of an empty tool agent loop response](https://github.com/verl-project/verl/pull/8479)
- **作者**: arun-elr  **时间**: 2026-11-01 09:18 CST
- **摘要**: ### What does this PR do?  Fixes `ToolAgentLoop.run` returning the prompt as the response when the first turn generates no tokens. Fixes #8478.  The final split is `response_ids = agent_data.prompt_ids[-len(agent_data.response_mask):]`. If the first generation is empty, `response_mask` is `[]` and t…

### #8477 — [[hardware, training_utils] fix: compute real Ascend CE entropy and tile backward](https://github.com/verl-project/verl/pull/8477)
- **作者**: Gxj230958  **时间**: 2026-10-31 23:42 CST
- **摘要**: ### What does this PR do?  Fix the entropy contract of Ascend fused linear cross-entropy, then reduce its mixed CE/entropy backward workspace in a separate commit. This extends #8143 and relates to #7950; it does not replace the existing CANN CE operators or introduce a new GEMM kernel.  With the te…

### #8476 — [[megatron] fix: allow inference MTP alongside fused log-prob kernels](https://github.com/verl-project/verl/pull/8476)
- **作者**: RitianD  **时间**: 2026-10-31 19:53 CST
- **摘要**: ### What does this PR do?  Relaxes the Megatron fused-kernel gate so inference-only MTP (`mtp.enable=True`, `mtp.enable_train=False`) can run together with fused `linear_cross_entropy` on the THD path. Training MTP and the bshd path stay unfused. Related to #8475.  This is narrower than draft #7826.…

### #8474 — [[trainer] fix: end v1 fit() through on_train_end() and stop the train dataloader workers](https://github.com/verl-project/verl/pull/8474)
- **作者**: 0z5a  **时间**: 2026-10-31 19:27 CST
- **摘要**: ### What does this PR do?  Fixes two problems at the end of the v1 trainer's `fit()`:  - When training reaches its last step, `fit()` returns from inside the loop. On that path `on_train_end()` never runs, so the hook only fires when the loop ends some other way (the epochs run out first). No class …

### #8473 — [[rollout, perf] fix: compress the MXFP4 / MXFP8 rollout experts of a layer one projection per call](https://github.com/verl-project/verl/pull/8473)
- **作者**: 0z5a  **时间**: 2026-10-31 19:07 CST
- **摘要**: ### What does this PR do?  Makes the trainer-side rollout exporter about 3× faster for MXFP4 / MXFP8 MoE rollouts, with bit-identical output.  `CompressedTensorsWeightQuantizer` compressed every expert weight on its own. On the GPU each call is launch-bound: on a B300 one call takes about 1.3 ms for…

### #8472 — [[ci] chore: validate Ascend Docker builds on pull requests](https://github.com/verl-project/verl/pull/8472)
- **作者**: lxb007981  **时间**: 2026-10-31 17:46 CST
- **摘要**: ### What does this PR do?  Replace push triggers with pull_request triggers for A2, A3, and A5. Use cache-only builds on PRs and skip registry login, image pushes, digest handling, and manifest publishing.  Preserve scheduled and manual builds with publishing enabled.  ### Checklist Before Starting …

### #8471 — [[veomni, rollout, training_utils] fix: replay only recorded routing rows via an explicit mask](https://github.com/verl-project/verl/pull/8471)
- **作者**: horcham  **时间**: 2026-10-31 15:09 CST
- **摘要**: ### What does this PR do?  Fix VeOmni R3 (rollout router replay) so it replays **only the token rows that actually carry rollout routing**, using an explicit `routed_experts_mask` produced by whoever zero-fills `routed_experts`, instead of inferring it from duplicate top-k expert ids.  The current i…

### #8470 — [[ci] chore: enable daily Ascend A5 Docker builds](https://github.com/verl-project/verl/pull/8470)
- **作者**: lxb007981  **时间**: 2026-10-31 15:02 CST
- **摘要**: ### What does this PR do?  Enable the existing cron schedule to build Ascend A5 Docker images daily at 19:00 UTC, matching the Ascend A3 workflow.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {…
