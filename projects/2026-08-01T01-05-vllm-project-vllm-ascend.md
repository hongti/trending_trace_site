# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-01 09:05 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🛠️ Pull Request (PR)

**1. 新特性与性能优化**
*   **支持 DeepSeek V4 推理力度**：前端新增对 DeepSeek-V4-Flash-0731 模型不同思考级别的支持（如 `low` 无推理前缀，`high` 保留前缀），该修复同时合入了主分支及 v0.25.1rc 分支 (#13313, #13314)。
*   **Ascend 310P 性能提升**：为 Atlas 300I (310P) 引入 Qwen3 的 DFlash 和 DSpark 并行投机采样支持，解决了 310P 无法单步提供完整 block 服务的痛点 (#13311)。

**2. Bug 修复**
*   **KV Pool 初始化断言报错修复**：修复了在 `lazy_init` 模式下，MemcacheBackend 未初始化时调用 `batch_get_key_info` 触发 `assert self.store is not None` 的崩溃问题。此修复被广泛回传至 v0.23.0、v0.24.0、v0.25.1 及 main 分支 (#13307, #13308, #13309, #13310)。

**3. 工程与文档**
*   **CI 流程更新**：为 v0.25.1rc 和 v0.26.0rc 版本发布添加了相应的 CI 选项 (#13312)。
*   **文档更新**：修改了 DCP（上下文并行）支持的场景范围说明，明确在 950 场景下 SFA 不支持与 CP 叠加，而 MLA 和 GQA 已实验性支持 (#13306, #13305)。

---

### 🐛 Issue
*   本次提供的动态中无 Issue 相关信息。

---

### 🚀 Release
*   本次提供的动态中无正式 Release 发布信息。（注：从 PR 动态可见，社区正在积极筹备 **v0.25.1rc** 和 **v0.26.0rc** 版本，并对 v0.25.1 进行了 DeepSeek V4 支持与 KV Pool 缺陷的重点回传修复）。

---

## 🔀 Pull Requests

### #13314 — [[v0.25.1rc][BugFix][Frontend] Support DeepSeek V4 0731 reasoning effort](https://github.com/vllm-project/vllm-ascend/pull/13314)
- **作者**: QwertyJack  **时间**: 2026-08-01 07:43 CST
- **摘要**: ### What this PR does / why we need it?  This is the v0.25.1 release backport of #13313.  DeepSeek-V4-Flash-0731 defines three thinking levels with distinct prompt behavior:  - `low`: no reasoning-effort prefix - `high`: the existing `Reasoning Effort: Absolute maximum` prefix - `max`: the new `Reas…

### #13313 — [[BugFix][Frontend] Support DeepSeek V4 0731 reasoning effort](https://github.com/vllm-project/vllm-ascend/pull/13313)
- **作者**: QwertyJack  **时间**: 2026-08-01 01:39 CST
- **摘要**: ### What this PR does / why we need it?  DeepSeek-V4-Flash-0731 defines three thinking levels with distinct prompt behavior:  - `low`: no reasoning-effort prefix - `high`: the existing `Reasoning Effort: Absolute maximum` prefix - `max`: the new `Reasoning Effort: Beyond maximum` prefix  The vLLM ve…

### #13312 — [[CI] Add options for releases/v0.25.1rc and releases/0.26.0rc](https://github.com/vllm-project/vllm-ascend/pull/13312)
- **作者**: zhangxinyuehfad  **时间**: 2026-07-31 23:45 CST
- **摘要**: <!--  Thanks for sending a pull request!  BEFORE SUBMITTING, PLEASE READ https://docs.vllm.ai/en/latest/contributing/overview.html  --> ### What this PR does / why we need it?  Add options for releases/v0.25.1rc and releases/0.26.0rc  ### Does this PR introduce _any_ user-facing change? <!-- Note th…

### #13311 — [[310P][Performance] Support Qwen3 dflash/dspark for Ascend 310P](https://github.com/vllm-project/vllm-ascend/pull/13311)
- **作者**: ChunnX  **时间**: 2026-07-31 22:52 CST
- **摘要**: ### What this PR does / why we need it? Add DFlash and DSpark parallel drafting support for Atlas 300I (310P) on model_runner_v1. Both methods draft a whole block per step, which 310P could not serve before for two independent reasons: the existing split-fuse attention path is causal, and the input-…

### #13310 — [[BugFix][v0.25.1][KV Pool] Guard batch_get_key_info before memcache backend init](https://github.com/vllm-project/vllm-ascend/pull/13310)
- **作者**: tyy0829  **时间**: 2026-07-31 22:25 CST
- **摘要**: fix(kv_pool): guard batch_get_key_info before memcache backend init  When the MemcacheBackend store is not yet initialized (lazy_init mode), batch_get_key_info hit `assert self.store is not None` and crashed the scheduler/worker. Return an empty list instead, matching the list[KeyInfo] return contra…

### #13309 — [[BugFix][v0.24.0][KV Pool] Guard batch_get_key_info before memcache backend init](https://github.com/vllm-project/vllm-ascend/pull/13309)
- **作者**: tyy0829  **时间**: 2026-07-31 22:25 CST
- **摘要**: fix(kv_pool): guard batch_get_key_info before memcache backend init  When the MemcacheBackend store is not yet initialized (lazy_init mode), batch_get_key_info hit `assert self.store is not None` and crashed the scheduler/worker. Return an empty list instead, matching the list[KeyInfo] return contra…

### #13308 — [[BugFix][main][KV Pool] Guard batch_get_key_info before memcache backend init](https://github.com/vllm-project/vllm-ascend/pull/13308)
- **作者**: tyy0829  **时间**: 2026-07-31 22:24 CST
- **摘要**: fix(kv_pool): guard batch_get_key_info before memcache backend init  When the MemcacheBackend store is not yet initialized (lazy_init mode), batch_get_key_info hit `assert self.store is not None` and crashed the scheduler/worker. Return an empty list instead, matching the list[KeyInfo] return contra…

### #13307 — [[BugFix][v0.23.0][KV Pool] Guard batch_get_key_info before memcache backend init](https://github.com/vllm-project/vllm-ascend/pull/13307)
- **作者**: tyy0829  **时间**: 2026-07-31 22:02 CST
- **标签**: ready
- **摘要**: fix(kv_pool): guard batch_get_key_info before memcache backend init  When the MemcacheBackend store is not yet initialized (lazy_init mode), batch_get_key_info hit `assert self.store is not None` and crashed the scheduler/worker. Return an empty list instead, matching the list[KeyInfo] return contra…

### #13306 — [[DOC]Modify the scope of scenarios supported by DCP](https://github.com/vllm-project/vllm-ascend/pull/13306)
- **作者**: weiguihua2  **时间**: 2026-07-31 21:35 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? Modify the scope of scenarios supported by CP. In scenario 950, SFA does not support the overlay of CP, while MLA and GQA are experimentally supported.  ### Does this PR introduce _any_ user-facing change? yes  ### How was this patch tested?  - vLLM version: v…

### #13305 — [[DOC]Modify the scope of scenarios supported by DCP](https://github.com/vllm-project/vllm-ascend/pull/13305)
- **作者**: weiguihua2  **时间**: 2026-07-31 21:30 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? Modify the scope of scenarios supported by CP. In scenario 950, SFA does not support the overlay of CP, while MLA and GQA are experimentally supported.  ### Does this PR introduce _any_ user-facing change? yes  ### How was this patch tested?  - vLLM version: v…
