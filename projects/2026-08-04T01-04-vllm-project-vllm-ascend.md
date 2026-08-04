# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-04 09:04 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue
- **高并发导致 PD 分离式部署服务崩溃** (#13428)：
  在开启 `VLLM_ASCEND_ENABLE_MLAPO=1` 的 PD（Prefill-Decode）分离式部署场景下，高并发请求会导致推理服务崩溃。根据反馈，20并发运行正常，但60及以上并发运行一段时间后即发生崩溃。涉及模型为微调后的 DeepSeekV3（v0.23.0rc1）。

### 🚀 Pull Request
**核心特性与性能优化**
- **Ascend 910B 启用 Fused W8A8 MC2** (#13426)：实现了动态 W8A8 `dispatch_ffn_combine` 核函数，支持核内激活量化和 INT8 GMM 计算等，提升计算性能。
- **SFA DCP sparse-index remap 性能优化** (#13432)：使用 Ascend C 自定义算子替换了原有的 NPU PyTorch float32/floor/sort/gather 路径，采用稳定的整数重映射与压缩，提升执行效率。

**Bug 修复**
- **修复 RecomputeScheduler 补丁错误** (#13423)：修复了核心调度器中 `in_flight_tokens` 的补丁错误，提升运行稳定性。

**代码同步**
- **同步 vLLM 主分支最新代码** (#13425, #13433)：分别将 vllm-ascend 适配至 7月24日 和 8月3日 的 vLLM main 分支最新提交。

**文档更新**
- **新增 DeepSeek-V4-Flash-0731 部署指南** (#13427)：为 A2 系列添加了该模型的 cookbook，包含在线服务部署文档和启动脚本。
- **重构 GLM-5 / GLM-5.2 部署指南** (#13429)：按部署场景（单节点/多节点/PD分离）重新梳理参数，新增推荐配置表和性能调优指南。
- **修复文档表格格式** (#13431)：修复 A2/A3 Pooling 模型支持矩阵表的列对齐和分隔符渲染问题。

**CI/测试**
- **启用 A5 每日夜间测试流水线** (#13430)。
- **新增 Qwen 模型常规测试** (#13424)：为 Qwen3.5-122B-A10B 和 Qwen3.6-35B-A3B 添加每周性能与精度测试用例。

### 📦 Release
- 近期无新的 Release 版本发布。

---

## 🐛 Issues

### #13428 — [[bug]In PD (Prefill-Decode) disaggregated deployments, VLLM_ASCEND_ENABLE_MLAPO=1,high concurrency can cause the inference service to crash](https://github.com/vllm-project/vllm-ascend/issues/13428)
- **作者**: yyyliu714  **时间**: 2026-08-03 23:07 CST
- **标签**: bug
- **摘要**: <describe> v0.23.0rc1 model: DeepSeekV3 after fine-tuning 20 concurrency - no problem 60 concurrency - crashes after running for a while... 70 concurrency - crashes after running for a while... 80 concurrency - crashes after running for a while...  60 concurrency in v0.18.0-no problem   ### Your cur…

## 🔀 Pull Requests

### #13433 — [[Misc]feat: adapt to vLLM main (5df9999f)](https://github.com/vllm-project/vllm-ascend/pull/13433)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-04 08:24 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to August 03.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | `tests/e2e/conftest.py`<br>`vllm_ascend/_310p/attention/metadata_builder…

### #13432 — [[Performance][SFA] Replace DCP sparse-index remap with Ascend C](https://github.com/vllm-project/vllm-ascend/pull/13432)
- **作者**: pisceskkk  **时间**: 2026-08-04 02:52 CST
- **标签**: module:tests
- **摘要**: ## Summary  - add an Ascend C custom operator for SFA DCP sparse-index remapping - replace the NPU PyTorch float32/floor/sort/gather path with stable integer remap and compaction - preserve a pure integer CPU reference fallback for unit tests and tooling - support dynamic top-k up to 8192, non-power…

### #13431 — [[Doc] Fix table formatting](https://github.com/vllm-project/vllm-ascend/pull/13431)
- **作者**: herizhen  **时间**: 2026-08-04 00:55 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? This PR fixes the column alignment and separator line in the A2/A3 Pooling Models support matrix table to ensure proper rendering.  ### Does this PR introduce _any_ user-facing change? No, this is a documentation-only update.  ### How was this patch tested? Do…

### #13430 — [[CI] enable A5 nightly test pipeline](https://github.com/vllm-project/vllm-ascend/pull/13430)
- **作者**: xqchen7  **时间**: 2026-08-04 00:00 CST
- **标签**: ci/build, module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/0351e9aa1fdf1a51329d1906881528dfe61fc88e

### #13429 — [[Doc][Misc] Update GLM-5 and GLM-5.2 deployment guides ](https://github.com/vllm-project/vllm-ascend/pull/13429)
- **作者**: Wyz-134  **时间**: 2026-08-03 23:19 CST
- **标签**: documentation
- **摘要**: - GLM5.md: restructure parameter descriptions by deployment scenario (single-node / multi-node / PD disaggregation), add per-scenario recommended configuration tables and a performance tuning guide - GLM5.2.md: reorganize document with numbered sections, add multi-node communication verification and…

### #13427 — [[Doc][Feature] Add DeepSeek-V4-Flash-0731 cookbook](https://github.com/vllm-project/vllm-ascend/pull/13427)
- **作者**: yiminghub2024  **时间**: 2026-08-03 22:37 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  This PR adds the DeepSeek-V4-Flash-0731 cookbook for the A2 series, providing documentation and startup scripts for online service deployment.  ### Does this PR introduce _any_ user-facing change?  No, this is a documentation-only update.  ### How was this pa…

### #13426 — [[Feature][Kernel] Enable fused W8A8 MC2 on Ascend 910B](https://github.com/vllm-project/vllm-ascend/pull/13426)
- **作者**: CubeLander  **时间**: 2026-08-03 21:59 CST
- **标签**: module:tests, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  The dynamic-W8A8 `dispatch_ffn_combine` kernel already implements in-kernel activation quantization, INT8 GMM1/GMM2, expert-granular GMM2 publication, and AIV combine. On Ascend 910B/910B2, however, it is neither built nor selected and its communication helpe…

### #13425 — [[Misc]feat: adapt to vLLM main (5d8e90a9)](https://github.com/vllm-project/vllm-ascend/pull/13425)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-03 21:26 CST
- **标签**: merge-conflicts
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to July 24.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | `vllm_ascend/worker/npu_input_batch.py` | [866fea2b](https://github.com/vl…

### #13424 — [[CI] Add weely performance and accuracy test cases for Qwen3.5-122B-A10B](https://github.com/vllm-project/vllm-ascend/pull/13424)
- **作者**: weixinAc  **时间**: 2026-08-03 21:10 CST
- **标签**: module:tests, model-dataset-download
- **摘要**: ### What this PR does / why we need it? This PR adds weekly performance and accuracy test cases for Qwen3.5-122B-A10B and Qwen3.6-35B-A3B models, as well as Qwen3.6-35B-A3B models, to ensure continuous validation of performance and accuracy.  ### Does this PR introduce _any_ user-facing change? NO  …

### #13423 — [[BugFix][v0.25.1][Core] RecomputeScheduler fix in_flight_tokens patch error](https://github.com/vllm-project/vllm-ascend/pull/13423)
- **作者**: nwpu-zxr  **时间**: 2026-08-03 21:02 CST
- **摘要**: ### What this PR does / why we need it? RecomputeScheduler fix in_flight_tokens patch error.  ### Does this PR introduce _any_ user-facing change? No.  ### How was this patch tested? By CI.  - vLLM version: v0.25.1 - vLLM main: https://github.com/vllm-project/vllm/commit/fe784ff22e630a31fd798f392b01…
