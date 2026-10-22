# verl-project/verl — 动态追踪

> 生成时间: 2026-10-22 13:15 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📋 Issue
* **#8324 [RFC] 现代 Megatron 集成方案**
  作者提出重构 verl 的 Megatron 集成，计划围绕当前的 Megatron-Core 和 Megatron-Bridge API 进行现代化改造，旨在减少下游补丁的维护成本，并使未来向上游升级更加顺畅。

### 🔧 Pull Request
近期 PR 主要集中在功能增强、Bug 修复及依赖升级：

**新特性与功能增强**
* **#8330 暴露低精度 ReduceScatter 通信配置**：VeOmni 已支持节点本地 BF16 梯度传输（配合 FP32 累加和输出），此 PR 在 verl 中暴露了该配置（默认关闭，命名为 `low_precision_r...`）。
* **#8327 MXFP8 Rollouts 的 Refit 修复与运行时检查**：完善了量化（MXFP8）rollouts 的支持，修复了 vLLM 中 MoE 层保持 BF16 等问题，并增加了运行时检查。
* **#8323 基于权重版本标记外部 KV 缓存**：优化了 Rollout 引擎的 KV 缓存处理机制，不再在每次权重更新时强制硬重置旧策略的 KV，而是通过权重版本进行标记，提升了效率。

**Bug 修复与 CI 改进**
* **#8328 捕获 Ascend 环境与权重传输诊断信息**：针对 Ascend Qwen3-VL 精度回退问题增加诊断证据收集，以协助排查 CI 中未能在本地复现的精度崩溃。
* **#8329 修复 ModelOpt QAT 模块忽略模式**：修复了忽略模式无法禁用子量化器的问题（如 `*self_attention.linear_proj` 匹配失败）。
* **#8325 修复 top-k 蒸馏张量的 THD 与 FP8 填充**：使 THD 填充逻辑与配方无关，修复了相关审查问题。
* **#8326 修复 L20 FP8 MoE 测试后端选择**：在端到端测试中显式指定使用 vLLM Triton Linear 和 MoE 后端。

**依赖升级 (Dependabot)**
* **#8322** 将开发依赖 vllm 从 0.29.0 升级至 0.31.0。
* **#8321** 将依赖 transformers 从 5.12.1 升级至 5.20.1。
* **#8320** 将开发依赖 spmd-types 从 0.2.3 升级至 0.3.0。

### 🚀 Release
* 本周期内无新的 Release 版本发布。

---

## 🐛 Issues

### #8324 — [[RFC] Modern Megatron integration](https://github.com/verl-project/verl/issues/8324)
- **作者**: ISEEKYAN  **时间**: 2026-10-22 10:40 CST
- **标签**: enhancement, megatron, RFC
- **摘要**: ### Feature request  I would like to modernize verl's Megatron integration around current Megatron-Core and Megatron-Bridge APIs, reduce the downstream patch surface, and make future upstream upgrades easier to maintain.  **This RFC asks for architecture and migration-plan review before submitting a…

## 🔀 Pull Requests

### #8330 — [[veomni] feat: expose low-precision ReduceScatter communication](https://github.com/verl-project/verl/pull/8330)
- **作者**: amenomori  **时间**: 2026-10-22 13:00 CST
- **摘要**: ### What does this PR do?  VeOmni already supports node-local BF16 gradient transport with FP32 accumulation and output, but verl does not expose its configuration. Add the default-off `low_precision_reduce_scatter_comm` option and forward it to the VeOmni model builder for training engines.  Implem…

### #8329 — [[megatron, ci] fix: honor module ignore patterns in ModelOpt QAT](https://github.com/verl-project/verl/pull/8329)
- **作者**: LZC-BELIEVER  **时间**: 2026-10-22 11:51 CST
- **摘要**: ### What does this PR do?  Fix ModelOpt QAT module ignore patterns so that they disable the module's child quantizers. For example, `*self_attention.linear_proj` currently matches neither `...linear_proj.input_quantizer` nor `...linear_proj.weight_quantizer`. Training therefore still applies fake qu…

### #8328 — [[ci, fsdp] feat: capture Ascend environment and weight transfer diagnostics](https://github.com/verl-project/verl/pull/8328)
- **作者**: dodatboii  **时间**: 2026-10-22 11:48 CST
- **摘要**: ### What does this PR do?  Capture evidence for the Ascend Qwen3-VL accuracy regression discussed around #8187 and #8188. Local runs of the failing CI revision have not reproduced the collapse, so the Qwen3-VL nightly job now saves environment/content fingerprints, bounded per-worker weight-path rec…

### #8327 — [[rollout, vllm, sglang, trainer] feat: refit fixes and runtime checks for MXFP8 rollouts](https://github.com/verl-project/verl/pull/8327)
- **作者**: wengeezhang  **时间**: 2026-10-22 11:38 CST
- **摘要**: ### What does this PR do?  The rest of #7519 after #8127: refit fixes and runtime checks for quantized (MXFP8) rollouts, on top of #8287, #8306 and #8308.  - **vLLM refit**: MoE layers kept in BF16 by `rollout.quantization.ignore` are re-derived after every quantized refit. Before, the quantized ref…

### #8326 — [[ci, vllm] fix: select Triton for L20 FP8 MoE test](https://github.com/verl-project/verl/pull/8326)
- **作者**: wuxibin89  **时间**: 2026-10-22 10:56 CST
- **摘要**: ### What does this PR do?  Follow up [#8308](https://github.com/verl-project/verl/pull/8308) by explicitly selecting the vLLM Triton Linear and MoE backends in the L20 FP8 MoE end-to-end test. #8308 made DeepGEMM the default for low-precision rollouts when no engine backend is configured, but L20 is…

### #8325 — [[megatron, trainer] fix: recipe-agnostic THD padding and FP8 padding of top-k distillation tensors](https://github.com/verl-project/verl/pull/8325)
- **作者**: wengeezhang  **时间**: 2026-10-22 10:54 CST
- **摘要**: ### What does this PR do?  Follow-up to #8127, for two of its review threads.  1. **Recipe-agnostic THD padding** ([review](https://github.com/verl-project/verl/pull/8127#discussion_r4336382125)). `preprocess_packed_seqs` and `preprocess_thd_engine` took two FP8-specific arguments, `use_fp8_padding`…

### #8323 — [[rollout, vllm, sglang, trtllm, cfg] feat: salt external KV caches by weight version instead of resetting them](https://github.com/verl-project/verl/pull/8323)
- **作者**: daegyu94  **时间**: 2026-10-22 10:14 CST
- **摘要**: ### What does this PR do?  Rollout engines increasingly keep prefix KV beyond GPU memory (KV connectors, offload tiers, HiCache storage). On every weight update verl drops old-policy KV with a hard reset (`reset_connector=True`, #6373), which does not reliably reach those tiers: - In vLLM 0.29 (and …

### #8322 — [build(deps-dev): bump vllm from 0.29.0 to 0.31.0](https://github.com/verl-project/verl/pull/8322)
- **作者**: dependabot[bot]  **时间**: 2026-10-22 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [vllm](https://github.com/vllm-project/vllm) from 0.29.0 to 0.31.0. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/vllm-project/vllm/releases">vllm's releases</a>.</em></p> <blockquote> <h1>v0.31.0</h1> <h2>Highlights</h2> <p>This release features 71…

### #8321 — [build(deps): bump transformers from 5.12.1 to 5.20.1](https://github.com/verl-project/verl/pull/8321)
- **作者**: dependabot[bot]  **时间**: 2026-10-22 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [transformers](https://github.com/huggingface/transformers) from 5.12.1 to 5.20.1. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/huggingface/transformers/releases">transformers's releases</a>.</em></p> <blockquote> <h1>Release v5.20.1</h1> <p>Specia…

### #8320 — [build(deps-dev): bump spmd-types from 0.2.3 to 0.3.0](https://github.com/verl-project/verl/pull/8320)
- **作者**: dependabot[bot]  **时间**: 2026-10-22 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [spmd-types](https://github.com/meta-pytorch/spmd_types) from 0.2.3 to 0.3.0. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/meta-pytorch/spmd_types/releases">spmd-types's releases</a>.</em></p> <blockquote> <h2>v0.3.0</h2> <p><strong>Full Changelog<…
