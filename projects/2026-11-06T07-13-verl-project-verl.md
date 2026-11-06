# verl-project/verl — 动态追踪

> 生成时间: 2026-11-06 15:13 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📦 Release
本期动态未提供 Release 相关信息。

### 📝 Issue
本期共有 2 个文档相关 Issue：
1. **RolloutTraceConfig 文档补全** (#8570)：建议在 `RolloutTraceConfig.backend` 的 docstring 中补充 `trackio` 的说明。
2. **PPO 损失函数文档错误** (#8567)：指出 `compute_policy_loss_vanilla` 函数的文档与实际签名不符，文档描述了不存在的参数 `rollout_log_probs`，但实际签名为 `rollout_is_weights`。

### 🔧 Pull Request
本期共有 10 个 PR，涵盖新特性、Bug 修复及文档与 CI 维护：

**✨ 重要新特性**
1. **支持 ROCm (MI355X) 上的 Kimi-K3.1 True On-policy GRPO** (#8574)：新增适用于 ROCm (gfx950) 的 LoRA GRPO 训练配方，确保 vLLM rollout 与 trainer 产生的 log-probs 达到按位级别的完全一致。
2. **FSDP 优化：仅对 Response Tokens 投影 LM Head** (#8571)：为 FSDP 引擎新增 `response_only_lm_head` 选项（默认关闭），避免对整序列 packed tokens 进行不必要的 LM head 投影，从而提升性能。
3. **TPU 配置开关** (#8568)：为 Qwen3.5 TPU 训练配方新增 `ENABLE_PREFIX_CACHING` 环境变量开关，方便控制 Prefix Caching 行为。

**🐛 重要修复**
1. **Gemma4 工具解析器修复** (#8575)：修复了无法正确解析嵌套数组/对象参数的问题，并修复了工具名包含 `-` 或 `.` 时调用被丢弃的 Bug。
2. **PPO Trainer 指标收集修复** (#8565)：修复了当 `use_kl_in_reward=false` 时的 PPO 指标收集问题，改为仅在生成 token_level_rewards 时按需获取，避免破坏原有数据写入行为。
3. **Rollout Log-prob 验证脚本修复** (#8564)：修复 `rollout_logprob_diff.py --refit-rounds N` 脚本的误报问题，确保每一轮 refit 都严格验证阈值和非有限指标。

**📚 文档与 CI 维护**
1. **Ascend 教程与指南大修** (#8573, #8569, #8566)：统一修复 Ascend 相关文档的导航与 Sphinx 交叉引用，明确要求在 Ascend 上使用 SGLang 时 Python 版本需 ≥ 3.12。
2. **Ascend CI 镜像统一** (#8572)：将所有 Ascend CI 镜像的引用统一至相同的 SWR 仓库前缀下，规范化管理。

---

## 🐛 Issues

### #8570 — [[Doc] Add `trackio` to the `RolloutTraceConfig.backend` docstring](https://github.com/verl-project/verl/issues/8570)
- **作者**: gula00  **时间**: 2026-11-06 10:18 CST
- **摘要**: ### Description  The `backend` parameter documentation in [`[rollout_trace.py](https://github.com/verl-project/verl/blob/main/verl/utils/rollout_trace.py)`](https://github.com/verl-project/verl/blob/main/verl/utils/rollout_trace.py) lists `weave`, `mlflow`, and `None`, but omits `trackio`.  The same…

### #8567 — [[doc] compute_policy_loss_vanilla documents rollout_log_probs, but the signature takes rollout_is_weights](https://github.com/verl-project/verl/issues/8567)
- **作者**: lxcxjxhx  **时间**: 2026-11-05 22:52 CST
- **摘要**: ## Summary  `compute_policy_loss_vanilla` is the reference PPO objective, so its docstring is what people read when they adapt the loss. It documents a parameter the function does not take, and describes it with the semantics of the argument that replaced it.  Current `main` (checked 2026-11-05, `e4…

## 🔀 Pull Requests

### #8575 — [[tool] fix: parse nested arguments in Gemma4 tool parser](https://github.com/verl-project/verl/pull/8575)
- **作者**: ZhuLinsen  **时间**: 2026-11-06 14:46 CST
- **摘要**: ### What does this PR do?  `Gemma4ToolParser` mangles array and object arguments, and drops calls whose tool name contains `-` or `.`. It matched arguments with a flat `key:value` regex, so a value was cut at the first `,` or `}`:  ``` call:tag{tags:[<|"|>a<|"|>,<|"|>b<|"|>],ids:[1,2,3]} before: {"t…

### #8574 — [[hardware, recipe, rollout, worker] feat: Kimi-K3.1 true on-policy GRPO on ROCm (MI355X)](https://github.com/verl-project/verl/pull/8574)
- **作者**: indianspeedster  **时间**: 2026-11-06 14:44 CST
- **摘要**: ### What does this PR do?  Adds a ROCm (MI355X, gfx950) recipe for Kimi-K3.1 LoRA GRPO where the vLLM rollout log-probs and the trainer log-probs of the same tokens are **bitwise equal** (`training/rollout_probs_diff_max` = 0.0), plus the verl changes the full model needs to sync weights at all on 2…

### #8573 — [[doc] fix: fix ascend tutorial links and navigation](https://github.com/verl-project/verl/pull/8573)
- **作者**: lxb007981  **时间**: 2026-11-06 14:36 CST
- **摘要**: ### What does this PR do?  Use Sphinx cross-references for documentation links and GitHub URLs for repository files and directories. Generate local tables of contents and enable MyST heading anchors.  Correct Qwen model names and URLs, and add the missing .sh extension to the MXFP8 Megatron example …

### #8572 — [[ci, doc] chore: use base_image for all Ascend CI images](https://github.com/verl-project/verl/pull/8572)
- **作者**: likaizheng666-sketch  **时间**: 2026-11-06 11:26 CST
- **摘要**: ### What does this PR do?  Ascend CI currently uses different SWR registry locations and namespaces. Temporarily unify all verl CI image references under this complete repository prefix:  ```text swr.cn-southwest-2.myhuaweicloud.com/base_image/ascend-ci/verl/verl ```  For example, PR-level vLLM jobs…

### #8571 — [[fsdp] feat: project LM head only for response tokens](https://github.com/verl-project/verl/pull/8571)
- **作者**: hxw12219  **时间**: 2026-11-06 11:01 CST
- **摘要**: ### What does this PR do?  Adds an opt-in FSDP engine option, `response_only_lm_head` (default `false`). RL losses read values only at response tokens, but the engine projects every packed token through the LM head. With the option on, the engine passes the packed rows that predict response tokens t…

### #8569 — [[doc] fix: require Python >=3.12 for SGLang on Ascend](https://github.com/verl-project/verl/pull/8569)
- **作者**: lxb007981  **时间**: 2026-11-06 09:53 CST
- **摘要**: ### What does this PR do?  Update the Ascend installation guides to recommend Python 3.12 and use it in the conda environment creation commands.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {de…

### #8568 — [[hardware, recipe] feat: ENABLE_PREFIX_CACHING knob for the Qwen3.5 TPU recipe](https://github.com/verl-project/verl/pull/8568)
- **作者**: jialei777  **时间**: 2026-11-06 05:13 CST
- **摘要**: ### What does this PR do?  Adds an `ENABLE_PREFIX_CACHING` environment knob (default `False`, so today's behavior is unchanged) to `examples/tpu/grpo/run_qwen3_5_torchtitan.sh`. The recipe used to hard-code `actor_rollout_ref.rollout.enable_prefix_caching=False` and `engine_kwargs.vllm.no_enable_pre…

### #8566 — [[doc] fix: repair navigation in Ascend installation guide](https://github.com/verl-project/verl/pull/8566)
- **作者**: lxb007981  **时间**: 2026-11-05 20:48 CST
- **摘要**: ### What does this PR do?  Replace manual anchors with a generated table of contents and use Sphinx document references for related guides. Link installation scripts to GitHub and use anonymous links to avoid duplicate targets.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at l…

### #8565 — [[trainer] fix: conditionally fetch token-level rewards](https://github.com/verl-project/verl/pull/8565)
- **作者**: ChibiQuest  **时间**: 2026-11-05 17:18 CST
- **摘要**: ### What does this PR do?  Fix PPO metrics collection when algorithm.use_kl_in_reward=false by only fetching token_level_rewards when they are produced. This preserves the original data-writing behavior and prevents missing-field errors without affecting KL-in-reward workflows.  ### Checklist Before…

### #8564 — [[training_utils] fix: validate every rollout log-prob refit round](https://github.com/verl-project/verl/pull/8564)
- **作者**: LZC-BELIEVER  **时间**: 2026-11-05 16:43 CST
- **摘要**: ### What does this PR do?  Fix false passes from `scripts/rollout_logprob_diff.py --refit-rounds N`: an earlier refit can exceed the mismatch threshold or produce non-finite metrics, yet the script returns success when the final round is healthy. The script records every round but previously checked…
