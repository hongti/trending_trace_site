# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-24 13:14 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 📌 Issue (议题)
- **#17455 [RFC] 围绕 MRV2 合约重构 Ascend 310P 边缘运行时**
  提议对 Ascend 310P 的 MoE MLP 实现进行重构，使其向“由量化方法接管权重访问和执行钩子”的架构演进，以更好地适配 MRV2 合约规范。

### 🔧 Pull Requests (合并请求)
近期 PR 主要集中在官方算子迁移、新特性支持及稳定性修复上：

**1. 重构与算子迁移**
- **#17464 & #17462 迁移至官方 CANN 算子**：将 DeepSeek V4 的 A5 和 A3 版本的 QLI V2 及 Metadata 调用，全面迁移至官方 CANN 算子接口（ACLNN 绑定及 `cann_ops_transformer.ops`），支持常规及上下文并行路径。A2/A3/A5 路由互不干扰。

**2. 新特性与功能优化**
- **#17456 [PCP] 核心解码请求分片隔离**：优化 PCP（并行上下文）解码请求分片机制，确保 Ascend attention 能正确复制每个 owner 的 KV 更新并处理空 owner 情况。
- **#17454 [Mamba] 调度器检查点协调**：在 Ascend 调度器中引入应用导向的 Mamba 检查点协调机制（目前为 Draft 状态，等待上游 vLLM 相关 PR 合并后推进）。
- **#17459 & #17458 ZeroMoE 与 GDN 融合**：在 A2 平台集成 ZeroMoE 特性，并实现 GDN 算子融合及相关 v0.26.0rc 版本的适配。

**3. Bug 修复**
- **#17463 修复内核编译错误**：为 `CompressorMetadataKernel::Init` 的 `tilingData` 参数添加 `const` 修饰符，修复静态编译错误并提升图构建稳定性。
- **#17461 修复 MRv2 MTP 状态标记**：修正了 MRv2 MTP 误将统一目标解码标记为 `SpecDecoding` 的问题，统一改用 `DecodeOnly` 以防状态继承错误。

**4. CI 改进**
- **#17457 提升 CI 分区并发数**：提高了 A3（双/四卡）和 A2（单卡）测试套件的负载均衡分区数量，以缩短全量测试的执行时间。

### 🚀 Release (版本发布)
- 本周期内无新版本发布。

---

## 🐛 Issues

### #17455 — [[RFC]: Refactor the Ascend 310P edge runtime around MRV2 contracts](https://github.com/vllm-project/vllm-ascend/issues/17455)
- **作者**: YangShuai52  **时间**: 2026-09-24 11:36 CST
- **标签**: RFC
- **摘要**: ### Motivation.  [PR #14496](https://github.com/vllm-project/vllm-ascend/pull/14496) moved the 310P MoE MLP implementation toward quantization-method-owned weight access and execution hooks. That is a useful local refactor, but the rest of the 310P execution path still has boundaries that are diffic…

## 🔀 Pull Requests

### #17464 — [[Refactor][A5] Use official CANN QLI V2 with isolated bindings](https://github.com/vllm-project/vllm-ascend/pull/17464)
- **作者**: massmass  **时间**: 2026-09-24 12:30 CST
- **标签**: documentation, module:tests
- **摘要**: ## Scope - Migrate only DeepSeek V4 Ascend 950 (A5) QLI V2 and Metadata call sites, including context parallel, to paired official CANN ACLNN bindings (FP8 quant_mode=1). - Keep A2/A3 on their existing calls. Keep the custom QLI V2 OPP for DeepSeek V4.1 candidate/V3 use. - Resolve official QLI and M…

### #17463 — [[BugFix][Kernel] Mark tilingData as const in CompressorMetadataKernl](https://github.com/vllm-project/vllm-ascend/pull/17463)
- **作者**: ZT-AIA  **时间**: 2026-09-24 12:13 CST
- **摘要**: ### What this PR does / why we need it? Add `const` qualifier to the `tilingData` parameter of `CompressorMetadataKernel::Init` to fix static compilation errors and improve graph-building stability in SK scenarios.  Cherry-pick from PR #16334.  ### Does this PR introduce _any_ user-facing change? No…

### #17462 — [[Refactor] Replace A3 QLI and metadata with official CANN operators](https://github.com/vllm-project/vllm-ascend/pull/17462)
- **作者**: massmass  **时间**: 2026-09-24 12:11 CST
- **标签**: module:tests
- **摘要**: ## Scope - Route DeepSeek V4 A3 QLI V2 and QLI Metadata to the official cann_ops_transformer.ops interface in normal and context-parallel paths. Keep A2/A5 routing unchanged. - Exclude only these two custom OPP kernels from the A3 build list. Retain compatibility bindings for other devices. - Add CP…

### #17461 — [[BugFix] Use DecodeOnly for MRv2 MTP attention state](https://github.com/vllm-project/vllm-ascend/pull/17461)
- **作者**: drslark  **时间**: 2026-09-24 12:00 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  MRv2 MTP currently labels uniform target decode as `SpecDecoding`, and the first draft step inherits that target metadata. Use `DecodeOnly` for both single-token decode and MTP verification batches with one valid token. The MLA attention layout already handle…

### #17460 — [Fix/gdn fla v0.26.0rc](https://github.com/vllm-project/vllm-ascend/pull/17460)
- **作者**: huoyibingli  **时间**: 2026-09-24 11:56 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/d02df748bf9efd99022f1a062597dc3cb3808485

### #17459 — [Feature/zercmoe a2 integration](https://github.com/vllm-project/vllm-ascend/pull/17459)
- **作者**: huoyibingli  **时间**: 2026-09-24 11:53 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/d02df748bf9efd99022f1a062597dc3cb3808485

### #17458 — [gdn 融合](https://github.com/vllm-project/vllm-ascend/pull/17458)
- **作者**: huoyibingli  **时间**: 2026-09-24 11:52 CST
- **标签**: documentation, ci/build, module:tests, module:ops, module:core, module:quantization, module:tools
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/ced6857afa0ea7b2e3f0846a62e1394e90f15607

### #17457 — [[CI] Raise partition counts for A3 two/four-card and A2 one-card suites](https://github.com/vllm-project/vllm-ascend/pull/17457)
- **作者**: zhangdiago  **时间**: 2026-09-24 11:47 CST
- **摘要**: ## What this PR does / why we need it?  Raises the load-balanced partition counts in `test_config.yaml` so full-suite runs (ready-all / main2main) spread the selected files across one extra runner per hardware partition:  - `a3-2`: 4 → 5 (slowest shard ~62.8 → ~51.2 min) - `a3-4`: 4 → 5 (slowest sha…

### #17456 — [[Feat][PCP] Isolate the core decode request sharding adaptation](https://github.com/vllm-project/vllm-ascend/pull/17456)
- **作者**: recky-c  **时间**: 2026-09-24 11:46 CST
- **标签**: module:core
- **摘要**: ### What this PR does / why we need it?  PCP decode request sharding assigns each decode request to one PCP rank. Ascend attention must still replicate every owner's KV update and allow empty owners to participate in collectives before running owner-local attention.  This is a separate, production-c…

### #17454 — [[Feature][Mamba] Application-directed Mamba checkpoint coordination in the Ascend scheduler (spike for #17352)](https://github.com/vllm-project/vllm-ascend/pull/17454)
- **作者**: curnane-lab  **时间**: 2026-09-24 11:23 CST
- **摘要**: > **Draft — based on vllm-project/vllm#55873 and vllm-project/vllm#55875 (unmerged). Do not merge until the upstream series lands.** This PR tracks the Ascend-side portion of the port proposed in #17352 so the rebase is ready when upstream merges.  ### What this PR does / why we need it?  vLLM core …
