# verl-project/verl — 动态追踪

> 生成时间: 2026-10-25 13:08 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🐛 Issue
* **#8362 [rollout, sglang] SGLang rollout 忽略 `logprobs_mode` 配置**
  指出 SGLang rollout 未正确处理 `rollout.logprobs_mode` 参数，导致在相同配置下，其返回的对数概率 与 vLLM 后端存在差异。

### 🔧 Pull Request
* **#8363 [sglang, rollout] 修复 SGLang rollout 忽略对数概率模式的问题**
  修复了上述 #8362 问题。使 SGLang rollout 能够正确遵循 `rollout.logprobs_mode` 配置，而不是始终返回经过温度缩放的全词汇表分布的对数概率。
* **#8361 [megatron, doc] 修复 QAT 模式下打包 THD 序列的因果掩码问题**
  修复了当序列被打包（`use_remove_padding=True`）且上下文并行度为 1 时，Megatron QAT 在首次前向传播中因注意力掩码断言而失败的问题。
* **#8360 [reward] 在 prime_code 中支持 APPS/TACO 列表格式的测试用例**
  修复了在无 `sandbox_fusion_url` 的情况下，`default_compute_score` 回退到 `prime_code` 时，无法正确解析 APPS/TACO 等数据源中列表格式的调用测试用例的问题。
* **#8359 [fsdp] 在未设置 mixed_precision 时遵循 fsdp_config.dtype**
  修复了 FSDP 配置解析问题，确保在未显式设置 `mixed_precision param_dtype` 时，系统会正确应用 `fsdp_config.dtype` 的设定。
* **#8358 [reward] 修复 PrimeRewardManager 并行评分时的字典提取问题**
  修复了当奖励评估函数返回字典格式（如内置的 `math_dapo.compute_score` 或 `rlla.compute_score`）时，`PrimeRewardManager` 无法正确提取 `['score']` 字段的问题。

### 🚀 Release
* 本期动态中无新版本发布信息。

---

## 🐛 Issues

### #8362 — [[rollout, sglang] rollout.logprobs_mode is ignored by the SGLang rollout, so rollout log-probs differ between backends](https://github.com/verl-project/verl/issues/8362)
- **作者**: flyingdoubleG  **时间**: 2026-10-25 11:10 CST
- **摘要**: ### What happens  `rollout.logprobs_mode` (default `processed_logprobs`) is passed to vLLM, but the SGLang rollout ignores it. Even though the config is identical, the rollout log-probs therefore mean different things on the two backends:  - **vLLM** honours the mode. `processed_logprobs` are taken …

## 🔀 Pull Requests

### #8363 — [[sglang, rollout] fix: honour rollout.logprobs_mode on the SGLang rollout](https://github.com/verl-project/verl/pull/8363)
- **作者**: flyingdoubleG  **时间**: 2026-10-25 11:10 CST
- **摘要**: ### What does this PR do?  Fixes #8362. The SGLang rollout ignores `rollout.logprobs_mode`. It always returns log-probs of the temperature-scaled full-vocabulary distribution, so: - `raw_logprobs` has no effect on SGLang; - `processed_logprobs` does not reflect top-k / top-p truncation, whereas vLLM…

### #8361 — [[megatron, doc] fix: keep the causal mask under QAT with packed THD sequences](https://github.com/verl-project/verl/pull/8361)
- **作者**: 0z5a  **时间**: 2026-10-25 00:22 CST
- **摘要**: ### What does this PR do?  Megatron QAT fails at the first forward whenever sequences are packed (`use_remove_padding=True`, the default) and context parallelism is 1:  ``` AssertionError: Attention mask type must be padding or padding_causal for qkv_format=thd! ```  With QAT enabled, the engine bui…

### #8360 — [[reward] fix: support APPS/TACO list-formatted call-based test cases in prime_code](https://github.com/verl-project/verl/pull/8360)
- **作者**: liangqianxing  **时间**: 2026-10-24 17:02 CST
- **摘要**: ## What does this PR do?  Fixes #8174.  For `data_source` in `("apps", "taco", "codecontests", "codeforces")` without a `sandbox_fusion_url`, `default_compute_score` falls back to `prime_code`. In `verl/utils/reward_score/prime_code/testing_util.py`, the `CODE_TYPE.call_based` branch of `run_test` a…

### #8359 — [[fsdp] fix: honor fsdp_config.dtype when mixed_precision param_dtype is unset](https://github.com/verl-project/verl/pull/8359)
- **作者**: liangqianxing  **时间**: 2026-10-24 17:01 CST
- **摘要**: ## What does this PR do?  Fixes #8076.  `FSDPEngineConfig` inherits `dtype: str = "bfloat16"` from `EngineConfig` (`verl/workers/config/engine.py`), documented as `"Mixed precision training param dtype, default 'bfloat16'"`.  However, `FSDPEngine._build_fsdp_module` and the QAT export path in `FSDPE…

### #8358 — [[reward] fix: extract dict['score'] in PrimeRewardManager parallel scoring](https://github.com/verl-project/verl/pull/8358)
- **作者**: liangqianxing  **时间**: 2026-10-24 15:44 CST
- **摘要**: ### What does this PR do?  Fixes #8216.  When a reward evaluation function returns a dictionary (such as the built-in `math_dapo.compute_score` or `rlla.compute_score` which return `{"score": ..., "acc": ..., "pred": ...}`), `PrimeRewardManager`'s `parallel_compute_score_async` previously fell throu…
