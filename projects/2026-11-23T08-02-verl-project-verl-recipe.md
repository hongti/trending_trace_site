# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-11-23 16:02 CST

## AI 总结

# verl-project/verl-recipe 近期动态摘要

## 📋 Issue

**#181 — [deepeyes] 失败的 crop 调用仍能获取工具奖励**（作者：archted，2026-11-22）

- **问题核心**：`compute_score()` 通过标签检测是否使用了工具；但即便 crop 操作失败（工具返回错误），这些标签依然存在，导致正确答案仍被错误地加上 **1.2 的工具奖励加成**。
- **影响**：奖励计算逻辑存在漏洞，可能干扰训练信号的准确性。
- **复现**：作者已提供可复现说明。

## 🔧 Pull Request

**#182 — [deepswe_tpu] 多项功能增强，支持开箱即用运行 35B/397B**（作者：jialei777，2026-11-23）

主要变更：
1. **注册 `grpo_loo`**：新增 GRPO Leave-One-Out 采样策略支持。
2. **逗号链式 ROLLOUT_RS**：支持以逗号分隔、引号包裹的方式串联 rollout-correction 阶段。
3. **BYPASS_MODE 支持**：在 Qwen3-5 **35B** 与 **397B** 两个规模上启用 bypass 模式。
4. **开箱即用**：`run_qwen3_5_35b_deepswe.sh` 与 `run_qwen3_5_397b_deepswe.sh` 两个脚本可直接干净运行，无需额外配置。

## 📦 Release

> 本周期内无新版本发布。

---

**总体点评**：本期活动聚焦于**奖励计算正确性修复**（Issue #181 揭示的工具奖励误判）与**大模型训练脚本的可用性增强**（PR #182 让 35B/397B 规模训练更易上手）。两者分别对应"质量保障"与"规模化能力扩展"两条主线。

---

## 🐛 Issues

### #181 — [[deepeyes] Failed crop calls can still receive the tool reward](https://github.com/verl-project/verl-recipe/issues/181)
- **作者**: archted  **时间**: 2026-11-22 15:10 CST
- **摘要**: ## Problem  `compute_score()` detects tool use from `<tool_call>` or `<tool_response>` tags. A failed crop still has these tags, so a correct answer receives the 1.2 tool bonus even when the tool returns an error.  ## Reproduction  Pass a trajectory containing a crop call, an error tool response, an…

## 🔀 Pull Requests

### #182 — [[deepswe_tpu] feat: register grpo_loo, quote comma-chained ROLLOUT_RS, and support BYPASS_MODE on 35B/397B](https://github.com/verl-project/verl-recipe/pull/182)
- **作者**: jialei777  **时间**: 2026-11-23 15:08 CST
- **摘要**: ## Summary  Updates `deepswe_tpu` so both `run_qwen3_5_35b_deepswe.sh` and `run_qwen3_5_397b_deepswe.sh` run cleanly out of the box with `grpo_loo`, `BYPASS_MODE`, and comma-chained rollout-correction filters on `verl` `main`:  - **`deepswe_tpu/deepswe.py`**: Registers `grpo_loo` (`grpo-loo`) in `AD…
