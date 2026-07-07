# verl-project/verl — 动态追踪

> 生成时间: 2026-07-07 09:11 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📌 Issue 概要
1. **GPU 内存泄漏（Megatron/TE FP8）**：使用 `offload_megatron_model_to_cpu` 时，未清理 Transformer-Engine 的 FP8 weight workspace 缓存，导致 GPU 内存泄漏（#6953）。
2. **Docker 兼容性异常**：Docker 镜像 `verlai/verl:vllm023.dev1` 在 B300/GB300 (sm_103) 硬件上运行崩溃（#6949）。
3. **MTP 训练权重未更新**：开启 MTP 训练后，保存的 MTP 权重始终保持不变，未发生预期更新（#6947）。
4. **特性请求：异步并发度可配置**：希望将 `fully_async` rollout 的最大并发数 `max_concurrent_samples` 从硬编码改为可配置项（#6946）。

### 🛠️ PR 概要
**重要修复：**
- **[megatron] 修复 FP8 内存泄漏**：在 CPU offload 时清理 TE 的 FP8 weight workspace，解决上述 #6953 提出的内存泄漏问题（#6952）。
- **[trainer] 修复 SPMD 与激活检查点兼容性**：修复 `spmd_types` + `selective activation checkpointing` + `未开启 torch compile` 组合下训练崩溃的 Bug（#6950）。
- **[fsdp] 修复 logits 温度缩放**：避免对 view tensor 执行 in-place 的温度缩放操作，防止 FSDP 训练中的潜在异常（#6945）。

**新特性与架构变更：**
- **[rollout/vllm] 引入 KV-cache 感知负载均衡器**：为 rollout 服务器新增基于 KV-cache 的请求路由负载均衡器，优化调度性能（#6940）。
- **[megatron] Megatron Bridge 成为默认**：弃用 vanilla mBridge 并添加弃用警告，正式将 Megatron Bridge 设为默认选项（#6951）。
- **[trainer] 支持 Titan Engine**：为 Titan Engine 添加 README 文档与 CI 测试（#6954）。

**性能与硬件计算修正：**
- **[hardware/perf] 更新 AMD GPU FLOPS**：修正 AMD CDNA GPU（新增 MI350X 等）的 BF16/FP16 峰值算力表，确保 MFU 计算准确（#6942）。
- **[flops_counter] 修正 Qwen3-VL 算力估算**：修复 Qwen-VL head_dim 一致性及 merger token 统计错误，并更新相关测试基准（#6944）。

**文档与配置优化：**
- **[doc] 移除 NPU eager 模式**：由于 eager 模式在 vllm018/ascend 上会导致融合算子回退引发性能劣化，从脚本中移除该配置（#6948）。
- **[doc] 修复 Ascend 文档链接**（#6941）。

### 🚀 Release 概要
*近期暂无新版本发布记录。*

---

## 🐛 Issues

### #6953 — [[megatron] Transformer-Engine FP8 weight workspaces not freed by offload_megatron_model_to_cpu (GPU memory leak on FP8 / native-FP8 checkpoints)](https://github.com/verl-project/verl/issues/6953)
- **作者**: YQ-Wang  **时间**: 2026-07-07 07:30 CST
- **标签**: bug
- **摘要**: ### System Info  VeRL from the official docker image.  ### Information  - [ ] The official example scripts - [x] My own modified scripts  ### Tasks  - [ ] An officially supported task in the `examples` folder (such as GLUE/SQuAD, ...) - [x] My own task or dataset (give details below)  ### Reproducti…

### #6949 — [[docker] verlai/verl:vllm023.dev1 breaks on B300/GB300 (sm_103)](https://github.com/verl-project/verl/issues/6949)
- **作者**: borisfom  **时间**: 2026-07-07 02:17 CST
- **标签**: bug
- **摘要**: ### System Info  VeRL installed from source/trunk.  Environment: verlai/verl:vllm023.dev1 container on B300/GB300   ### Information  - [x] The official example scripts - [ ] My own modified scripts  ### Tasks  - [x] An officially supported task in the `examples` folder (such as GLUE/SQuAD, ...) - [ …

### #6947 — [Has anyone noticed that the saved MTP weights remain unchanged after enabling MTP training?](https://github.com/verl-project/verl/issues/6947)
- **作者**: zwc163  **时间**: 2026-07-06 20:31 CST
- **摘要**: Has anyone noticed that the saved MTP weights remain unchanged after enabling MTP training? I tested it on verl-0.7.1 actor_rollout_ref.model.mtp.detach_encoder=True

### #6946 — [[Feature request] Make fully_async rollouter max_concurrent_samples configurable (follow-up to #6306)](https://github.com/verl-project/verl/issues/6946)
- **作者**: YolandaLyj  **时间**: 2026-07-06 20:00 CST
- **摘要**: ## Feature request  Expose the `fully_async_policy` rollouter's in-flight concurrency cap `max_concurrent_samples` as an optional config field, instead of the current hardcoded `num_replicas * 16`.  Current code ([`fully_async_rollouter.py#L441`](https://github.com/verl-project/verl/blob/9d5188a8793…

## 🔀 Pull Requests

### #6954 — [[trainer] fix: Add readme and CI test for Titan Engine](https://github.com/verl-project/verl/pull/6954)
- **作者**: acisseJZhong  **时间**: 2026-07-07 08:21 CST
- **摘要**: As title

### #6952 — [[megatron] fix: free Transformer-Engine FP8 weight workspaces on CPU offload](https://github.com/verl-project/verl/pull/6952)
- **作者**: alexxu-roblox  **时间**: 2026-07-07 05:11 CST
- **摘要**: Co-authored with @YQ-Wang.  ### What does this PR do?  `offload_megatron_model_to_cpu` frees DDP buffers and walks `.parameters()`, but never clears Transformer-Engine's FP8 weight-workspace caches (`module._fp8_workspaces`). With an FP8 param dtype or a natively-FP8 checkpoint, TE `Linear` / `Group…

### #6951 — [[megatron] chore: deprecate vanilla mBridge and make Megatron Bridge default](https://github.com/verl-project/verl/pull/6951)
- **作者**: HollowMan6  **时间**: 2026-07-07 05:11 CST
- **摘要**: ### What does this PR do?  Add vanilla mBridge deprecation warning and make Megatron Bridge default.  ### Checklist Before Starting  - [X] Search for similar PRs. Paste at least one query link here: ... - [X] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)…

### #6950 — [[trainer] fix: spmd_types and activation checkpointing composability bug ](https://github.com/verl-project/verl/pull/6950)
- **作者**: acisseJZhong  **时间**: 2026-07-07 03:59 CST
- **摘要**: With `spmd_backend=spmd_types` + `activation_checkpoint=selective` + `use_torch_compile=False`, training crashes during the actor backward:  ``` spmd_types.types.SpmdTypeError: assert_type(...) requires an active mesh, but no current mesh is set. ```  ## Root cause spmd.assert_type needs the thread-…

### #6948 — [[doc] fix: no use eager in npu scrip](https://github.com/verl-project/verl/pull/6948)
- **作者**: yyyy2000  **时间**: 2026-07-06 22:17 CST
- **摘要**: ### What does this PR do?   eager模式在vllm018上存在性能劣化，因为vllm-ascend为了易用性，将融合算子split_qkv_rmsnorm_rope_kernel替换成了小算子  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked…

### #6945 — [[fsdp] fix: FSDP logits temperature scaling for view tensors](https://github.com/verl-project/verl/pull/6945)
- **作者**: zjchenn  **时间**: 2026-07-06 19:00 CST
- **摘要**: ### What does this PR do?  This PR updates the FSDP language-model eager logits path to avoid in-place temperature scaling on a squeezed logits view.  `output.logits.squeeze(0)` can return a view. For models whose logits are produced through custom autograd functions, applying `div_()` to that view …

### #6944 — [fix(flops_counter): correct Qwen-VL head_dim consistency and merger token count (#6903)](https://github.com/verl-project/verl/pull/6944)
- **作者**: chethanuk  **时间**: 2026-07-06 18:38 CST
- **摘要**: ### What does this PR do?  Corrects two objective inaccuracies in the Qwen3-VL FLOPs estimators in `verl/utils/flops_counter.py`, and refreshes the affected test goldens. Addresses verl-project/verl#6903 (claims 1 and 4 only).  1. **`head_dim` sourcing (`_estimate_qwen3_vl_flops`)** — read `head_dim…

### #6942 — [[hardware, perf] fix: update AMD GPU FLOPS values](https://github.com/verl-project/verl/pull/6942)
- **作者**: Vivicai1005  **时间**: 2026-07-06 17:35 CST
- **摘要**: Updates the _DEVICE_FLOPS table in verl/utils/flops_counter.py to reflect AMD's official BF16/FP16 dense matrix peak for CDNA GPUs, so MFU is computed correctly on these devices:    - Adds MI350X (2.3 PFLOPS dense) and MI355X (2.5 PFLOPS dense) — previously unlisted, so get_device_flops() returned i…

### #6941 — [[doc] chore: fix ascend doc link](https://github.com/verl-project/verl/pull/6941)
- **作者**: hustmf  **时间**: 2026-07-06 17:25 CST
- **摘要**: ### What does this PR do?  fix ascend doc link  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`,…

### #6940 — [[rollout, vllm] feat: KV-cache-aware request load balancer](https://github.com/verl-project/verl/pull/6940)
- **作者**: zhaizhiqiangA  **时间**: 2026-07-06 17:23 CST
- **摘要**: ### What does this PR do?  Add a **KV-cache-aware request load balancer** as a new routing option for verl's rollout servers, migrated from the standalone `uni-agent` LLM router. The new balancer routes each request by combining prefix-cache hit rates (GPU/CPU/SSD tiers) with live load metrics (KV-c…
