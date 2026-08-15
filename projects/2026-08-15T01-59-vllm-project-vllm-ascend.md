# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-15 09:59 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📌 Issue 动态
1. **CI 自动化流程中断** (#14319)：`main2main` 自动同步流程未能完成所有计划步骤，当前需人工介入审查。
2. **SpecDecode 导致 KV Cache 严重膨胀** (#14314)：使用 DSpark 投机解码时，Kimi-K3（MLA + KDA 混合架构）的 KV Cache padding 膨胀至 26%，导致单张 Ascend 910 A3 (64GB HBM) 无法支持 1M (`max-model-len=1048576`) 的上下文长度。

### 📌 PR 动态
**🚀 新特性**
- **支持 MLA Context Parallel 组合** (#14316)：在 Ascend 上实现了组合 MLA PCP 与 DCP，补齐了 Dense MLA Model Runner V2 的上下文并行路径。
- **MRV2 支持 MTP+SFA FullGraph** (#14310)：在 ModelRunner V2 中支持带 SFA 的多 Token 预测（MTP）FullGraph 模式，修复了 Eagle MTP draft 图重放时的参数重建问题。

**⚡ 性能优化**
- **移除冗余 NPU-CPU 同步** (#14313)：在 DSA-CP QLI metadata 构建路径中避免了不必要的 NPU-to-CPU 同步，提升注意力机制性能。

**🛠️ Bug 修复**
- **修复 DeepSeek V4 推理提前终止** (#14308)：修复了 DSV4 在 Ascend NPU 上推理未完成就过早生成 EOS 的问题（通过增加 Ascend 本地扩展实现）。
- **对齐 DeepSeek V4 Thinking 默认行为** (#14317)：回溯移植修复，确保遗漏的 thinking 控制默认使用高效推理模式。
- **修复 Cudagraph 调度验证缺失** (#14307, #14311, #14312)：修复了在无 `recompute_scheduler` 且开启 `mix_cudagraph_mode` 的 D 节点上，因缺少 cudagraph 模式验证导致不同 DP ranks 行为异常的问题。

**🤖 CI 与适配**
- **适配 vLLM Main 分支** (#14318)：尝试适配 vLLM main 分支 (对应 v0.27.1)，但目前自动化适配步骤全部失败。
- **更新 CI 测试模型矩阵** (#14315)：新增了 glm5.1/5.2 (w8a8c8) 及 qwen3.5/3.6 (35B/w4a8) 等量化/大模型版本的 CI 测试。

### 📌 Release 动态
- 近期无新的 Release 版本发布。

---

## 🐛 Issues

### #14319 — [[main2main] main2main manual review required ()](https://github.com/vllm-project/vllm-ascend/issues/14319)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-15 00:39 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/14318 - Commit range: `1d2d83a07fd3180068917a031c28dcc83141d0be`...`` - Status: `failed`  ## Final Summary  main2main adaptation failed — no steps c…

### #14314 — [[Bug][SpecDecode] DSpark inflates Kimi-K3 KV cache padding to 26%, blocking 1M max-model-len](https://github.com/vllm-project/vllm-ascend/issues/14314)
- **作者**: linnea-lin-00638949  **时间**: 2026-08-14 20:53 CST
- **摘要**: ## Summary  Kimi-K3 (MLA + KDA hybrid, 24 full-attention + 69 KDA layers) with DSpark speculative decoding cannot fit `max-model-len=1048576` (1M) on a single Ascend 910 A3 card (61.28 GiB HBM): the KV cache reservation fails with a 26% padding waste that is **introduced by DSpark**, not by the mode…

## 🔀 Pull Requests

### #14318 — [[Misc]feat: adapt to vLLM main (3ac95255)](https://github.com/vllm-project/vllm-ascend/pull/14318)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-15 00:38 CST
- **标签**: module:tests, module:ops
- **摘要**: main2main adaptation failed — no steps completed.  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14317 — [[v0.25.1rc][BugFix][Frontend] Align DeepSeek V4 parser thinking default](https://github.com/vllm-project/vllm-ascend/pull/14317)
- **作者**: zhangshuai404  **时间**: 2026-08-15 00:10 CST
- **标签**: module:tests
- **摘要**: Backport vLLM-Ascend PR #14074 from releases/v0.26.0rc to releases/v0.25.1rc.  The DeepSeek V4 Flash 0731 tokenizer backport defaults omitted thinking controls to thinking mode with high reasoning effort, while the vLLM v0.25.1 parser still starts in CONTENT mode. Because the rendered prompt already…

### #14316 — [[Attention][Feature] Support combined MLA PCP and DCP on Ascend](https://github.com/vllm-project/vllm-ascend/pull/14316)
- **作者**: imsatoshi  **时间**: 2026-08-14 21:42 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This PR completes the dense MLA Model Runner V2 context-parallel path requested by #10660 and RFC #12622. It is stacked on #14008 and keeps upstream `PCPManager` as the single owner of virtual-batch partitioning, expanded slot mappings, decode-write de-duplic…

### #14315 — [[CI] update glm5.1-w8a8c8 glm5.2-w8a8c8 qwen3.6-35B qwen3.5-122B-w4a8](https://github.com/vllm-project/vllm-ascend/pull/14315)
- **作者**: guxin108  **时间**: 2026-08-14 21:41 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? update glm5.1-w8a8c8 glm5.2-w8a8c8 qwen3.6-35B qwen3.5-122B-w4a8  ### Does this PR introduce _any_ user-facing change? NO  ### How was this patch tested? run the cases weekly or nightly  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm…

### #14313 — [[Performance][Attention] Avoid DSA CP QLI metadata sync](https://github.com/vllm-project/vllm-ascend/pull/14313)
- **作者**: ltdo111  **时间**: 2026-08-14 20:52 CST
- **摘要**: ### What this PR does / why we need it? This PR avoids an unnecessary NPU-to-CPU synchronization in the DSA-CP QLI metadata build path.  `_build_qli_metadata` previously derived `max_seqlen_q` and `max_seqlen_k` from device tensors with `.max().item()`. On Ascend NPU, calling `.item()` on a device t…

### #14312 — [[bugfix] fix bug when D node with mix_cudagraph_mode and without reco…](https://github.com/vllm-project/vllm-ascend/pull/14312)
- **作者**: zzzzwwjj  **时间**: 2026-08-14 19:40 CST
- **标签**: module:core, ready
- **摘要**: ### What this PR does / why we need it?  When D node with mix_cudagraph_mode and without recompute_scheduler, We overlooked the verification of the cudagraph mode earlier, which may cause different DP running on different cudagraph mode.  ### Does this PR introduce _any_ user-facing change?  None.  …

### #14311 — [[bugfix] fix bug when D node with mix_cudagraph_mode and without reco…](https://github.com/vllm-project/vllm-ascend/pull/14311)
- **作者**: zzzzwwjj  **时间**: 2026-08-14 19:40 CST
- **标签**: module:core, ready
- **摘要**: ### What this PR does / why we need it?  When D node with mix_cudagraph_mode and without recompute_scheduler, We overlooked the verification of the cudagraph mode earlier, which may cause different DP running on different cudagraph mode.  ### Does this PR introduce _any_ user-facing change?  None.  …

### #14310 — [[Feature][MRV2] Support MTP with SFA in FullGraph mode](https://github.com/vllm-project/vllm-ascend/pull/14310)
- **作者**: jiajinzhu2  **时间**: 2026-08-14 19:36 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This PR fixes Eagle MTP draft full-graph replay with SFA on ModelRunner V2.  During draft graph replay, SFA rebuilds graph parameters using the current vLLM configuration. However, the Eagle draft replay path only sets the forward context and does not set the…

### #14308 — [[Bugfix]mask premature eos during reasoning on DSV4](https://github.com/vllm-project/vllm-ascend/pull/14308)
- **作者**: doubinfx  **时间**: 2026-08-14 19:26 CST
- **标签**: module:tests, module:core
- **摘要**: Fixes #14031  ### What this PR does / why we need it?    This PR fixes the premature EOS issue during unfinished DeepSeek V4 reasoning on Ascend NPU.    The change adds an Ascend-local extension to the upstream thinking budget state holder. When explicitly enabled with:    {"premature_eos_policy": "…

### #14307 — [[bugfix] fix bug when D node with mix_cudagraph_mode and without recompute_scheduler](https://github.com/vllm-project/vllm-ascend/pull/14307)
- **作者**: zzzzwwjj  **时间**: 2026-08-14 19:12 CST
- **标签**: module:core, ready
- **摘要**: ### What this PR does / why we need it?  When D node with mix_cudagraph_mode and without recompute_scheduler, We overlooked the verification of the cudagraph mode earlier, which may cause different DP running on different cudagraph mode.  ### Does this PR introduce _any_ user-facing change?  None.  …
