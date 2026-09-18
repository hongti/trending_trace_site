# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-18 13:04 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要。本次更新主要包含 10 个 Pull Request，未检测到相关的 Issue 和 Release 动态。

### 📋 Pull Request (PR) 摘要

近期 PR 主要围绕新特性引入、缺陷修复以及文档与测试完善展开，具体归纳如下：

#### ✨ 新特性与功能增强
*   **GLM-5.3-Flash KV Pool 传输支持** (#16854, #16858)：新增了对 GLM-5.3-Flash 模型的 KV Cache 传输支持。非分层传输 (#16854) 处理了 NoPE MLA、压缩索引器及请求私有循环尾等复杂状态；分层传输 (#16858) 则在此基础上实现了 Memcache 的分层适配。
*   **Qwen3 MoE 序列并行回移** (#16857)：为 Qwen3 MoE 启用了 FlashComm1，在 MoE 块内对 token 进行分片，解决了原先解码器 RMSNorm 操作无法并行的问题，提升了模型执行效率。

#### 🛠️ 重要缺陷修复
*   **ModelLoader 连接健壮性修复** (#16859)：修复了 `ElasticClient.__init__` 中 `sock` 仅在 `try` 块内赋值导致异常处理程序引用失败的缺陷。
*   **适配 vLLM 0.28.0 的 MRV2 调度** (#16853)：修复了在 vLLM 0.28.0 版本下，Ascend MRV2 PCP+DP 在并行配置校验时被拒的问题，并修正了调度同步逻辑。
*   **A5 平台自定义算子加载修复** (#16851)：为 A5 平台补充了缺失的 `RUNTIME_CUSTOM_OPS`，使 `enable_custom_op()` 能够正确尝试加载已安装的自定义扩展。
*   **Kimi K3 启动失败日志优化** (#16850)：修复了 Kimi K3 + DSpark 启动时在 Ascend 钩子运行前失败的问题，现在会打印 KV page 布局日志以便于排查。

#### 📚 文档、测试与 CI
*   **文档重构** (#16856)：基于统一的功能文档模板重构了 `kv_pool.md`，并新增了英文版 `kv_pool_en.md`。
*   **FLA 算子测试补全** (#16855)：为 Triton 的 FLA 层归一化算子补充了 NPU 正确性覆盖测试和算子文档。
*   **CI 基线更新** (#16852)：更新了 CI 测试基线。

---

### 📝 Issue 摘要
*本次输入数据中未包含相关 Issue 动态。*

---

### 🚀 Release 摘要
*本次输入数据中未包含相关 Release 动态。*

---

## 🔀 Pull Requests

### #16859 — [[BugFix][ModelLoader] Harden NetLoader client connection setup](https://github.com/vllm-project/vllm-ascend/pull/16859)
- **作者**: joeqth  **时间**: 2026-09-18 12:58 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Two defects in the per-source connection loop of `ElasticClient.__init__`:  1. `sock` is only assigned inside the `try` block, but the exception handler    references it. When `socket.socket()` itself raises (for example on    descriptor exhaustion), the hand…

### #16858 — [[Feature][KV Pool] Support GLM-5.3-Flash Memcache layerwise transfer](https://github.com/vllm-project/vllm-ascend/pull/16858)
- **作者**: Pz1116  **时间**: 2026-09-18 12:44 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Depends on #16854, which supplies the private-tail, hash/alignment and empty-RoPE fixes. This is the separate Memcache layerwise adaptation for GLM-5.3-Flash; the first commit is shared with that PR.  KDA state is TP-sharded even though MLA KV is replicated. …

### #16857 — [[Feat][Model] Backport Qwen3 MoE sequence parallelism](https://github.com/vllm-project/vllm-ascend/pull/16857)
- **作者**: jiaqi-lee  **时间**: 2026-09-18 12:42 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  With Qwen3 MoE, enabling FlashComm1 currently shards tokens inside the MoE block but leaves the decoder RMSNorm operations replicated. Backport the model-level sequence-parallel behavior from https://github.com/vllm-project/vllm/pull/57337 through a worker pa…

### #16856 — [docs: rewrite KV pool guide based on unified feature doc template](https://github.com/vllm-project/vllm-ascend/pull/16856)
- **作者**: LeiW777  **时间**: 2026-09-18 12:36 CST
- **标签**: documentation
- **摘要**: - Restructure kv_pool.md and add English version kv_pool_en.md following the newly established feature documentation writing template, which unifies writing standards and structure across feature guides - VOC analysis showed that feature stacking constraints (e.g. layerwise applicability per backend…

### #16855 — [[Test][Doc][Ops] Add FLA layer norm Triton coverage](https://github.com/vllm-project/vllm-ascend/pull/16855)
- **作者**: drslark  **时间**: 2026-09-18 12:34 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Adds focused NPU correctness coverage and operator documentation for vllm_ascend/ops/triton/fla/layernorm_guard.py.  The three test cases cover standard LayerNorm with bias, grouped gated RMSNorm, and row counts beyond MAX_CORES. The Markdown document records…

### #16854 — [[Feature][KV Pool] Support GLM-5.3-Flash non-layerwise transfer](https://github.com/vllm-project/vllm-ascend/pull/16854)
- **作者**: Pz1116  **时间**: 2026-09-18 12:34 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  GLM-5.3-Flash combines NoPE MLA, a compressed indexer, request-private circular tails, and KDA state. AscendStore currently counts the tail's four-token ring as a hash group and registers the empty RoPE view at address zero, so the model cannot use the non-la…

### #16853 — [[BugFix][MRV2] Adapt PCP+DP dispatch for vLLM 0.28.0](https://github.com/vllm-project/vllm-ascend/pull/16853)
- **作者**: wzx0726  **时间**: 2026-09-18 12:10 CST
- **摘要**: ### What this PR does / why we need it?  On vLLM 0.28.0, Ascend MRV2 PCP+DP is rejected during parallel configuration validation. Allowing configuration alone is insufficient: dispatch still synchronizes the global token count before PCP partitioning, so a 44-token prefill with PCP2 reaches `DPMetad…

### #16852 — [[CI] update baseline](https://github.com/vllm-project/vllm-ascend/pull/16852)
- **作者**: guxin108  **时间**: 2026-09-18 11:59 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? update baseline  ### Does this PR introduce _any_ user-facing change? no  ### How was this patch tested? run the case  - vLLM main: https://github.com/vllm-project/vllm/commit/84030bbe3d74d99bad477a3d2e37a973ccd8865c

### #16851 — [[BugFix][Platform] Enable A5 custom ops through the shared loader](https://github.com/vllm-project/vllm-ascend/pull/16851)
- **作者**: Foriv  **时间**: 2026-09-18 11:55 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  A5 currently lacks `RUNTIME_CUSTOM_OPS`, so `enable_custom_op()` returns `False` before attempting to load an installed custom extension. Add this capability to let A5 use the existing shared initialization path.  Remove A5's `BGMV_SGMV_META_REGISTRATION` cap…

### #16850 — [[Bugfix][Kimi K3] Log KV page layouts on DSpark startup failure](https://github.com/vllm-project/vllm-ascend/pull/16850)
- **作者**: qijiajin  **时间**: 2026-09-18 11:51 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  A Kimi K3 + DSpark startup can fail in vLLM's KV page-size unification before the Ascend Kimi K3 mixed-grouping hook runs. The exception identifies the first incompatible target layer but does not show the target attention, draft attention, or Mamba page layo…
