# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-08 14:42 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### Issue
*本期无公开的 Issue 动态。*

### Release
*本期无新的 Release 动态。*

### Pull Request (PR)
近期 PR 活动主要围绕新模型适配、功能增强、性能优化及文档更新展开，具体如下：

*   **新模型支持**
    *   **MiniCPM3-4B 适配**：为 `openbmb/MiniCPM3-4B` 补齐 Ascend（A2, BF16, TP1）使能材料，由于上游 vLLM 已内置该模型支持，无需额外专用补丁。(#21265)

*   **新特性**
    *   **Mooncake KV 存储支持**：使上游 vLLM 的 `MooncakeStoreConnector` 能够在 Ascend 上运行，通过可选的 worker 兼容补丁处理遗留 Fabric 初始化。(#21258)
    *   **GLM5.3 Flash Prefill 卸载**：为 PD（Prefill-Decode）producers 引入 GLM5.3 Flash prefill 卸载功能，使用 `AscendStoreConnector` 并支持共享 `KVCacheConfig`。(#21256)
    *   **EPLB 专家放置导出**：在 Model Runner V2 (MRV2) 中导出已提交的 EPLB 专家物理到逻辑映射，支持可复用的专家放置。(#21260)

*   **性能优化**
    *   **DeepSeek V4 Decode 提速**：在完整 decode 图中捕获 DeepSeek V4 目标模型元数据，避免重复的主机执行器构造和 CPU 标量准备。(#21259)
    *   **KV Pool 批量加载**：跨缓存组批量加载层级元数据，分别获取读租约和刷新地址，以提升处理效率。(#21262)

*   **Bug 修复**
    *   **EPLB 放置评估修正**：修复 STAIR 在评分时因交替的专家热点在时间分箱内互相抵消，导致放置方案变差的问题，现基于每步负载进行验证。(#21261)
    *   **KVTransfer Prefix 查找修复**：在调度器查询本地前缀缓存之前，规范化混合 PD producer prompts，修复缓存匹配回调问题。(#21257)

*   **文档更新**
    *   **GLM-5.3-Flash A2 配置指南**：更新 A2 Prefill/Decode (PD) 和 KV 池配置指导，细化 DP/TP/PCP 参数、FlashComm1 设置、调度器与编译选项，并将 A2 Prefill 和 Decode 的 MTP 推测性 token 设为 3。(#21263, #21264)

---

## 🔀 Pull Requests

### #21265 — [[Model] Add MiniCPM3-4B support on Ascend (A2, BF16, TP1)](https://github.com/vllm-project/vllm-ascend/pull/21265)
- **作者**: name-jiu  **时间**: 2026-11-08 14:27 CST
- **标签**: documentation, module:tests
- **摘要**: ## 目的  为 **openbmb/MiniCPM3-4B** 补齐 Ascend 使能材料。该模型为 62 层 / 隐藏维度 2560 的稠密模型，采用 MLA 注意力、SwiGLU MLP 和 LongRoPE（4B 参数，理论上下文 32K）。  上游 vLLM 已内置 `MiniCPM3ForCausalLM`（注意力、权重映射、采样集成、LongRoPE 处理），因此**无需任何模型专用 kernel 或建模层补丁**。缺的是将其在 Ascend 上验收合格所需的全部材料：一份 E2E 配置、一份生成用例、一篇教程，以及支持矩阵 / 每日任务（nightly）的条目。  ## 本 …

### #21264 — [[Doc] Update GLM-5.3-Flash A2 KV pool configuration](https://github.com/vllm-project/vllm-ascend/pull/21264)
- **作者**: panther-zhu  **时间**: 2026-11-08 13:40 CST
- **标签**: documentation
- **摘要**: ## Summary     - Update A2 Prefill guidance for DP2/TP1/PCP8, including the batch-token limit and FlashComm1/shared-expert settings.   - Set MTP speculative tokens to 3 for A2 Prefill and Decode.   - Replace the A2 Mooncake pool backend guidance with MemCache over RoCE, while retaining MooncakeConne…

### #21263 — [[Doc] Update GLM-5.3-Flash A2 PD and KV pool settings](https://github.com/vllm-project/vllm-ascend/pull/21263)
- **作者**: panther-zhu  **时间**: 2026-11-08 13:38 CST
- **标签**: documentation
- **摘要**: ## Summary     - Update A2 Prefill/Decode guidance to use Prefill DP2/TP1/PCP8 and Decode DP8/TP2/PCP1.   - Set Prefill FlashComm1 and shared-expert DP on; update Decode scheduler, memory, and compilation settings.   - Document MemCache pooling over A2 RoCE while retaining MooncakeConnectorV2 for Pr…

### #21262 — [[Performance][KV Pool] Batch layerwise load metadata across cache groups](https://github.com/vllm-project/vllm-ascend/pull/21262)
- **作者**: bowgneo  **时间**: 2026-11-08 02:05 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Layerwise loads query metadata, acquire a read lease and refresh addresses separately for every cache group. Collect each request's group keys first, issue these calls as one batch, then scatter addresses to the original full and partial-page destinations. Fo…

### #21261 — [[BugFix][EPLB] Validate STAIR placements against per-step load](https://github.com/vllm-project/vllm-ascend/pull/21261)
- **作者**: WanhaoZhang  **时间**: 2026-11-07 23:26 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Refs #20759. STAIR currently averages chronological bins before scoring a candidate. Alternating expert hot spots can cancel inside each bin, making a worse placement look better.  Keep one row per recorded step through the existing EPLB load reduction. The C…

### #21260 — [[Feature][EPLB] Export committed MRV2 expert placement](https://github.com/vllm-project/vllm-ascend/pull/21260)
- **作者**: WanhaoZhang  **时间**: 2026-11-07 23:09 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Refs #20602, workstream 3. This draft covers the export side of reusable EPLB placement in Model Runner V2.  Export the active per-layer physical-to-logical map from EP rank 0 after the asynchronous workspace commit and routing refresh complete. The writer wa…

### #21259 — [[Performance][Model] Capture DeepSeek V4 decode metadata in full graphs](https://github.com/vllm-project/vllm-ascend/pull/21259)
- **作者**: zzt93  **时间**: 2026-11-07 22:40 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Capture DeepSeek V4 target-model SAS/QLI metadata producers in FULL decode graphs, avoiding repeated host executor construction and CPU scalar preparation. Persistent inputs are refreshed before replay; producer outputs retain stable storage and existing strea…

### #21258 — [[Feature][KVStore] Support upstream vLLM MooncakeStoreConnector on Ascend](https://github.com/vllm-project/vllm-ascend/pull/21258)
- **作者**: LCAIZJ  **时间**: 2026-11-07 20:33 CST
- **标签**: documentation, module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  Support using upstream vLLM's `MooncakeStoreConnector` on Ascend through an optional worker compatibility patch for legacy Fabric initialization and NPU transfer-thread device binding.  **Depends on [vllm-project/vllm#66271](https://github.com/vllm-project/vl…

### #21257 — [[BugFix][KVTransfer] Normalize layerwise producer prompts before prefix lookup](https://github.com/vllm-project/vllm-ascend/pull/21257)
- **作者**: bowgneo  **时间**: 2026-11-07 19:50 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Normalize hybrid PD producer prompts in the existing `on_new_request` hook, before the scheduler queries the local prefix cache. The cache-matching callback then keeps the request length unchanged.  Previously, a repeated prompt could first match all but its …

### #21256 — [[Feature][KV Pool] Add GLM5.3 Flash prefill offload for PD producers](https://github.com/vllm-project/vllm-ascend/pull/21256)
- **作者**: bowgneo  **时间**: 2026-11-07 19:34 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  Add **GLM5.3 Flash prefill offload for PD producers** using `AscendStoreConnector` and shared `KVCacheConfig` backing.  - Layers **0/1/2/3** retain one independent pack: KDA 0/1/2 and main KV 3 share one tensor. - Later main packs rotate through **four shared…
