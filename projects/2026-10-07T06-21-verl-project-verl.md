# verl-project/verl — 动态追踪

> 生成时间: 2026-10-07 14:21 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🐛 Issue
- **#8123 奖励函数计算 Bug**：`math_dapo` 在提取 `\boxed{}` 格式的备选答案时，执行了严格的逐字节字符串比对。如果与标准答案存在微小格式差异，会被直接判错并打分为 -1.0。
- **#8122 蒸馏数据预处理 Bug**：在去填充预处理重复运行两次时，嵌套的 `teacher_logprobs` 张量结构会被破坏，触发 `split_with_sizes` 断言错误。

### 🛠️ Pull Request
- **#8125 修复蒸馏预处理张量损坏问题**：针对 Issue #8122，在去填充预处理逻辑中为 `teacher_logprobs` / `teacher_ids` 增加了 `is_nested` 守卫，防止对已嵌套的张量进行重复转换。
- **#8124 修复奖励函数匹配逻辑**：针对 Issue #8123，规范化了 `math_dapo` 中 `\boxed{}` 备选答案的提取与比对逻辑，不再使用严苛的原始字符串比较。
- **#8121 新增 Intel XPU 真实 FP8 训练支持**：引入在 XPU 硬件上进行真实 FP8 精度训练的特性（目前处于 Draft 草稿状态，待进一步完善）。

### 🚀 Release
- 本期动态内无新版本发布。

---

## 🐛 Issues

### #8123 — [[reward] math_dapo: \boxed{} answers score -1.0 unless matching ground truth byte-for-byte](https://github.com/verl-project/verl/issues/8123)
- **作者**: KardeniaPoyu  **时间**: 2026-10-07 01:08 CST
- **摘要**: ### System Info  ``` verl 0.10.0.dev, main @ 8718ca3 Python 3.12.12, Windows 11 (CPU only; issue is pure string handling) torch 2.13.0+cpu, ray 2.59.0 ```  ### Information  - [ ] The official example scripts - [x] My own modified scripts  ### Tasks  - [x] An officially supported task in the `example…

### #8122 — [[Bug] Distillation: nested teacher_logprobs tensor corrupted when rmpad preprocessing runs twice (split_with_sizes assertion)](https://github.com/verl-project/verl/issues/8122)
- **作者**: NGDBZ  **时间**: 2026-10-06 21:42 CST
- **摘要**: ### System Info  - verl **v0.9.1** (commit `1876b06d0a3e4e71e06230be10af14492ca8a75b`); the missing guard described below is still present on `main` as of 2026-10-06 - Hardware: 8× NVIDIA H20 96GB - Python 3.11.11 (miniconda env), FSDP worker with `use_dynamic_bsz=True`, vLLM rollout - Model: Qwen3.…

## 🔀 Pull Requests

### #8125 — [fix: guard against re-conversion of already-nested teacher_logprobs in rmpad preprocessing](https://github.com/verl-project/verl/pull/8125)
- **作者**: NGDBZ  **时间**: 2026-10-07 10:48 CST
- **摘要**: ### What does this PR do?  Adds the missing `is_nested` guard for `teacher_logprobs`/`teacher_ids` in the rmpad preprocessing (`verl/workers/utils/padding.py`), mirroring the adjacent `routed_experts` handling in the same function.  ### Context  Filed alongside #8122 (crash: `split_with_sizes expect…

### #8124 — [[reward] fix: normalize the boxed fallback answer in math_dapo](https://github.com/verl-project/verl/pull/8124)
- **作者**: KardeniaPoyu  **时间**: 2026-10-07 01:08 CST
- **摘要**: ### What does this PR do?  In `math_dapo.verify()`, when a response doesn't contain an `Answer:` line, it falls back to extracting `\boxed{}`. However, the fallback currently does a raw string comparison against the ground truth. Meanwhile, the `Answer:` path normalizes both the prediction and the g…

### #8121 — [[draft] Feat/xpu real fp8 training](https://github.com/verl-project/verl/pull/8121)
- **作者**: kahlun  **时间**: 2026-10-06 19:19 CST
- **摘要**: to be finalized or use different branch . . .
