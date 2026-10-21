# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-21 13:12 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📦 Pull Request (PR) 摘要

近期 PR 活动主要围绕**特性跨版本回退**、**性能优化**以及**代码重构**展开：

1. **ModelRunner V1 Runtime Guard 特性大范围回退**
   作者 `djh95` 向多个历史发布分支（`v0.30.0`, `v0.28.0rc`, `v0.27.1rc`, `v0.26.0rc`, `v0.23.0`）批量提交了 Cherry-pick PR。旨在为这些版本线接入 ModelRunner V1 的 Runtime Guard 特性（涵盖控制平面与 worker 空闲钩子）。

2. **EPLB 性能优化**
   作者 `freyfwt` 提交了 HIXL 缓冲区重用机制的优化（#19342）。取消了在重用共享专家缓冲区前对整个 EPLB 流的同步，改为仅等待所需的 consumed event，从而提升执行效率。

3. **Attention 后端代码重构**
   作者 `recky-c` 统一了注意力后端中 PCP 和 DCP 的配置标志与辅助函数命名（#19334），使其与上游命名规范保持一致。

4. **LongSpark 文档补充**
   作者 `HaizhouPeng` 添加了关于 LongSpark 投机解码集成的最小化占位 README 及相关链接（#19335）。

5. **自动化夜间测试**
   作者 `LQDLove` 提交了两个用于 Nightly 自动化验证的空 PR（#19336, #19333），基于 vLLM main 分支的最新提交进行测试。

### 🐛 Issue 摘要
近期未观察到公开的 Issue 动态。

### 🚀 Release 摘要
近期未发布新版本。

---

## 🔀 Pull Requests

### #19342 — [[Performance][EPLB] Wait for consumed event before HIXL buffer reuse](https://github.com/vllm-project/vllm-ascend/pull/19342)
- **作者**: freyfwt  **时间**: 2026-10-21 12:00 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  HIXL READs currently synchronize the entire EPLB stream before reusing the shared expert buffer. For subsequent changed layers, the required dependency is the previous foreground consumption of that buffer.  Pass the recorded consumption event from the existi…

### #19341 — [[Cherry-pick][releases/v0.30.0][Feature] Wire runtime_guard for ModelRunner V1](https://github.com/vllm-project/vllm-ascend/pull/19341)
- **作者**: djh95  **时间**: 2026-10-21 11:59 CST
- **标签**: module:core
- **摘要**: ## Summary Wire **Model Runner V1** Runtime Guard (control plane + `model_runner_v1.py` / worker idle hooks) onto this release line.  - Source: `feat/runtime-guard-config-v1` @ `9343c9859` adapted per docs pin in `runtime-guard-docs/05_mrv1_backport/` - **MRV1 only** — does **not** modify `worker/v2…

### #19340 — [[Cherry-pick][releases/v0.28.0rc][Feature] Wire runtime_guard for ModelRunner V1](https://github.com/vllm-project/vllm-ascend/pull/19340)
- **作者**: djh95  **时间**: 2026-10-21 11:59 CST
- **标签**: module:core
- **摘要**: ## Summary Wire **Model Runner V1** Runtime Guard (control plane + `model_runner_v1.py` / worker idle hooks) onto this release line.  - Source: `feat/runtime-guard-config-v1` @ `9343c9859` adapted per docs pin in `runtime-guard-docs/05_mrv1_backport/` - **MRV1 only** — does **not** modify `worker/v2…

### #19339 — [[Cherry-pick][releases/v0.27.1rc][Feature] Wire runtime_guard for ModelRunner V1](https://github.com/vllm-project/vllm-ascend/pull/19339)
- **作者**: djh95  **时间**: 2026-10-21 11:59 CST
- **标签**: module:core
- **摘要**: ## Summary Wire **Model Runner V1** Runtime Guard (control plane + `model_runner_v1.py` / worker idle hooks) onto this release line.  - Source: `feat/runtime-guard-config-v1` @ `9343c9859` adapted per docs pin in `runtime-guard-docs/05_mrv1_backport/` - **MRV1 only** — does **not** modify `worker/v2…

### #19338 — [[Cherry-pick][releases/v0.26.0rc][Feature] Wire runtime_guard for ModelRunner V1](https://github.com/vllm-project/vllm-ascend/pull/19338)
- **作者**: djh95  **时间**: 2026-10-21 11:59 CST
- **标签**: module:core
- **摘要**: ## Summary Wire **Model Runner V1** Runtime Guard (control plane + `model_runner_v1.py` / worker idle hooks) onto this release line.  - Source: `feat/runtime-guard-config-v1` @ `9343c9859` adapted per docs pin in `runtime-guard-docs/05_mrv1_backport/` - **MRV1 only** — does **not** modify `worker/v2…

### #19337 — [[Cherry-pick][releases/v0.23.0][Feature] Wire runtime_guard for ModelRunner V1](https://github.com/vllm-project/vllm-ascend/pull/19337)
- **作者**: djh95  **时间**: 2026-10-21 11:59 CST
- **标签**: module:core
- **摘要**: ## Summary Wire **Model Runner V1** Runtime Guard (control plane + `model_runner_v1.py` / worker idle hooks) onto this release line.  - Source: `feat/runtime-guard-config-v1` @ `9343c9859` adapted per docs pin in `runtime-guard-docs/05_mrv1_backport/` - **MRV1 only** — does **not** modify `worker/v2…

### #19336 — [[Verify] Nightly test on main](https://github.com/vllm-project/vllm-ascend/pull/19336)
- **作者**: LQDLove  **时间**: 2026-10-21 11:55 CST
- **标签**: documentation
- **摘要**: Auto-created empty PR for nightly verification. - vLLM main: https://github.com/vllm-project/vllm/commit/a98247ab4db686ee03c66d5feb3c761e52a2f8ab

### #19335 — [[Doc] Add LongSpark ](https://github.com/vllm-project/vllm-ascend/pull/19335)
- **作者**: HaizhouPeng  **时间**: 2026-10-21 11:52 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  This draft tracks the planned LongSpark speculative decoding integration.  It adds a minimal placeholder README with links to:  - [LongSpark: Efficient speculative decoding with a fixed-cost parallel drafter](https://arxiv.org/abs/2609.37029) - [LongSpark mod…

### #19334 — [[Refactor][Attention] Unify PCP/DCP flag and helper naming](https://github.com/vllm-project/vllm-ascend/pull/19334)
- **作者**: recky-c  **时间**: 2026-10-21 11:47 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  PCP and DCP configuration flags have inconsistent names across the attention backends. This PR standardizes the names to match upstream conventions and distinguish configuration flags from helper functions:  - Use `use_pcp` and `use_dcp` for configuration fla…

### #19333 — [[Verify] Nightly test on main](https://github.com/vllm-project/vllm-ascend/pull/19333)
- **作者**: LQDLove  **时间**: 2026-10-21 11:43 CST
- **标签**: documentation
- **摘要**: Auto-created empty PR for nightly verification. - vLLM main: https://github.com/vllm-project/vllm/commit/a98247ab4db686ee03c66d5feb3c761e52a2f8ab
