# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-21 13:20 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 最近动态的中文摘要：

### 📦 Release（版本发布）
*   **准备 v0.23.0.post1 版本** (#17063)
    *   **版本亮点**：主要修复了在 DP-all-to-all 通信期间出现的 KV-cache slot（缓存槽）元数据过时/失效的问题，并更新了相关发布文档。

### 🔀 Pull Requests（代码合并请求）

**1. 新特性与模型支持**
*   **UNO 投机解码** (#17069)：新增 UNO 投机解码支持。复用目标 Qwen3 模型并结合门控 draft LoRA，无需单独引入 draft 模型，并支持 `FULL_DECODE_ONLY` 树状 draft 图。
*   **Kimi K3 MLA Prolog V3 支持** (#17065)：将 `MlaPrologV3` 实现移植到 `releases/v0.27.1rc` 分支，并在 A3 架构上启用符合条件的 Kimi K3 解码路径。
*   **MoE 序列并行增强** (#17061)：在 `DP=1`（数据并行大小为1）的拓扑结构下启用 MoE 序列并行（SP），利用 Ascend FlashComm 支持 TP/EP 拓扑。

**2. 性能优化**
*   **DFlash 投机解码算子融合** (#17070)：将 DFlash 路径中的 `context-KV` 预计算操作融合进批处理中，将每个 prefill 步骤的算子启动次数从约 16 次大幅减少。
*   **MoE Router 精度与性能优化** (#17068)：对于原始为 bf16 的权重，改用 bf16 权重矩阵乘法（GEMM）配合 fp32 累加，替代之前全程强制使用 fp32 计算的方式。
*   **DeepSeek V4 RoPE 查找优化** (#17067)：移除了冗余的 DeepSeek V4 RoPE 表索引，支持通过层过滤器复用已过滤的查找结果。
*   **Block-table 提交优化** (#17066)：在模型运行器 v1 中，将每步全宽度的 block-table H2D（主机到设备）拷贝替换为仅提交“脏数据”范围，大幅减少数据传输量。

**3. Bug 修复**
*   **修复 MRV2 下的 DCP 元数据缺失问题** (#17064)：修复了 MRV2 在 MLA 后端消费 `context_parallel_metadata` 前未完全填充的问题，解决了报错 `DCP metadata must be populated.` 的错误。

**4. 测试与基础设施**
*   **DSV4 Pro 夜间测试配置更新** (#17062)：更新了 DeepSeek-V4-Pro 的夜间测试配置，测试将 FlashComm1 的启用从旧版环境变量中剥离出来的影响。

### 🐛 Issue（问题反馈）
*   *近期暂无公开的 Issue 动态。*

---

## 🔀 Pull Requests

### #17070 — [[Performance][Spec Decode] Fuse DFlash context-KV precompute into batched operations](https://github.com/vllm-project/vllm-ascend/pull/17070)
- **作者**: curnane-lab  **时间**: 2026-09-21 12:34 CST
- **摘要**: ## What this PR does / why we need it?  `precompute_and_store_context_kv` in the DFlash speculative decoding path currently issues ~16 kernel launches per prefill step for 5 draft layers:  | Operation | Launches | Notes | |---|---|---| | hidden RMSNorm | 1 | | | Fused KV GEMM | 1 | Already KV-only (…

### #17069 — [[Feature][Spec Decode] Add UNO speculative decoding with FULL_DECODE_ONLY tree draft graphs](https://github.com/vllm-project/vllm-ascend/pull/17069)
- **作者**: Liuchenbing-2026  **时间**: 2026-09-21 12:11 CST
- **标签**: module:tests, module:ops, module:core, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  Adds UNO speculative decoding support to vllm-ascend: the target Qwen3 model is reused with a gated draft LoRA, so no separate draft model is needed.  - **Draft graphs.** Separate `FULL_DECODE_ONLY` draft graphs are captured with   stable LoRA masks, exact re…

### #17068 — [[Performance][MoE] Use bf16 router GEMM with fp32 accumulation for bf16-origin weights](https://github.com/vllm-project/vllm-ascend/pull/17068)
- **作者**: Levi-JQ  **时间**: 2026-09-21 12:03 CST
- **标签**: module:ops
- **摘要**: ## What this PR does / why we need it  `AscendGateLinear` (PR #9617) forces fp32 weights and fp32 computation for the MoE router gate, because the previous `bf16×bf16→bf16` path (bf16 accumulation, no fp32 output) caused expert-selection errors in agent workloads on NPU.  However, the fp32 path (`ac…

### #17067 — [[Performance][DSV4] Reuse filtered RoPE lookups](https://github.com/vllm-project/vllm-ascend/pull/17067)
- **作者**: pisceskkk  **时间**: 2026-09-21 12:02 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  This PR removes redundant DeepSeek V4 RoPE table indexing that remains on current main.  - adds an optional layer filter to `get_cos_and_sin_dsa` so draft attention queries only the RoPE config consumed by its layers - resolves draft `*.swa_cache` layer names…

### #17066 — [[Performance] Commit only dirty block-table ranges](https://github.com/vllm-project/vllm-ascend/pull/17066)
- **作者**: pisceskkk  **时间**: 2026-09-21 12:02 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  This PR replaces the per-step full-width block-table H2D copy in model runner v1 with dirty-range commits.  - tracks the changed interval for each request row - packs changed block IDs and compact scatter metadata into double-buffered staging buffers - scatte…

### #17065 — [[Feat][Model] Enable Kimi K3 MLA Prolog V3 on A3](https://github.com/vllm-project/vllm-ascend/pull/17065)
- **作者**: MaybeChz  **时间**: 2026-09-21 11:58 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Ports the A2/A3 (arch22) `MlaPrologV3` implementation from #13844 onto the `releases/v0.27.1rc` branch and enables the eligible Kimi K3 A3 decode path.  - Adds the 12 `op_kernel/arch22/*` source files byte-for-byte from #13844. - Registers and builds `mla_pro…

### #17064 — [[BugFix][MRV2] Build missing DCP metadata in MLA backend](https://github.com/vllm-project/vllm-ascend/pull/17064)
- **作者**: recky-c  **时间**: 2026-09-21 11:56 CST
- **标签**: module:tests
- **摘要**: MRV2 does not always populate `AscendCommonAttentionMetadata.context_parallel_metadata` before the MLA DCP builder consumes it, which causes `DCP metadata must be populated.` during metadata construction.  This keeps the fix local to `AscendMlaDCPMetadataBuilder`: when the runner has not supplied DC…

### #17063 — [[Doc][Release] Prepare v0.23.0.post1](https://github.com/vllm-project/vllm-ascend/pull/17063)
- **作者**: yiz-liu  **时间**: 2026-09-21 11:54 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Prepare the vLLM Ascend v0.23.0.post1 release documentation on top of the `releases/v0.23.0` branch. The primary fix is stale KV-cache slot metadata during DP-aligned dummy runs from [#15362](https://github.com/vllm-project/vllm-ascend/pull/15362). The releas…

### #17062 — [[Test] Update DSV4 Pro Nightly FlashComm1 configuration](https://github.com/vllm-project/vllm-ascend/pull/17062)
- **作者**: pgzddxx  **时间**: 2026-09-21 11:53 CST
- **标签**: module:tests
- **摘要**: ## Purpose  This is a configuration-only measurement carrier for the `DeepSeek-V4-Pro-w4a8-prefix-cache-PD` Nightly case. It tests whether moving FlashComm1 enablement from the legacy environment variable path to `--additional-config` changes performance.  This PR is not a confirmed performance fix …

### #17061 — [[Feature][SP] Enable MoE SP with DP=1](https://github.com/vllm-project/vllm-ascend/pull/17061)
- **作者**: jiaqi-lee  **时间**: 2026-09-21 11:52 CST
- **标签**: module:tests
- **摘要**: ## What this PR does / why we need it?  Upstream enables MoE sequence parallelism only when `data_parallel_size > 1`. However, Ascend FlashComm also supports the TP/EP, `DP=1` topology, which requires the same rank-local token sharding layout.  This PR adds an Ascend platform patch for `ParallelConf…
