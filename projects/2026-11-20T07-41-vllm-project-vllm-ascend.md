# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-20 15:41 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🐛 Issue (问题)
*   **#22616 [Bug][Upstream]**：在 vLLM 0.33.0 与 Ascend NPU 环境下，`granite-swash-3b-a600m` 模型在图捕获阶段触发 `aclnnFusedInferAttentionScoreV4` 报错（error 561），导致 `test_granite.py` 测试失败。该问题被标记为上游缺陷。

### 🔧 Pull Request (拉取请求)
**1. 新特性与模型支持**
*   **#22623**：为 `Qwen/Qwen3.5-122B-A10B` 添加了华为 Ascend 硬件的 Overlay 配置，包含 NPU 专属的启动参数与环境设置。
*   **#22617**：为 `MiniMax-M3.1-Flash` 添加了 Ascend Overlay 配置文件，方便 recipes.vllm.ai 渲染。

**2. 重要重构**
*   **#22624**：重构 DeepSeek V4.1 的 KV 规划逻辑，将其分组、布局及容量计算委托给上游 vLLM（main128/SWA32 pages），同时保留 Ascend 侧的组件视图、跨步压缩机环及完整的 Mooncake 传输特性。

**3. 关键 Bug 修复**
*   **#22621**：修复 KV Transfer 中 AscendStore 保存事件 ID 冲突问题，改用 `uuid.uuid4().int` 循环生成唯一 ID，避免重复注册。
*   **#22620**：修复 v0.30.0 运行时依赖问题，声明 `ml_dtypes>=0.5.4,<0.7` 以拉取受支持的依赖版本，并升级 memfabric 等组件。

**4. 文档与杂项**
*   **#22622**：移除了 17 个模型教程 Docker 示例中的 `--privileged` 特权模式（共 36 处）。
*   **#22618**：更新贡献指南，要求在 commit message 中明确记录性能测试环境（如 A3/A5 硬件、Prefill/Decode 阶段）与验证结果。
*   **#22619**：将上游 vLLM (codex/vllm 61452) 的变更同步合并至 Ascend main 分支。
*   **#22615**：实现 Direct reload（直接重载）机制。
*   **#22614**：优化 850E 相关文档的翻译。

### 🚀 Release (版本发布)
*   本期动态中暂无新版本发布。

---

## 🐛 Issues

### #22616 — [[Bug][Upstream]: granite-swash-3b-a600m fails in aclnnFusedInferAttentionScoreV4 (error 561) during graph capture case failed: tests/models/language/generation/test_granite.py](https://github.com/vllm-project/vllm-ascend/issues/22616)
- **作者**: meiling6226  **时间**: 2026-11-20 15:09 CST
- **摘要**: ### Your current environment  vllm 0.33.0 (upstream release) vllm-ascend commit from upstream-test weekly CI image NPU backend (Ascend)  Job: `upstream-test` weekly CI v0.33.0 shard.  ### 🐛 Describe the bug  `tests/models/language/generation/test_granite.py`  Result: 1 failed, 6 passed.  1. `test_mo…

## 🔀 Pull Requests

### #22624 — [[Refactor]use upstream KV planning for DeepSeek V4.1](https://github.com/vllm-project/vllm-ascend/pull/22624)
- **作者**: Angazenn  **时间**: 2026-11-20 15:39 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: Delegate V4.1 grouping, placement and capacity accounting to upstream vLLM with main128/SWA32 pages. Keep Ascend component views, strided compressor rings and complete-page Mooncake transfers without extending upstream cache descriptors.  Validation: Ruff lint/format, Python AST syntax, whitespace, …

### #22623 — [[Ops][Feature] Add Ascend overlay for Qwen/Qwen3.5-122B-A10B](https://github.com/vllm-project/vllm-ascend/pull/22623)
- **作者**: jinersu-06  **时间**: 2026-11-20 15:34 CST
- **标签**: recipe
- **摘要**: ### What this PR does / why we need it?  Add the Huawei Ascend hardware overlay for `Qwen/Qwen3.5-122B-A10B` under `recipes/overlays/`, carrying the NPU-specific launch arguments, environment, image pin and strategy support captured from a live vLLM-Ascend deployment.  ### Does this PR introduce _an…

### #22622 — [[Doc] Remove privileged mode from model Docker examples](https://github.com/vllm-project/vllm-ascend/pull/22622)
- **作者**: shuomaster  **时间**: 2026-11-20 15:33 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Remove `--privileged=true` and `--privileged` from Docker commands in 17 model tutorials (36 occurrences), following #22360. The examples already pass the required NPU devices explicitly, so they do not need privileged mode.  ### Does this PR introduce _any_ …

### #22621 — [[BugFix][KV Transfer] Allocate unique AscendStore save event IDs](https://github.com/vllm-project/vllm-ascend/pull/22621)
- **作者**: cocobronie2333-dot  **时间**: 2026-11-20 15:28 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Generate AscendStore in-flight save event IDs with `uuid.uuid4().int`. If a generated ID is already tracked, generate another before registering the save. Worker completions still release the matching KV blocks after all workers acknowledge the save.  ### Doe…

### #22620 — [[BugFix][Dependency] Update runtime requirements for v0.30.0](https://github.com/vllm-project/vllm-ascend/pull/22620)
- **作者**: tangdafu  **时间**: 2026-11-20 15:28 CST
- **摘要**: ### What this PR does / why we need it?  - Declare `ml_dtypes>=0.5.4,<0.7` so dependency installation pulls in a supported version. - Include the existing release-branch commit that bumps memfabric and memcache hybrid to 1.3.1.  ### Does this PR introduce _any_ user-facing change?  Yes. The versions…

### #22619 — [[Misc][Main2main]Codex/vllm 61452 ascend main](https://github.com/vllm-project/vllm-ascend/pull/22619)
- **作者**: Angazenn  **时间**: 2026-11-20 15:25 CST
- **标签**: module:tests, module:ops, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…

### #22618 — [[Doc] Document commit message content and validation results](https://github.com/vllm-project/vllm-ascend/pull/22618)
- **作者**: zhenwenqi2024  **时间**: 2026-11-20 15:21 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  The contribution guide currently shows a subject-only commit. A reviewer reading a reported speedup cannot tell whether it was measured on A3 or A5, or with prefill and decode colocated or disaggregated.  Document what to put in the commit body under Problem,…

### #22617 — [[Doc][Recipe] Add Ascend overlay for MiniMax-M3.1-Flash](https://github.com/vllm-project/vllm-ascend/pull/22617)
- **作者**: U1stRsouland  **时间**: 2026-11-20 15:12 CST
- **标签**: recipe
- **摘要**: ### What this PR does / why we need it? Add `recipes/overlays/MiniMaxAI/MiniMax-M3.1-Flash.yaml`, migrated from `docs/source/tutorials/models/MiniMax-M3.1-Flash.md`, so recipes.vllm.ai can render the Ascend deployment lanes for this model on top of the upstream base recipe.  - Two Ascend variants: W…

### #22615 — [Direct reload](https://github.com/vllm-project/vllm-ascend/pull/22615)
- **作者**: Liccol  **时间**: 2026-11-20 15:09 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…

### #22614 — [[Doc][Misc] Optimize the translation of 850E](https://github.com/vllm-project/vllm-ascend/pull/22614)
- **作者**: ella1107  **时间**: 2026-11-20 15:07 CST
- **摘要**: ### What this PR does / why we need it? Optimize the translation of 850E  ### Does this PR introduce _any_ user-facing change? No  ### How was this patch tested? No - vLLM main: https://github.com/vllm-project/vllm/commit/c410543d9ffff03f5f6c4a8c0ae1d3ad082c4a15
