# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-09 15:03 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 📋 总览
近期仓库活动主要以代码贡献（PR）为主，涉及性能分析工具引入、底层 KV 池逻辑重构、多项 Bug 修复以及 CI/文档更新。近期未捕捉到新的 Issue 讨论与正式版本发布动态。

---

### 🚀 PR 动态（代码合并与提交）

**1. 新特性与重构**
*   **新增层级分析工具** (#21323)：引入了一个可选的层级接收队列分析器，用于排查 Ascend Store 中的队头阻塞 问题。
*   **重构 KV Pool 逻辑** (#21314)：去重了层级 AscendStore 数据平面的逻辑，将 mooncake 和 memcache 后端的通用代码进行了合并，无行为变更。

**2. 重要 Bug 修复**
*   **修复 EPLB 布局问题** (#21316)：针对 FRACTAL_NZ 物理布局问题，使用目标转换参数重建 Gloo NZ 接收逻辑，修复了权重转换带来的潜在错误。
*   **修复 mrope 编码预处理** (#21315)：支持 mrope 编码在 PCP 阶段的预处理。
*   **修复 CI 流水线问题** (#21319, #21320)：修复了二分查找基线路径匹配不精确的问题（按完整组件匹配），并优化了缓存的 PR head 刷新机制。

**3. 文档与弃用管理**
*   **刷新模型部署文档** (#21321, #21322)：更新了 GLM-5.3-Flash 950DT 的 PD 示例配置，采用 1M 配置并保持原有的 Prefill/Decode 启动器设置。
*   **功能弃用警告** (#21317)：为即将进入停止维护阶段的功能（动态 chunk size CPP 和 GQA DCP）添加了废弃警告及相关文档说明。
*   **杂项** (#21318)：清理了多余的描述文档。

---

### 📝 Issue 动态
近期无公开的 Issue 动态。

### 🏷️ Release 动态
近期无新的 Release 发布。（*注：PR #21322 标题中虽带有 `v0.32.0rc` 字样，但仅为更新该候选版本的配置文档，非正式发布。*）

---

## 🔀 Pull Requests

### #21323 — [[Feature][Profiler] Add layerwise receive queue profiling](https://github.com/vllm-project/vllm-ascend/pull/21323)
- **作者**: jryxxx  **时间**: 2026-11-09 15:02 CST
- **摘要**: ### What this PR does / why we need it?  Add an opt-in profiler for the layerwise receive queue to investigate head-of-line (HOL) blocking in Ascend Store.  Each task is associated with its request, layer and group. The timeline separates queue residence, save CPU/NPU waits, gate condition/NPU waits…

### #21322 — [[Doc][v0.32.0rc] Refresh GLM-5.3-Flash 950DT PD configuration](https://github.com/vllm-project/vllm-ascend/pull/21322)
- **作者**: Wyz-134  **时间**: 2026-11-09 14:57 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Refresh the GLM-5.3-Flash 950DT ordinary PD example with the supplied 1M configuration. Keep Prefill DP1/TP1/PCP8 and Decode DP8/TP1, the existing launcher, and proxy endpoints. Update role-specific scheduler limits, communication settings, optimization optio…

### #21321 — [[Doc] Refresh GLM-5.3-Flash 950DT PD configuration](https://github.com/vllm-project/vllm-ascend/pull/21321)
- **作者**: Wyz-134  **时间**: 2026-11-09 14:57 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Refresh the GLM-5.3-Flash 950DT ordinary PD example with the supplied 1M configuration. Keep Prefill DP1/TP1/PCP8 and Decode DP8/TP1, the existing launcher, and proxy endpoints. Update role-specific scheduler limits, communication settings, optimization optio…

### #21320 — [[BugFix][CI] Refresh cached bisect PR heads before resolving](https://github.com/vllm-project/vllm-ascend/pull/21320)
- **作者**: tianming2009  **时间**: 2026-11-09 14:57 CST
- **标签**: module:tests, module:tools
- **摘要**: ### What this PR does / why we need it?  Refresh the temporary PR head ref when resolving a PR number for bisect. Accept non-fast-forward updates to this cache ref and read it only after a successful fetch. This prevents a force-pushed or unavailable PR head from resolving to a previously cached com…

### #21319 — [[BugFix][CI] Match bisect baseline paths by complete components](https://github.com/vllm-project/vllm-ascend/pull/21319)
- **作者**: tianming2009  **时间**: 2026-11-09 14:57 CST
- **标签**: module:tests, module:tools
- **摘要**: ### What this PR does / why we need it?  Match bisect good-table YAML paths by complete path components. A query for `alpha/model.yaml` currently accepts `beta/model.yaml`, and `model.yaml` accepts `other-model.yaml`. A newer success for either unrelated configuration can supply both the Ascend base…

### #21318 — [[Doc][Misc] Delete the description](https://github.com/vllm-project/vllm-ascend/pull/21318)
- **作者**: ella1107  **时间**: 2026-11-09 14:52 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?   Delete the description  ### Does this PR introduce _any_ user-facing change?  No  ### How was this patch tested?  No - vLLM main: https://github.com/vllm-project/vllm/commit/544f182d9f0c9bfe8bd7645525951d5e3692a542

### #21317 — [[Platform][Doc] Add sunset deprecation warnings and documentation for dynamic chunk CPP and GQA DCP](https://github.com/vllm-project/vllm-ascend/pull/21317)
- **作者**: pisceskkk  **时间**: 2026-11-09 14:30 CST
- **标签**: documentation, module:core
- **摘要**: ### What this PR does / why we need it?  This PR introduces sunset deprecation warnings and documentation updates for two features that are entering the sunset process:  1. **Dynamic chunk size Chunked Pipeline Parallelism (CPP)**:    - Deprecation rationale: Dense models with long sequences previou…

### #21316 — [[BugFix][EPLB] Rebuild Gloo NZ receives with the destination's cast parameters](https://github.com/vllm-project/vllm-ascend/pull/21316)
- **作者**: tgrhn  **时间**: 2026-11-09 14:27 CST
- **标签**: module:tests, module:ops, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  RFC #20602 item 5.   FRACTAL_NZ is not a single physical layout: `npu_format_cast` derives the C0 tiling from `customize_dtype`/`input_dtype`, so a weight cast with them reports a C0 variant. A W4A8MXFP expert weight is cast with `customize_dtype=float8_e4m3f…

### #21315 — [[Bugfix] Support the preprocess of pcp for mrope encodings](https://github.com/vllm-project/vllm-ascend/pull/21315)
- **作者**: baolongsun  **时间**: 2026-11-09 14:26 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…

### #21314 — [[Refactor][KV Pool] Deduplicate layerwise AscendStore data-plane logic](https://github.com/vllm-project/vllm-ascend/pull/21314)
- **作者**: LeiW777  **时间**: 2026-11-09 14:23 CST
- **标签**: module:tests
- **摘要**: ## Summary  Deduplicate the plane-specific logic shared by the mooncake (block-key) and memcache (GVA) backends of the layerwise AscendStore KV pool. No behavior change is intended: both block-key session protocols were moved verbatim (verified by normalized diff against the pre-move code), and wire…
