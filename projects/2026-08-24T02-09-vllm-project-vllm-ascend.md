# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-24 10:09 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库最近动态的中文摘要：

### 🐛 Issue 动态
- **CI 流程异常**：`main2main` 自动化流程执行失败，停留在中间状态，需人工审查介入（#14785）。
- **模型 Bug 反馈**：在 NPU 910 A3 环境下，按官方文档部署 Qwen 3.8 模型时，Tool Call（工具调用）功能执行失败（#14781）。

### 🔧 PR 动态
**1. 新特性与模型支持**
- **支持 MiniMax M3 模型**：适配 CANN MegaMoE 路径，为其转发特定的 `swigluoai_uninterleave` 激活函数及相关参数（#14784）。
- **支持 Kimi-K3 DCP 解码**：将 Kimi-K3 的 DCP decode 功能移植到 main 分支（#14782）。
- **支持 A5 MegaMoE**：新增对 A5 架构 MegaMoE 的支持（#14780）。

**2. 架构与功能重构**
- **KV Cache 量化参数化**：重构量化逻辑，新增 `--kv-quant-dtype` 启动参数，允许用户指定 KV Cache 的量化数据类型（#14786）。

**3. 性能优化**
- **融合 DCP 索引重映射与压缩**：针对 DCP 稀疏注意力路径，将原本需要 25 个算子的 PyTorch `sort + gather` 链融合优化，显著提升性能（#14787）。
- **优化 GLM-5.1 IndexCache**：跳过 GLM-5.1 静态 S 层的逐层索引器缓存（因其运行时复用其他层的 top-k 索引），减少冗余计算（#14783）。

### 🚀 Release 动态
- 近期**无**新版本发布。

---

## 🐛 Issues

### #14785 — [[main2main] main2main manual review required ()](https://github.com/vllm-project/vllm-ascend/issues/14785)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-24 00:04 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR:  - Commit range: `27ec8ac626345498fdac0527a4fcd1451c24bebc`...`` - Status: `failed`  ## Final Summary  main2main adaptation failed — no steps completed.

### #14781 — [[Bug]: Qwen 3.8 tool call failed!](https://github.com/vllm-project/vllm-ascend/issues/14781)
- **作者**: gbdjxgp  **时间**: 2026-08-23 16:33 CST
- **标签**: bug
- **摘要**: ### Your current environment  npu:910 A3 follow the qwen 3.8 deployment guide: https://docs.vllm.ai/projects/ascend/en/latest/tutorials/models/Qwen3.8-27B.html#31-model-weight  ### 🐛 Describe the bug  ```bash  curl -X POST http://localhost:9010/v1/chat/completions \ >   -H "Content-Type: application…

## 🔀 Pull Requests

### #14787 — [perf: fuse DCP index remap and stable compaction](https://github.com/vllm-project/vllm-ascend/pull/14787)
- **作者**: qqq1357962  **时间**: 2026-08-24 10:00 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  The DCP sparse-attention path currently remaps and stably compacts every TopK index row with a 25-kernel PyTorch `sort + gather` chain. This occurs once per attention layer and adds about 12 ms to every GLM-5.2 decode iteration at TP16/DCP16/MTP3.  This PR:  …

### #14786 — [[Refactor][Quantization]KVCache quantization dtype specified by the --kv-quant-dtype parameter](https://github.com/vllm-project/vllm-ascend/pull/14786)
- **作者**: lcfenglinwan  **时间**: 2026-08-24 09:34 CST
- **标签**: module:core, module:quantization, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/cdc4824a21eaa986d4d1fee90a7e6465c9f706e6

### #14784 — [[Feature][Model] Support MiniMax M3 with CANN MegaMoE](https://github.com/vllm-project/vllm-ascend/pull/14784)
- **作者**: CXY-Katrina  **时间**: 2026-08-23 23:59 CST
- **标签**: module:tests, module:ops, module:core, module:quantization, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  MiniMax M3 uses the `swigluoai_uninterleave` activation, so its MegaMoE path must forward the activation name together with the clamp, alpha, and beta parameters.  Assuming the latest `cann_ops_transformer` package provides `swigluoai` support, this PR keeps …

### #14783 — [[Performance] skip per-layer indexer cache for GLM-5.1 IndexCache S layers](https://github.com/vllm-project/vllm-ascend/pull/14783)
- **作者**: lijiahang226  **时间**: 2026-08-23 18:04 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  GLM-5.1 IndexCache keeps an `Indexer` module on static S layers for weight-loading compatibility, but those layers reuse another layer's top-k indices at runtime. Previously they still allocated and wrote a per-layer indexer KV cache even though the computed …

### #14782 — [[Feature][MLA] Support Kimi-K3 DCP decode on main](https://github.com/vllm-project/vllm-ascend/pull/14782)
- **作者**: foraxe  **时间**: 2026-08-23 16:59 CST
- **标签**: module:tests
- **摘要**: # [Feature][MLA] Support Kimi-K3 DCP decode on main  ### What this PR does / why we need it?  This is the current-main port of vLLM Ascend PR [#14126](https://github.com/vllm-project/vllm-ascend/pull/14126), which targeted `releases/v0.23.0` and validated the Kimi-K3 DCP path on Ascend A3.  Kimi-K3 …

### #14780 — [support A5 megamoe](https://github.com/vllm-project/vllm-ascend/pull/14780)
- **作者**: LookAround0301  **时间**: 2026-08-23 16:14 CST
- **标签**: documentation, module:ops, module:core, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/d02df748bf9efd99022f1a062597dc3cb3808485
