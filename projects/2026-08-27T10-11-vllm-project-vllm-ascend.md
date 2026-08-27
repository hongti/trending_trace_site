# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-27 18:11 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库最近动态的中文摘要：

### 📌 Issue 动态

*   **架构演进提议 (RFC #15119)**：提议支持上游 vLLM 的 batch-sharded sampling 机制，并废弃传统的 reduce-sampling 路径，以保持与上游架构对齐。
*   **性能优化诉求 (Feature #15110)**：提议将 LoRA 的 `shrink` 和 `expand` 操作融合为单一的 `sgmv_lora` / `bgmv_lora` kernel，以避免 decode 阶段中间数据在全局内存（GM）中的往返，提升小秩（8/16/32/64）场景性能。
*   **文档缺陷 (#15116)**：指出 v0.23.0 官方文档中 GLM5.2 在开启 PP（Pipeline Parallelism）特性下的 PD 分离配置存在错误，会导致请求失败并返回 Internal Server Error。

---

### 🔀 Pull Request 动态

**🚀 核心新特性与性能优化**
*   **LoRA 算子融合 (PR #15111)**：响应 Issue #15110，正式实现 LoRA shrink+expand 算子融合，显著减少 decode 阶段的全局内存读写开销。
*   **通信算法可配置 (PR #15113)**：新增 `additional_config.mc2_comm_alg` 参数，支持 `fullmesh` / `hierarchy` / `fullmesh_v2` 等多种通信算法，其中 `fullmesh_v2` 在特定场景下可带来性能提升。

**🐛 重要 Bug 修复**
*   **负载均衡代理错误静默 (PR #15108)**：修复了 load-balance proxy 吞噬 decode 后端错误（如 4xx/5xx/OOM）并返回空 200 状态码的问题，现在会正确将错误转发给客户端。
*   **P/D 分离调度修复 (PR #15109)**：在 `recompute_scheduler` 上为 fully spec tokens 添加 padding，修复了 Prefill/Decode 分离场景下的调度问题。
*   **推测解码冗余更新 (PR #15105)**：移除了 `AscendAutoRegressiveSpeculator` 中废弃的 `_ascend_update_seq_lens` 方法调用，清理了序列长度更新的冗余逻辑。

**🛠️ 重构与工程化**
*   **Attention 后端重构 (PR #15114)**：去重了 5 个 AscendDSA paging 变体后端（C4/C128/SWA 等）中的样板代码（如 `get_name()` 和 `get_supported_kernel_block_sizes`），提升代码可维护性。
*   **Docker 增强 (PR #15115)**：在 Docker 镜像中引入了 Rust tool parser。
*   **Watchdog 机制 (PR #15117)**：针对 v0.23.0 引入 Watchdog 机制。

**🧪 CI 与测试**
*   **重新启用长期测试 (PR #15118)**：修复了此前因 Ascend 向量核超时而跳过的 `test_deepseek_v4_mtp_eager` 测试，并通过配置 `ASCEND_LAUNCH_BLOC` 重新启用。
*   **补充 MC2 测试 (PR #15112)**：增加了 `mc2=0` 场景的 CI 测试覆盖。

---

### 🚀 Release 动态

*   本期暂无新的 Release 版本发布。

---

## 🐛 Issues

### #15119 — [[RFC]: Support upstream batch-shared and remove reduce sample.](https://github.com/vllm-project/vllm-ascend/issues/15119)
- **作者**: zouzy5137  **时间**: 2026-08-27 18:10 CST
- **标签**: RFC
- **摘要**: ### Motivation.  Apply upstream vLLM's batch-sharded sampling ([vllm-project/vllm#50465](https://github.com/vllm-project/vllm/pull/50465)) to vllm-ascend, and retire the legacy reduce-sampling path.  Upstream `#50465` changes the sampling dataflow under tensor parallelism:  - Before: every TP rank m…

### #15116 — [[Doc]: 0.23 官方文档 GLM5.2 5.1.1.3章节  在开启PP特性下 PD分离配置存在问题](https://github.com/vllm-project/vllm-ascend/issues/15116)
- **作者**: Patrickpan9  **时间**: 2026-08-27 17:44 CST
- **标签**: documentation, glm5, llm-model
- **摘要**: ### 📚 The doc issue  https://docs.vllm.ai/projects/ascend/zh-cn/v0.23.0/tutorials/models/GLM5.2.html#prefill-decode-disaggregation 参考章节5.1.1.3  PD分离配置部署，会出现请求失败的问题 curl proxy 会回显Internal Server Error ### Suggest a potential alternative/fix  在P节点开启PP并行策略下，P1节点作为从节点未加入--headless配置，此外load_balance_proxy…

### #15110 — [[Feature] Fuse LoRA shrink+expand into a single sgmv_lora/bgmv_lora kernel](https://github.com/vllm-project/vllm-ascend/issues/15110)
- **作者**: Agoni-02  **时间**: 2026-08-27 16:59 CST
- **摘要**: ## Feature request  Fuse per-slice LoRA `sgmv_shrink` + `sgmv_expand` into one vector kernel `y += (x @ A) @ B` so decode does not round-trip the rank intermediate through GM.  - Rank 8/16/32/64; otherwise keep shrink/expand. - Fully-sharded LoRA: all-gather A at `set_lora`. - Measured on Qwen3.5-27…

## 🔀 Pull Requests

### #15118 — [[CI][MRV2] Re-enable DeepSeek V4 MTP eager test](https://github.com/vllm-project/vllm-ascend/pull/15118)
- **作者**: jiajinzhu2  **时间**: 2026-08-27 17:53 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Re-enables `test_deepseek_v4_mtp_eager`, which was temporarily skipped by #15007 after an Ascend vector core timeout.  The test now runs with `ASCEND_LAUNCH_BLOCKING=1` in its existing test-scoped environment patch. This keeps blocking launch behavior limited…

### #15117 — [Dev2608/watch dog/0.23.0](https://github.com/vllm-project/vllm-ascend/pull/15117)
- **作者**: wenjinhust  **时间**: 2026-08-27 17:52 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389

### #15115 — [Rust tool parser in docker](https://github.com/vllm-project/vllm-ascend/pull/15115)
- **作者**: FORFuture37  **时间**: 2026-08-27 17:38 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/ba07e4a48fc951300d97eb506217dd530583dea3

### #15114 — [[Refactor][Attention] Deduplicate DSA paging backend boilerplate](https://github.com/vllm-project/vllm-ascend/pull/15114)
- **作者**: Woolgathererer  **时间**: 2026-08-27 17:24 CST
- **摘要**: ### What this PR does / why we need it?  The five AscendDSA paging-variant backends (C4/C128/SWA/C4State/C128State) each override `get_name()` and `get_supported_kernel_block_sizes()` with staticmethods that only return literals. Replace this boilerplate with two class attributes (`_backend_name` an…

### #15113 — [[feature] add param additional_config.mc2_comm_alg](https://github.com/vllm-project/vllm-ascend/pull/15113)
- **作者**: zzzzwwjj  **时间**: 2026-08-27 17:23 CST
- **标签**: documentation, module:tests, module:ops, module:core, ready-precise, ready
- **摘要**: ### What this PR does / why we need it?  add param additional_config.mc2_comm_alg, support `""/"fullmesh"/"hierarchy"/"fullmesh_v2"` four types, and "fullmesh_v2" may get better performance in some cases.  ### Does this PR introduce _any_ user-facing change?  + Add new param: additional_config.mc2_c…

### #15112 — [[test][ci]mc2=0](https://github.com/vllm-project/vllm-ascend/pull/15112)
- **作者**: U1stRsouland  **时间**: 2026-08-27 17:18 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/ba07e4a48fc951300d97eb506217dd530583dea3

### #15111 — [[Performance] Fuse shrink and expand into sgmv_lora/bgmv_lora](https://github.com/vllm-project/vllm-ascend/pull/15111)
- **作者**: Agoni-02  **时间**: 2026-08-27 16:59 CST
- **摘要**: ### What this PR does / why we need it?  Linear LoRA currently launches `sgmv_shrink` then `sgmv_expand` per slice, writing the rank buffer to GM. This PR adds one vector kernel `sgmv_lora` / `bgmv_lora` that computes `y += scale * (x @ A) @ B` with the rank intermediate kept in UB.  - Supported ran…

### #15109 — [[v0.26.0][BugFix][P/D] Add fully spec tokens padding on recompute_scheduler](https://github.com/vllm-project/vllm-ascend/pull/15109)
- **作者**: nwpu-zxr  **时间**: 2026-08-27 16:55 CST
- **标签**: ready
- **摘要**: ### What this PR does / why we need it? Add fully spec tokens padding on `recompute_scheduler`.  ### Does this PR introduce _any_ user-facing change? No.  ### How was this patch tested? By CI.   - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/d02df748bf9efd99022f1a06…

### #15108 — [[BugFix] [load_balance_proxy] Forward decode backend errors to client instead of empty 200](https://github.com/vllm-project/vllm-ascend/pull/15108)
- **作者**: xlshaoscu  **时间**: 2026-08-27 16:53 CST
- **摘要**: ﻿## Description  Fixes #12166.  When the decode backend returns an error (e.g. 4xx because `prompt_tokens + max_tokens > max_model_len`, or 5xx/OOM/connection), the load-balance proxy currently swallows the exception inside `generate_stream()` and ends the `StreamingResponse` generator without yield…

### #15105 — [[BugFix][Spec Decode] Remove obsolete sequence length update](https://github.com/vllm-project/vllm-ascend/pull/15105)
- **作者**: jiajinzhu2  **时间**: 2026-08-27 16:46 CST
- **标签**: ready-precise
- **摘要**: ### What this PR does / why we need it?  This PR removes the unused `_ascend_update_seq_lens` method and its invocation in `_run_model` within `AscendAutoRegressiveSpeculator`.  The method runs after the draft model execution has completed, and the generated `seq_len_list` is not consumed by the sub…
