# verl-project/verl — 动态追踪

> 生成时间: 2026-09-26 13:20 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🐛 Issue 动态
1. **代码奖励计算逻辑错误 (#8021)**：发现在 codecontests 等代码任务中，`prime_code` 评分器在某项测试失败后，仅重新运行前 10 个测试并按 `passed/10` 评分。这导致只要前 10 个测试通过，即使后续测试失败，程序也会错误地获得 1.0 满分。
2. **截断采样导致序列被全部拒绝 (#8020)**：当使用 `top_p`、`top_k` 或 `min_p` 进行概率分布截断时，vLLM 返回的 `processed_logprobs` 会在保留的 token 上重新归一化。这导致 Geo-RS 和 Seq-MIS rollout-correction 预设拒绝了所有生成的序列，作者建议重放采样支持集大小以修复此问题。

### 🚀 PR 动态
1. **新特性：配置 HF 导出精度 (#8019)**：新增配置 Hugging Face 检查点导出数据类型（如 BF16/FP16）的功能。评估和部署通常不需要 FSDP 的完整 FP32 主权重，此更新让用户可以直接导出所需精度，省去了离线转换的步骤。
2. **修复：GRPO 组状态机判定漏洞 (#8024)**：修复了 V1 TransferQueue agent-loop 适配器中的状态机问题。此前如果一个 GRPO 组的所有 session 都返回空输出，会被错误标记为 `finished`（完成），现在将其正确标记为 `failed`（失败）。
3. **修复：适配 TRL >= 1.13 模型加载 (#8023)**：TRL 1.13 版本移除了 `trl.experimental.ppo` 及相关的 `AutoModelForCausalLMWithValueHead` 类，此 PR 修复了由此导致的模型加载失败问题。
4. **修复：代码奖励计分逻辑 (#8022)**：针对 Issue #8021 提交的修复，解决代码评分器在第 10 个测试后对已知错误答案仍计分的问题。
5. **修复：NPU 上 Qwen3-VL 激活值卸载问题 (#8017, #8018)**：修复了在华为昇腾 NPU（Ascend NPU）上训练 Qwen3-VL 模型时激活值卸载失败的问题（关联 Issue #5498），提升了异构硬件兼容性。

### 📦 Release 动态
* 近期暂无版本发布信息。

---

## 🐛 Issues

### #8021 — [[reward] code reward gives 1.0 to a program that fails a test after the first 10](https://github.com/verl-project/verl/issues/8021)
- **作者**: no-hup  **时间**: 2026-09-26 06:02 CST
- **摘要**: for codecontests / apps / codeforces / taco, prime_code runs all tests once and if anything fails re-runs only the first 10 and returns passed/10. the first run already found the failing test but that result is thrown away, so a program right on 1-10 and wrong on 11 gets 1.0 like a correct one. sand…

### #8020 — [With `top_p` / `top_k` / `min_p` truncation, the Geo-RS and Seq-MIS rollout-correction presets reject every sequence; proposal: replay the sampling support size](https://github.com/verl-project/verl/issues/8020)
- **作者**: GuoCheng24  **时间**: 2026-09-26 03:42 CST
- **摘要**: ### What happens  verl asks vLLM for `processed_logprobs` by default (`rollout.logprobs_mode`). With `top_p`, `top_k` or `min_p` truncating the distribution, those log-probs are normalised over the kept set, while `old_log_probs` from the actor are normalised over the full vocabulary. So `old_log_pr…

## 🔀 Pull Requests

### #8024 — [[trainer] fix: a group whose every session returned no output is a failed group, not a finished one](https://github.com/verl-project/verl/pull/8024)
- **作者**: dafu-wu  **时间**: 2026-09-26 12:01 CST
- **摘要**: ### What does this PR do?  Fix a status-machine gap in the V1 TransferQueue agent-loop adapter: a GRPO group whose **every** session returned an empty output list was published as `finished` although it holds zero trajectories. With `algorithm.filter_groups.enable=True` (DAPO dynamic sampling) the `…

### #8023 — [[model] fix: keep model loading working with TRL >= 1.13](https://github.com/verl-project/verl/pull/8023)
- **作者**: GuoCheng24  **时间**: 2026-09-26 09:28 CST
- **摘要**: ### What does this PR do?  TRL 1.13 removed `trl.experimental.ppo`, the last place that shipped `AutoModelForCausalLMWithValueHead`. Every release I checked from 0.29.1 to 1.12.0 has the class there, and 1.13.0 and 1.14.0 do not have it anywhere. `apply_monkey_patch` imports the class whenever TRL i…

### #8022 — [[reward] fix: code reward counts a known wrong answer after test 10](https://github.com/verl-project/verl/pull/8022)
- **作者**: no-hup  **时间**: 2026-09-26 06:02 CST
- **摘要**: ### What does this PR do?  Fixes #8021. for codecontests/apps/codeforces/taco the code scorer re-runs only the first 10 tests after a failure and scores passed/10, so a program right on 1-10 and wrong on 11 got 1.0. now a wrong answer the full run already saw past test 10 is counted (10/11). timeout…

### #8019 — [[ckpt] feat: configure Hugging Face export dtype](https://github.com/verl-project/verl/pull/8019)
- **作者**: lbaolin  **时间**: 2026-09-26 03:17 CST
- **摘要**: ### What does this PR do?  Evaluation and serving usually need BF16 or FP16 Hugging Face weights, not FSDP's full FP32 master weights. Today users must save verl checkpoint shards and run the [offline FSDP merger](https://github.com/verl-project/verl/blob/main/verl/model_merger/fsdp_model_merger.py#…

### #8018 — [[npu] fix Qwen3-VL activation offload deepstack views](https://github.com/verl-project/verl/pull/8018)
- **作者**: LZC-BELIEVER  **时间**: 2026-09-25 16:07 CST
- **摘要**: # [model, hardware] fix: support Qwen3-VL activation offloading on NPU  ## What does this PR do?  Fixes [verl#5498](https://github.com/verl-project/verl/issues/5498), where Qwen3-VL training on Ascend NPU fails when activation offloading is enabled:  ```text RuntimeError: Output 0 of SliceBackward0 …

### #8017 — [[model, hardware] fix: support Qwen3-VL activation offloading on NPU](https://github.com/verl-project/verl/pull/8017)
- **作者**: LZC-BELIEVER  **时间**: 2026-09-25 15:34 CST
- **摘要**: # [model, hardware] fix: support Qwen3-VL activation offloading on NPU  ## What does this PR do?  Fixes [verl#5498](https://github.com/verl-project/verl/issues/5498), where Qwen3-VL training on Ascend NPU fails when activation offloading is enabled:  ```text RuntimeError: Output 0 of SliceBackward0 …
