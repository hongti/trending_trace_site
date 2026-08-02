# verl-project/verl — 动态追踪

> 生成时间: 2026-08-02 09:05 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近的动态摘要：

### 📌 Issue
近期无显著 Issue 动态。

### 📌 Pull Request
近期 PR 主要集中在**修复核心算法 Bug**、**DeepSeek V4 适配**以及**Megatron 后端并行能力扩展**：

*   **算法与 Agent 修复**
    *   **#7225**：修复了蒸馏损失（distillation loss）的微批次归一化问题，解决了 issue #7200 中的报错。
    *   **#7222**：修复 Agent Loop 中跨轮次路由专家丢失的问题，确保多轮对话中连续路由对齐，仅追加新覆盖的后缀。
*   **DeepSeek V4 与 vLLM 增强**
    *   **#7224**：增强 DeepSeek V4 的 FP8/FP4 线性层和 MoE 权重重置。修复了 FP8 scale 重置导致 rollout 权重同步出错的 Bug，并将 FP8 和 MXFP4 的量化权重同步代码拆分，不再共用文件。
*   **Megatron 并行能力扩展**
    *   **#7223**：为 Megatron 的 steady delta export（`delta_sharded` 检查点引擎）增加流水线并行（PP/VPP）支持，移除了此前 PP=1 的断言限制。
    *   **#7221**：为 DeepSeek V4 在 Megatron 后端新增连续上下文并行（contiguous CP）序列布局，与原有的 zigzag 布局并存。

### 📌 Release
近期无新版本发布动态。

---

## 🔀 Pull Requests

### #7225 — [[algo] fix: micro-batch normalization for distillation loss](https://github.com/verl-project/verl/pull/7225)
- **作者**: JacobHelwig  **时间**: 2026-08-02 05:20 CST
- **摘要**: ### What does this PR do?  Applies fix proposed by @wuxibin89 for issue #7200 found by @yph22    ### Test  <img width="454" height="276" alt="image" src="https://github.com/user-attachments/assets/b7954153-39e8-42d7-b7c4-0c11176c46e6" />  - Ran Qwen3-0.6B student w/ Qwen3-1.7B teacher on gsm8k for a…

### #7224 — [[vllm] feat: enhance DeepSeek V4 fp8/fp4 linear and moe weight refit](https://github.com/verl-project/verl/pull/7224)
- **作者**: wuxibin89  **时间**: 2026-08-01 21:08 CST
- **摘要**: ### What does this PR do?  Repairs FP8 scale refit for rollout weight sync, and splits the quantized weight-sync code by quantization scheme so the FP8 and MXFP4 paths stop sharing one file.  The bug: `_copy_param_subclass_attrs` copied every attribute off the source parameter, including *bound meth…

### #7223 — [[megatron] PP/VPP support for the steady delta export](https://github.com/verl-project/verl/pull/7223)
- **作者**: gxlvera  **时间**: 2026-08-01 11:32 CST
- **摘要**: Adds pipeline-parallel (PP/VPP) support to the Megatron steady delta export of the `delta_sharded` checkpoint engine (follow-up to #7060; PP support was previously guarded by a PP=1 assert).  ## Design  The bridge's `build_conversion_tasks` already maintains a **global parameter directory**: it allg…

### #7222 — [[agent-loop] fix: preserve routed experts across turns](https://github.com/verl-project/verl/pull/7222)
- **作者**: YAO-001  **时间**: 2026-08-01 11:32 CST
- **摘要**: ## Summary  - preserve routed-expert rows captured by earlier tool-agent turns - append only the newly covered suffix from each later full-sequence routing snapshot - keep routing aligned when Continuous Token removes already-covered prefix tokens - add focused CPU regression coverage for NumPy, Tor…

### #7221 — [[megatron] feat: support contiguous context-parallel layout for DeepSeek V4](https://github.com/verl-project/verl/pull/7221)
- **作者**: ISEEKYAN  **时间**: 2026-08-01 10:02 CST
- **摘要**: ### What does this PR do?  Adds a `contiguous` context-parallel (CP) sequence layout to the Megatron backend, alongside the existing `zigzag` layout, and wires it up for DeepSeek V4 (`experimental_attention_variant == "dsv4_hybrid"`).  DeepSeek V4's attention expects each CP rank to own **one consec…
