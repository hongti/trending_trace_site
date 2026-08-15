# verl-project/verl — 动态追踪

> 生成时间: 2026-08-15 09:58 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📌 Issue
- **#7424 提议使用 Liger-Kernel 替换现有 FL-PPO 实现**
  作者建议引入 Liger-Kernel 的融合线性 PPO 等效内核。据测试，该内核可将吞吐量提升约 **3 倍**，并将显存占用降低约 **50%**。

---

### 🔧 Pull Requests
**性能与通信优化（Ulysses QKV）**
- **#7417, #7418, #7419 优化 Ulysses QKV 通信**：将原本 3 次独立的 Q/K/V all-to-all 通信合并为 1 次打包通信（#7417）；新增预打包 QKV API（#7418）；并将该优化扩展支持 Q/KV 头数不同的 GQA/MQA 布局（#7419），大幅提升通信效率。

**异步训练与 Megatron 修复**
- **#7420 实现异步 Megatron 大模型训练可用**：整合此前多项修复，使异步 Megatron 大模型训练正式可用。
- **#7423 修复 NCCL 死锁**：解决了在 Megatron（流水线+专家并行）异步分离式权重同步中出现的交错 NCCL 死锁问题。
- **#7422 修复 SGLang 加载格式被覆盖**：修复了在分离式（异步）训练中，SGLang 的 `dummy` 加载格式被静默覆盖为 `auto` 的问题。
- **#7421 修复 DSA 兼容性**：修复了 MultiLatentAttention (DSA) 与 mcore >= 0.16.2 的兼容性问题，并为没有 `fast_hadamard_transform` 的环境增加了纯 PyTorch Walsh-Hadamard 回退方案。

**新特性与配置修复**
- **#7425 新增逆向 KL Student Top-K 蒸馏损失**：添加了 `reverse_kl_student_topk` 蒸馏模式，支持 FSDP/VeOmni 和 Megatron 后端。
- **#7415 新增可插拔 Rollout 数据后端**：扩展了 V1 trainer 的数据路径，支持将 Mooncake 作为可选后端，实现控制/数据分离及 ReplayBuffer 工作流。
- **#7426 修复 Checkpoint 配置暴露不全**：在 trainer YAML 中补全了 `CheckpointConfig` 的缺失字段，使其能通过命令行正确配置。

---

### 🚀 Release
- **v0.9.0 版本发布**
  **核心亮点**：支持基于 **Megatron-Bridge**（actor/ref）与 **vLLM rollout** 的 **DeepSeek-V4 GRPO 端到端训练**，并引入了 **FP8/MXFP4 权重转换/重排** 支持，进一步增强了大规模模型训练的性能与灵活性。

---

## 🐛 Issues

### #7424 — [Liger fused linear PPO to replace the current FL-PPO implementation.](https://github.com/verl-project/verl/issues/7424)
- **作者**: kolehma8  **时间**: 2026-08-15 00:35 CST
- **摘要**: ### Feature request  Hi team,  We recently added fused linear PPO equivalent kernels to [Liger-Kernel](https://github.com/linkedin/Liger-Kernel). We observe ~3x higher throughput and ~50% reduction in memory usage on both Hopper and Blackwell series GPUs with an fallback option to identical implemen…

## 🔀 Pull Requests

### #7426 — [[cfg, ckpt] fix: expose every CheckpointConfig field in the trainer YAMLs](https://github.com/verl-project/verl/pull/7426)
- **作者**: Nas01010101  **时间**: 2026-08-15 09:13 CST
- **摘要**: ### What does this PR do?  Adds the `CheckpointConfig` fields that exist on the dataclass but are missing from the trainer YAMLs, so they can be set from the command line.  Hydra composes these configs in struct mode, so a key that is not in the backing YAML cannot be set with a plain `a.b.c=value` …

### #7425 — [[distillation] Add reverse KL student top-k loss](https://github.com/verl-project/verl/pull/7425)
- **作者**: YichenZW  **时间**: 2026-08-15 06:03 CST
- **摘要**: ## What  Add `reverse_kl_student_topk` as a top-k distillation loss mode.  This includes: - FSDP/VeOmni reverse KL student top-k support - Megatron reverse KL student top-k support - teacher log-prob lookup on student top-k ids - raw mass metrics before clamp - missing teacher fallback excluded from…

### #7423 — [[worker] fix: NCCL deadlock in async disaggregated weight sync](https://github.com/verl-project/verl/pull/7423)
- **作者**: theely  **时间**: 2026-08-14 22:38 CST
- **摘要**: ### What does this PR do?  Fixes two interleaved-NCCL deadlocks that occur during async disaggregated weight sync with Megatron (pipeline + expert parallelism).  In async training, update_actor and update_weights are dispatched to the same ActorRolloutRefWorker instance via different execution conte…

### #7422 — [[sglang] Fix: preserve dummy load_format in disaggregated SGLang rollout](https://github.com/verl-project/verl/pull/7422)
- **作者**: theely  **时间**: 2026-08-14 22:35 CST
- **摘要**: ### What does this PR do?  Removes a guard in SGLangHttpServer.__init__ that silently overrode load_format = "dummy" to "auto" in non-hybrid rollout modes.  In disaggregated (async) training the rollout workers are intentionally started with load_format: dummy — they skip loading model weights at st…

### #7421 — [[model] fix: DSA for mcore >= 0.16.2: route x/qr kwargs and add PyTorch WHT fa…](https://github.com/verl-project/verl/pull/7421)
- **作者**: theely  **时间**: 2026-08-14 22:33 CST
- **摘要**: ### What does this PR do?  Fixes MultiLatentAttention (DSA variant) compatibility with mcore ≥ 0.16.2 and adds a pure-PyTorch Walsh-Hadamard fallback for environments that don't have the fast_hadamard_transform CUDA extension.  **Background**. verl patches MultiLatentAttention.forward to forward ext…

### #7420 — [Working async megatron large models](https://github.com/verl-project/verl/pull/7420)
- **作者**: theely  **时间**: 2026-08-14 21:51 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #7419 — [[perf, model] feat: pack GQA Ulysses all-to-all](https://github.com/verl-project/verl/pull/7419)
- **作者**: 0z5a  **时间**: 2026-08-14 20:26 CST
- **摘要**: <!-- Title: [perf, model] feat: pack GQA Ulysses all-to-all -->  ### What does this PR do?  Extend packed Ulysses QKV communication to GQA/MQA-style layouts where Q and KV head counts differ. Per-destination Q/K/V chunks are flattened into one payload, redistributed with one autograd-capable `all_to…

### #7418 — [[perf] feat: add pre-packed Ulysses QKV API](https://github.com/verl-project/verl/pull/7418)
- **作者**: 0z5a  **时间**: 2026-08-14 20:15 CST
- **摘要**: <!-- Title: [perf] feat: add pre-packed Ulysses QKV API -->  ### What does this PR do?  Add `gather_packed_qkv_seq_scatter_heads`, an API for producers that already own a dimension-zero packed QKV buffer. It validates the Q/K/V extents, runs one Ulysses collective, and returns split views without an…

### #7417 — [[perf, model] feat: pack Ulysses QKV into one all-to-all](https://github.com/verl-project/verl/pull/7417)
- **作者**: 0z5a  **时间**: 2026-08-14 20:08 CST
- **摘要**: <!-- Title: [perf, model] feat: pack Ulysses QKV into one all-to-all -->  ### What does this PR do?  Replace three independent Ulysses Q/K/V all-to-all calls with one packed collective when all three tensors have the same shape, dtype, and device. Q/K/V are concatenated along dimension zero, redistr…

### #7415 — [[rollout, trainer] feat: add pluggable rollout data backends](https://github.com/verl-project/verl/pull/7415)
- **作者**: zxpdemonio  **时间**: 2026-08-14 17:03 CST
- **摘要**: ### What does this PR do?  Extends the V1 trainer's existing TransferQueue-based rollout data path so Mooncake can be selected as an optional backend. The control/data separation, ReplayBuffer workflow, and incremental production and consumption of rollout fields remain unchanged.  TransferQueue rem…

## 🚀 Releases

### [v0.9.0](https://github.com/verl-project/verl/releases/tag/v0.9.0)
- **作者**: wuxibin89  **时间**: 2026-08-14 15:55 CST
- **摘要**: # v0.9.0  ## Highlights  ### Training  #### Megatron - DeepSeek-V4 GRPO end-to-end with Megatron-Bridge actor/ref, vLLM rollout and FP8/MXFP4 weight transfer (#6473), plus a contiguous context-parallel layout (#7221) and CP fixes that make long-context DeepSeek-V4 runnable (#7297). - [Megatron Lite …
