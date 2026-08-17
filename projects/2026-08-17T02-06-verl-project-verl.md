# verl-project/verl — 动态追踪

> 生成时间: 2026-08-17 10:06 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🐛 Issue
*   **#7438**：`delta_sharded` 检查点引擎与 `one_step_off_policy` 在 Qwen3.5 模型上配合使用时报错。原因在于多模态处理器与 `ContinuousToken` 家族不匹配，影响了相关实验功能的正常运行。

### 🔀 Pull Request
**1. 新增昇腾 训练支持与配方（#7441, #7442）**
*   **#7442**：新增 Qwen2.5-0.5B-Instruct **CISPO** 训练示例，采用 Megatron（actor/reference 训练）+ vLLM-Ascend（rollout）在昇腾 NPU 上运行。
*   **#7441**：新增 Qwen2.5-7B **GMPO** 训练示例，同样基于 Megatron + vLLM-Ascend 架构，并引入了可选的基于文件的 mmap 后端以优化处理。

**2. 修复 Reward 计算相关问题（#7440, #7437）**
*   **#7440**：修复数学题评分超长问题。将 math-DAPO 的答案截断限制在最后 300 个字符内，与文档中 MATH-500 最大 159 字符的长度预算保持一致，防止溢出。
*   **#7437**：修复符号等式判断的超时失效问题。修正了 `symbolic_equal()` 错误地将 `timeout_limit()` 当作上下文管理器（实际应为装饰器工厂）的问题，确保解析、化简和数值比较操作能正确触发超时限制。

**3. 修复内核计算稳定性（#7439）**
*   **#7439**：修复融合熵核在处理全负数 logits 时的不稳定问题。改用负无穷大作为 max-reduction 的恒等值和掩码值，以确保移位不变性，并新增了单设备及张量并行（TP）的回归测试。

### 🚀 Release
*   近期无新版本发布。

---

## 🐛 Issues

### #7438 — [delta_sharded + one_step_off_policy fails with Qwen3.5 due to multimodal processor / ContinuousToken family mismatch](https://github.com/verl-project/verl/issues/7438)
- **作者**: yuxuan-z19  **时间**: 2026-08-16 21:15 CST
- **标签**: bug
- **摘要**: ### System Info  Hi, thanks for the great work on verl.  I encountered an issue when trying to use the experimental delta_sharded checkpoint engine together with `one_step_off_policy.main_ppo` on Qwen3.5 models.  The same model/config works with the standard `verl.trainer.main_ppo` FSDP training scr…

## 🔀 Pull Requests

### #7442 — [[megatron, vllm, recipe] feat: add Qwen2.5-0.5B CISPO training on Ascend](https://github.com/verl-project/verl/pull/7442)
- **作者**: RordChang  **时间**: 2026-08-17 02:00 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Adds a Qwen2.5-0.5B-Instruct CISPO training example using Megatron for actor/reference training and vLLM-Ascend for rollout on Ascend NPUs.  - Adds a canonical Megatron + vLLM-Ascend launcher with environment-variable overrides, automatic checkpoint resume, and timestamped…

### #7441 — [[megatron, vllm, recipe] feat: add Qwen2.5-7B GMPO training on Ascend](https://github.com/verl-project/verl/pull/7441)
- **作者**: RordChang  **时间**: 2026-08-16 23:31 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Adds a Qwen2.5-7B GMPO training example using Megatron for actor/reference training and vLLM-Ascend for rollout on Ascend NPUs.  - Adds an optional file-backed mmap backend for bucketed vLLM weight transfer when `VERL_WEIGHT_TRANSFER_DIR` is set, avoiding `/dev/shm` capaci…

### #7440 — [[reward] fix: keep boxed answers within the documented length budget](https://github.com/verl-project/verl/pull/7440)
- **作者**: JimmyWang0417  **时间**: 2026-08-16 23:20 CST
- **摘要**: ### What does this PR do?  `compute_score()` already limits math-DAPO solutions to their final 300 characters, matching the documented maximum MATH-500 answer length of 159 characters. `is_correct_strict_box()` then applies another 100-character slice, which can remove the opening `\boxed{` marker f…

### #7439 — [fix(kernel): stabilize fused entropy for negative logits](https://github.com/verl-project/verl/pull/7439)
- **作者**: WangBD2023hdu  **时间**: 2026-08-16 21:29 CST
- **摘要**: Use negative infinity as the max-reduction identity and masked value so all-negative logits remain shift invariant. Add single-device and tensor-parallel regression coverage.  ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHu…

### #7437 — [[reward] fix: enforce symbolic equality timeouts](https://github.com/verl-project/verl/pull/7437)
- **作者**: JimmyWang0417  **时间**: 2026-08-16 19:55 CST
- **摘要**: ### What does this PR do?  `symbolic_equal()` treats `timeout_limit()` as a context manager even though it is a decorator factory. Each parsing, simplification, and numerical comparison attempt therefore raises before the guarded function runs; the surrounding exception handlers silently convert tho…
