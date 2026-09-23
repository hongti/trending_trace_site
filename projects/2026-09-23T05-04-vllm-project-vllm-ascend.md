# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-23 13:04 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### Issue 摘要
1. **Mamba 前缀检查点特性移植 (#17352)**
   - 作者: curnane-lab
   - 内容: 请求将 vLLM 主干中针对混合 Mamba/GDN 模型的“应用导向前缀检查点”机制（由 3 个 PR 组成的系列）移植到 Ascend NPU 上。
2. **混合模型的 int8 KV Cache 支持 (#17347)**
   - 作者: qys4123
   - 内容: 请求为 GLM-5.3-Flash / DeepSeek-V4 风格的混合架构模型提供端到端的 `--kv-cache-dtype int8` 支持。目前预填充路径已能正常运行，但解码路径在 `aclnnSparseFlashAttention` 处失败。

### PR 摘要
**🚀 新特性与适配**
- **#17355**: 使 MooncakeConnector V2 支持 PCP（预填充上下文提供者）。
- **#17350**: 将 vllm-ascend 适配至 9 月 16 日的 vLLM 主干代码。
- **#17345**: 补充 MiniMax-M3 的 A2 系列 (W8A8) 多节点部署脚本与文档。

**🐛 Bug 修复**
- **#17344**: 修复长上下文预填充中融合 MC2 MoE 路径的静默输出损坏问题，为其增加防容量截断保护。
- **#17348**: 修复 Engram HOST_UVA 卸载场景下，CPU 绑定时因错误执行页面迁移 (`migratepages`) 导致的问题。
- **#17351**: 修复 CI 每日构建中，基础镜像分支错误地从 PR head SHA 解析导致构建失败的问题。

**⚡ 性能优化**
- **#17353**: 优化融合槽位映射，避免针对“请求计数”进行特化，从而减少额外的 Triton 编译开销。
- **#17346**: 优化 DSA 上下文并行元数据内核，同样避免请求计数特化，防止每次请求计数生成独立内核。

**🧪 测试与其他**
- **#17349**: 将 MiniMax-M2.7 的端到端测试用例从每日测试套件移至每周测试套件。
- **#17354**: 替换 sas（细节未提供）。

### Release 摘要
近期无正式版本发布。

---

## 🐛 Issues

### #17352 — [[Feature] Ascend NPU follow-up: application-directed Mamba prefix checkpoints (port of vllm#55697 series)](https://github.com/vllm-project/vllm-ascend/issues/17352)
- **作者**: curnane-lab  **时间**: 2026-09-23 12:21 CST
- **摘要**: ## Background  vLLM main is landing application-directed prefix checkpoints for hybrid Mamba/GDN models via a three-PR series:  - RFC: vllm-project/vllm#55697 — mechanism + L40S benchmark (Qwen3.5 35B: 2.1x throughput, 100% logit parity, -48.6% TTFT, -53.4% prefill FLOPs) - vllm-project/vllm#55873 —…

### #17347 — [[Feature]: int8 KV cache support for GLM-5.3-Flash / DeepSeek-V4-style hybrid models — decode path fails at aclnnSparseFlashAttention (EZ1009) while prefill path already works](https://github.com/vllm-project/vllm-ascend/issues/17347)
- **作者**: qys4123  **时间**: 2026-09-23 11:55 CST
- **摘要**: ### 🚀 The feature, motivation and pitch  **The feature we need**: end-to-end support for `--kv-cache-dtype int8` on GLM-5.3-Flash (`Glm5NextForConditionalGeneration`, w8a8, hybrid architecture: MLA + linear-attention/Mamba + compressed-indexer sparse attention (DSA-style), with an MTP head for specu…

## 🔀 Pull Requests

### #17355 — [[Feature][P/D] MooncakeConnector V2 supports PCP](https://github.com/vllm-project/vllm-ascend/pull/17355)
- **作者**: nwpu-zxr  **时间**: 2026-09-23 13:02 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? MooncakeConnectorV2 supports PCP.  ### Does this PR introduce _any_ user-facing change? No.  ### How was this patch tested? By CI.  - vLLM main: https://github.com/vllm-project/vllm/commit/84030bbe3d74d99bad477a3d2e37a973ccd8865c

### #17354 — [Replace sas](https://github.com/vllm-project/vllm-ascend/pull/17354)
- **作者**: lcfenglinwan  **时间**: 2026-09-23 12:59 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/84030bbe3d74d99bad477a3d2e37a973ccd8865c

### #17353 — [[Performance][Worker] Avoid request-count specialization in fused slot mapping](https://github.com/vllm-project/vllm-ascend/pull/17353)
- **作者**: pisceskkk  **时间**: 2026-09-23 12:31 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Both fused slot-mapping kernels currently specialize on `NUM_REQS` to unpack a flattened grid. A new request count therefore creates another Triton compilation variant in decode and multi-request prefill, even when the tile configuration is unchanged.  Use a …

### #17351 — [fix(ci): resolve nightly base image branch from PR target instead of …](https://github.com/vllm-project/vllm-ascend/pull/17351)
- **作者**: JavaPythonAIForBAT  **时间**: 2026-09-23 12:12 CST
- **标签**: ci/build
- **摘要**: …head SHA  The PR-triggered nightly image build set vllm_ascend_branch to the PR head SHA, which was also used to construct the base image tag (e.g. nightly-<sha>-310p). That tag never exists, so the build failed with "not found". Separate the checkout ref from the base image branch by introducing a…

### #17350 — [[Misc]feat: adapt to vLLM main (903285fb)](https://github.com/vllm-project/vllm-ascend/pull/17350)
- **作者**: dev-submitter  **时间**: 2026-09-23 11:59 CST
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to September 16.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | — | [5392fbca](https://github.com/vllm-project/vllm/commit/5392fbca2a…

### #17349 — [[Test] Move MiniMax-M2.7 nightly case to weekly suite](https://github.com/vllm-project/vllm-ascend/pull/17349)
- **作者**: fanyihua0309  **时间**: 2026-09-23 11:56 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? This PR moves the MiniMax-M2.7 E2E test case from the nightly suite to the weekly suite.  - Remove `Minimax_m2.7_w8a8_A3` entry from nightly config and delete its nightly case file - Move the case to `tests/e2e/weekly` with updated envs and server args (enforc…

### #17348 — [[BugFix][Engram] Skip HOST_UVA page migration during CPU binding](https://github.com/vllm-project/vllm-ascend/pull/17348)
- **作者**: QwertyJack  **时间**: 2026-09-23 11:56 CST
- **标签**: module:tests, module:core, ready-precise
- **摘要**: ### What this PR does / why we need it?  Fixes #17286.  With Engram HOST_UVA offload, post-warmup CPU binding runs `migratepages` after the large host tables have already been registered and pinned. Migrating the whole worker can stall startup for many minutes, and DP replicas sharing a backing can …

### #17346 — [[Performance] Avoid request-count specialization in DSA-CP metadata kernel](https://github.com/vllm-project/vllm-ascend/pull/17346)
- **作者**: pisceskkk  **时间**: 2026-09-23 11:54 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  The DSA context-parallel metadata kernel currently uses the request-count block size as a tl.constexpr, producing a separate compiled kernel for each request-count tier.  This change:  - uses a fixed 1024-request tile and scales the launch grid for larger bat…

### #17345 — [[Doc][Feature] Add A2 W8A8 multi-node deployment scripts for MiniMax-M3](https://github.com/vllm-project/vllm-ascend/pull/17345)
- **作者**: taochong123456  **时间**: 2026-09-23 11:54 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  This PR extends the MiniMax-M3 deployment tutorial with an A2 series (W8A8) multi-node deployment example in Section 5.2. - Add the node 0 startup command for an 8-NPU A2 node. - Add the node 1 headless startup command and its data-parallel rank configuration…

### #17344 — [[v0.26.0rc][BugFix][MoE] Guard fused MC2 dispatch against silent capacity truncation](https://github.com/vllm-project/vllm-ascend/pull/17344)
- **作者**: Liuchenbing-2026  **时间**: 2026-09-23 11:53 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Fixes silent output corruption in the fused MC2 MoE path (`enable_fused_mc2=1`) for long-context prefill.  **Root cause.** The `dispatch_ffn_combine` op sizes its internal per-rank token buffers from `max_output_size` and **silently discards work** once the p…
