# verl-project/verl — 动态追踪

> 生成时间: 2026-07-25 09:04 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📌 Issue 动态
*   **#7141 提出新评估指标提案**：建议增加名为 **REFUTE** 的评估指针，用于衡量模型在科学批判上的诚实度与校准能力。该指标基于近期论文摘要，无需人工评判，采用 Brier/ECE 评分，且能有效区分批判技能与校准度。
*   **#7138 FSDP 主机名解析 Bug**：在 verl v0.5.0 版本中启动 dapo 训练脚本并使用 FSDP 框架时，出现 FSDP 无法解析主机名的错误，导致多卡通信挂起（暂停），影响训练正常进行。

### 🔧 PR 动态（突出重要修复与新特性）
**1. 硬件与底层支持扩展**
*   **#7142 【新特性】支持 Ascend NPU**：为昇腾 NPU 添加了 `megatron_adaptor+te-npu` 引擎后端，作为 MindSpeed 路径的替代方案，注册策略为 `megatron_adaptor`，复用了通用 Megatron 流程。
*   **#7140 【文档更新】ROCm 7.14 支持**：更新了 ROCm 7.14 的支持文档，补充了可复现的 Docker 及 Dockerfile 示例。

**2. 关键框架与模型修复/改进**
*   **#7136 【重要修复】vLLM 0.20.x FP8 权重同步**：修复了在 vLLM 0.20.x 上进行 FP8 rollout 权重重同步的 Bug。通过驱动 vLLM 原生的逐层重载生命周期，配合 verl 的分桶 IPC 机制，解决了权重更新问题。
*   **#7137 【重要改进】Ulysses FA-varlen 路径优化**：在 Ulysses Flash Attention varlen 路径中显式传递 `cu_seqlens`，**消除了对 `position_ids` 的依赖**。此改进直接支持了如 Qwen3.5 等使用 3D `position_ids`（mrope）模型的训练。
*   **#7139 【重要修复】修复 sglang NCCL 缓冲区竞争**：修复了 `_compact_for_bucket` 中的 NCCL 缓冲区竞态条件。当 bucket 大小恰好等于 tensor 占用空间时，原有启发式方法无法检测 NCCL recv_buf 视图 tensor，导致其逃逸克隆操作引发竞争；现通过 `_base` guard 机制予以解决。

### 🚀 Release 动态
*   近期**无新版本发布**动态。

---

## 🐛 Issues

### #7141 — [Related eval: REFUTE scientific critique + calibration](https://github.com/verl-project/verl/issues/7141)
- **作者**: connerlambden  **时间**: 2026-07-25 07:51 CST
- **摘要**: ## Proposal  Related eval pointer: **REFUTE** measures scientific critique honesty + calibration on recent paper summaries (judge-free; Brier/ECE). Critique skill and calibration often come apart.  - https://bgpt.pro/refute - https://github.com/connerlambden/refute-inspect - https://bio.tools/bgpt-r…

### #7138 — [After the dapo training job is started, FSDP cannot parse the host name. As a result, the FSDP communication is suspended.](https://github.com/verl-project/verl/issues/7138)
- **作者**: iiiiice-seabird  **时间**: 2026-07-24 16:09 CST
- **标签**: bug
- **摘要**: ### System Info  When starting the dapo training script in version verl v0.5.0 and using the FSDP training framework, an error occurs indicating that FSCP cannot resolve the hostname, and the FSDP communication becomes stuck.  (WorkerDict pid=16904, ip=10.17.138.10) [W331 04:59:41.148164733 socket.c…

## 🔀 Pull Requests

### #7142 — [[megatron] feat(npu): add megatron_adaptor+te-npu backend for Ascend NPU](https://github.com/verl-project/verl/pull/7142)
- **作者**: Bruce-rl-hw  **时间**: 2026-07-25 08:10 CST
- **摘要**: ### What does this PR do?  Adds a megatron_adaptor engine backend for Ascend NPU, as an alternative to the MindSpeed path. It registers under strategy="megatron_adaptor" and reuses the generic MegatronEngineWithLMHead/ValueHead (a thin ~80-line shell), applying the Ascend NPU patches once at import …

### #7140 — [[docker] chore: Rocm714 doc update](https://github.com/verl-project/verl/pull/7140)
- **作者**: mingjielu  **时间**: 2026-07-24 18:51 CST
- **摘要**: ### What does this PR do?  > Update the ROCm 7.14 support documentation to provide reproducible Docker and Dockerfile examples.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here:  https://github.com/verl-project/verl/pull/6388 - [x] Format the PR title …

### #7139 — [[sglang] fix: use _base guard in _compact_for_bucket to prevent NCCL buffer race](https://github.com/verl-project/verl/pull/7139)
- **作者**: zpltys  **时间**: 2026-07-24 16:31 CST
- **摘要**: The original nbytes == numel*element_size heuristic fails to detect NCCL recv_buf view tensors when the bucket size (e.g. 1024 MB) exactly equals the tensor footprint.  Such tensors escape the clone, remain views into the recv_buf, and get silently overwritten by the next NCCL broadcast before the C…

### #7137 — [[model] Pass explicit cu_seqlens through the Ulysses FA-varlen path, eliminating position_ids dependency](https://github.com/verl-project/verl/pull/7137)
- **作者**: Zhanli-Li  **时间**: 2026-07-24 15:18 CST
- **摘要**: # [model] Pass explicit cu_seqlens through the Ulysses FA-varlen path, eliminating position_ids dependency  ## Motivation  Training mrope models such as Qwen3.5 (3D `position_ids` of shape `(mrope_dim, bsz, seq)`) with Ulysses SP + FlashAttention varlen crashes in the monkey-patched `_ulysses_flash_…

### #7136 — [[rollout, vllm] fix: FP8 rollout weight resync on vLLM 0.20.x via native layerwise reload lifecycle](https://github.com/verl-project/verl/pull/7136)
- **作者**: KeitaW  **时间**: 2026-07-24 09:28 CST
- **摘要**: ### What does this PR do?  Fixes FP8 rollout weight resync (`actor_rollout_ref.rollout.quantization=fp8`) on vLLM 0.20.x by driving vLLM's native layerwise reload lifecycle around verl's bucketed IPC weight sync.  Related to #5683 (a failure of the same FP8-resync family on a newer pin). This PR doe…
