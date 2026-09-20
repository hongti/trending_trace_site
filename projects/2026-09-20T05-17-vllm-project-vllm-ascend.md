# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-20 13:17 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
- **#16976 GLM-5.3-w8a8c8 乱码问题**：有用户反馈，在昇腾 A2 八节点 PD 分离部署中，运行 GLM-5.3-w8a8c8 模型并启用 `enable_sparse_sfa_c8: true` 参数时，在 `v0.26.0rc1` 和 `v0.26.0rc2` 版本下会出现输出乱码的 Bug。

### 🔧 PR 动态
近期的 PR 主要集中在**性能优化、新特性支持及 Bug 修复**上，重点围绕注意力机制、MoE 架构和昇腾硬件特性展开：

**✨ 新特性**
- **#16973 AscendStore 支持 DeepSeek V4.1**：为 DeepSeek V4.1 的混合 C1/C2、滑动窗口和私有 circular-state 布局做准备，并引入了推测性缓存复用。
- **#16967 MRv2 支持 O_proj 张量并行**：在 model runner v2 中启用了 `oproj_tensor_parallel_size` 功能，主要针对 PD decode 节点进行优化。

**🚀 性能优化**
- **#16977 & #16972 DCP 解码优化**：优化 DCP decode 流和 V-up 投影（保留 history attention dtypes，融合排列操作）；融合单行 DCP16 打包交换与合并，减少 BF16 到 FP32 的类型转换开销。
- **#16975 & #16974 注意力性能提升**：针对大型 DSA-CP 缓存写入，改用块拷贝替代通用的 scatter 更新；保持 eager DSA-CP prefill 的 token-sharded 状态，避免重复输出物的冗余生成。
- **#16969 EPLB 开销降低**：减少了 MRv2 EPLB 中逐层的逻辑到物理专家映射及小批量负载收集的开销。

**🐞 Bug 修复**
- **#16970 CUDAGraph 捕获修复**：修复了昇腾平台上非默认 CUDAGraph 捕获大小导致最大解码批次出错的 Bug。
- **#16968 MoE 激活生命周期修复**：修复了 MoE MLP 重构引入的 W4A4 MXFP4 激活生命周期回归问题，在 GMM1 前释放重量化激活。

**🧪 CI 与测试**
- **#16971**：修改了 DSV4 的测试用例。

*注：本次动态未包含 Release 相关信息。*

---

## 🐛 Issues

### #16976 — [[Bug]: GLM-5.3-w8a8c8 with parameter "enable_sparse_sfa_c8: true" which deployed on vllm-ascend:0.26.0rc1 &vllm-ascend:v0.26.0rc2 has garbled text problem](https://github.com/vllm-project/vllm-ascend/issues/16976)
- **作者**: hawkmenz  **时间**: 2026-09-20 12:41 CST
- **标签**: bug, glm5, llm-model
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text [W920 04:36:26.356237470 FunctionLoader.cpp:48] Warning: LD_PRELOAD detected, FunctionLoader prefers RTLD_DEFAULT for symbol resolution. (function operator()) INFO 09-20 04:36:32 [__init__.py:52…

## 🔀 Pull Requests

### #16977 — [[Refactor] [DCP] optimize DCP decode streams and V-up projection](https://github.com/vllm-project/vllm-ascend/pull/16977)
- **作者**: weiguihua2  **时间**: 2026-09-20 12:55 CST
- **标签**: module:tests
- **摘要**: Preserve history attention dtypes through DCP packing, overlap current-token attention on a side stream, and fuse V-up projection permutations. Extend stream-order and dtype regression coverage.  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How w…

### #16975 — [[Performance][Attention] Use block copies for large DSA-CP cache writes](https://github.com/vllm-project/vllm-ascend/pull/16975)
- **作者**: lrf-vm  **时间**: 2026-09-20 12:22 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Large DSA-CP prefill writes spend substantial time in generic scatter updates. Reuse the existing block-copy kernel for supported large main and indexer cache writes.  Build main-cache grouping metadata once from its own physical slot mapping during the initi…

### #16974 — [[Performance][Attention] Keep eager DSA-CP prefill token-sharded](https://github.com/vllm-project/vllm-ascend/pull/16974)
- **作者**: lrf-vm  **时间**: 2026-09-20 12:22 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Eligible DSA-CP MoE layers gather token-sharded hidden states that attention immediately shards again. After full O-proj, they materialize a replicated output and reduce-scatter it even though each rank has already computed its complete local token output.  K…

### #16973 — [[Feature][AscendStore] Prepare V4.1 and speculative cache reuse](https://github.com/vllm-project/vllm-ascend/pull/16973)
- **作者**: Pz1116  **时间**: 2026-09-20 12:18 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Prepare non-layerwise Ascend Store for DeepSeek V4.1's mixed C1/C2, sliding-window and private circular-state layout. Private state currently participates in hash-size selection, hit lookup and transfer: it can choose a 32-token hash unit instead of 128, reje…

### #16972 — [[Perf][MLA] Fuse single-row DCP16 packed exchange and merge](https://github.com/vllm-project/vllm-ascend/pull/16972)
- **作者**: foraxe  **时间**: 2026-09-20 12:13 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Single-row MLA decode currently converts BF16 attention output to FP32 before DCP exchange and materializes received output/LSE tensors before the native merge. This adds traffic and launches on the decode critical path.  This patch transports BF16 output and…

### #16971 — [[CI] modify dsv4 testcase](https://github.com/vllm-project/vllm-ascend/pull/16971)
- **作者**: lcfenglinwan  **时间**: 2026-09-20 12:05 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/84030bbe3d74d99bad477a3d2e37a973ccd8865c

### #16970 — [[BugFix] Add off-stride graph capture size ceilings](https://github.com/vllm-project/vllm-ascend/pull/16970)
- **作者**: yiz-liu  **时间**: 2026-09-20 11:48 CST
- **标签**: module:tests, module:core
- **摘要**: ## What this PR does / why we need it?  Fix off-stride default CUDAGraph capture sizes on Ascend. Fixes #16049 .  When `max_num_seqs` is not on the default capture-size grid, the largest decode batches could fall back to eager execution. This change preserves Ascend's reduced capture ceiling for mem…

### #16969 — [[Performance][EPLB] Reduce routing and small-batch load collection overhead](https://github.com/vllm-project/vllm-ascend/pull/16969)
- **作者**: zhenwenqi2024  **时间**: 2026-09-20 11:48 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Reduce the per-layer cost of logical-to-physical expert mapping and load collection in MRv2 EPLB.  - Build a one-row routing table when the physical and logical expert counts are equal. This uses existing host metadata and preserves in-place table refresh for…

### #16968 — [[BugFix][MoE] Release requantized W4A4 MXFP4 activations before GMM1](https://github.com/vllm-project/vllm-ascend/pull/16968)
- **作者**: ParadiseHeaven  **时间**: 2026-09-20 11:47 CST
- **标签**: module:tests, module:quantization
- **摘要**: ### What this PR does / why we need it?  Fix the W4A4 MXFP4 MoE activation lifetime regression introduced by the MoE MLP refactor. When the dispatcher does not provide a dynamic scale, local dynamic MXFP quantization produces a separate FP4 activation tensor. Release the original BF16 activation bef…

### #16967 — [[Feature][MRV2] Support o_proj TP in model runner v2 (graph-mode scope)](https://github.com/vllm-project/vllm-ascend/pull/16967)
- **作者**: xuchi-0808  **时间**: 2026-09-20 11:46 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Enables the `oproj_tensor_parallel_size` knob of fine-grained TP on model runner v2, scoped to the deployment it actually serves: a PD decode node (`tp == 1`, `dp > 1`) running a graph mode with the recompute scheduler.  o_proj TP shards the attention output …
