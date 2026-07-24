# verl-project/verl — 动态追踪

> 生成时间: 2026-07-24 09:02 CST

## AI 总结

以下是 **verl-project/verl** 仓库最近动态的中文摘要：

### 🛠 Issue 动态
* **#7130 [vllm] vLLM 0.12 因 `reload_weights` 签名不匹配导致崩溃**
  作者报告在使用 `.[vllm]` 安装 vLLM 0.12.0 后，程序由于 `reload_weights` 函数签名不匹配而崩溃。受影响环境包括 verl 0.9.0.dev、PyTorch 2.9.0+cu128 及 Ray 2。

### 🚀 Pull Request 动态

**✨ 重要新特性**
* **#7135 [megatron] 支持 packed cu_seqlens**：为 Megatron 后端增加显式的 packed-sequence (`cu_seqlens`) 支持，使 THD/remove-padding forwards 能在 packed nested rows 内保留序列边界。
* **#7131 [gemma4] 支持 Flash Attention 2**：允许 Gemma4 加载 `flash_attention_2`。针对其 text tower head dimension 超过 256 导致原 FlashAttention 崩溃的问题，实现了全局层回退到 SDPA 的机制。
* **#7127 [misc] uv 集成**：为 vllm、sglang x fsdp 及 megatron 等组合环境引入了对 `uv` 包管理器的支持。
* **#7129 [docker] 支持社区 CI 镜像构建**：新增构建 vllm/sglang x megatron 社区 CI 镜像的能力。

**🐛 重要修复**
* **#7134 [npu] 纯 CPU 环境下的 NPU 可用性检查修复**：修复了在 Ray head 节点为纯 CPU 而 worker 节点拥有 NPU 的集群中，提交 GRPO 训练任务时在 driver 导入阶段失败的问题。
* **#7133 [trainer] TaskRunner 日志隔离修复**：解决 vLLM server 初始化时重置进程 root logger，导致 `verl.*` INFO/WARNING 日志消失的问题。
* **#7132 [platform] 静默纯 CPU Ray worker 警告**：当确认 Ray task/actor 无 GPU/NPU 资源时，将平台自动检测警告降级为 DEBUG 级别，减少日志干扰。
* **#7128 [rollout, hardware] rollout replica 设备抽象修复**：移除了硬编码的 `"cuda"/"npu"` 设备选择逻辑，替换为通用的 `get_device_name()`，完善了硬件设备抽象层。

**🔧 CI 与发布分支维护**
* **#7125 & #7126 [ci] NPU CI 触发逻辑修复**：修复了在发布分支（v0.8.0 更新）上触发 NPU CI 的逻辑，并强调对发布分支实施更严格的检查。

### 📦 Release 动态
* **近期无新版本发布**。

---

## 🐛 Issues

### #7130 — [[vllm] vLLM 0.12 from .[vllm] crashes on reload_weights signature mismatch](https://github.com/verl-project/verl/issues/7130)
- **作者**: azusa-nami  **时间**: 2026-07-23 18:00 CST
- **摘要**: ### System Info  ```text Installation command: uv pip install -e ".[vllm]" Python: 3.12.13 verl: 0.9.0.dev verl commit: a35908ca3c9632859c58d6a2855d858918ae21dc vLLM: 0.12.0 PyTorch: 2.9.0+cu128 Ray: 2.56.1 Platform: Linux x86_64 Training backend: FSDP2 Rollout backend: vLLM V1 Model: Qwen3-0.6B Tas…

## 🔀 Pull Requests

### #7135 — [[megatron] feat: support packed cu_seqlens](https://github.com/verl-project/verl/pull/7135)
- **作者**: kolehma8  **时间**: 2026-07-24 00:21 CST
- **摘要**: ### What does this PR do?  Adds explicit packed-sequence `cu_seqlens` support to the Megatron backend so THD/remove-padding forwards can preserve sequence boundaries inside packed nested rows instead of treating each row as one attention segment.  This is not duplicating an existing PR. I searched o…

### #7134 — [fix(npu): make NPU availability checks robust in CPU-only environments](https://github.com/verl-project/verl/pull/7134)
- **作者**: chenin-wang  **时间**: 2026-07-24 00:12 CST
- **摘要**: ## Problem  On Ray clusters where the head node is CPU-only but worker nodes have NPUs, submitting a GRPO training job via `ray job submit` fails during the import stage on the driver (head node).  **Root cause**: `torch.npu.is_available()` raises an exception when torch_npu is installed but no NPU …

### #7133 — [[trainer] fix: isolate TaskRunner logs from root logger changes](https://github.com/verl-project/verl/pull/7133)
- **作者**: Begunner  **时间**: 2026-07-23 21:29 CST
- **摘要**: ### What does this PR do?  This PR prevents `verl.*` INFO and WARNING logs in `TaskRunnerV1` from disappearing after vLLM server initialization. The vLLM OpenAI/SageMaker reconfigures the process root logger and its existing handlers to `ERROR` by default. Since verl loggers previously propagated to…

### #7132 — [[platform] fix: quiet CPU-only Ray worker warnings](https://github.com/verl-project/verl/pull/7132)
- **作者**: Wooonster  **时间**: 2026-07-23 19:33 CST
- **摘要**: ### What does this PR do?  Downgrade two platform auto-detection warnings to DEBUG only when verl can safely confirm that the current process is a Ray task or actor with no assigned GPU/NPU resources:  - `No supported accelerator detected ... Falling back to 'nvidia'.` - `Platform 'nvidia' (Platform…

### #7131 — [[gemma4] Support attn_implementation=flash_attention_2 (global layers fall back to SDPA)](https://github.com/verl-project/verl/pull/7131)
- **作者**: nanastassacos  **时间**: 2026-07-23 18:41 CST
- **摘要**: ## What this does  Lets Gemma4 load with `attn_implementation="flash_attention_2"`. Today it crashes with `FlashAttention forward only supports head dimension at most 256`, because the text tower alternates 25 sliding layers at `head_dim=256` with 5 global layers at `head_dim=512`, and flash caps `h…

### #7129 — [[docker] feat: support community CI image building](https://github.com/verl-project/verl/pull/7129)
- **作者**: ETOgaosion  **时间**: 2026-07-23 12:00 CST
- **摘要**: ### What does this PR do?  support community CI image building of vllm/sglang x megatron  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modul…

### #7128 — [[rollout, hardware] fix: fix rollout replica.py for device abstraction](https://github.com/verl-project/verl/pull/7128)
- **作者**: kahlun  **时间**: 2026-07-23 11:33 CST
- **摘要**: ### What does this PR do?  Replace the hardcoded `"cuda"/"npu"` device selection in RolloutReplica with `get_device_name()` (→ `get_platform().device_name`, verl/utils/device.py#L95).  The old expression `"cuda" if not is_torch_npu_available() else "npu"` can only emit "cuda" or "npu", so any other …

### #7127 — [[misc] feat: uv integration](https://github.com/verl-project/verl/pull/7127)
- **作者**: ETOgaosion  **时间**: 2026-07-23 11:21 CST
- **摘要**: ### What does this PR do?  Support uv with vllm, sglang x fsdp, megatron  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fs…

### #7126 — [[ci] fix: tigger npu ci on the release branch (v0.8.0 update)](https://github.com/verl-project/verl/pull/7126)
- **作者**: tardis-key  **时间**: 2026-07-23 11:14 CST
- **摘要**: ### What does this PR do?  as title  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, `sglang`, …

### #7125 — [[ci] fix: tigger npu ci on the release branch](https://github.com/verl-project/verl/pull/7125)
- **作者**: tardis-key  **时间**: 2026-07-23 11:07 CST
- **摘要**: ### What does this PR do?  as title. it needs even stricter checks on the release branch  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modul…
