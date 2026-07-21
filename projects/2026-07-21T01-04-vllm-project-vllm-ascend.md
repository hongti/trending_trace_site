# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-21 09:04 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文简洁摘要：

### 📌 Issue 动态
- **CI 流程异常** (#12444)：`main2main` 自动化同步流程未完成全部计划步骤而中断，目前需要人工审核介入（关联 Draft PR #12443）。
- **文档 404 错误** (#12437)：用户反馈 v0.23.0rc1 版本的 Ascend 英文官方文档页面（`/en/latest/`）出现 404 Not Found，需排查修复。

### 🛠️ PR 动态
**1. 核心特性与适配**
- **适配 vLLM 主分支** (#12443)：升级 vLLM 底层依赖至 commit `62286308`，并对单类型 KV cache manager 及分布式 KV transfer 等模块进行适配改造。
- **MRV2 支持 MTP** (#12442)：为 MRV2 架构引入 Multi-Token Prediction (MTP) 支持，包含 eager 模式与 FullGraph 模式。
- **Minimax m3 模型同步** (#12438, #12445)：同步适配 vLLM v0.24.0 版本的 Minimax m3 模型相关变更。

**2. Bug 修复**
- **HunyuanVL 处理器补丁重构** (#12440)：重构 HunyuanVL processor 的兼容层，使其遵循现有补丁结构与版本策略，并将生效范围限定至 v0.25.1 版本。

**3. 文档更新**
- **Qwen 3.5 397B 部署指南** (#12435)：补充并更新了 Qwen 3.5 397B 模型在 Ascend 950DT 上的部署信息。
- **GLM5.2 文档适配** (#12441)：更新 GLM5.2 模型文档以适配最新 vllm-ascend 版本。
- **术语重命名 Cherry-pick** (#12439)：将文档中的 "Support Matrix" 重命名为 "Features and Models"，并合入 `releases/v0.24.0rc` 分支。

**4. 杂项 / 测试**
- **DCO 签名测试** (#12434, #12436)：两次提交用于测试 DCO (Developer Certificate of Origin) 检查流程。

### 🚀 Release 动态
- **本次动态中无新版本发布**。但根据 PR 活动可以看出，仓库正在积极为 **v0.24.0rc** 版本做准备（如 Minimax m3 模型同步、文档 Cherry-pick 合入等），下一版本发布后预计将包含上述模型支持与修复更新。

---

## 🐛 Issues

### #12444 — [[main2main] main2main manual review required (62286308)](https://github.com/vllm-project/vllm-ascend/issues/12444)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-21 02:10 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/12443 - Commit range: `54503ecec0f3ac31e5ecfc5f28652e4cc42307b5`...`62286308c9e30adfef3780c6fbefeca5cf8f36ae` - Status: `failed`  ## Final Summary  …

### #12437 — [[Doc]: Feedback for `/en/latest/`](https://github.com/vllm-project/vllm-ascend/issues/12437)
- **作者**: Castielzhe  **时间**: 2026-07-20 22:13 CST
- **标签**: documentation
- **摘要**: ### 📚 The doc issue  https://docs.vllm.ai/projects/ascend/en/v0.23.0rc1/   pages 404 Not Found， please check  ### Suggest a potential alternative/fix  _No response_

## 🔀 Pull Requests

### #12445 — [Minimax m3 sync release 0 24 0](https://github.com/vllm-project/vllm-ascend/pull/12445)
- **作者**: CXY-Katrina  **时间**: 2026-07-21 08:03 CST
- **标签**: documentation, module:tests, module:ops, module:quantization
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.24.0 - vLLM main: https://github.com/vllm-project/vllm/commit/85c09e9885e346ea1612da30ebff5a75f67d2350

### #12443 — [[Misc]feat: adapt to vLLM main (62286308)](https://github.com/vllm-project/vllm-ascend/pull/12443)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-21 02:10 CST
- **摘要**: ### What this PR does / why we need it?  Upgrade vLLM commit to `62286308`  1. Adapt `vllm_ascend/core/single_type_kv_cache_manager.py`, `vllm_ascend/distributed/kv_transfer/kv_pool/ascend_store/coordinator.py`, `vllm_ascend/distributed/kv_transfer/kv_pool/ascend_store/pool_scheduler.py`, `vllm_asce…

### #12442 — [[Feature][MRV2] Support MTP](https://github.com/vllm-project/vllm-ascend/pull/12442)
- **作者**: jiajinzhu2  **时间**: 2026-07-21 00:09 CST
- **摘要**: ### What this PR does / why we need it? Support eager and FullGraph for MTP in MRV2.  ### Does this PR introduce _any_ user-facing change? No  ### How was this patch tested?  set -x export HCCL_OP_EXPANSION_MODE="AIV" export PYTORCH_NPU_ALLOC_CONF="expandable_segments:True" export PYTHONPATH=/home/j…

### #12441 — [[Doc]Update GLM5.2.md](https://github.com/vllm-project/vllm-ascend/pull/12441)
- **作者**: aisong1988  **时间**: 2026-07-20 23:16 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? The GLM5.2 model documentation needs to be adapted to the latest vllm-ascend version.  ### Does this PR introduce _any_ user-facing change? No, this is a documentation-only update.  ### How was this patch tested?  Documentation changes only.  - vLLM version: v…

### #12440 — [[BugFix][Model] Scope Hunyuan processor patch to v0.25.1](https://github.com/vllm-project/vllm-ascend/pull/12440)
- **作者**: QwertyJack  **时间**: 2026-07-20 22:44 CST
- **标签**: module:tests, module:core, ready
- **摘要**: ### What this PR does / why we need it?  Refactor the HunyuanVL processor compatibility layer to follow the current vLLM Ascend patch structure and version policy.  - Move the compatibility module into `vllm_ascend/patch/platform/` and install it only for vLLM `v0.25.1`. - Keep the verified vLLM mai…

### #12439 — [[Cherry-pick][releases/v0.24.0rc][Doc][Misc] Rename Support Matrix to Features and Models (from #12419)](https://github.com/vllm-project/vllm-ascend/pull/12439)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-20 22:31 CST
- **标签**: documentation
- **摘要**: Cherry-pick of PR #12419 onto `releases/v0.24.0rc`.  Original PR: #12419 Original author: @herizhen  --- ### What this PR does / why we need it? Rename Support Matrix to Features and Models  ### Does this PR introduce _any_ user-facing change? No, this is a documentation-only update.  ### How was th…

### #12438 — [[Feature] Minimax m3 sync release 0 24 0 07201920](https://github.com/vllm-project/vllm-ascend/pull/12438)
- **作者**: Bill845514379  **时间**: 2026-07-20 22:19 CST
- **标签**: documentation, module:tests, module:ops, module:quantization
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.24.0 - vLLM main: https://github.com/vllm-project/vllm/commit/85c09e9885e346ea1612da30ebff5a75f67d2350

### #12436 — [test_DCO](https://github.com/vllm-project/vllm-ascend/pull/12436)
- **作者**: czydyy  **时间**: 2026-07-20 21:49 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.24.0 - vLLM main: https://github.com/vllm-project/vllm/commit/85c09e9885e346ea1612da30ebff5a75f67d2350

### #12435 — [【Doc】update qwen 3.5 397b doc, fill ascend 950DT info](https://github.com/vllm-project/vllm-ascend/pull/12435)
- **作者**: Karryking3  **时间**: 2026-07-20 21:29 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? This PR primarily updates the deployment guide for qwen 3.5 397b on the Ascend 950DT.  ### Does this PR introduce _any_ user-facing change? No  ### How was this patch tested? Documentation-only change, no testing required.  vLLM version: v0.25.0 vLLM main: htt…

### #12434 — [test_Dco](https://github.com/vllm-project/vllm-ascend/pull/12434)
- **作者**: czydyy  **时间**: 2026-07-20 21:13 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.24.0 - vLLM main: https://github.com/vllm-project/vllm/commit/85c09e9885e346ea1612da30ebff5a75f67d2350
