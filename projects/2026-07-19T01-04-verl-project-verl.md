# verl-project/verl — 动态追踪

> 生成时间: 2026-07-19 09:04 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📋 Issue 动态
*   **[#7092] 模型加载忽略 `_keep_in_fp32_modules` 配置，导致静默精度退化**
    *   **详情**：作者 SelfSL 报告了一个严重 Bug。当前模型加载机制未正确处理 `_keep_in_fp32_modules` 参数，导致像 Inkling、Qwen3-Next 等需要将特定模块保留为 FP32 精度的模型，在加载时发生了**无声的精度降级**（即没有报错提示，但实际精度受损），这可能会对模型训练和推理效果产生隐蔽的负面影响。

### 🔀 PR 动态
*   近期暂无提供的重要 PR 动态。

### 🚀 Release 动态
*   近期暂无新版本发布信息。

*(注：当前提供的数据仅包含一条 Issue 记录，PR 与 Release 暂无相关输入数据。)*

---

## 🐛 Issues

### #7092 — [[BUG] Model loading ignores _keep_in_fp32_modules — silent precision degradation for fp32-keep models (Inkling, Qwen3-Next)](https://github.com/verl-project/verl/issues/7092)
- **作者**: SelfSL  **时间**: 2026-07-19 05:18 CST
- **标签**: bug
- **摘要**: ### System Info  ``` ----------Python Info---------- Version      : 3.12.3 ------------Pip Info----------- vllm         : 0.24.1.dev0+gee0da84ab.d20260706.cu129 sglang       : 0.0.0.dev200+g6a60304af ray          : 2.56.0 torch        : 2.12.0a0+git220ecc2 ----------verl Info----------- Version     …
