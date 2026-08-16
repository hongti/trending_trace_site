# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-16 10:09 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue 动态

*   **核心 Bug：DeepSeek V4 模型输出异常** (#14363)
    DeepSeek V4 适配器未将 `swiglu_limit` 参数传递给 routed expert（路由专家），这是导致模型生成中文及特殊 token 异常的根本原因。
*   **兼容性 Bug：CANN 9.1.0 Profiling 分析失败** (#14359)
    在 CANN 9.1.0 环境下，虽然能采集原始 Profiling 数据，但后续的数据分析步骤会失败。
*   **性能提议：扩展 `use_layerwise` 支持** (#14357)
    社区提出目前 `use_layerwise` 特性仅支持 memcache，询问是否有计划使其支持 Mooncake 后端以提升性能。
*   **CI 异常：main2main 自动化中断** (#14361)
    main2main 自动化流程未能完成所有计划步骤，需要人工介入审查。

---

### 🔀 Pull Request 动态

*   **[重要修复] 传递 DeepSeek V4 SwiGLU limit** (#14364)
    修复上述 #14363 Bug，将 `config.swiglu_limit` 正确传递给 routed `FusedMoE`，解决中文/特殊 token 生成异常问题（针对 v0.25.1 版本）。
*   **[重要修复] 修复 MegaMoe prefill 缓冲区大小计算** (#14358)
    在 `enable_prefill_mc2` 被禁用的情况下，修正了 CANN MegaMoe 对称通信缓冲区的分配逻辑（针对 v0.26.0rc 版本）。
*   **[依赖管理] 锁定 KV Pool 依赖版本** (#14352, #14353, #14354, #14355, #14356)
    为提升构建稳定性，跨多个版本分支（v0.22.1rc / v0.23.0 / v0.25.1rc / v0.26.0rc / main）锁定了 `memfabric_hybrid` 和 `memcache_hybrid` 的版本。其中 v0.22.1rc 锁定至 1.1.4，其余分支锁定至 1.2.0。
*   **[文档] 自动翻译更新** (#14365)
    自动翻译并更新了 9 个中文（zh_CN）文档文件。
*   **[CI/构建] 适配与测试调整** (#14360, #14362)
    尝试适配 vLLM main 分支 (3ac95255)，但自动化流程失败未完成步骤；另外在 main_verify 测试中跳过了 pre-commit hooks 检查。

---

### 🚀 Release 动态

*   本期暂无新版本发布。

---

## 🐛 Issues

### #14363 — [[Bug]: DeepSeek V4 adapter drops routed expert swiglu_limit](https://github.com/vllm-project/vllm-ascend/issues/14363)
- **作者**: QwertyJack  **时间**: 2026-08-16 09:38 CST
- **标签**: llm-model, deepseek
- **摘要**: ### Bug description  This is the root cause of the abnormal Chinese/special tokens reported in #13615.  DeepSeek V4 checkpoints configure `swiglu_limit=10.0`. The Ascend `DeepseekV4MoE` adapter passes this value to the shared expert, but omits it when constructing the routed `FusedMoE`:  ```python s…

### #14361 — [[main2main] main2main manual review required ()](https://github.com/vllm-project/vllm-ascend/issues/14361)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-16 02:02 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/14360 - Commit range: `1d2d83a07fd3180068917a031c28dcc83141d0be`...`` - Status: `failed`  ## Final Summary  main2main adaptation failed — no steps c…

### #14359 — [[Bug]: Profiling data analysis fails with CANN 9.1.0](https://github.com/vllm-project/vllm-ascend/issues/14359)
- **作者**: yiz-liu  **时间**: 2026-08-16 01:44 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Collecting environment information... PyTorch version: 2.10.0+cpu Is debug build: False  OS: Ubuntu 26.04 LTS (aarch64) GCC version: (Ubuntu 15.2.0-16ubuntu1) 15.2.0 Clang version: Could not col…

### #14357 — [[Performance]: Currently, the use_layerwise feature only supports memcache. Does the community have any plans to enable use_layerwise to support Mooncake-backend?](https://github.com/vllm-project/vllm-ascend/issues/14357)
- **作者**: weim0000  **时间**: 2026-08-15 21:39 CST
- **标签**: performance
- **摘要**: ### Proposal to improve performance  Currently, the use_layerwise feature only supports memcache. Does the community have any plans to enable use_layerwise to support Mooncake-backend?  ### Report of performance regression  Currently, the use_layerwise feature only supports memcache. Does the commun…

## 🔀 Pull Requests

### #14365 — [[Doc] Translated Doc files 2026-08-16](https://github.com/vllm-project/vllm-ascend/pull/14365)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-16 09:58 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **9** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/slash-commands.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/tutorials/models/DeepSeek-V4-Flas…

### #14364 — [[BugFix][v0.25.1][Model] Propagate DeepSeek V4 routed SwiGLU limit](https://github.com/vllm-project/vllm-ascend/pull/14364)
- **作者**: QwertyJack  **时间**: 2026-08-16 09:40 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  DeepSeek V4 reads `config.swiglu_limit` and already passes it to the shared expert, but the Ascend model adapter does not pass it to the routed `FusedMoE`. As a result, checkpoints with `swiglu_limit=10.0` execute routed experts without the required gate/up c…

### #14362 — [main_verify:test: skip pre-commit hooks](https://github.com/vllm-project/vllm-ascend/pull/14362)
- **作者**: czydyy  **时间**: 2026-08-16 03:19 CST
- **标签**: module:tools
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14360 — [[Misc]feat: adapt to vLLM main (3ac95255)](https://github.com/vllm-project/vllm-ascend/pull/14360)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-16 02:02 CST
- **标签**: module:tests, module:ops
- **摘要**: main2main adaptation failed — no steps completed.  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14358 — [[v0.26.0rc][BugFix][MoE] Fix MegaMoe prefill buffer sizing](https://github.com/vllm-project/vllm-ascend/pull/14358)
- **作者**: jianzs  **时间**: 2026-08-15 23:52 CST
- **标签**: module:ops, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  CANN MegaMoe allocates its symmetric communication buffer once from the MC2 token capacity. When `enable_prefill_mc2` is disabled, that capacity was derived from the decode graph capture size even though eager prefill could still select MegaMoe through fused …

### #14356 — [[v0.26.0rc][Misc] Pin MemFabric and MemCache dependency versions](https://github.com/vllm-project/vllm-ascend/pull/14356)
- **作者**: zouyida2052  **时间**: 2026-08-15 20:08 CST
- **摘要**: ## What this PR does  Pins the KV Pool dependencies to exact versions:  - `memfabric_hybrid==1.2.0` - `memcache_hybrid==1.2.0`  ## Changes  - Update only the root `requirements.txt`.  ## Validation  - `git diff --check` - Verified the commit changes only `requirements.txt`.  - vLLM version: v0.26.0 …

### #14355 — [[Misc] Pin MemFabric and MemCache dependency versions](https://github.com/vllm-project/vllm-ascend/pull/14355)
- **作者**: zouyida2052  **时间**: 2026-08-15 20:08 CST
- **摘要**: ## What this PR does  Pins the KV Pool dependencies to exact versions:  - `memfabric_hybrid==1.2.0` - `memcache_hybrid==1.2.0`  ## Changes  - Update only the root `requirements.txt`.  ## Validation  - `git diff --check` - Verified the commit changes only `requirements.txt`.  - vLLM version: v0.27.1 …

### #14354 — [[v0.25.1rc][Misc] Pin MemFabric and MemCache dependency versions](https://github.com/vllm-project/vllm-ascend/pull/14354)
- **作者**: zouyida2052  **时间**: 2026-08-15 20:08 CST
- **摘要**: ## What this PR does  Pins the KV Pool dependencies to exact versions:  - `memfabric_hybrid==1.2.0` - `memcache_hybrid==1.2.0`  ## Changes  - Update only the root `requirements.txt`.  ## Validation  - `git diff --check` - Verified the commit changes only `requirements.txt`.  - vLLM version: v0.25.1 …

### #14353 — [[v0.22.1rc][Misc] Pin MemFabric and MemCache dependency versions](https://github.com/vllm-project/vllm-ascend/pull/14353)
- **作者**: zouyida2052  **时间**: 2026-08-15 20:08 CST
- **摘要**: ## What this PR does  Pins the KV Pool dependencies to exact versions:  - `memfabric_hybrid==1.1.4` - `memcache_hybrid==1.1.4`  ## Changes  - Update only the root `requirements.txt`.  ## Validation  - `git diff --check` - Verified the commit changes only `requirements.txt`.  - vLLM version: v0.22.1 …

### #14352 — [[v0.23.0][Misc] Pin MemFabric and MemCache dependency versions](https://github.com/vllm-project/vllm-ascend/pull/14352)
- **作者**: zouyida2052  **时间**: 2026-08-15 20:08 CST
- **摘要**: ## What this PR does  Pins the KV Pool dependencies to exact versions:  - `memfabric_hybrid==1.2.0` - `memcache_hybrid==1.2.0`  ## Changes  - Update only the root `requirements.txt`.  ## Validation  - `git diff --check` - Verified the commit changes only `requirements.txt`.  - vLLM version: v0.23.0 …
