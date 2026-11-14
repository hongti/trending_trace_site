# verl-project/verl — 动态追踪

> 生成时间: 2026-11-14 14:55 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📦 Pull Requests (PR)
本期共有 7 个 PR 更新，主要集中在硬件适配、核心算法修复及性能优化：

**1. 新特性与硬件支持**
*   **#8686 [Ascend PD 分离]**：通过 `MooncakeConnectorV1` 启用华为 Ascend 平台的预填充/解码分离，基于现有的单节点 1P+ND 和非对称 TP 实现进行扩展。
*   **#8683 [TPU v7x 性能优化]**：为 Qwen3.5-35B-A3B 模型提供更快的 TPU v7x GRPO 训练默认配置，并为参考模型引入静态形状优化（注：该 PR 当前被 `torchtitan#142` 阻塞，暂不可合并）。

**2. 重要 Bug 修复**
*   **#8681 [MTP 权重同步]**：修复了 vLLM ModelRunner V2 下的 MTP（多 Token 预测）权重同步问题，确保 V1 和 V2 版本的 speculator/drafter 模型均能正常进行权重加载和更新。
*   **#8685 [依赖升级修复]**：将 `flash-linear-attention` 升级至 0.6.0，修复了旧版本在 Blackwell 架构与 Triton < 3.8 环境下 GDN 反向内核的竞态问题。
*   **#8684 [PPO 逻辑修复]**：修复 `FusedLinearForPPO`（默认融合后端）中 log-probs 与 rollout 结果不一致的问题，现在会在升精度至 fp32 之前正确地用温度值去除 logits。
*   **#8680 [vLLM 停止原因保留]**：修复 vLLM rollout 中停止原因丢失的问题，现在能正确区分自然停止与达到长度上限，并在 `TokenOutput` 中保留 `finish_reason="length"`。

**3. 文档与维护**
*   **#8682**：更新并改进了 `AGENTS.md` 文档。

### 🐛 Issues
*   近期无公开的 Issue 动态。

### 🚀 Releases
*   近期无新版本发布。

---

## 🔀 Pull Requests

### #8686 — [[rollout, vllm] feat: enable Ascend PD with MooncakeConnectorV1](https://github.com/verl-project/verl/pull/8686)
- **作者**: Mengyuyang  **时间**: 2026-11-14 14:44 CST
- **摘要**: ### What does this PR do?  Enable Ascend prefill/decode disaggregation through `MooncakeConnectorV1` on top of verl's existing single-node 1P+ND and asymmetric-TP implementation. The current base rejects NPU at launch and configures GPU connectors. This change supplies the Ascend connector/port conf…

### #8685 — [[fsdp, megatron, veomni] fix: bump flash-linear-attention to 0.6.0](https://github.com/verl-project/verl/pull/8685)
- **作者**: JonSnow1807  **时间**: 2026-11-14 12:55 CST
- **摘要**: ### What does this PR do?  verl pins `flash-linear-attention==0.5.2`, whose GDN backward kernel `chunk_bwd_kernel_dv_local` offers 8-warp configs that race on Blackwell with Triton < 3.8 (fla-org/flash-linear-attention#1228, fixed by fla-org/flash-linear-attention#1229 in 0.6.0). On a B300 with verl…

### #8684 — [[training_utils] fix: divide fp32 logits by the temperature in FusedLinearForPPO](https://github.com/verl-project/verl/pull/8684)
- **作者**: JonSnow1807  **时间**: 2026-11-14 12:02 CST
- **摘要**: ### What does this PR do?  The torch backend of `FusedLinearForPPO` (the default fused backend) divides the bf16 logits by the temperature before upcasting to fp32, so log-probs differ from the rollout sampler’s formula (fp32 logits / T) by up to 0.054 at T=0.7, and the returned entropy is bf16. Thi…

### #8683 — [[hardware, recipe, perf] feat: faster Qwen3.5-35B-A3B defaults for the TPU v7x GRPO recipe, static shapes for the reference model (blocked on torchtitan#142)](https://github.com/verl-project/verl/pull/8683)
- **作者**: wenjung2007  **时间**: 2026-11-14 03:28 CST
- **摘要**: > [!CAUTION] > **Blocked on google-pytorch/torchtitan#142.** Do not merge until torchtitan#142 is checked in and `TORCHTITAN_COMMIT` in `examples/tpu/grpo/Dockerfile` points at a commit that contains it.  ### What does this PR do?  Makes `examples/tpu/grpo/run_qwen3_5_torchtitan.sh` default to the f…

### #8682 — [[doc] chore: update and improve AGENTS.md](https://github.com/verl-project/verl/pull/8682)
- **作者**: Inference1  **时间**: 2026-11-13 21:19 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  The existing `AGENTS.md` is somewhat rudimentary and basic, so it needs to be updated and improved.  The root `AGENTS.md` introduce…

### #8681 — [[vllm] fix: support MTP weight sync with vllm ModelRunner V2](https://github.com/verl-project/verl/pull/8681)
- **作者**: lxb007981  **时间**: 2026-11-13 16:39 CST
- **摘要**: ### What does this PR do?  Resolve both V1 drafter.model and V2 speculator.model so MTP weight loading, buffer updates, and post-processing work with both vLLM ModelRunner versions. Apply the same fallback to quantized weight loading and parameterize the buffer-sync test over both owners.  ### Check…

### #8680 — [[vllm, rollout] fix: preserve the length stop reason in token outputs](https://github.com/verl-project/verl/pull/8680)
- **作者**: yyDing1  **时间**: 2026-11-13 14:50 CST
- **摘要**: ### What does this PR do?  Preserve vLLM's `finish_reason="length"` in `TokenOutput.stop_reason`. Today the vLLM rollout server reports `completed` both when generation stops naturally and when the output budget cuts it off. A caller translating this value to the Anthropic Messages API consequently …
