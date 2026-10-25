# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-25 13:09 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue (问题反馈)
近期共报告了 3 个重要 Bug，主要集中在多模态模型和特定硬件的内存/算子异常：
1. **M-RoPE 多模态模型与 Prefix caching 冲突崩溃 (#19727)**：在同时开启前缀缓存（Prefix caching）和负载均衡调度（`BalanceScheduler`）时，由于 `NewRequestData.from_request` 遗漏了 `uses_mrope` 标志，导致 Qwen 系列多模态模型（如 Qwen3-VL 等）在遇到前缀缓存命中时直接崩溃。
2. **310P 设备可用 KV Cache 为负数 (#19726)**：`NPUWorker310.determine_available_memory()` 混淆了设备级和实例级的内存统计范围，错误地将其他进程占用的内存扣减至当前实例，导致可用 KV 缓存出现负值。
3. **A3 平台算子异常导致 TP8 Workers 全部崩溃 (#19722)**：在 Atlas A3 平台运行 DeepSeek-V4.1-Flash 并发长提示词任务时，`grouped_matmul_swiglu_quant_v2` 算子触发 Vector core 异常（错误码 507035），导致所有 TP8 workers 被强制终止。

---

### 🚀 Pull Request (代码合并)
本期共有 10 个 PR 合并，涵盖了关键 Bug 修复、新特性及文档完善：

**✨ 重要新特性:**
1. **投机解码优化 (#19724)**：为 MRV2 支持了 `draft_window_size` 滑动窗口特性。解决了草稿模型在长上下文（如 32k）下接受率骤降的问题，有效维持推理速度。
2. **稀疏卸载与 MLAPO 优化 (#19719)**：支持直接 MLAPO KV 输出和融合 CPU 回写，允许在 Decode 阶段进行带有融合重叠稀疏 KV 卸载的 MLAPO 预处理，减少 NPU 与 CPU 间的写回开销。
3. **INT2 后端架构重构 (#19728, #19729)**：oscar 系列重构的第 5、6 部分，将原先单体架构中的 INT2 共享辅助函数（如 R150/R166 slot 分配器）抽离至独立文件，并添加了后端模块的显式共享导入中心。

**🛠 修复与 CI 改进:**
1. **M-RoPE 崩溃修复 (#19731)**：针对上述 Issue #19727，在 `BalanceScheduler`/`DyntraLB` 中向 `NewRequestData.from_request` 正确传递 `uses_mrope`，修复多模态模型崩溃问题。
2. **多节点 CI 门禁修复 (#19725)**：修复了 A2 夜间测试门禁逻辑漏洞，原先只要有一个多节点 leader 准备就绪就会错误地放行单节点任务，现已修正为等待所有已启用的多节点 leaders 就绪后再释放。

**📖 文档更新:**
1. **模型部署指南**：新增 MiniMax-M3.1-Flash 的部署指南与支持矩阵 (#19723)；为 DeepSeek-V4-Pro 文档补充了 950DT Memcache KV Cache Pool PD 部署章节 (#19717, #19718)。
2. **文档本地化**：自动翻译并合并了 33 个中文文档文件 (#19730)。

---

### 📦 Release (版本发布)
本期未观察到正式的版本发布记录。
*注：PR #19717 中标题带有 `[v0.30.0]` 标签，推测仓库正在为 v0.30.0 版本进行文档收尾和前置部署指南的补充工作，近期可能会有新版本发布。*

---

## 🐛 Issues

### #19727 — [[Bug]: Prefix caching + M-RoPE models crash with balance scheduling — BalanceScheduler/DyntraLB omit uses_mrope in NewRequestData.from_request](https://github.com/vllm-project/vllm-ascend/issues/19727)
- **作者**: amazingapple  **时间**: 2026-10-25 11:36 CST
- **摘要**: ## Summary  With `--enable-prefix-caching` and an M-RoPE multimodal model (Qwen3.5 / Qwen3-VL / Qwen2.5-VL), enabling `enable_balance_scheduling` (`BalanceScheduler`) crashes the engine as soon as a request hits the prefix cache — typically the 2nd/3rd question about the same video/image.  The proxi…

### #19726 — [[Bug]: [310P] determine_available_memory() charges other processes' memory to the current instance, yielding negative  available KV cache](https://github.com/vllm-project/vllm-ascend/issues/19726)
- **作者**: Paulx209  **时间**: 2026-10-25 11:13 CST
- **标签**: bug, 310p
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text    Collecting environment information...    ==============================            System Info    ==============================    OS                           : Ubuntu 22.04.5 LTS (aarch64)…

### #19722 — [[Bug]: Vector core exception (507035) in grouped_matmul_swiglu_quant_v2 kills all TP8 workers under concurrent long-prompt load (Atlas A3, DeepSeek-V4.1-Flash)](https://github.com/vllm-project/vllm-ascend/issues/19722)
- **作者**: rockjimi  **时间**: 2026-10-25 01:23 CST
- **标签**: llm-model, deepseek
- **摘要**: ## Environment  - **vllm-ascend**: quay.io/ascend/vllm-ascend:deepseek-v4.1-flash-a3 (vLLM 0.27.1+empty) - **CANN**: 9.1.0 (built-in OPP) - **Platform**: Atlas 800I A3, 8×910C (16 logical devices), dual TP8 instances - **Model**: DeepSeek-V4.1-Flash W4A8 (INT8 Engram) - **torch_npu**: 2.10.0.post2 -…

## 🔀 Pull Requests

### #19731 — [[BugFix] Forward uses_mrope to NewRequestData.from_request in BalanceScheduler/DyntraLB](https://github.com/vllm-project/vllm-ascend/pull/19731)
- **作者**: amazingapple  **时间**: 2026-10-25 12:44 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  `NewRequestData.from_request()` strips the payload of multimodal items that are fully covered by a prefix-cache hit. With the upstream default `uses_mrope=False` it drops every payload field, including the `keep_on_cpu` `image_grid_thw` / `video_grid_thw` tha…

### #19730 — [[Doc] Translated Doc files 2026-10-25](https://github.com/vllm-project/vllm-ascend/pull/19730)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-25 12:34 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **33** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents/context_parallel.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/develop…

### #19729 — [[Attention][Feature] oscar: add the explicit shared import hub for the backend modules [6/22]](https://github.com/vllm-project/vllm-ascend/pull/19729)
- **作者**: nqz1997  **时间**: 2026-10-25 11:53 CST
- **标签**: module:tests
- **摘要**: ## Summary  Adds `vllm_ascend/attention/int2/ctx.py`, the explicit import hub the later INT2 backend modules use to reach:  - the config surface / diag / rotation loader, - the context-parallel metadata types (main keeps them in `attention.context_parallel.attention_cp`), - scheduler / attention-sta…

### #19728 — [[Attention][Feature] oscar: relocate the shared INT2 helpers out of the monolith [5/22]](https://github.com/vllm-project/vllm-ascend/pull/19728)
- **作者**: nqz1997  **时间**: 2026-10-25 11:53 CST
- **标签**: module:tests
- **摘要**: ## Summary  Extracts the shared INT2 helpers from the monolithic backend into `vllm_ascend/attention/int2/helpers.py`:  - **R150/R166 slot allocator** — fail-closed acquire/release with cover invariants, generation counters, optional identity-row reservation (`reserve0`) - **R114 steady-state trace*…

### #19725 — [[CI] Wait for all enabled multi-node leaders before releasing single-node](https://github.com/vllm-project/vllm-ascend/pull/19725)
- **作者**: CuixyHW  **时间**: 2026-10-25 10:41 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it?  **Problem.** The A2 nightly gate `wait-multi-node-pods-ready` released the single-node / accuracy jobs as soon as *any* leader pod was Ready (`READY >= 1`), and it counted `role=leader` pods cluster-wide. Two issues:  1. A single ready leader released single-…

### #19724 — [[Feature][Spec Decode][MRV2] Support draft_window_size sliding window](https://github.com/vllm-project/vllm-ascend/pull/19724)
- **作者**: krell11  **时间**: 2026-10-25 03:24 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Draft models trained on short contexts lose acceptance as the running context grows (e.g. collapse from ~5 to ~1 at 32k). `draft_window_size` already restores draft acceptance on Model Runner V1 by capping **draft-only** KV attention to a recent window while …

### #19723 — [[Doc][Model] Add MiniMax-M3.1-Flash deployment guide and support matrix](https://github.com/vllm-project/vllm-ascend/pull/19723)
- **作者**: CXY-Katrina  **时间**: 2026-10-25 02:40 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Consolidate the MiniMax-M3.1-Flash documentation from #19665 and #19652 into one tutorial with Experimental navigation labels and the original support-matrix entries.  - Document Ascend-adapted W8A8/MXFP8 weights, platform-specific Docker images, model mounts…

### #19719 — [[Feature] Support Direct MLAPO KV outputs and fused CPU writeback for sparse offload](https://github.com/vllm-project/vllm-ascend/pull/19719)
- **作者**: zmc1997  **时间**: 2026-10-24 21:51 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Enable MLAPO preprocessing with fused-overlap sparse KV offload on Decode. The existing MLAPO writes to an NPU KV cache, while offload needs the current token's KV tensors and a completed CPU-pool write before attention can consume historical KV.  This PR:  -…

### #19718 — [[Doc] Add 950DT Memcache KV Cache Pool PD deployment section to DeepSeek-V4-Pro docs](https://github.com/vllm-project/vllm-ascend/pull/19718)
- **作者**: lenghuixing0330  **时间**: 2026-10-24 20:56 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…

### #19717 — [[Doc][v0.30.0] Add 950DT Memcache KV Cache Pool PD deployment section to DeepSeek-V4-Pro docs](https://github.com/vllm-project/vllm-ascend/pull/19717)
- **作者**: lenghuixing0330  **时间**: 2026-10-24 20:54 CST
- **标签**: documentation
- **摘要**: <!--  Thanks for sending a pull request!  BEFORE SUBMITTING, PLEASE READ https://docs.vllm.ai/en/latest/contributing/overview.html  --> ### What this PR does / why we need it? <!-- - Please clarify what changes you are proposing. The purpose of this section is to outline the changes and how this PR …
