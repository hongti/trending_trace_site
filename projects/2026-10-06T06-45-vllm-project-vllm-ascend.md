# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-06 14:45 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📌 Issue
- **#17923 KV-pool 查找服务线程异常时静默退出**：通过代码审查发现，当外部查找服务器的 handler 发生异常时，线程会静默崩溃，导致调度器无限挂起。

### 🛠 Pull Request (PR)
近期合并的 PR 主要集中在 **Bug 修复**，重点覆盖 KV Cache、模型结构及算子通信等方面：
- **KV Cache 优化与修复**：
  - **#17924** 保持 Paged Attention 的 K/V 内存连续性，修复非连续分配导致的解码性能下降问题。
  - **#17917** 将非连续 KV Cache 布局限制于混合 Attention/Mamba 模型，修复了纯 GQA 模型的吞吐量回退。
  - **#17918** 恢复使用 `auto` dtype 时的静态量化 MLA Cache 规划，修复了 Kimi-K2.6 (W4A4C8 + Eagle3) 的容量计算失败。
- **模型适配与修复**：
  - **#17922** 修复 GLM-5.3-Flash 在 C8 KV Cache 下缓存分组报错的问题，保持混合 block 对齐。
  - **#17919** 修复指定自定义推测模型时 GLM C8 的异常，增加对 `model_config` 和 `use_mla` 的检查。
  - **#17921** 优化 DeepSeek V4 IndexCache 初始化逻辑，使其支持跨 PP 阶段切分的重用组。
- **MoE 与量化相关**：
  - **#17916** 修复 MoE 通信方式解析错误，解决 AscendMoERunner 注册特定形状通信导致的问题。
  - **#17915** 修复 W4A8 MXFP EPLB 运行时问题，启用针对 W4A8 MoE 权重的 EPLB 并修正 Float4 专家权重物化路径。
- **算子与 CI**：
  - **#17914** 修复 `dispatch_ffn_combine` 系列算子中隐式查询 HCCL 组大小导致的 GE 拓扑依赖问题，改为显式传递 `worldSize`。
  - **#17920** 修复 CI 周期测试用例。

### 🚀 Release
- 近期无新版本发布。

---

## 🐛 Issues

### #17923 — [[Bug]: KV-pool lookup server thread dies silently on any handler exception, hanging the scheduler indefinitely](https://github.com/vllm-project/vllm-ascend/issues/17923)
- **作者**: do420  **时间**: 2026-10-06 13:52 CST
- **标签**: bug
- **摘要**: ### Your current environment  Found by code inspection on `main` (`f7cac7a`) while assessing #14151. Affects the `ascend_store` KV-pool path (`AscendStoreConnector` with the external lookup server enabled). No NPU needed to reproduce — the failure is in the ZMQ REQ/REP control path.  ### Suggested f…

## 🔀 Pull Requests

### #17924 — [[BugFix][KV Cache] Keep KV cache contiguous for paged attention](https://github.com/vllm-project/vllm-ascend/pull/17924)
- **作者**: zhaochuang001  **时间**: 2026-10-06 14:21 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Keep K/V contiguous for ordinary FullAttention layers that may use paged attention (PA). The non-contiguous block-major allocation causes a substantial decode performance regression in the QwQ-32B PA configuration.  Follow-up to #14340. Related failing CI: [Q…

### #17922 — [[BugFix][GLM] Preserve packed C8 KPool hybrid block alignment](https://github.com/vllm-project/vllm-ascend/pull/17922)
- **作者**: YanpengDing  **时间**: 2026-10-06 11:55 CST
- **标签**: module:tests, module:core, ready-precise
- **摘要**: ### What this PR does / why we need it?  GLM-5.3-Flash with C8 KV cache can fail during cache grouping with:  ```text ValueError: GLM-Next main MLA and compressed indexer caches must use one logical block size. ```  Ascend's initial hybrid configuration includes the packed C8 scale bytes and chooses…

### #17921 — [[BugFix][Model] Initialize V4 IndexCache at the first C4 of each PP stage](https://github.com/vllm-project/vllm-ascend/pull/17921)
- **作者**: zhenwenqi2024  **时间**: 2026-10-06 11:15 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Make DeepSeek V4 IndexCache work with PP partitions that split a reuse group, while keeping the existing reuse schedule when the partition is already aligned.  V4 attaches an Indexer only to C4 layers. The model's frequency/pattern schedule uses C4 Indexer or…

### #17920 — [[CI] fix weekly cases](https://github.com/vllm-project/vllm-ascend/pull/17920)
- **作者**: chen-commits  **时间**: 2026-10-06 10:51 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/ced6857afa0ea7b2e3f0846a62e1394e90f15607

### #17919 — [[BugFix] Fix GLM C8 When Given Custom Speculative Model](https://github.com/vllm-project/vllm-ascend/pull/17919)
- **作者**: lcfenglinwan  **时间**: 2026-10-06 09:56 CST
- **标签**: module:quantization, ready-precise
- **摘要**: ### What this PR does / why we need it? This PR updates `enable_fa_quant` in `vllm_ascend/quantization/utils.py` to check if `model_config` is present and if `use_mla` is enabled before proceeding. It also disables FlashAttention quantization if `enable_sfa(vllm_config)` is true.  ### Does this PR i…

### #17918 — [[BugFix][KV Cache] Restore static quantized MLA cache planning with auto dtype](https://github.com/vllm-project/vllm-ascend/pull/17918)
- **作者**: ParadiseHeaven  **时间**: 2026-10-06 07:45 CST
- **标签**: module:tests, module:quantization
- **摘要**: ### What this PR does / why we need it?  Fix the FA MLA cache capacity failure with the unchanged Kimi-K2.6 W4A4C8 + Eagle3 command and default KV cache dtype `auto`. On paired vLLM `ced6857afa0ea7b2e3f0846a62e1394e90f15607` and vLLM-Ascend `9b8fc5d728e1ea54dc277562b296112d905e15c1`, planning the wh…

### #17917 — [[BugFix][KV Cache] Restrict non-contiguous KV cache to hybrid models to fix pure-GQA throughput regression](https://github.com/vllm-project/vllm-ascend/pull/17917)
- **作者**: chenzhh69  **时间**: 2026-10-06 04:36 CST
- **摘要**: ### What this PR does / why we need it? This PR restricts the non-contiguous KV cache layout to hybrid Attention/Mamba models: `strided_attention_cache_layers` is gated on `self.hybrid_with_attn_and_mamba` in `model_runner_v1.py`, and the combined KV allocation is gated on any `MambaSpec` in `attn_u…

### #17916 — [[BugFix][MoE] Resolve communication methods with MoE config](https://github.com/vllm-project/vllm-ascend/pull/17916)
- **作者**: hust17yixuan  **时间**: 2026-10-06 04:35 CST
- **标签**: module:tests, module:ops, ready-precise
- **摘要**: ### What this PR does / why we need it?  The MoE communication registry is keyed by both communication type and the expert shape from FusedMoEConfig. AscendMoERunner registered shape-specific communication instances but retrieved FusedMC2 and AlltoAll without the config, which looked up the (0, 0) k…

### #17915 — [[BugFix][Quantization] Fix W4A8 MXFP EPLB runtime](https://github.com/vllm-project/vllm-ascend/pull/17915)
- **作者**: hust17yixuan  **时间**: 2026-10-06 03:56 CST
- **标签**: module:tests, module:quantization
- **摘要**: ### What this PR does / why we need it?  This PR enables EPLB for W4A8 MXFP MoE weights and fixes the Float4 expert-weight materialization path used by EPLB.  - Exposes persistent per-expert weight and scale tensors for EPLB. - Adds W4A8 MXFP expert mappings and private-format receive buffers to the…

### #17914 — [[Ops][BugFix] Pass worldSize explicitly to dispatch_ffn_combine ops to avoid GE topology dependency](https://github.com/vllm-project/vllm-ascend/pull/17914)
- **作者**: ZT-AIA  **时间**: 2026-10-06 01:00 CST
- **标签**: module:tests, module:ops, ready-precise
- **摘要**: ## Motivation  Fixes #17904.  The `dispatch_ffn_combine` / `dispatch_ffn_combine_w4_a8` / `dispatch_ffn_combine_bf16` tiling queried the HCCL group rank size via `ge::HcomTopoInfo::GetGroupRankSize`. On the ACLNN path, the GE graph execution environment may not be initialized and the HCCL group may …
