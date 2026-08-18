# verl-project/verl — 动态追踪

> 生成时间: 2026-08-18 10:00 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🚀 Release（版本发布）
近期无新版本发布。

### 🐛 Issue（问题反馈）
近期无新增公开 Issue。

### 🔧 Pull Request（代码合并请求）
近期的 PR 主要集中在**Ascend NPU 生态功能扩展**以及**核心训练/推理逻辑的修复**，具体归纳如下：

**✨ 新特性与功能支持**
*   **Ascend NPU 生态全面扩展**：
    *   新增基于 Ascend NPU 的 **MiMo-7B MTP 训练**支持，采用 Megatron/MindSpeed 训练 + SGLang 推理 + HCCL 权重同步的方案 (#7446)。
    *   新增基于 Ascend NPU 的 **Qwen2.5-1.5B GDPO 训练**示例，采用 Megatron/MBridge (Actor/Ref) + vLLM-Ascend (Rollout) 方案 (#7445)。
    *   文档更新：增加了 Ascend 950 对 kimi-ckpt-engine 的支持说明 (#7452)。
*   **新接口与动态**：
    *   新增对 Trajectory（轨迹）接口的支持 (#7448)。
    *   添加了 verl-Tinker 的最新新闻动态 (#7454)。

**🛠️ 重要修复**
*   **vLLM LoRA 索引错位修复**：修复了堆叠 QKV 投影导致的 vLLM LoRA 同步索引未对齐问题，通过非堆叠权重映射器处理张量并保留 3D LoRA 元数据，防止了张量映射错误 (#7453)。
*   **THD 上下文并行兼容性修复**：修复了多轨迹的 `padding row` 在 THD 上下文并行下无法正常工作的问题（原先合成的 2 token 行低于上下文并行拆分下限） (#7449)。
*   **数据一致性校验强化**：在 `DataProtoItem` 构造时增加不变量校验，并通过 `check_consistency()` 暴露验证逻辑，确保未批处理的 TensorDict 数据一致性 (#7450)。

**🧪 维护与测试**
*   新增 CI 正确性测试及相关的 CI 测试维护 (#7451, #7447)。

---

## 🔀 Pull Requests

### #7454 — [[recipe] chore: Add in news for verl-Tinker](https://github.com/verl-project/verl/pull/7454)
- **作者**: wyettzeng  **时间**: 2026-08-18 09:09 CST
- **摘要**: ### What does this PR do?  Add in news for verl-Tinker  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `…

### #7453 — [[vllm, rollout] fix: vLLM Lora sync index misalignment fix](https://github.com/verl-project/verl/pull/7453)
- **作者**: wyettzeng  **时间**: 2026-08-18 06:13 CST
- **摘要**: ### What does this PR do?  Fix vLLM LoRA index resolution for stacked QKV projections by mapping tensors through the unstacked weight mapper and preserving 3D LoRA metadata. This prevents incorrect tensor selection during adapter loading and ensures weights are resumed before vLLM LoRA synchronizati…

### #7452 — [[doc] chore: Update ckpt engine readme](https://github.com/verl-project/verl/pull/7452)
- **作者**: LeoYao123  **时间**: 2026-08-17 20:53 CST
- **摘要**: ### What does this PR do? add kimi-ckpt-engine support in ascend 950 > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query lin…

### #7451 — [test ci correctness](https://github.com/verl-project/verl/pull/7451)
- **作者**: lxb007981  **时间**: 2026-08-17 20:01 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #7450 — [[misc] fix: validate DataProtoItem consistency](https://github.com/verl-project/verl/pull/7450)
- **作者**: 0z5a  **时间**: 2026-08-17 18:05 CST
- **摘要**: ### What does this PR do?  Validate `DataProtoItem` invariants at construction time and expose the same validation through `check_consistency()`. A `DataProtoItem` now requires an unbatched `TensorDict` (`num_batch_dims == 0`) plus dictionary-valued `non_tensor_batch` and `meta_info` fields.  This r…

### #7449 — [[trainer] fix: make the multi-trajectory padding row survive THD context parallel](https://github.com/verl-project/verl/pull/7449)
- **作者**: gaohongkui  **时间**: 2026-08-17 17:06 CST
- **摘要**: ## What does this PR do?  `construct_minimal_padding_template` builds the synthetic divisibility row as **2 tokens** (one prompt + one response). That is below what the THD context-parallel split can handle.  `preprocess_packed_seqs` pads each row to `align_size = tp * cp * 2` and then hands CP rank…

### #7448 — [Support for trajectory interfaces](https://github.com/verl-project/verl/pull/7448)
- **作者**: tardis-key  **时间**: 2026-08-17 16:46 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #7447 — [Ci test](https://github.com/verl-project/verl/pull/7447)
- **作者**: lxb007981  **时间**: 2026-08-17 15:38 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #7446 — [[megatron, sglang, ckpt, recipe] feat: add MiMo-7B MTP training on Ascend](https://github.com/verl-project/verl/pull/7446)
- **作者**: RordChang  **时间**: 2026-08-17 12:21 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Adds MiMo-7B-RL MTP training on Ascend NPU with Megatron/MindSpeed for training, SGLang for rollout, and direct HCCL weight synchronization between the two sides.  - Adds the registered `sglang_hccl` checkpoint engine for direct Megatron-to-SGLang weight updates. - Pairs e…

### #7445 — [[megatron, vllm, recipe] feat: add Qwen2.5-1.5B GDPO training on Ascend](https://github.com/verl-project/verl/pull/7445)
- **作者**: RordChang  **时间**: 2026-08-17 12:21 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Adds a canonical Qwen2.5-1.5B-Instruct GDPO training example for Ascend NPU, using Megatron/MBridge for the actor and reference workers and vLLM-Ascend for rollout.  - Adds `examples/gdpo_trainer/run_qwen2_5_1_5b_megatron.sh` with environment overrides, auto resume, timest…
