# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-11-10 15:40 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的简洁摘要：

### 🚀 Pull Request 动态
*   **PR #170: 将 SPPO 配方迁移至当前 verl 核心 (V1 trainer)**
    *   **作者**: yuki1ssad
    *   **核心变更**: 废弃了此前依赖旧版子类实现（固定绑定 verl `bcb6386` 版本）的 SPPO 配方。
    *   **新特性/亮点**: 通过注册优势估计器和策略损失函数，成功将 SPPO 适配并移植到最新的 verl 核心（V1 trainer）上，同时淘汰了旧的 `sppo_ray_trainer` 文件。

### 📝 Issue 动态
*   最近无公开的 Issue 动态。

### 🏷️ Release 动态
*   最近无新的版本发布。

---

## 🔀 Pull Requests

### #170 — [[recipe] feat: port SPPO to the current verl core (v1 trainer) via registered adv estimator and policy loss](https://github.com/verl-project/verl-recipe/pull/170)
- **作者**: yuki1ssad  **时间**: 2026-11-10 15:32 CST
- **摘要**: ### What does this PR do?  Ports the SPPO recipe off the legacy subclass implementation (pinned to verl `bcb6386` in `REQUIRED_VERL.txt`) onto the current verl core. The legacy files (`sppo_ray_trainer.py` / `sppo_worker.py` / `dp_actor.py`) subclass `ActorRolloutRefWorker`, `DataParallelPPOActor` a…
