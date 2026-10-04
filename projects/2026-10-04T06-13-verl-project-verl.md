# verl-project/verl — 动态追踪

> 生成时间: 2026-10-04 14:13 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🚀 Release（发布）
*近期无版本发布动态。*

### 🐛 Issue（议题）
*近期无 Issue 动态。*

### 🔧 Pull Request（拉取请求）
近期共合并/提交了 6 个 PR，主要集中在 **vLLM/LoRA 机制修复**、**评估指标精确化** 以及 **FSDP 包装修复**。重要变更如下：

**1. vLLM 与 LoRA 同步优化（3 项）**
* **[新特性] #8105**：在张量同步期间转发模型的 LoRA adapter hooks。修复了 VERL 内存路径直接调用 vLLM tensor loader 从而绕过磁盘加载器中 adapter 转换钩子的问题。
* **[修复] #8106**：修复了通过 IPC 接收数据后 LoRA 张量被重复克隆拷贝的问题，避免了不必要的内存冗余。
* **[修复] #8101**：修复了多输出轨迹中因 prompt 长度不一致导致的异步奖励输入 padding 未对齐的问题。

**2. Trainer 评估指标精确计算（2 项）**
* **[修复] #8102**：精确计算任意数值评分（包含负分和部分给分）下的 `best@k` 和 `worst@k`，用精确计算替代了原先 1000 次有放回的 bootstrap 估计。
* **[修复] #8104**：改进了 `maj@k` 的计算方式，使用无放回均匀子集的解析均值和标准差替代了原先的 bootstrap 估计。

**3. FSDP 包装修复（1 项）**
* **[修复] #8103**：修复了 FSDP2 模块包装逻辑，改为通过模块的 MRO（方法解析顺序）来正确选择 ABC wrapper，解决了 PEFT 模块被误判替代的问题。

---

## 🔀 Pull Requests

### #8106 — [[rollout, vllm] fix: avoid duplicate LoRA tensor copy after IPC receipt](https://github.com/verl-project/verl/pull/8106)
- **作者**: aoshen02  **时间**: 2026-10-04 12:46 CST
- **摘要**: ## Problem  The LoRA branch of `update_weights_from_ipc` accumulates bucketed tensors until the adapter is complete. Its callback clones each tensor out of the receiver's reused IPC buffer, because vLLM retains the tensors after the callback. `_update_weights` then clones the complete adapter again …

### #8105 — [[rollout, vllm] feat: forward model LoRA adapter hooks during tensor sync](https://github.com/verl-project/verl/pull/8105)
- **作者**: aoshen02  **时间**: 2026-10-04 12:17 CST
- **摘要**: ## Problem  VERL's in-memory `TensorLoRARequest` path calls vLLM's tensor loader directly. That bypasses a model's adapter conversion hook used by the on-disk loader. Also, the rollout server currently overwrites an explicit `engine_kwargs.vllm.max_lora_rank` with a value derived only from the train…

### #8104 — [[trainer] fix: compute exact maj@k without replacement](https://github.com/verl-project/verl/pull/8104)
- **作者**: tongyx361  **时间**: 2026-10-04 12:10 CST
- **摘要**: ### What does this PR do?  Replace validation `maj@k` bootstrap estimates with analytic means and standard deviations over uniformly selected subsets of `k` distinct responses, without replacement. Select uniformly among tied plurality answers, then uniformly among the sampled rows with the selected…

### #8103 — [[fsdp] fix: select ABC wrapper by module MRO](https://github.com/verl-project/verl/pull/8103)
- **作者**: aoshen02  **时间**: 2026-10-04 10:32 CST
- **摘要**: ## Problem  FSDP2 temporarily substitutes `FSDPModule` while wrapping an `ABC`-first `nn.Module`. The previous `isinstance(model, ABC)` condition also selects that substitute for a PEFT module whose MRO puts `nn.Module` before `ABC`. Its class swap then fails with `TypeError: __class__ assignment: o…

### #8102 — [[trainer] fix: compute exact best/worst@k for numeric scores](https://github.com/verl-project/verl/pull/8102)
- **作者**: tongyx361  **时间**: 2026-10-04 02:00 CST
- **摘要**: ### What does this PR do?  Compute validation `best@k` and `worst@k` exactly for arbitrary numeric scores, including negative scores and partial credit. Replace 1000 with-replacement bootstrap iterations with the exact mean and standard deviation of the maximum/minimum over uniformly selected subset…

### #8101 — [[rollout, reward] fix: align padding for asynchronous reward inputs](https://github.com/verl-project/verl/pull/8101)
- **作者**: zupengwang  **时间**: 2026-10-03 18:39 CST
- **摘要**: ### What does this PR do?  Fix asynchronous reward inputs for multi-output trajectories with different prompt lengths. `_compute_score` currently pads concatenated sequences independently from prompts/responses, so the response boundary in `attention_mask` can disagree with `responses`. Default rewa…
