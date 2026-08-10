# verl-project/verl — 动态追踪

> 生成时间: 2026-08-10 10:42 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🛠️ Pull Request (PR)
近期 PR 主要集中在修复配置兼容性、性能分析工具及 Docker 构建问题：

*   **配置兼容性修复 (#7331)**：恢复了主 PPO 入口点中的 `migrate_legacy_reward_impl(config)` 调用。该修复确保了已弃用的顶层 reward 配置键能在验证前被正确映射到 `config.reward.*`，提升了旧版配置的向前兼容性。
*   **性能分析 (Profiling) 修复 (#7330)**：修复了在连续（非离散）模式下进行性能分析时，nsys `.nsys-rep` 文件无法正常收尾（finalize）的问题。同时调整了 `RayWorkerGroup` 的默认行为，将 `capture-range-end` 默认设为 `repeat-shutdown:{6 * len(profile_steps)}`。
*   **Docker 构建修复 (#7329)**：修复了 `Dockerfile.stable.vllm` 中导致 `fast-hadamard-transform` 安装失败的 pip 标志拼写错误（将单短划线 `-no-build-isolation` 更正为双短划线 `--no-build-isolation`）。

### 🐛 Issue
*   近期无新增重要 Issue 动态。

### 🚀 Release
*   近期无新的版本发布。

---

## 🔀 Pull Requests

### #7331 — [[trainer] fix: migrate legacy reward config on the main PPO entrypoint](https://github.com/verl-project/verl/pull/7331)
- **作者**: nataliekung  **时间**: 2026-08-10 07:36 CST
- **摘要**: ### What does this PR do?  Restore the `migrate_legacy_reward_impl(config)` call on the main PPO entrypoint so the deprecated top-level reward keys are mapped onto `config.reward.*` before validation and use.  `verl/trainer/config/ppo_trainer.yaml` still composes `legacy_reward_impl` ("legacy reward…

### #7330 — [[single_controller, trainer] fix: finalize nsys capture range in continuous profiling](https://github.com/verl-project/verl/pull/7330)
- **作者**: nataliekung  **时间**: 2026-08-10 04:49 CST
- **摘要**: ### What does this PR do?  Finalize the nsys `.nsys-rep` when profiling in continuous (non-discrete) mode.  `RayWorkerGroup` defaults `capture-range-end` to `repeat-shutdown:{6 * len(profile_steps)}`, which assumes discrete profiling emits six `cudaProfilerApi` ranges per profiled step. With `discre…

### #7329 — [[docker] fix: pip flag typo breaks fast-hadamard-transform install](https://github.com/verl-project/verl/pull/7329)
- **作者**: ji-huazhong  **时间**: 2026-08-09 21:33 CST
- **摘要**: ### What does this PR do?  Fixes a flag typo in `docker/Dockerfile.stable.vllm` introduced by #7101: the fast-hadamard-transform install step uses `-no-build-isolation` (single dash) instead of `--no-build-isolation`. This is a build-breaking fix, not a cosmetic typo change.  ### Checklist Before St…
