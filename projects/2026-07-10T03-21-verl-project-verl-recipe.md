# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-07-10 11:21 CST

## AI 总结

**GitHub 仓库 verl-project/verl-recipe 近期动态摘要**

### 💡 Pull Request (PR)
* **PR #119**：新增 W4A8 FSDP 量化感知训练（QAT）模拟配方
  * **重要变更**：为 Dense 架构的 `Qwen3-8B-Base` 和 MoE 架构的 `Qwen3-30B-A3B-Base` 模型引入了 FSDP W4A8 数值模拟的运行脚本。
  * **新增特性**：添加了全模型量化模拟脚本（`qat/run_qwen3_8b_w4a8.sh`）以及仅针对 FFN 层的量化模拟脚本（`qat/run_qwen3_8b_w4a8_FFN_only.sh`）等，扩展了 verl 在低比特量化训练方面的recipe支持。

### 🐛 Issue
* 近期无相关动态。

### 🚀 Release
* 近期无新版本发布。

---

## 🔀 Pull Requests

### #119 — [[recipe] feat: add W4A8 FSDP QAT simulation recipes](https://github.com/verl-project/verl-recipe/pull/119)
- **作者**: zhangyimi  **时间**: 2026-07-09 17:01 CST
- **摘要**: ## Summary  This draft adds FSDP W4A8 numerical-simulation recipes for dense Qwen3-8B-Base and MoE Qwen3-30B-A3B-Base:  - `qat/run_qwen3_8b_w4a8.sh` - `qat/run_qwen3_8b_w4a8_FFN_only.sh` - `qat/run_qwen3_30b_w4a8.sh` - `qat/run_qwen3_30b_w4a8_FFN_only.sh` - README configuration, limitations, and the…
