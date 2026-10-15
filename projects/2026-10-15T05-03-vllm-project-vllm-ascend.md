# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-15 13:03 CST

## AI 总结

## vllm-ascend 仓库动态摘要（2026-10-15）

---

### 📌 Issue

两条 Issue 均为 **Nightly Daily 测试失败报告**，由 `czydyy` 提交：

- **#18815 / #18814**：2026-10-15 08:00 的夜间测试在 **a3-2 卡（part 5-5）、a3-4 卡（part 4-5、part 2-5）** 以及 **Kimi-K2.6-w4a8-A3** 模型上失败，涉及 `test_hccl_weight` 等测试用例。需相关开发者排查修复。

---

### 🔧 Pull Request（共 10 条）

#### Bug 修复类
| PR | 摘要 |
|---|---|
| #18823 | **[KVTransfer]** 修复 Mooncake HMA 在多组逻辑缓存或需 HMA 场景下，通过 `failed_recving` 按请求报告 P2P 接收失败 |
| #18817 | **[WeightTransfer]** 修复打包 HCCL 权重传输路径中，单个 chunk 跨多个 NPU 流导致丢失流顺序的问题 |
| #18816 | **[Compilation]** 回退外部 DP（数据并行）静态 kernel worker 推断（#18720），恢复原有 `LOCAL_WORLD_SIZE` 计算，为受控回滚 |
| #18813 / #18812 | **[KVPool]** 修复 GVA 逐层传输线程在非零返回码时直接抛异常的问题——因池缓存为可选项，瞬时失败不应中断流程。#18813 为 v0.30.0 分支的 backport |

#### 性能优化类
| PR | 摘要 |
|---|---|
| #18821 | **[v0.30.0rc2][Feature]** 将 MegaMoe 引入 A2 prefill，支持可配置 token 阈值，并将 `moe_intermediate_size` 下限从 1024 降至 512 |
| #18820 | **[MoE]** 在 v0.30.0 分支上移除强制 FP32 路由权重预转换，与 main 分支对齐 |
| #18818 | **[MoE]** 对 Kimi K3 混合量化 MoE，将共享专家 W8A8 Gate-Up 矩阵乘与路由专家 W4A8 GMM1 并行启动，提升 Cube/AIC 资源利用率 |

#### 文档与工程类
| PR | 摘要 |
|---|---|
| #18822 | **[Doc]** 更新 CPP 相关文档，使其与当前调度器和配置行为一致（如 ShortRequestFirst 与 profiling-chunk 的组合说明） |
| #18819 | **[Community]** 新增 release note fragments 工作流——贡献者在同一 PR 中声明面向用户的变更片段，替代发布时全量回溯 commit 历史 |

---

### 🚀 Release

> 本次数据中 **无新版本发布**。但多个 PR 围绕 **v0.30.0rc2 / v0.30.0** 发布分支进行 backport 和优化，表明该版本正在积极收尾中，重点方向包括：
> - **MoE 性能**：MegaMoe 支持、FP32 路由权重优化、Kimi K3 并行调度
> - **KV/权重传输健壮性**：Mooncake HMA 错误报告、层式传输容错、流顺序修复
> - **工程流程**：release note 自动化片段机制

---

## 🐛 Issues

### #18815 — [Nightly Daily test failure on 2026-10-15 08:00 - please check](https://github.com/vllm-project/vllm-ascend/issues/18815)
- **作者**: czydyy  **时间**: 2026-10-15 12:11 CST
- **摘要**: The nightly Daily test for vllm-ascend reported failures at 08:00 on 2026-10-15.  **Failed on:** a3-2 card-(part 5-5), a3-4 card-(part 4-5), a3-4 card-(part 2-5)  **Failed test cases:** - test_hccl_weight_transfer_transaction — AssertionError: qwen3.5-35b-a3b-moe-layout: repeated live reload changed…

### #18814 — [Nightly Daily test failure on 2026-10-15 08:00 - please check](https://github.com/vllm-project/vllm-ascend/issues/18814)
- **作者**: czydyy  **时间**: 2026-10-15 11:52 CST
- **摘要**: The nightly Daily test for vllm-ascend reported failures at 08:00 on 2026-10-15.  **Failed on:** a3-2 card-(part 5-5), a3-4 card-(part 4-5), a3-4 card-(part 2-5), Kimi-K2.6-w4a8-A3 **Failed test cases:** test_hccl_weight_transfer_transaction, test_sequence_parallel_moe_dp2_tp2_functional, test_glm53…

## 🔀 Pull Requests

### #18823 — [[BugFix][KVTransfer] Report Mooncake HMA receive failures by request](https://github.com/vllm-project/vllm-ascend/pull/18823)
- **作者**: zhao-stack  **时间**: 2026-10-15 12:52 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Report P2P Mooncake receive failures through `KVConnectorTransferResults.failed_recving` when the logical cache has multiple groups or requires HMA. Flat block IDs do not identify the affected request across independent cache groups; vLLM 0.31 rejects such re…

### #18822 — [[Doc][CPP] Align CPP documentation with scheduler behavior](https://github.com/vllm-project/vllm-ascend/pull/18822)
- **作者**: dianWang325  **时间**: 2026-10-15 12:47 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Align the CPP-related documentation with the current scheduler and configuration behavior:  - document that ShortRequestFirst can be composed with profiling-chunk scheduling and recommend Model Runner V2 for CPP deployments that may use asynchronous schedulin…

### #18821 — [[v0.30.0rc2][Ops][Feature] Enable MegaMoe for A2 prefill with configurable token threshold](https://github.com/vllm-project/vllm-ascend/pull/18821)
- **作者**: wxhhuawei  **时间**: 2026-10-15 12:46 CST
- **标签**: module:core
- **摘要**: This PR ports #16772 onto the `v0.30.0rc2` release branch.  ### Changes - Relax MegaMoe `moe_intermediate_size` lower bound from 1024 to 512. - Enable `CANN_MEGAMOE` capability on Ascend A2. - Select `FUSED_MC2` for A2 prefill/profile_run only when token count > 64; otherwise keep existing MC2/ALLGA…

### #18820 — [[Performance][MoE] Stop forcing FP32 router weights on v0.30.0](https://github.com/vllm-project/vllm-ascend/pull/18820)
- **作者**: Karryking3  **时间**: 2026-10-15 12:40 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Align releases/v0.30.0 with main by removing the unconditional gate.precast_fp32_weight = True assignment from AscendMoERunner.__init__. Gates without weight_fp32 are no longer forced by the runner to materialize an FP32 weight at load time. Models that alrea…

### #18819 — [[Community] add release note fragments workflow](https://github.com/vllm-project/vllm-ascend/pull/18819)
- **作者**: wangxiyuan  **时间**: 2026-10-15 12:38 CST
- **标签**: documentation
- **摘要**: Contributors now declare user-facing changes as a fragment in the same PR, so release notes are assembled from fresh context instead of re-analyzing the whole commit history at release time.  - Add release_notes/ with the fragment template, principles, and archive - Add collect_release_notes.py: val…

### #18818 — [[Performance][MoE] Stage Kimi shared Gate-Up after routed GMM1](https://github.com/vllm-project/vllm-ascend/pull/18818)
- **作者**: zhenwenqi2024  **时间**: 2026-10-15 12:36 CST
- **标签**: module:tests, module:ops, module:quantization
- **摘要**: ### What this PR does / why we need it?  Kimi K3's mixed-quantization MoE can launch the shared-expert W8A8 Gate-Up matmul alongside the routed-expert W4A8 GMM1. Both use Cube/AIC resources, so that overlap may be contention rather than useful Cube/Vector overlap.  This draft stages shared Gate-Up a…

### #18817 — [[BugFix][WeightTransfer] Run each packed chunk on its buffer slot's stream](https://github.com/vllm-project/vllm-ascend/pull/18817)
- **作者**: Ronald1995  **时间**: 2026-10-15 12:32 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  The packed HCCL weight-transfer path spread one chunk's work over several NPU streams and lost the stream ordering that the layerwise reload depends on. This ports the scheme upstream vLLM already uses for the same path (`vllm/distributed/weight_transfer/pack…

### #18816 — [[BugFix][Compilation] Revert external DP static kernel worker inference](https://github.com/vllm-project/vllm-ascend/pull/18816)
- **作者**: LQDLove  **时间**: 2026-10-15 12:29 CST
- **摘要**: ### What this PR does / why we need it?  Revert #18720 and restore the previous `LOCAL_WORLD_SIZE` calculation for static kernel compilation. This is a controlled rollback while the external DP behavior and recent nightly failures are investigated.  The Qwen3.5-397B-A17B-w8a8-mtp AIME2025 result was…

### #18813 — [[BugFix][KVPool] Recover ordinary layerwise pool transfer failures (v0.30.0)](https://github.com/vllm-project/vllm-ascend/pull/18813)
- **作者**: ader47  **时间**: 2026-10-15 11:52 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  Backport of https://github.com/vllm-project/vllm-ascend/pull/18812 to `releases/v0.30.0`.  GVA layerwise transfer threads currently raise on nonzero copy/write-completion return codes even when the pool is only an optional cache. A transient pool failure can …

### #18812 — [[BugFix][KVPool] Recover ordinary layerwise pool transfer failures](https://github.com/vllm-project/vllm-ascend/pull/18812)
- **作者**: ader47  **时间**: 2026-10-15 11:51 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  GVA layerwise transfer threads currently raise on nonzero copy/write-completion return codes even when the pool is only an optional cache. A transient pool failure can therefore become a fatal worker error.  - Make ordinary layerwise pool saves best-effort wh…
