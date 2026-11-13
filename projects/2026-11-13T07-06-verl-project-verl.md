# verl-project/verl — 动态追踪

> 生成时间: 2026-11-13 15:06 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🛑 Issue 动态
- **数学奖励函数评分缺陷 (#8675)**：有用户反馈 `math_dapo` 奖励函数在处理多变量 Minerva 答案时存在逻辑漏洞。评分器仅截取最后一个 `=` 后的内容，导致像 `x=2, y=3` 这样的多变量赋值，在与参考答案 `x=1, y=3` 比较时会被错误地给予满分。

### 🔧 Pull Request 动态
近期的 PR 主要集中在修复核心奖励逻辑、提升 Rollout 性能、增强硬件兼容性及优化 CI 测试：

**1. 核心算法与奖励修复**
- **修复多变量数学评分 (#8676)**：针对上述 Issue #8675，保留了逗号分隔的多重赋值逻辑，修复了多变量答案的误判。
- **新增 Rollout 校正选项 (#8677)**：引入了 `seq_mask_tis`（Rollout IS）和 `seq_mean_abs_exp`（Rollout RS）两个选项，用于处理训练与推理间的 mismatch，以实现对齐 Tunix / MLPerf DeepSWE 的参考规范。
- **新增异步 GRPO MFU 监控 (#8678)**：添加了异步 GRPO 的模型算力利用率仪表化功能。

**2. 硬件兼容与 FSDP 修复**
- **修复 log-prob 内核与静默梯度 Bug (#8672)**：为不支持 flash-attn 交叉熵的设备添加了设备无关的融合 CE 回退方案；同时修复了 FSDP 引擎在 top-k 蒸馏路径中存在的静默梯度计算 Bug。
- **修复分层 LoRA 参数收集 (#8673)**：恢复了 FSDP1 布局下（含扁平化参数或独立包装适配器）的有界分层 LoRA 参数收集功能。

**3. Rollout 与性能优化**
- **保留 vLLM 长度停止原因 (#8680)**：修复了 vLLM rollout 服务器在自然停止和达到长度限制时均报告 `completed` 的问题，现在会准确保留 `finish_reason="length"` 标记。
- **跳过 Hydra 提升 Agent 循环性能 (#8674)**：在构建内置 Agent 循环时跳过了 Hydra 的构造过程，从而优化了单样本生成前的性能开销。

**4. 环境文档与 CI 测试**
- **Ascend 环境文档对齐 (#8679)**：修复并统一了 Ascend 源码安装文档，使其与 Dockerfiles 保持一致。
- **NPU CI 测试扩容 (#8671)**：使 vLLM Ascend 奖励循环 CI 能够在 16 个 NPU 上运行，并支持通过环境变量配置测试设备数。
- **Qwen3-8B PPO 故障隔离诊断 (#8670)**：在 CI 中直接注入 HCCL 诊断信息，用于复现并排查 Qwen3-8B 夜间 PPO/FSDP 测试失败的问题。

### 🚀 Release 动态
本期动态中未包含版本发布信息。

---

## 🐛 Issues

### #8675 — [[reward] math_dapo ignores earlier assignments in multi-variable Minerva answers](https://github.com/verl-project/verl/issues/8675)
- **作者**: shx815  **时间**: 2026-11-13 01:05 CST
- **标签**: bug
- **摘要**: ### System Info  verl source: math_dapo.py from upstream main Platform: macOS (arm64) Python: 3.9.6 Reproduced on CPU by running the scorer directly with synthetic answers. No model or training run was involved.  ### Information  - [ ] The official example scripts - [ ] My own modified scripts  ### …

## 🔀 Pull Requests

### #8680 — [[vllm, rollout] fix: preserve the length stop reason in token outputs](https://github.com/verl-project/verl/pull/8680)
- **作者**: yyDing1  **时间**: 2026-11-13 14:50 CST
- **摘要**: ### What does this PR do?  Preserve vLLM's `finish_reason="length"` in `TokenOutput.stop_reason`. Today the vLLM rollout server reports `completed` both when generation stops naturally and when the output budget cuts it off. A caller translating this value to the Anthropic Messages API consequently …

### #8679 — [[env, doc] fix: align Ascend source installation with Dockerfiles](https://github.com/verl-project/verl/pull/8679)
- **作者**: renjunying2  **时间**: 2026-11-13 12:03 CST
- **摘要**: ### What does this PR do?  [env, doc] fix: align Ascend source installation with Dockerfiles  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{m…

### #8678 — [Add async GRPO MFU instrumentation](https://github.com/verl-project/verl/pull/8678)
- **作者**: zqc214  **时间**: 2026-11-13 10:07 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #8677 — [[algo, doc] feat: seq_mask_tis rollout IS and seq_mean_abs_exp rollout RS (Tunix / MLPerf DeepSWE parity)](https://github.com/verl-project/verl/pull/8677)
- **作者**: jialei777  **时间**: 2026-11-13 02:23 CST
- **摘要**: ### What does this PR do?  Adds two rollout-correction options used by the Tunix / MLPerf DeepSWE reference recipe, so verl runs can match its training-inference mismatch handling:  - **`rollout_is="seq_mask_tis"`** (Tunix `seq-mask-tis`): sequence-masked truncated importance sampling.   - Each toke…

### #8676 — [[reward] fix: preserve multiple assignments in math_dapo](https://github.com/verl-project/verl/pull/8676)
- **作者**: shx815  **时间**: 2026-11-13 01:25 CST
- **摘要**: ### What does this PR do?  Fixes #8675.  The Minerva scorer keeps only the text after the last `=`, so `x=2, y=3` receives full credit against `x=1, y=3`. This patch preserves comma-separated assignments during normalization so the scorer compares both values.  ### Checklist Before Starting  - [x] S…

### #8674 — [[rollout, perf] fix: skip Hydra when constructing built-in agent loops](https://github.com/verl-project/verl/pull/8674)
- **作者**: Hang-get  **时间**: 2026-11-12 23:50 CST
- **摘要**: ### What does this PR do?  Built-in agent loops are constructed once per sample inside `AgentLoopWorker._run_agent_loop`, on the event loop, before that sample can call `generate`. `register()` stores only `{"_target_": "<class path>"}`, but every sample still goes through `hydra.utils.instantiate`,…

### #8673 — [[fsdp] fix: collect layered LoRA parameters from owning units](https://github.com/verl-project/verl/pull/8673)
- **作者**: MrBlue-1996  **时间**: 2026-11-12 23:25 CST
- **摘要**: ### What does this PR do?  Restore bounded layered LoRA collection for FSDP1 layouts with flattened parameters or independently wrapped adapter leaves. The collector currently checks local parameter names, passes local keys to PEFT, and recursively summons nested units. These behaviors can drop adap…

### #8672 — [[fsdp, hardware] fix: logits-preserving log-prob kernels and a device-agnostic fused CE fallback](https://github.com/verl-project/verl/pull/8672)
- **作者**: kahlun  **时间**: 2026-11-12 23:08 CST
- **摘要**: ### What does this PR do?  Adds a fused log-prob fallback for devices without flash-attn's cross-entropy, and fixes a silent gradient bug in the FSDP engine's top-k distillation path.  Fallback: without flash-attn CE (or with `VERL_DISABLE_FLASH_ATTN_CE=1`), `logprobs_from_logits_v2` loops over toke…

### #8671 — [[ci] fix: run vLLM Ascend reward-loop CI on 16 NPUs](https://github.com/verl-project/verl/pull/8671)
- **作者**: lxb007981  **时间**: 2026-11-12 17:23 CST
- **摘要**: ### What does this PR do?  Make test device counts configurable through VERL_TEST_N_GPUS_PER_NODE, preserving the eight-device default. Split standalone rollout and reward pools evenly and scale reward-model samples with the replica count.  ### Checklist Before Starting  - [x] Search for similar PRs…

### #8670 — [[ci] test: isolate Qwen3-8B PPO HCCL initialization diagnostics](https://github.com/verl-project/verl/pull/8670)
- **作者**: fangweii1  **时间**: 2026-11-12 16:05 CST
- **摘要**: ### What does this PR do?  Reproduce the Qwen3-8B nightly PPO/FSDP failure with HCCL diagnostics directly in the GitHub Actions console. The original main-branch failure is https://github.com/verl-project/verl/actions/runs/43065248441/job/130678351649: critic FSDP parameter synchronization calls `hc…
