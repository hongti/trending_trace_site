# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-27 09:07 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📌 Issue
*   **CI 自动化流程中断** (#12889)：`main2main` 自动化同步流程未完成所有计划步骤即停止，目前需要人工审查介入。

---

### 🚀 Pull Request
本次 PR 活动主要围绕**新特性支持、Bug 修复及性能优化**，重点加强了对 DeepSeek V4、Gemma4 及混合架构模型的支持：

**🆕 新特性与支持**
*   **DeepSeek V4 block_size 扩展** (#12882)：新增支持 `block_size=16`（此前仅支持 32/64/128），为 DS V4 模型提供更细粒度的内存管理。
*   **线性混合模型 KV Offload** (#12881)：重计算卸载支持线性混合模型（如 Qwen3.5），并修复了 MTP 接受长度变化时 Mamba 缓存块的处理问题。
*   **MegaMoe 移植** (#12887)：将 CANN MegaMoe 融合 MC2 支持 Backport 到 `v0.23.0` 分支，针对 Atlas A3 硬件。

**🐛 Bug 修复**
*   **GLM-5.2 推测解码修复** (#12886)：修复了 GLM-5.2 DFlash 配合 QuaRot 量化目标时，推测解码 token 接受率接近零的问题。
*   **Gemma4 模型支持** (#12884)：修复并支持 Gemma4 E2B 和 E4B 模型，通过兼容回退机制处理 256/512 维度的 TND prefill attention，并新增精度配置。
*   **NetLoader P2P 修复** (#12885)：修复 `INT8_CACHE=no` 场景下 processed-layout 的 HCCL P2P 传输问题，强化了弹性回退机制。

**⚡ 性能优化**
*   **DSV4 DSA Attention 优化** (#12883)：优化 `mla_prolog_multistream`，通过共享 wq_a 和 wk 之间的量化 hidden_states 减少重复计算，并移除冗余流同步，提升双流并行度。

**🤖 CI/适配**
*   **vLLM main 适配失败** (#12888)：尝试适配 vLLM v0.25.1 的 main2main 流程失败，未完成任何步骤。

---

### 📦 Release
*   近期无新的 Release 版本发布。

---

## 🐛 Issues

### #12889 — [[main2main] main2main manual review required ()](https://github.com/vllm-project/vllm-ascend/issues/12889)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-27 00:27 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/12888 - Commit range: `7ca017778fc04cfb1a96080e7c484b1b44fc3a64`...`` - Status: `failed`  ## Final Summary  main2main adaptation failed — no steps c…

## 🔀 Pull Requests

### #12888 — [[Misc]feat: adapt to vLLM main (97a98006)](https://github.com/vllm-project/vllm-ascend/pull/12888)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-27 00:27 CST
- **摘要**: main2main adaptation failed — no steps completed.  - vLLM version: v0.25.1 - vLLM main: https://github.com/vllm-project/vllm/commit/fe784ff22e630a31fd798f392b01e0a75c18f047

### #12887 — [[Backport][v0.23.0][Feature] Add CANN MegaMoe fused MC2 support for Atlas A3](https://github.com/vllm-project/vllm-ascend/pull/12887)
- **作者**: Liuchenbing-2026  **时间**: 2026-07-26 20:09 CST
- **标签**: module:tests, module:ops, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  This PR **backports CANN MegaMoe support** from upstream PR [#11701](https://github.com/vllm-project/vllm-ascend/pull/11701) to the `releases/v0.23.0` branch for Atlas A3 inference products.  CANN 9.1 introduces `cann_ops_transformer.ops.mega_moe` as the main…

### #12886 — [[Bugfix][SpecDecode] Fix GLM-5.2 DFlash acceptance for QuaRot targets](https://github.com/vllm-project/vllm-ascend/pull/12886)
- **作者**: Liuchenbing-2026  **时间**: 2026-07-26 19:27 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This PR fixes near-zero speculative token acceptance when a GLM-5.2 DFlash checkpoint is used with a QuaRot-quantized GLM-5.2 target on Ascend.  The DFlash path was active before this fix: draft, draft-token, and accepted-token counters all increased. However…

### #12885 — [[BugFix][NetLoader] Fix processed-layout P2P for INT8_CACHE=no](https://github.com/vllm-project/vllm-ascend/pull/12885)
- **作者**: yilunh998  **时间**: 2026-07-26 18:57 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? This PR adds processed-layout HCCL transfer for NetLoader `INT8_CACHE=no`, and then hardens the elastic fallback / P2P implementation built on top of that path. **1. Support processed-layout P2P when `INT8_CACHE=no`** Raw `named_parameters()` tensors are not v…

### #12884 — [[BugFix] Support Gemma4 E2B and E4B on Ascend](https://github.com/vllm-project/vllm-ascend/pull/12884)
- **作者**: ZhaXionghui  **时间**: 2026-07-26 18:24 CST
- **标签**: documentation, module:tests
- **摘要**: ## Summary  - Route 256- and 512-dimension TND prefill attention through the compatible `npu_fusion_attention` fallback. - Add Gemma 4 E2B-IT and E4B-IT accuracy configurations, including 1,000-sample GSM8K baselines. - Document supported Gemma 4 serving scope and text, image, and audio request exam…

### #12883 — [[Ascend] Optimize DSV4 DSA attention mla_prolog: share quantize + remove redundant stream syncsMla prolog multistream overlap](https://github.com/vllm-project/vllm-ascend/pull/12883)
- **作者**: frankie-ys  **时间**: 2026-07-26 17:58 CST
- **摘要**: ## Summary  Optimize `AscendDSAImpl._mla_prolog_multistream()` with two changes that reduce duplicate work and increase dual-stream parallelism:  1. **Share quantized hidden_states** between wq_a and wkv when their    quantization schemes are identical and communication status matches.    Saves one …

### #12882 — [[Ascend] Support block_size=16 for DeepSeek V4 models](https://github.com/vllm-project/vllm-ascend/pull/12882)
- **作者**: frankie-ys  **时间**: 2026-07-26 17:52 CST
- **标签**: module:core
- **摘要**: ## Summary  Add support for `--block-size 16` when deploying DeepSeek V4 models on Ascend NPU. Previously only block sizes [32, 64, 128] were supported.  ## Motivation  Smaller block_size enables finer-grained prefix caching granularity. For DeepSeek V4 with compress_ratio=4 and 128: - block_size=32…

### #12881 — [[Feature][KVOffload][P/D] Recompute offload supports linear hybrid model](https://github.com/vllm-project/vllm-ascend/pull/12881)
- **作者**: nwpu-zxr  **时间**: 2026-07-26 15:46 CST
- **摘要**: ### What this PR does / why we need it? Recompute offload supports linear hybrid model, such as Qwen3.5. When the MTP accept length changes, the offload Mamba cache block corresponding to the accept length needs to be placed at the start position of mamba cache groups, and this logic is implemented …
