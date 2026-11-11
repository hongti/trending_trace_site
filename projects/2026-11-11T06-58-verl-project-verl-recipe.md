# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-11-11 14:58 CST

## AI 总结

## verl-project/verl-recipe 近期动态摘要

---

### 📦 Release
近期无 Release 发布。

---

### 🔀 Pull Request

**#171 — 将全异步与单步离线 trainer 从 verl 迁移至本仓库**
- 作者：Begunner（2026-11-10）
- **核心变更**：将 `fully_async_policy`、`one_step_off_policy` 及其共享的 separation 代码从 verl 主仓库迁移到 verl-recipe 仓库。
- **背景**：为 verl 主仓库即将移除 V0 trainer 做前置准备。两个 recipe 均对 verl 版本进行了锁定。

**#170 — SPPO 适配迁移至 verl v1 trainer 核心**
- 作者：yuki1ssad（2026-11-10）
- **核心变更**：将 SPPO recipe 从旧的子类实现方式（依赖 `REQUIRED_VERL.txt` 中锁定的 verl `bcb6386`）迁移至当前 verl 核心（v1 trainer）。
- **实现方式**：通过注册自定义的 advantage estimator 和 policy loss 来接入，弃用原有的 `sppo_ray_trainer` 等遗留文件。

---

### 📌 Issue
近期无 Issue 活动。

---

### 总结
两条 PR 的共同主题是 **「脱离旧版 V0 trainer，适配 verl 新核心架构」**：
1. **基础设施迁移**（#171）：异步/离线 trainer 代码从 verl 主仓库下沉到 recipe 仓库。
2. **算法适配升级**（#170）：SPPO 从遗留子类模式改为基于 v1 trainer 的注册式实现，与新架构对齐。

这表明 verl 生态正在进行一次系统性的 trainer 架构升级，recipe 仓库将承担更多算法实现的维护职责。

---

## 🔀 Pull Requests

### #171 — [[recipe] refactor: migrate fully async and one-step-off trainers from verl](https://github.com/verl-project/verl-recipe/pull/171)
- **作者**: Begunner  **时间**: 2026-11-10 21:10 CST
- **摘要**: ## What does this PR do?  Move fully_async_policy, one_step_off_policy, and their shared separation code from verl into this repository ahead of V0 trainer removal from verl main.  Both recipes pin verl v0.9.1 at 1876b06d0a3e4e71e06230be10af14492ca8a75b and continue using its V0 trainer. The V0 trai…

### #170 — [[recipe] feat: port SPPO to the current verl core (v1 trainer) via registered adv estimator and policy loss](https://github.com/verl-project/verl-recipe/pull/170)
- **作者**: yuki1ssad  **时间**: 2026-11-10 15:32 CST
- **摘要**: ### What does this PR do?  Ports the SPPO recipe off the legacy subclass implementation (pinned to verl `bcb6386` in `REQUIRED_VERL.txt`) onto the current verl core. The legacy files (`sppo_ray_trainer.py` / `sppo_worker.py` / `dp_actor.py`) subclass `ActorRolloutRefWorker`, `DataParallelPPOActor` a…
