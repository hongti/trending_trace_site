# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-17 12:53 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要。本次提供的数据仅包含 Pull Request (PR) 活动，未包含 Issue 和 Release 记录。

### 🛠️ Pull Request (PR) 活动摘要

**1. v0.30.0 版本热修复与合入**
近期多项关键修复被 backport 到 `releases/v0.30.0` 分支，为版本稳定发布做准备：
*   **KVPool 与 Mamba 修复**：同步了 Mamba 状态拷贝的执行顺序，并修复了由于池元数据中断导致的普通逐层可读性探测失败问题。
*   **CI 修复**：通过设置 `TORCH_EXTENSIONS_DIR=/tmp/t` 缓解了 GLM-5.3-Flash PD 夜间测试的启动卡死问题。

**2. 新特性与功能支持**
*   **Qwen3.8 序列并行 (SP)**：为 Qwen3.8 模型添加了 dense 序列并行支持，引入了显式的 all-gather 和 reduce-scatter 边界，并集成了 FlashComm 功能。
*   **Model Runner V2 (MRv2) 预热优化**：为 MRv2 引入了早期 kernel 预热和 NZ 格式转换预热功能，提升运行效率。

**3. 模型与调度 Bug 修复**
*   **Kimi K3 调度恢复**：恢复了 v0.27.1 版本中的共享专家多流提交顺序，以支持 latent W4A8 序列并行。
*   **GLM5 配置回退**：将 `GLM5_1_W4A4_A5` 模型恢复至 V1 服务器配置，以修复前期提交带来的影响。

**4. 测试与文档**
*   **测试覆盖**：为 Gemma 4 MXFP4 模型（31B 和 26B-A4B）添加了 A5 夜间 GPQA-D 精度测试覆盖。
*   **文档修复**：更新了 README 中的快速入门与安装链接，将其指向最新的 `v0.30.0rc2` 版本，修复了重定向循环问题。

---
*注：本次输入数据未包含 Issue 和 Release 相关动态。*

---

## 🔀 Pull Requests

### #19060 — [[BugFix][KVPool][v0.30.0] Backport synchronous Mamba state-copy ordering](https://github.com/vllm-project/vllm-ascend/pull/19060)
- **作者**: LifeOfCoffee  **时间**: 2026-10-17 12:10 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Backport #18901 (`35d5934deb7afdfb577768a1ada2c16686650db7`) to `releases/v0.30.0`, based on `169823b8cca0eebb4db20811f1582dd28104ecaa`. Related issue: #18898.  With MRV2 and Mamba `align`, preprocessing can copy a cached source state before a synchronous, no…

### #19059 — [[v0.30.0][CI][BugFix] Fix GLM-5.3-Flash PD nightly hangs](https://github.com/vllm-project/vllm-ascend/pull/19059)
- **作者**: lrf-vm  **时间**: 2026-10-17 11:58 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Mitigate GLM-5.3-Flash PD nightly startup hangs in `GLM-5.3-Flash-W8A8-130k-PD.yaml`:  - Incorporate the workaround from #19049: set `TORCH_EXTENSIONS_DIR=/tmp/torch_extensions/glm53` in `env_common`, so both prefill and decode use pod-local Torch/CANN JIT bu…

### #19058 — [[CI][BugFix] Fix GLM-5.3-Flash PD nightly hangs](https://github.com/vllm-project/vllm-ascend/pull/19058)
- **作者**: lrf-vm  **时间**: 2026-10-17 11:57 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Mitigate GLM-5.3-Flash PD nightly startup hangs in `GLM-5.3-Flash-W8A8-130k-PD.yaml`:  - Incorporate the workaround from #19049: set `TORCH_EXTENSIONS_DIR=/tmp/torch_extensions/glm53` in `env_common`, so both prefill and decode use pod-local Torch/CANN JIT bu…

### #19057 — [[BugFix][Kimi K3] Restore v0.27.1 shared-expert multistream schedule](https://github.com/vllm-project/vllm-ascend/pull/19057)
- **作者**: shaopeng-666  **时间**: 2026-10-17 11:31 CST
- **标签**: module:tests, module:ops
- **摘要**: ## Summary  - restore the v0.27.1-style deferred shared-expert submission order for latent W4A8 sequence parallelism - keep router input token-local while shared-expert input may already be sequence-parallel gathered - pass the Kimi K3 gate into `FusedMoEFactory` and preserve FP32 router logits for …

### #19056 — [[Feature] add dense SP support for Qwen3.8](https://github.com/vllm-project/vllm-ascend/pull/19056)
- **作者**: jiaqi-lee  **时间**: 2026-10-17 11:06 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: Add a scoped Qwen3.5 decoder layer override with explicit sequence-parallel all-gather and reduce-scatter boundaries. Wire opt-in FlashComm enablement and add unit, collective, engine, graph-mode, MTP, and full-weight coverage.  ### What this PR does / why we need it?  ### Does this PR introduce _an…

### #19055 — [[BugFix][CI] Restore GLM5_1_W4A4_A5 to the V1 server config](https://github.com/vllm-project/vllm-ascend/pull/19055)
- **作者**: yjyang62  **时间**: 2026-10-17 10:29 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  #17681 moved GLM5_1_W4A4_A5 onto a Model Runner V2 server shape (TP=2, prefill-context-parallel-size=4, HCCL_BUFFSIZE=1024, gpu-memory-utilization=0.90, CLI EPLB). A follow-up only pinned VLLM_USE_V2_MODEL_RUNNER=0. V1 rejects prefill context parallel greater…

### #19054 — [[Ops][Feature][MRV2] Add early kernel and NZ warmup for Model Runner V2](https://github.com/vllm-project/vllm-ascend/pull/19054)
- **作者**: zouzy5137  **时间**: 2026-10-17 10:21 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  This PR enables early kernel warmup and NZ format-cast warmup for Model Runner V2 (MRv2). Specifically: - It integrates `start_early_kernel_warmup`, `join_early_kernel_warmup`, `start_nz_warm_thread`, and `join_nz_warm_thread` into `NPUModelRunner.load_model`…

### #19053 — [[Test] Add A5 nightly GPQA-D coverage for Gemma 4 MXFP4](https://github.com/vllm-project/vllm-ascend/pull/19053)
- **作者**: Alex-stack-hub  **时间**: 2026-10-17 09:35 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Add A5 nightly GPQA-D accuracy coverage for Gemma 4 MXFP4 models: - 31B: baseline 75%, tolerance ±5 percentage points. - 26B-A4B: baseline 72%, tolerance ±5 percentage points.  ### Does this PR introduce _any_ user-facing change?  No.  ### How was this patch …

### #19052 — [[Doc][BugFix] Update README links to v0.30.0rc2](https://github.com/vllm-project/vllm-ascend/pull/19052)
- **作者**: lianxiaozhi  **时间**: 2026-10-17 02:24 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  The v0.30.0rc1 Quick Start and Installation links in the Getting Started tables loop through redirects. Update the version and both documentation links to v0.30.0rc2 in README.md and README.zh.md.  ### Does this PR introduce _any_ user-facing change?  No. Thi…

### #19051 — [[BugFix][KVPool] Recover ordinary layerwise readability probe failures (v0.30.0)](https://github.com/vllm-project/vllm-ascend/pull/19051)
- **作者**: lizy124  **时间**: 2026-10-17 00:46 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  Backport of https://github.com/vllm-project/vllm-ascend/pull/19050 to `releases/v0.30.0`. Follow-up to #18813.  A pool metadata outage can make `batch_is_readable` raise `BatchResultShapeError` (for example, when the metadata query returns zero results for a …
