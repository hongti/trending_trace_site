# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-09-22 13:18 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的中文简洁摘要：

### 🐛 Issue (议题)
* 近期暂无相关动态。

### 🔀 Pull Request (拉取请求)
* **PR #155：[script] 设置 optimizer_offload 为 False**
  * **作者:** fangweii1
  * **重要修复/变更：** 解决了在 A2 双节点（two-node）环境下，开启优化器卸载（optimizer offload）会导致训练时出现显存溢出（OOM）的问题。
  * **具体操作：** 在脚本中通过显式设置 `actor_rollout_ref.actor.fsdp_config.optimizer_offload=False` 来禁用该功能。

### 🚀 Release (版本发布)
* 近期暂无新版本发布。

---

## 🔀 Pull Requests

### #155 — [[script] set optimizer_offload to False](https://github.com/verl-project/verl-recipe/pull/155)
- **作者**: fangweii1  **时间**: 2026-09-22 09:59 CST
- **摘要**: On A2 two-node setup, enabling optimizer offload causes OOM during training.  This change disables optimizer offload by setting: actor_rollout_ref.actor.fsdp_config.optimizer_offload=False  With optimizer offload disabled, the A2 two-node configuration can run successfully without OOM.
