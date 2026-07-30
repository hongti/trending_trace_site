# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-07-30 09:00 CST

## AI 总结

以下是 **verl-project/verl-recipe** 仓库近期动态的中文摘要：

### 📥 Pull Request (合并请求)
*   **#127 feat(low_precision): 新增 Qwen3 MoE MXFP8 端到端配方** 
    *   **作者**: jQizhang | **时间**: 2026-07-29
    *   **新特性与重要变更**：为 Qwen3-30B-A3B 模型新增了端到端的 DAPO 训练配方。该配方的重要亮点在于**全面支持 MXFP8 低精度计算**，涵盖了 Megatron 训练阶段与 SGLang 推理阶段。具体配置了 Transformer Engine 的 MXFP8 训练，以及动态 MXFP8 推理权重，进一步拓展了 MoE 架构在低精度下的端到端优化能力。

### 🚨 Issue (问题)
*   近期暂无新增动态。

### 🚀 Release (版本发布)
*   近期暂无新版本发布。

---

## 🔀 Pull Requests

### #127 — [feat(low_precision): Add Qwen3 MoE MXFP8 end-to-end recipe](https://github.com/verl-project/verl-recipe/pull/127)
- **作者**: jQizhang  **时间**: 2026-07-29 13:43 CST
- **摘要**: ## Summary    Add an end-to-end DAPO recipe for Qwen3-30B-A3B using MXFP8 for both Megatron training and SGLang rollout.    The recipe configures Transformer Engine MXFP8 training, dynamic MXFP8 rollout weight quantization, and Blackwell-optimized SGLang backends.    ## Changes    New file:    - `lo…
