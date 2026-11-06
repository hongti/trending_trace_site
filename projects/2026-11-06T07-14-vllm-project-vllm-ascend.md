# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-06 15:14 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 📋 Issue 动态
近期共有 2 个 Issue 提出，主要涉及新特性请求与潜在 Bug 修复：
*   **新特性请求 (#21171)**：提出在 Ascend NPU 上为包含池化层的模型支持全图（CUDAGraph/TorchGraph）捕获功能。
*   **Bug 报告 (#21166)**：通过静态代码检查发现，hybrid/layerwise mooncake connectors 在 prompt 截断后会跳过 `block_hashes` 的重建（此为 #20213 问题的后续遗留）。

---

### 🔨 Pull Request 动态
近期 PR 活动主要集中在性能优化、新模型/特性适配、Bug 修复以及文档改进上：

**1. 性能优化与新特性**
*   **ACLGraph 性能提升 (#21172)**：在 Ascend A2 上，将 DeepSeek V4.1-Flash 解码元数据的准备工作移至辅助 ACL Graph stream 上捕获，减少了主机端提交开销，提升执行效率。
*   **Gemma4 注意力机制支持 (#21169)**：为 Gemma4 添加了 512 维注意力机制的支持，包含 B1 paged verify、FULL cudagraph 和 FIA TND 特性。

**2. Bug 修复**
*   **DeepSeek V4 权重映射修复 (#21168)**：修复了 DeepSeek V4 DSpark 在构建 MoE 层时，检查点加载器未正确处理冗余专家的映射问题。
*   **AscendStore 页面传输修复 (#21167)**：修复了 AscendStore 独立推导哈希粒度导致与前缀匹配单元不兼容的问题，现统一使用 Core 解析的哈希大小和 APC 命中对齐。

**3. 代码清理与重构**
*   **移除无用算子 (#21162)**：清理并移除了未使用的 `kv_quant` sparse attention shared-KV 自定义算子。

**4. 文档与测试**
*   **文档更新 (#21174, #21175, #21170)**：将 GLM-5.3 的服务示例全面切换至 Model Runner V2；同时改进了 MiniMax、Qwen3 和 Step 模型的部署教程，填补了文档空白。
*   **自动化验证 (#21163, #21173)**：在 `main` 分支和 `releases/v0.32.0rc` 分支上创建了 Nightly 自动化测试 PR。

---

### 🚀 Release 动态
*   **无新版本发布**：本期动态中未包含正式的 Release 版本发布信息，但从部分 PR 标题（如 `releases/v0.32.0rc`）可以看出，项目正在积极筹备 **v0.32.0** 版本。

---

## 🐛 Issues

### #21171 — [[Feature]: Support Pooling Model Full Graph Capture](https://github.com/vllm-project/vllm-ascend/issues/21171)
- **作者**: zhangwenhan12  **时间**: 2026-11-06 14:55 CST
- **标签**: feature request
- **摘要**: ### 🚀 The feature, motivation and pitch  ### Feature Proposal Enable full graph (CUDAGraph/TorchGraph) capture for models with pooling layers on Ascend NPU. Previously, forcing full graph mode on pooling models resulted in error `507011` during replay. This PR resolves the underlying issues related …

### #21166 — [[Bug]: hybrid/layerwise mooncake connectors skip block_hashes rebuild after prompt truncation (follow-up of #20213)](https://github.com/vllm-project/vllm-ascend/issues/21166)
- **作者**: Esir98  **时间**: 2026-11-06 14:28 CST
- **摘要**: ### Your current environment  Found by static code inspection against `main` @ `e617b70` (2026-11-06). No NPU runtime reproduction yet.  ### 🐛 Describe the bug  #20213 fixed stale `block_hashes` after P-side prompt truncation in the mooncake path, adding the hash rebuild in two places — `kv_p2p/moon…

## 🔀 Pull Requests

### #21175 — [[Doc][v0.32.0rc] Switch GLM-5.3 serving examples to Model Runner V2](https://github.com/vllm-project/vllm-ascend/pull/21175)
- **作者**: Wyz-134  **时间**: 2026-11-06 15:10 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Switch all 32 serving examples in the shared GLM-5.2/GLM-5.3 guide to explicit Model Runner V2. Update existing runner selections, add the missing selections to four examples, and remove the obsolete `enable_dsa_cp` switch and related descriptions. Update run…

### #21174 — [[Doc] Switch GLM-5.3 serving examples to Model Runner V2](https://github.com/vllm-project/vllm-ascend/pull/21174)
- **作者**: Wyz-134  **时间**: 2026-11-06 15:10 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Switch all 32 serving examples in the shared GLM-5.2/GLM-5.3 guide to explicit Model Runner V2. Update existing runner selections, add the missing selections to four examples, and remove the obsolete `enable_dsa_cp` switch and related descriptions. Update run…

### #21173 — [[Verify] Nightly test on main](https://github.com/vllm-project/vllm-ascend/pull/21173)
- **作者**: LQDLove  **时间**: 2026-11-06 15:04 CST
- **标签**: documentation
- **摘要**: Auto-created empty PR for nightly verification. - vLLM main: https://github.com/vllm-project/vllm/commit/24420b7a341af37a9bc3186e6fc6062512e0aa58

### #21172 — [[Performance][ACLGraph] Capture decode metadata preparation on an auxiliary ACL Graph stream](https://github.com/vllm-project/vllm-ascend/pull/21172)
- **作者**: jwkkkkkkdada  **时间**: 2026-11-06 14:57 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? On Ascend A2, DeepSeek V4.1-Flash prepares cache slot mappings and attention/indexer metadata before replaying the decode graph. This adds host submissions on every decode step and puts metadata preparation on the critical path.  This PR captures C2 compressed…

### #21170 — [[Doc] Improve MiniMax, Qwen3 and Step model tutorials](https://github.com/vllm-project/vllm-ascend/pull/21170)
- **作者**: U1stRsouland  **时间**: 2026-11-06 14:48 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Several model tutorials have gaps between their deployment examples, chapter structure, and verification instructions. This PR makes those workflows easier to follow while retaining the existing serving configurations and reported accuracy results.  - **MiniM…

### #21169 — [[Attention][Feature] Gemma4 512-dim attention: B1 paged verify,    FULL cudagraph and FIA TND](https://github.com/vllm-project/vllm-ascend/pull/21169)
- **作者**: swimming2007-doge  **时间**: 2026-11-06 14:44 CST
- **标签**: module:core
- **摘要**: [ops_transformer_fia_tnd_hdim512_d3235e3.patch](https://github.com/user-attachments/files/34465934/ops_transformer_fia_tnd_hdim512_d3235e3.patch) # ops-transformer FIA TND + headDim=512 补丁使用说明  （2026-11-06，适用 CANN 9.2.0 / ascend910b）  > 目标读者：需要在另一套环境里复现「FusedInferAttentionScore（FIA）支持 TND 布局 + head_…

### #21168 — [[BugFix][DSpark] Fix DeepSeek V4 expert weight mapping with redundant experts](https://github.com/vllm-project/vllm-ascend/pull/21168)
- **作者**: JudgingH  **时间**: 2026-11-06 14:37 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  DeepSeek V4 DSpark constructs its MoE layers with the configured redundant experts, but its checkpoint loader builds the expert mapping with `num_redundant_experts=0`. With MRv2's distributed initial layout, the loader can therefore write a logical expert int…

### #21167 — [[BugFix] Keep AscendStore page transfers compatible with prefix-match-unit](https://github.com/vllm-project/vllm-ascend/pull/21167)
- **作者**: iKeybot-code  **时间**: 2026-11-06 14:35 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  AscendStore previously derived hash granularity independently of vLLM Core. Use the Core-resolved hash size and local APC hit alignment for both scheduler and worker, while deriving the bulk transfer boundary from the effective physical pages of the cacheable…

### #21163 — [[Verify] Nightly test on releases/v0.32.0rc](https://github.com/vllm-project/vllm-ascend/pull/21163)
- **作者**: LQDLove  **时间**: 2026-11-06 14:25 CST
- **标签**: documentation
- **摘要**: Auto-created empty PR for nightly verification. - vLLM main: https://github.com/vllm-project/vllm/commit/544f182d9f0c9bfe8bd7645525951d5e3692a542

### #21162 — [[Ops][Refactor]Remove unused kv_quant sparse attention shared-KV operators](https://github.com/vllm-project/vllm-ascend/pull/21162)
- **作者**: lcfenglinwan  **时间**: 2026-11-06 14:06 CST
- **标签**: documentation, module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  Remove the unused `npu_kv_quant_sparse_attn_sharedkv` and `npu_kv_quant_sparse_attn_sharedkv_metadata` custom operators.  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary …
