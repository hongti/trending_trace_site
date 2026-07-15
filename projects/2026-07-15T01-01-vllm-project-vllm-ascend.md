# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-15 09:01 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🚀 Pull Requests (PR)

本次 PR 动态以**缺陷修复**和**架构重构**为主，重点解决了 v0.23.0 版本引入的多个关键 Bug，并继续推进算子向 Triton 的统一迁移。

**🛠️ 重要缺陷修复**
*   **GDN Decode 输出乱码修复 (#12040)**：解决了当 PCP 结合 `FULL_DECODE_ONLY` 图执行模式，且活跃请求数小于捕获的图批次大小时，Qwen3.5/Qwen3.6 模型 GDN decode 输出损坏的问题。
*   **GQA 内存连续性错误修复 (#12032)**：针对 v0.23.0，修复了 `fused_infer_attention` 算子在 GQA（Grouped Query Attention）场景下的 contiguous 错误。
*   **DCP/DP 服务卡死修复 (#12036, #12034)**：修复了在处理 DCP 覆盖 DP 问题时，因 curl 请求导致服务卡死的情况（特别是在 mtp=3 且仅有单个请求的 prefill 阶段）。
*   **Mamba EAGLE Lookup 逻辑修复 (#12038)**：防止 v0.23.0 混合 KV-cache 协调器为 EAGLE/MTP 组额外增加的缓存块导致 lookup 上限误扩展。

**🔄 重构与算子优化**
*   **`fused_gdn_gating` 迁移至 Triton (#12035)**：废弃了先前由 PR #9601 引入的自定义 AscendC 算子，将其路由统一至现有的 Triton 实现，提升维护性。
*   **移除 `dispatch_gmm_combine_decode` 算子 (#12031)**：清理并移除了该融合算子及其相关的代码、测试和文档，精简项目结构。

**🧪 适配、测试与 CI (Misc / CI / Test)**
*   **适配 vLLM 主分支 (#12039)**：同步 vLLM 上游最新代码 (54503ece)，主要涉及 Mooncake Connector 及 QwenGatedDelta 的适配改动。
*   **新增 Gemma4 E2E 测试 (#12037)**：为 **Gemma4-26B-A4B-it** 模型（bf16, 非量化 MoE）添加了单节点夜间端到端基准测试配置。
*   **CI Pre-commit 优化 (#12033)**：调整 CI 流程，连续运行两次 pre-commit 以确保 ruff-format 自动修复后代码格式完全合规。

---

### 📋 Issues
*   本次提供的数据中无近期 Issue 动态。

---

### 🎉 Release
*   本次提供的数据中无近期 Release 发布。（注：当前 PR 动态频繁提及 v0.23.0，预计团队正集中修复该版本遗留问题并为下个版本做准备）。

---

## 🔀 Pull Requests

### #12040 — [[BugFix] Fix GDN graph padding state indices with PCP](https://github.com/vllm-project/vllm-ascend/pull/12040)
- **作者**: maoxx241  **时间**: 2026-07-15 04:52 CST
- **标签**: module:tests, module:ops
- **摘要**: ## What this PR does  Fixes corrupted Qwen3.5/Qwen3.6 GDN decode output when PCP is combined with `FULL_DECODE_ONLY` graph execution and the active request count is smaller than the captured graph batch size.  The GDN metadata builder now:  - keeps non-spec decode metadata at the padded graph reques…

### #12039 — [[Misc]feat: adapt to vLLM main (54503ece)](https://github.com/vllm-project/vllm-ascend/pull/12039)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-15 03:53 CST
- **摘要**: ### What this PR does / why we need it?  vllm upstream `e5588e49...54503ece` (4/4 steps).  #### vllm_ascend/distributed/kv_transfer/kv_p2p/mooncake_connector.py  - Cause: Upstream changed QwenGatedDeltaNetAttention.forward signature from (hidden_states, output) to (hidden_states) -> Tensor. The vllm…

### #12038 — [[BugFix] Prevent Mamba EAGLE lookup from expanding hybrid prefix hits](https://github.com/vllm-project/vllm-ascend/pull/12038)
- **作者**: underfituu  **时间**: 2026-07-15 02:46 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  The v0.23.0 hybrid KV-cache coordinator adds one cache block to the lookup upper bound for EAGLE/MTP groups. This margin is intended for managers that match one lookahead block and then consume `drop_eagle_block=True` by dropping that block.  Mamba cache find…

### #12037 — [test: add nightly e2e config for Gemma4-26B-A4B-it](https://github.com/vllm-project/vllm-ascend/pull/12037)
- **作者**: 0moyi0-2024  **时间**: 2026-07-14 23:42 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Add a nightly single-node e2e benchmark config for the **Gemma4-26B-A4B-it** model (bf16, non-quantized MoE), under `tests/e2e/nightly/single_node/models/configs/gemma4-26B.yaml`, modeled on the existing `GLM-4.7.yaml`.  This enables nightly accuracy (gsm8k-l…

### #12036 — [[BugFix]Fix the issue of DCP and DP services getting stuck](https://github.com/vllm-project/vllm-ascend/pull/12036)
- **作者**: weiguihua2  **时间**: 2026-07-14 23:07 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it? When fixing the issue of dcp overlaying dp, a curl request causes the service to get stuck.  When mtp=3 and there is only one request, for prefill, during the execution of mtp by dp0, only step0 is executed. The other two steps are copies of step0. dp1 perform…

### #12035 — [[Refactor] replace fused_gdn_gating from ascendc to triton](https://github.com/vllm-project/vllm-ascend/pull/12035)
- **作者**: ningjingbengxiaohai  **时间**: 2026-07-14 22:46 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Revert the AscendC fused_gdn_gating custom operator introduced by PR #9601 and route the common fused_gdn_gating path to the existing Triton implementation instead.  This removes the AscendC custom op sources, build registration, torch/meta bindings, and the A…

### #12034 — [[BugFix]Fix the issue of DCP and DP services getting stuck](https://github.com/vllm-project/vllm-ascend/pull/12034)
- **作者**: weiguihua2  **时间**: 2026-07-14 22:32 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it? When fixing the issue of dcp overlaying dp, a curl request causes the service to get stuck.  When mtp=3 and there is only one request, for prefill, during the execution of mtp by dp0, only step0 is executed. The other two steps are copies of step0. dp1 perform…

### #12033 — [[CI] Run pre-commit twice to handle ruff-format auto-fix](https://github.com/vllm-project/vllm-ascend/pull/12033)
- **作者**: wjunLu  **时间**: 2026-07-14 22:18 CST
- **标签**: ci/build, merge-conflicts
- **摘要**: First pass auto-fixes files (ruff format), second pass verifies everything is clean. Matches the standard pre-commit-in-CI pattern. - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/e5588e49bc2642670116664a7fc4096e27adb179

### #12032 — [[v0.23.0][BugFix]Fix fused_infer_attention ops contiguous err in GQA](https://github.com/vllm-project/vllm-ascend/pull/12032)
- **作者**: zzzzzz198  **时间**: 2026-07-14 21:57 CST
- **标签**: ready
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389

### #12031 — [[Refactor] Remove dispatch_gmm_combine_decode ops](https://github.com/vllm-project/vllm-ascend/pull/12031)
- **作者**: ningjingbengxiaohai  **时间**: 2026-07-14 21:53 CST
- **标签**: documentation, module:tests, module:ops, module:core, ready
- **摘要**: ### What this PR does / why we need it? Remove the dispatch_gmm_combine_decode fused operator and its related code, tests, and documentation.  This drops the custom op registration/build entries, Python call paths, feature gating logic, dedicated e2e test, and user docs that described VLLM_ASCEND_EN…
