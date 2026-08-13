# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-13 11:10 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
* **DeepSeek-V4-Pro 默认参数破坏性变更 (#52083)**：有用户反馈 PR #50580 的合并导致 DeepSeek-V4-Pro 模型的默认推理努力程度发生破坏性变更，影响了模型表现。

### 🔀 Pull Request 动态

**🚀 性能优化与新特性**
* **DeepSeek-V4 Prefill 提速 (#52084)**：优化了 sparse top-k metadata 内核，显著提升了 prefill 阶段的吞吐量。
* **Kimi-K3 架构支持增强 (#52079, #52080)**：
  * 为 Blackwell 架构新增 GEMM-RS 内核，以支持序列并行 (#52079)。
  * 针对 AMD ROCm 平台，将 Kimi-K3 MLA 中的 `q-a` 和 `kv-a` 两个 RMSNorm 融合为单次启动，减少了内核开销 (#52080)。
* **Attention 计算精简 (#52078)**：修复了 GDN metadata 构建过程中 mask 被重复计算的问题，避免了冗余运算。

**🛠️ 缺陷修复与回退**
* **撤回 ViT 阻塞优化 (#52081, #52082)**：撤回了 PR #51841（旨在避免 ViT 中长时间阻塞的 H2D 拷贝），因为该 PR 导致 Arm CPU 测试持续失败；同时修复了该 PR 引入的 Qwen3_VisionTransformer 在非 GPU 环境下的内存绑定问题。
* **YAML 配置解析崩溃修复 (#52077)**：修复了当 `--config` 传入空 YAML 文件或仅含注释的文件时，引发 `AttributeError` 崩溃的问题。
* **并行后端错误提示优化 (#52075)**：改进了 context-parallel 后端不兼容时的报错信息，为用户提供具体的恢复步骤（如切换 attention 后端或禁用 DCP）。

**📝 文档与核心代码**
* **文档更正 (#52074)**：更新了 `extract_hidden_states` 的文档，移除了“不兼容 chunked prefill”的过时描述（该功能现已支持）。
* **代码注释优化 (#52076)**：澄清并完善了 `BlockPool.free_blocks()` 中关于块驱逐优先级的注释，消除歧义。

### 📦 Release 动态
* 近期暂无新版本发布信息。

---

## 🐛 Issues

### #52083 — [[Bug]: Breaking change to DeepSeek-V4-Pro default reasoning effort](https://github.com/vllm-project/vllm/issues/52083)
- **作者**: benchislett  **时间**: 2026-08-13 10:51 CST
- **标签**: bug
- **摘要**: ### Your current environment  vllm main, bisected to https://github.com/vllm-project/vllm/pull/50580  ### 🐛 Describe the bug  This PR (https://github.com/vllm-project/vllm/pull/50580), which changes the DSV4 encoder for DSV4-Flash-0731 support, changes the default behaviour for DSV4-Pro.  Previously…

## 🔀 Pull Requests

### #52084 — [[Perf][DSV4] Optimize sparse top-k metadata kernels for higher prefill throughput](https://github.com/vllm-project/vllm/pull/52084)
- **作者**: chaunceyjiang  **时间**: 2026-08-13 10:57 CST
- **摘要**: ## Purpose Optimize sparse top-k metadata kernels for higher prefill throughput  ## Test ``` vllm bench serve \     --backend vllm \     --base-url http://localhost:8000 \     --model deepseek-ai/DeepSeek-V4-Flash-0731 \     --dataset-name random \     --random-input-len 1024 \     --random-output-l…

### #52082 — [[BugFix] Fix memory pinning in Qwen3_VisionTransformer for non-gpu](https://github.com/vllm-project/vllm/pull/52082)
- **作者**: njhill  **时间**: 2026-08-13 10:35 CST
- **标签**: bug, ready, qwen
- **摘要**: Fix for CI failure introduced by https://github.com/vllm-project/vllm/pull/51841

### #52081 — [Revert "Avoid long-blocking H2D copies in ViT" (#51841)](https://github.com/vllm-project/vllm/pull/52081)
- **作者**: vllm-agent  **时间**: 2026-08-13 10:06 CST
- **标签**: qwen
- **摘要**: Reverts the changes from #51841 ("Avoid long-blocking H2D copies in ViT").  ## Why  `Arm CPU Test` has failed on every build since #51841 merged (nightly [#83608](https://buildkite.com/vllm/ci/builds/83608), plus per-commit postmerge builds 83603 and 83607 on the same commit). It passed on build 835…

### #52080 — [[ROCm][Perf] Kimi-K3 AMD MLA: fuse the q-a and kv-a RMSNorms](https://github.com/vllm-project/vllm/pull/52080)
- **作者**: mpashkovskii  **时间**: 2026-08-13 09:43 CST
- **标签**: rocm, kimi, k3
- **摘要**: ## Purpose  On the Kimi-K3 AMD MLA front-end, every token runs **two** separate RMSNorm launches — `q_a_layernorm(q_c)` then `kv_a_layernorm(kv_c)` — once per MLA layer.  This PR collapses them into a single `fused_q_kv_rmsnorm` call (`models/common/ops/fused_qk_rmsnorm.py`), a **portable Triton** k…

### #52079 — [[Kimi-K3] Add GEMM-RS for sequence parallelism](https://github.com/vllm-project/vllm/pull/52079)
- **作者**: gau-nernst  **时间**: 2026-08-13 09:15 CST
- **标签**: performance, ci/build, kimi, k3
- **摘要**: ## Purpose  Add GEMM-RS kernel for Blackwell, based on https://github.com/NVIDIA/cutlass/blob/dcf215a/examples/python/CuTeDSL/cute/blackwell/kernel/distributed/distributed_gemm_reduce_scatter_blackwell.py (`multimem.ld_reduce`) - Supports any value of M e.g. M=1023. However, only uses GEMM-RS when M…

### #52078 — [[Attention] Avoid redundant mask compute in GDN metadata build](https://github.com/vllm-project/vllm/pull/52078)
- **作者**: xyang16  **时间**: 2026-08-13 09:02 CST
- **摘要**: ## Purpose  The spec decode detection in GDNAttentionMetadataBuilder.build() computed tensors multiple times:  1. computed the `num_decode_draft_tokens_cpu >= 0` mask twice in line 192 and 200: spec_sequence_masks_cpu = num_decode_draft_tokens_cpu >= 0. Each >= 0 creates a fresh boolean tensor. 2. c…

### #52077 — [[Bugfix] Handle empty YAML config in `--config` parsing](https://github.com/vllm-project/vllm/pull/52077)
- **作者**: veerareddyvishal144  **时间**: 2026-08-13 08:58 CST
- **标签**: bug
- **摘要**: ## Description Passing an empty or comments-only YAML file to `--config` caused `yaml.safe_load()` to return `None`, which then crashed with `AttributeError: 'NoneType' object has no attribute 'items'` in `FlexibleArgumentParser.load_config_file`.  This PR: - Treats `None` (empty/comments-only YAML)…

### #52076 — [[Core] Clearer comments in `BlockPool.free_blocks()`](https://github.com/vllm-project/vllm/pull/52076)
- **作者**: njhill  **时间**: 2026-08-13 08:42 CST
- **标签**: ready
- **摘要**: The comments explaining eviction precedence in `BlockPool.free_blocks()` were ambiguous/confusing. Make them clearer / more explicit.

### #52075 — [[Bugfix] Improve context-parallel backend error guidance](https://github.com/vllm-project/vllm/pull/52075)
- **作者**: veerareddyvishal144  **时间**: 2026-08-13 08:33 CST
- **标签**: bug
- **摘要**: ## Summary  Improve context-parallel compatibility errors so they give users concrete recovery steps.  - tell DCP users to select another backend with `--attention-backend`, or disable DCP with `--decode-context-parallel-size 1` - add equivalent actionable guidance to the PCP compatibility error - a…

### #52074 — [[Docs] extract_hidden_states supports chunked prefill](https://github.com/vllm-project/vllm/pull/52074)
- **作者**: aminsamir45  **时间**: 2026-08-13 08:19 CST
- **标签**: documentation
- **摘要**: ## Summary  The `extract_hidden_states` docs state:  > Chunked prefill is not compatible with this feature and must be disabled.  That appears to be stale. The feature's own integration test exercises chunked prefill deliberately, in `tests/v1/kv_connector/extract_hidden_states_integration/test_extr…
