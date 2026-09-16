# verl-project/verl — 动态追踪

> 生成时间: 2026-09-16 13:08 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要。由于本期数据仅包含 Pull Requests，未涉及 Issue 和 Release，故重点对 PR 进行分类归纳：

### 📦 Pull Request 活动归纳

近期 PR 主要集中在 **Ascend (NPU) 硬件适配与 CI 修复**、**训练引擎与性能优化**以及**检查点机制改进**三个方面：

**1. 硬件适配与 CI 修复 (Ascend/NPU)**
*   **CI 修复**：[#7877] 修复了 Ascend 镜像升级导致 `reward_model_vllm_ascend` 报错缺失 `__version__` 的问题；[#7875] 通过提前导入 `megatron_adaptor` 修复了 Ascend 测试中 `megatron.core` 的导入错误；[#7876] 修正了 Ascend CI 指南中的工作流文件名拼写错误。
*   **设备检查**：[#7871] 为 Megatron 设置随机种子时，增加了对非 GPU/NPU 设备的可用性检查。

**2. 训练引擎与性能优化**
*   **内存泄漏修复**：[#7874] 修复了 Megatron 后端中 `model_output` 未从自动求导图中 detach 导致的内存泄漏问题（与之前 FSDP 引擎的修复同类）。
*   **性能提升**：[#7873] 移除了权重更新（weight refit）路径中残留的全量垃圾回收机制，以优化性能，这是前期 rollout 权重恢复性能优化的后续跟进。

**3. 检查点机制与测试补充**
*   **检查点修复**：[#7870] 修复了检查点引擎在复用全局集合通信组时，因未提前定义 rank 而错误匹配到非自身创建的 group 的问题。
*   **测试补充**：[#7869] 为增量暂存生命周期和双 GPU GRPO 新增了冒烟测试，确保 CuPy 内存池能被正确释放，并覆盖真实的训练与推演循环。

**4. 文档更新**
*   [#7872] 新增了 Ascend NPU v1 trainer 的单独异步指南，文档说明了 `nccl` 后端会自动解析为 `HCCL` 后端，并记录了对 `mooncake` 后端的支持。

---
*注：本期数据未包含 Release 发版记录与 Issue 动态。*

---

## 🔀 Pull Requests

### #7877 — [[ci] fix: patch nvidia-resiliency-ext missing __version__ in reward_model_vllm_ascend](https://github.com/verl-project/verl/pull/7877)
- **作者**: aass-79  **时间**: 2026-09-16 12:47 CST
- **摘要**: ## What this PR fixes  `reward_model_vllm_ascend` has been failing on main since the Ascend image upgrade in #7604 (same root-cause family as #7838). Example: [run 34981536576](https://github.com/verl-project/verl/actions/runs/34981536576), job fails at step "Running vllm discriminative reward model…

### #7876 — [[doc] fix: correct npu_unit_test.yml to npu_unit_tests.yml in the Ascend CI guide](https://github.com/verl-project/verl/pull/7876)
- **作者**: linhongyu510  **时间**: 2026-09-16 12:05 CST
- **摘要**: ### What does this PR do?  The Ascend NPU CI guide names the unit-test workflow `npu_unit_test.yml` in three places, but the file is [`npu_unit_tests.yml`](https://github.com/verl-project/verl/blob/main/.github/workflows/npu_unit_tests.yml) — plural. The singular path doesn't exist:  ``` .github/wor…

### #7875 — [[ci] fix: fix megatron.core import error on Ascend tests](https://github.com/verl-project/verl/pull/7875)
- **作者**: lxb007981  **时间**: 2026-09-16 11:32 CST
- **摘要**: ### What does this PR do?  `import megatron_adaptor` before the test directly import `megatron.core` to apply the patching first.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (Th…

### #7874 — [[trainer] fix: detach model_output from autograd graph](https://github.com/verl-project/verl/pull/7874)
- **作者**: Dmovic  **时间**: 2026-09-15 21:40 CST
- **摘要**: ### What does this PR do?  Megatron backend follow-up to #6699, which fixed the same class of leak in the FSDP engine.  `MegatronEngineWithLMHead.postprocess_micro_batch_func` returns `model_output` still attached to the autograd graph. That dict becomes `loss_reduced`, which Megatron appends to `fo…

### #7873 — [[perf] fix: remove full gc during weight refit](https://github.com/verl-project/verl/pull/7873)
- **作者**: wuxibin89  **时间**: 2026-09-15 21:26 CST
- **摘要**: ### What does this PR do?  Follow-up to #7864 / #7848, which removed the full Python GC *before* rollout weight resume. This PR removes the remaining full GC on the colocated weight-refit path — but the naive removal is not safe, and finding out why turned up an actual bug.  Torch memory snapshots o…

### #7872 — [[doc] chore: add NPU guide for v1 trainer separate async](https://github.com/verl-project/verl/pull/7872)
- **作者**: dodatboii  **时间**: 2026-09-15 17:20 CST
- **摘要**: ### What does this PR do?  Document that on Ascend NPU, checkpoint_engine.backend=nccl is automatically resolved to the HCCL backend, and that the mooncake backend is supported, with parameter tables and usage examples.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one…

### #7871 — [[hardware] fix: add device_available check for devices other than GPU/NPU](https://github.com/verl-project/verl/pull/7871)
- **作者**: uqyxx  **时间**: 2026-09-15 16:03 CST
- **摘要**: ### What does this PR do?  > Add device_available check for devices other than GPU or NPU when set random seed for megatron  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This wil…

### #7870 — [[misc] fix: checkpoint engine define rank before the collective-group reuse guard](https://github.com/verl-project/verl/pull/7870)
- **作者**: ETOgaosion  **时间**: 2026-09-15 14:40 CST
- **摘要**: ### What does this PR do?  `ray.util.collective` groups are process-global and keyed only by name, so a `NCCLCheckpointEngine` can find a group it never created — for example a second engine constructed in the same process with the default `group_name`. `init_process_group` then takes the "already i…

### #7869 — [[ckpt, test] Cover delta staging lifecycle and two-GPU GRPO smoke](https://github.com/verl-project/verl/pull/7869)
- **作者**: 0z5a  **时间**: 2026-09-15 14:30 CST
- **摘要**: The delta staging lifecycle currently has no focused two-end regression asserting that its CuPy pool is released, and the existing gather/loader probes do not run an actual training-and-rollout loop. This adds two opt-in GPU checks for those boundaries, related to #7060.  The focused test executes r…
