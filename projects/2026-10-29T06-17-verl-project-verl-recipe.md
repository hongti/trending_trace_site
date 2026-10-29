# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-10-29 14:17 CST

## AI 总结

# verl-project/verl-recipe 近期动态摘要

## 📋 Issue（1 项）

- **#167** | 作者: super-yuki | 10-29
  - **deepeyes 配方需支持 Qwen3.5-4B/9B（Ascend NPU）**
  - 当前配方仅针对 Qwen2.5-VL，已无法在最新 verl 上运行，主要原因：`verl.tools` 中的 image zoom 工具被移除，数据集在 AgentLoop 接管前已内部 tokenized。

## 🔀 Pull Request（2 项）

- **#166** | 作者: super-yuki | 10-29
  - **[deepeyes] 新增 Qwen3.5-4B/9B 在 Ascend NPU 上的支持**
  - 与 Issue #167 对应的修复/新特性 PR，适配最新 verl API 变更（image zoom 工具移除、tokenization 流程变更、Qwen3.5 适配）。

- **#165** | 作者: 1903943257 | 10-28
  - **新增 Tree Prefix Reuse (TPR) 训练配方**
  - 为 RFC #7561 提供 Recipe 实现与执行入口，引入训练侧的前缀复用机制（TPR Runtime），旨在提升训练效率。

## 🚀 Release

- 近期无 Release 发布。

---

### 要点总结
> 本期核心围绕**硬件/模型适配**与**训练效率优化**两条主线：deepeyes 配方向 Qwen3.5 + Ascend NPU 迁移（#166/#167），同时引入 Tree Prefix Reuse 训练配方（#165）以降低重复前缀计算开销。

---

## 🐛 Issues

### #167 — [deepeyes: Qwen3.5-4B/9B support on Ascend NPU](https://github.com/verl-project/verl-recipe/issues/167)
- **作者**: super-yuki  **时间**: 2026-10-29 12:28 CST
- **摘要**: ### Why  The deepeyes recipe only targets Qwen2.5-VL. On current verl it no longer runs: the image zoom tool was removed from `verl.tools`, the dataset tokenized internally before the AgentLoop took over, and Qwen3.5 uses a different chat template and XML tool-call format.  ### What  Implemented in …

## 🔀 Pull Requests

### #166 — [[deepeyes] feat: add Qwen3.5-4B/9B support on Ascend NPU](https://github.com/verl-project/verl-recipe/pull/166)
- **作者**: super-yuki  **时间**: 2026-10-29 11:11 CST
- **摘要**: ### Why  The recipe targets Qwen2.5-VL and no longer runs on current verl: `verl.tools.image_zoom_in_tool` was removed, the dataset tokenized internally before the AgentLoop took over, and Qwen3.5 uses a different chat template and XML tool-call format.  ### What changed  - Added `configs/deepeyes_m…

### #165 — [feat(tpr): add Tree Prefix Reuse training recipe](https://github.com/verl-project/verl-recipe/pull/165)
- **作者**: 1903943257  **时间**: 2026-10-28 18:47 CST
- **摘要**: # Add Tree Prefix Reuse (TPR) for Training-Side Prefix Reuse  ## Checklist Before Starting  - This PR provides the Recipe implementation and execution entry point for RFC #7561. - The TPR Runtime has been moved to the standalone `tpr/verl_tpr/` package. Package imports have been verified in the exis…
