# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-14 16:28 CST

## AI 总结

以下是为您整理的 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态摘要。
*注：本次提供的动态数据中仅包含 Pull Request (PR)，未包含 Issue 和 Release 信息，故重点对 PR 进行分类归纳。*

### 🛠️ Pull Request (PR) 活动摘要
近期 PR 主要围绕性能优化、投机解码修复、新模型/特性支持以及 CI 测试改进展开：

**1. 重要性能优化与 Kernel 修复**
*   **MiniMax-M3 A5 Decode 性能优化** (#16515)：在 `v0.27.1rc` 分支中启用了优化的 AscendC `MsaIndexScore` 路径，提升 MiniMax-M3 A5 的解码性能。
*   **修复 EPLB 精度下降问题** (#16507)：修复了 MoE 模型在启用 EPLB（专家并行负载均衡）时，由于 `grouped_matmul_swiglu_quant_v2` 中 Tensor List 地址解析错误导致的严重精度下降问题。

**2. 投机解码 修复**
*   **DSpark Drafting 超限问题** (#16512, #16513, #16514)：修复了 DSpark 在目标模型步骤后追加并行 draft query 组时，因采样配置导致数量超限的问题。该修复已同步至主分支、`v0.27.1rc` 和 `v0.26.0rc` 版本。

**3. 新模型与新特性支持**
*   **Kimi K3 跨 PP 阶段序列并行** (#16511)：移除了原先 PP > 1 时禁用序列并行的限制，支持 Kimi-K3 模型在流水线并行 (PP) 阶段间正确使用序列并行，避免了输入被错误重复分片。
*   **P/D 抢占式卸载** (#16510)：为 MRV2 在 P/D 解码节点中引入了 preempt offload 支持，并移除了不再适用的 P-recompute 功能。

**4. 其他 Bug 修复**
*   **xlite 后端配置修复** (#16506)：修复了 `v0.26.0rc1` 中使用 `xlite` 作为 DP 部署后端时，由于 `local_rank` 语义变更导致的设备 ID 配置错误及 token 数量设置问题。

**5. 文档与 CI 测试**
*   **文档更新** (#16509)：补充了 MiniMax-M3 的 EAGLE3 权重链接与 KV cache pool PD 分离部署指南，并修改了最大模型长度限制。
*   **CI 测试拆分** (#16508)：将耗时较长的 MRv2 基础 E2E 测试（14 个参数化用例，约 2650 秒）进行拆分，以支持 CI 的并行调度，提升测试效率。

---

## 🔀 Pull Requests

### #16515 — [[v0.27.1rc][Performance] Optimize MiniMax-M3 A5 AscendC index score decode](https://github.com/vllm-project/vllm-ascend/pull/16515)
- **作者**: HaoxinZong  **时间**: 2026-09-14 16:06 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  This PR enables the optimized AscendC `MsaIndexScore` path for MiniMax-M3 A5 decode on `releases/v0.27.1rc`.  It contains two related changes:  - selectively ports the missing `kvChunks` short-M/long-KV optimization and A2/A5 fixes from ops-transformer PR 117…

### #16514 — [[v0.26.0rc][BugFix][SpecDecode] Skip over-limit DSpark drafting with DP alignment](https://github.com/vllm-project/vllm-ascend/pull/16514)
- **作者**: zhenwenqi2024  **时间**: 2026-09-14 16:03 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  DSpark appends one complete parallel draft query group after the target-model step. Depending on the configured sampling mode, that group is `num_speculative_tokens` or `num_speculative_tokens + 1` tokens wide. Near `max_model_len`, the target step can still …

### #16513 — [[v0.27.1rc][BugFix][SpecDecode] Skip over-limit DSpark drafting with DP alignment](https://github.com/vllm-project/vllm-ascend/pull/16513)
- **作者**: zhenwenqi2024  **时间**: 2026-09-14 16:03 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  DSpark appends one complete parallel draft query group after the target-model step. Depending on the configured sampling mode, that group is `num_speculative_tokens` or `num_speculative_tokens + 1` tokens wide. Near `max_model_len`, the target step can still …

### #16512 — [[BugFix][SpecDecode] Skip over-limit DSpark drafting with DP alignment](https://github.com/vllm-project/vllm-ascend/pull/16512)
- **作者**: zhenwenqi2024  **时间**: 2026-09-14 15:56 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  DSpark appends one complete parallel draft query group after the target-model step. Depending on the configured sampling mode, that group is `num_speculative_tokens` or `num_speculative_tokens + 1` tokens wide. Near `max_model_len`, the target step can still …

### #16511 — [[Feature][Model] Support Kimi K3 sequence parallelism across PP stages](https://github.com/vllm-project/vllm-ascend/pull/16511)
- **作者**: Dawn952  **时间**: 2026-09-14 15:49 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Kimi-K3 currently disables sequence parallelism when PP > 1. Removing that guard alone would shard already-sharded inputs again and reject the residual received from the preceding stage.  This change shards embeddings only on the first PP stage and carries hi…

### #16510 — [[Misc][P/D][KVOffload] Support preempt offload for MRV2 in P/D decoder nodes && Drop P-recompute function](https://github.com/vllm-project/vllm-ascend/pull/16510)
- **作者**: nwpu-zxr  **时间**: 2026-09-14 15:48 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it? This PR has the following major changes: 1. The recompute offload connector is renamed as preempt offload connector, which is more appropriate to its actual behavior. That is, the preempted requests are offloaded to the host DRAM to avoid recomputation. 2. The…

### #16509 — [[doc]: add EAGLE3 weight link and KV cache pool PD guide for MiniMax-M3 and modifyed max model len](https://github.com/vllm-project/vllm-ascend/pull/16509)
- **作者**: FORFuture37  **时间**: 2026-09-14 15:39 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  This PR supersedes #16173 and #16399. #16173 (squash commit `bc7c2fe`) already landed the MiniMax-M3 PD disaggregation tutorial on `releases/v0.27.1rc`, but the same fork branch was later reused for follow-up changes, which made #16399 diverge (12 ahead / 21 …

### #16508 — [[Test][CI] Split MRv2 basic E2E tests for parallel scheduling](https://github.com/vllm-project/vllm-ascend/pull/16508)
- **作者**: Liamup777  **时间**: 2026-09-14 15:37 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  The ModelRunner V2 basic E2E test file contains 14 parameterized cases and takes approximately 2650 seconds to run. Since CI schedules tests at file granularity, all cases in the combined file run serially on one NPU runner.  Split the combined test into six …

### #16507 — [[Kernel] Fix EPLB Tensor List address resolution for weight_assist_matrix in grouped_matmul_swiglu_quant_v2](https://github.com/vllm-project/vllm-ascend/pull/16507)
- **作者**: baolongsun  **时间**: 2026-09-14 15:20 CST
- **摘要**: ### What this PR does / why we need it?  **Symptom:** When running MoE models with **EPLB** (Expert Parallelism Load Balancing) enabled, severe accuracy or logits degradation occurs during expert switching. Disabling EPLB resolves the issue.  **Root Cause:** When EPLB is enabled in Model Runner V2, …

### #16506 — [[v0.26.0rc][BugFix][xlite] Fix xlite device id configuration (local_rank semantic changed), and set max_num_tokens/num_tokens correctly](https://github.com/vllm-project/vllm-ascend/pull/16506)
- **作者**: SijieFu  **时间**: 2026-09-14 15:17 CST
- **摘要**: ### What this PR does / why we need it?  This PR fixes a concurrent issue in `v0.26.0rc1` when using `xlite` as the backend under DP deployment - due to the semantic changes in variable definitions such as `local_rank` and `num_tokens`.  PR #13378 included the bug fixes but they were only merged int…
