# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-23 09:07 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库最近动态的简洁摘要：

### 📌 Issue 概览
1. **CI/自动化异常** (#12664)：`main2main` 自动化流程未能完成所有计划步骤，需人工审查介入。
2. **模型推理 Bug** (#12658)：在 Atlas 800I A2 环境及 v0.21.0rc1 版本下，DeepSeek-V4-Flash-w8a8-mtp 模型在 Non-think 模式中出现 tool call 为空的异常。
3. **部署与使用咨询** (#12657)：用户询问 GLM5.2 (v0.23.0) 1M PD 分离部署配置是否可参考 128K 的分布式部署文档。

### 🚀 PR 概览
**1. 核心特性与适配**
* **适配 vLLM 主分支** (#12656)：将 vllm-ascend 适配至 vLLM main 分支（截至7月22日的提交），保持与上游同步。
* **新增 CANN MegaMoe 支持** (#12659)：引入 CANN 原生融合 MoE 通信内核，增加 MegaMoe fused MC2 通信支持，提升 MoE 模型通信效率。
* **支持 dspark 推测解码** (#12662)：新增对 Qwen3.5 dspark 推测解码方法在 Ascend NPU 上的支持，并简化了 reorder batch 阈值计算逻辑。

**2. 性能优化（EPLB 相关）**
* **向量化 EPLB 映射生成** (#12663)：用张量化副本选择替换 `generate_log2phy_map` 中的嵌套 Python 循环和逐元素 `.item()` 调用，显著提升性能。
* **优化 EPLB 专家更新规划** (#12661)：预计算专家的首个发送 rank 和持有者，替换循环中的标量提取与微小张量扫描，优化调度效率。

**3. Bug 修复与兼容性**
* **修复量化模块适配** (#12665)：将 vLLM v0.25 的 RoutedExperts 模块映射到 Linear compressed-tensors 方案，处理未匹配目标避免 KeyError，并为 Kimi-K2-Thinking MoE 路径增加回归测试。
* **修复 Kimi-K2 Thinking 问题** (#12655)：修复 AOP bisect 过程中与 Kimi-K2 Thinking 相关的问题。

**4. 文档更新**
* **更新 GLM-5.2 Cookbook** (#12660)：修正 A2 环境下的镜像配置（解决默认 0.22.1rc1 镜像导致的 KeyError 问题），并新增 A2 单节点部署指南。

**5. CI 与维护**
* **CI 自动化** (#12653)：自动更新 `test_config.yaml` 中的预估测试时间。
* **配置同步** (#12652)：添加分支标记并同步 `nightly_config.yaml`。

### 🎉 Release 概览
本次提供的动态中**无新版本 Release 信息**。

---

## 🐛 Issues

### #12664 — [[main2main] main2main manual review required (27c3e579)](https://github.com/vllm-project/vllm-ascend/issues/12664)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-23 02:09 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR:  - Commit range: `54503ecec0f3ac31e5ecfc5f28652e4cc42307b5`...`27c3e579f0e5f345a86e512e26e1231d3689931f` - Status: `failed`  ## Final Summary  main2main adaptation failed — no steps completed.

### #12658 — [[Bug]: [v0.21.0rc1]DeepSeek-V4-Flash-w8a8-mtp, Non-think, tool call empty](https://github.com/vllm-project/vllm-ascend/issues/12658)
- **作者**: yxl6202  **时间**: 2026-07-22 21:58 CST
- **标签**: bug, advanced-features, mtp/speculative-decode, llm-model, deepseek
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Your output of above commands here ```  </details> Atlas 800I A2  vllm ascend v0.21.0rc1  ### 🐛 Describe the bug  Bad case:  [bad_case.json](https://github.com/user-attachments/files/30271332/ba…

### #12657 — [[Usage]: GLM5.2 0.23.0文档中1M PD分离的配置是否可以参考128K的分布式PD分离部署](https://github.com/vllm-project/vllm-ascend/issues/12657)
- **作者**: Sfeching  **时间**: 2026-07-22 21:52 CST
- **标签**: glm5, llm-model
- **摘要**: ### Your current environment  ```text The output of above commands ```   ### How would you like to use vllm on ascend  I want to run inference of a [specific model](put link here). I don't know how to integrate it with vllm.

## 🔀 Pull Requests

### #12665 — [fix(quantization): match routed experts compressed tensors scheme](https://github.com/vllm-project/vllm-ascend/pull/12665)
- **作者**: czydyy  **时间**: 2026-07-23 08:41 CST
- **标签**: module:tests, module:quantization
- **摘要**: Map the vLLM v0.25 RoutedExperts module to Linear compressed-tensors schemes and handle unmatched targets without raising KeyError. Add regression coverage for the Kimi-K2-Thinking MoE selection path.  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ###…

### #12663 — [[Performance] Vectorize EPLB log2phy map generation](https://github.com/vllm-project/vllm-ascend/pull/12663)
- **作者**: ShuhaoZhangTony  **时间**: 2026-07-23 01:51 CST
- **标签**: module:tests
- **摘要**: ## Summary  - replace nested Python loops and per-element `.item()` calls in `generate_log2phy_map` with tensorized replica selection - preserve rank-order replica selection for both the fallback and TP-aware policies - retain support for both a list of per-rank tensors and an already-stacked tensor…

### #12662 — [[Ops][Misc] Support dspark speculative method and simplify reorder batch threshold calculation](https://github.com/vllm-project/vllm-ascend/pull/12662)
- **作者**: PHOEBEMOON0802  **时间**: 2026-07-23 01:45 CST
- **标签**: module:ops
- **摘要**: What this PR does / why we need it?    This PR adds support for the Qwen3.5 dspark speculative decoding method on Ascend NPUs. Two changes are required:    1. vllm_ascend/ops/gdn_attn_builder.py — AscendGDNAttentionMetadataBuilder previously only recognized speculative_config.method == "dflash" when…

### #12661 — [[Performance] Optimize EPLB expert update planning](https://github.com/vllm-project/vllm-ascend/pull/12661)
- **作者**: ShuhaoZhangTony  **时间**: 2026-07-23 01:41 CST
- **标签**: module:tests
- **摘要**: ## Summary  - precompute the first sending rank and first current holder for each expert - replace repeated scalar extraction and tiny `torch.isin` / `torch.where` scans in the receive-planning loop with batched tensor-to-list conversion and dictionary lookups - preserve the existing first-source se…

### #12660 — [[Doc]update glm-5.2-w4a8c8 cookbook with a2  correct image & add a2 Single-node deployment](https://github.com/vllm-project/vllm-ascend/pull/12660)
- **作者**: yiminghub2024  **时间**: 2026-07-23 00:41 CST
- **标签**: documentation
- **摘要**: 1.update glm-5.2-w4a8c8 cookbook with a2  correct image if use default 0.22.1rc1 ,it will report error with a2 "KeyError: 'model.layers.3.self_attn.indexer.wq_b.weight'" i tested a2 with image  quay.io/ascend/vllm-ascend:nightly-main start successful 2.add a2 Single-node deployment with glm-5.2-w4a8…

### #12659 — [[Feature] Add CANN MegaMoe fused MC2 support](https://github.com/vllm-project/vllm-ascend/pull/12659)
- **作者**: Liuchenbing-2026  **时间**: 2026-07-22 23:29 CST
- **标签**: module:tests, module:ops, module:core, module:quantization, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  Add the CANN MegaMoe fused MC2 communication support for vllm-ascend on top of the current `main`. MegaMoe is the CANN-native fused MoE communication kernel (dispatch + combine collapsed into a single kernel invocation) that shortens MoE all-to-all critical p…

### #12656 — [[Misc]feat: adapt to vLLM main (27c3e579)](https://github.com/vllm-project/vllm-ascend/pull/12656)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-22 21:32 CST
- **标签**: ready, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to July 22.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | `vllm_ascend/core/single_type_kv_cache_manager.py`<br>`vllm_ascend/distrib…

### #12655 — [Fix/aop bisect kimi k2 thinking](https://github.com/vllm-project/vllm-ascend/pull/12655)
- **作者**: czydyy  **时间**: 2026-07-22 20:24 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.25.1 - vLLM main: https://github.com/vllm-project/vllm/commit/54503ecec0f3ac31e5ecfc5f28652e4cc42307b5

### #12653 — [[CI] Auto-update estimated test times in test_config.yaml](https://github.com/vllm-project/vllm-ascend/pull/12653)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-22 19:57 CST
- **摘要**: ## Summary  This PR was auto-generated by the **Update estimated test times** [workflow](https://github.com/vllm-project/vllm-ascend/actions/runs/29911396306).  It updates the `estimated_times` values in `.github/workflows/scripts/test_config.yaml` based on actual elapsed times collected from CI wor…

### #12652 — [chore: add branch marker and sync nightly_config.yaml](https://github.com/vllm-project/vllm-ascend/pull/12652)
- **作者**: czydyy  **时间**: 2026-07-22 19:49 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.25.1 - vLLM main: https://github.com/vllm-project/vllm/commit/54503ecec0f3ac31e5ecfc5f28652e4cc42307b5
