# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-13 09:07 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文简洁摘要：

### 📌 Issue 动态
近期共暴露 3 个核心 Bug 和 1 个 CI 流程问题：
1. **PD/Mooncake 非对称 TP 映射 Bug (#11885)**：在 32 卡 NPU 多节点部署中，`FullAttentionSpec` 在张量并行分片后使用了 rank-local 的 KV heads，而 Mooncake 将其误当作全局 head count，导致异构 P/D 映射出错。
2. **PD Decode Worker 崩溃 Bug (#11882)**：在 PD 分离部署中，当 MLAPO 释放了 `kv_consumer` 的 prefill 权重后，参数错误的请求会导致整个 Decode Worker 进程因 `UndefinedTensorImpl` 崩溃。
3. **模型服务稳定性 Bug (#11880)**：`glm5.1-w4a8` 模型在 v0.22.1rc1 版本下，多请求并发时会出现稳定崩溃现象。
4. **CI 流程中断 (#11889)**：main2main 自动化流程未完成所有计划步骤，需人工介入审查。

### 🛠️ Pull Request 动态
近期 PR 主要集中在**关键 Bug 修复**、**新特性支持**与**上游适配**：

**1. 重要 Bug 修复**
- **修复 Mooncake KV Heads 映射 (#11886, #11887)**：主分支及 v0.23.0 回溯修复，恢复 Mooncake 传输组的全局 KV heads，解决上述 Issue #11885。
- **修复 MLAPO Decode Worker 崩溃 (#11883)**：针对 Issue #11882，将 `kv_consumer` 的 prefill 权重卸载至 CPU，并在重计算时按需懒加载恢复，避免参数错误请求导致进程崩溃。
- **修复 eplb 功能异常 (#11879)**：修复 main2main 同步后引发的 eplb 功能问题。

**2. 新特性与功能支持**
- **SFA DCP 支持 PD 分离与 KV Pool (#11876)**：为 SFA DCP 的 replicate-indexer 路径增加 PD 分离部署支持及 KV Pool 功能（回溯至 v0.23.0）。
- **新增 Phi-4-mini-instruct 模型支持 (#11881)**：添加该模型在 Ascend NPU 上的适配文件及 E2E 测试配置。

**3. CI 适配与上游同步**
- **适配 vLLM Main (#11888)**：推进 vllm 上游同步（24/37步），涉及 `deepseekv4_tool_par` 等上游删除代码的适配。
- **CI 升级至 v0.24 (#11875)**：将 release lane 升级至 vLLM v0.24.0，保持 main lane 和 Transformers 版本不变。

**4. 文档与测试调整**
- **更新 Qwen3-VL 文档 (#11884)**：重构 Qwen3-VL-235B 及 30B 的部署文档，使其符合最新教程结构。
- **移除未适配的夜间测试配置 (#11874)**：在 C8 SFA 适配完成前，暂时移除 `DeepSeek-V3.2-W8A8-DCP-replicated-indexer` 的夜间测试。

### 🚀 Release 动态
*近期无新版发布记录。*

---

## 🐛 Issues

### #11889 — [main2main manual review required (a5d19cbb)](https://github.com/vllm-project/vllm-ascend/issues/11889)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-13 07:13 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/11888 - Commit range: `1f486d96a17303ce8db8e02be39545b2be338446`...`a5d19cbb95872c4b426c06735733568542fa33db` - Status: `failed`  ## Final Summary  …

### #11885 — [[Bug][Mooncake][PD] FullAttentionSpec uses rank-local KV heads for asymmetric TP mapping](https://github.com/vllm-project/vllm-ascend/issues/11885)
- **作者**: QwertyJack  **时间**: 2026-07-13 02:13 CST
- **摘要**: ### Your current environment  The issue was reproduced on a four-node Ascend deployment:  - Hardware: 32 Ascend NPUs - Docker image: `dev-26.1.0.cann9.1.0.day20260709` - vLLM: `0.23.0` - vLLM Ascend: `0.19.1rc2.dev872+g092841466` - torch-npu: `2.10.0.post3.dev20260708` - CANN: `9.1.0 B223` - KV conn…

### #11882 — [[P/D] Mis-parametrized request crashes decode worker (UndefinedTensorImpl) when MLAPO freed prefill weights on kv_consumer](https://github.com/vllm-project/vllm-ascend/issues/11882)
- **作者**: xlshaoscu  **时间**: 2026-07-12 21:27 CST
- **摘要**: ## Summary  In PD disaggregation, a single request with incorrect `kv_transfer_params` crashes the **entire decode (D) worker process** with `NotImplementedError: Cannot access storage of UndefinedTensorImpl`, taking down all in-flight requests on that D instance. The engine should fail the single b…

### #11880 — [[Bug]: glm5.1-w4a8在v0.22.1rc1版本运行一段时间会崩溃](https://github.com/vllm-project/vllm-ascend/issues/11880)
- **作者**: nutriver  **时间**: 2026-07-12 20:09 CST
- **标签**: bug, glm5, llm-model, wait-feedback
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text  ```  </details>   ### 🐛 Describe the bug  <details> <summary>启动脚本</summary>  ```text docker run \ --name glm-5.1 \ --net=host \ --privileged \ --shm-size=100g \ --device /dev/davinci0 \ --devic…

## 🔀 Pull Requests

### #11888 — [[Misc]feat: adapt to vLLM main (a5d19cbb)](https://github.com/vllm-project/vllm-ascend/pull/11888)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-13 07:12 CST
- **标签**: module:ops
- **摘要**: ### What this PR does / why we need it?  vllm upstream `1f486d96...a5d19cbb` (24/37 steps).  #### vllm_ascend/ops/fused_moe/gate_linear.py  - Cause: Upstream deleted vllm/tool_parsers/deepseekv4_tool_parser.py (DeepSeekV4ToolParser) and deepseekv32_tool_parser.py, replacing them with engine-based pa…

### #11887 — [[v0.23.0][BugFix][PD] Restore global KV heads for Mooncake transfer groups](https://github.com/vllm-project/vllm-ascend/pull/11887)
- **作者**: QwertyJack  **时间**: 2026-07-13 02:18 CST
- **标签**: module:tests
- **摘要**: Backport of #11886 to `releases/v0.23.0`.  ### What this PR does / why we need it?  `FullAttentionSpec.num_kv_heads` is rank-local after tensor-parallel sharding, but Mooncake used it as a model-global head count when deriving heterogeneous P/D rank pulls. With Prefill TP8 and Decode TP2, the same g…

### #11886 — [[BugFix][PD] Restore global KV heads for Mooncake transfer groups](https://github.com/vllm-project/vllm-ascend/pull/11886)
- **作者**: QwertyJack  **时间**: 2026-07-13 02:18 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  `FullAttentionSpec.num_kv_heads` is rank-local after tensor-parallel sharding, but Mooncake used it as a model-global head count when deriving heterogeneous P/D rank pulls. With Prefill TP8 and Decode TP2, the same global eight-head draft KV group was seriali…

### #11884 — [[Doc][Misc] Update Qwen3-VL documents](https://github.com/vllm-project/vllm-ascend/pull/11884)
- **作者**: LoganJane  **时间**: 2026-07-13 02:02 CST
- **标签**: documentation
- **摘要**: ## Summary  - Rework `Qwen3-VL-235B-A22B-Instruct.md` to match the current deployment tutorial structure. - Rework `Qwen3-VL-30B-A3B-Instruct.md` with complete sections for prerequisites, installation, online deployment, functional verification, evaluation, tuning, and FAQ. - Add Qwen3-VL-specific g…

### #11883 — [[MLAPO] Offload kv_consumer prefill weights to CPU, restore lazily on recompute (fixes #11882)](https://github.com/vllm-project/vllm-ascend/pull/11883)
- **作者**: xlshaoscu  **时间**: 2026-07-12 22:24 CST
- **摘要**: ## What & Why  Fixes #11882.  On `kv_consumer` (decode-only D) with MLAPO enabled and `max_num_batched_tokens <= MLAPO_MAX_SUPPORTED_TOKENS` (1024), `_process_weights_for_fused_mlapo` freed the original `fused_qkv_a_proj`/`q_proj` weights (set to `None`) to save NPU memory.  This breaks **local pref…

### #11881 — [[Doc][Feature] Add Phi-4-mini-instruct support for Ascend NPU](https://github.com/vllm-project/vllm-ascend/pull/11881)
- **作者**: Love-learning-Li  **时间**: 2026-07-12 21:09 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  This PR adds model adaptation artifacts and documentation for `microsoft/Phi-4-mini-instruct` on Ascend NPU. Specifically, it: - Adds the E2E test configuration (`Phi-4-mini-instruct.yaml`) with GSM8K accuracy metrics. - Adds a comprehensive tutorial document…

### #11879 — [[Bugfix] fix eplb after main2main](https://github.com/vllm-project/vllm-ascend/pull/11879)
- **作者**: Spicy-Stick  **时间**: 2026-07-12 18:05 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it? Fix functional issues with eplb after main2main.  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389

### #11876 — [[v0.23.0][Feature][P/D][KVPool] Support PD disaggregation and KV_Pool for DCP with replicate-indexer for SFA](https://github.com/vllm-project/vllm-ascend/pull/11876)
- **作者**: pisceskkk  **时间**: 2026-07-12 17:02 CST
- **标签**: module:tests, ready
- **摘要**: Cherry-pick of https://github.com/vllm-project/vllm-ascend/pull/11696 onto `releases/v0.23.0`.  What changed: - Adds P/D disaggregation support for the SFA DCP replicated-indexer path. - Adds KV Pool replicate-K cache-role handling for SFA DCP, including token database metadata, pool worker transfer…

### #11875 — [[CI] main2mainv0.24](https://github.com/vllm-project/vllm-ascend/pull/11875)
- **作者**: zhao-stack  **时间**: 2026-07-12 16:56 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization, ready
- **摘要**: ## Summary  Upgrade the release lane to vLLM `v0.24.0` while keeping the main lane and Transformers version unchanged.  - PR base: `vllm-project/vllm-ascend:main` - PR head: `zhao-stack:m2m-024` (`db5a50f46a9126280250fac31c3803d8e8d50f99`) - The branch was created in an independent worktree from the…

### #11874 — [[Test] remove DeepSeek-V3.2-W8A8-DCP-replicated-indexer from nightly config](https://github.com/vllm-project/vllm-ascend/pull/11874)
- **作者**: pisceskkk  **时间**: 2026-07-12 15:29 CST
- **摘要**: ### What this PR does / why we need it? remove DeepSeek-V3.2-W8A8-DCP-replicated-indexer from nightly config before C8 sfa adapted.  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446
