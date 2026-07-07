# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-07 17:11 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 最近动态的中文摘要：

### 🚀 Release 动态
近期无新版本发布信息。

---

### 🐛 Issue 动态
本期 Issue 主要聚焦于**性能劣化、错误提示优化及底层加载机制缺陷**：
1. **性能劣化问题 (#11561)**：`qwen3_30b` 模型在 A2/A3 环境下，自 main2main 升级（从 v0.20.1 至 v0.20.2 / Commit 26437c1）后出现明显的性能下降。
2. **报错提示不清晰 (#11557)**：当用户请求的 tokens 数超过模型最大上下文长度时，系统返回的错误信息未明确指出具体的输入 token 长度限制，体验不佳，需优化提示。
3. **底层机制缺陷 (#11553)**：D2D netloader 发生 fallback 时，缺失 `static_forward_context` 的清理操作；同时 MTP（Multi-Token Prediction）权重加载效率较低。

---

### 🔧 PR 动态
本期 PR 涵盖**关键 Bug 修复、新模型适配、LoRA 与特性增强，以及文档/CI 优化**：

**🛠️ 关键 Bug 修复**
- **回退修复 TP 挂起 (#11562)**：回退了 Pre-KV ACL graph profiling (#9865) 的提交，以修复 lmhead 在 TP（Tensor Parallel）场景下的挂起问题。
- **修复 KV Transfer 端口映射 (#11559)**：修复了 Mooncake PD KV transfer 握手监听端口无法安全推断的问题，确保 Prefill worker 正常握手。

**🌟 新特性与模型适配**
- **LoRA 补全注册 (#11555)**：在 `v0.21.0rc` 版本中，为 TP>1 场景下的 MLP 和 `o_proj` dense linear 层注册 Ascend LoRA wrappers（此前仅注册了四个 QKV wrappers）。
- **Dynamic SD 兼容 Full Graphs (#11552)**：[WIP] 正在开发使 Dynamic SD（Speculative Decoding）兼容全图模式的功能。
- **新增 QwQ-32B 适配教程 (#11556)**：QwQ-32B（基于 Qwen2ForCausalLM 的 32B 推理模型）已原生支持上游，本 PR 补充了其在 vLLM-Ascend 的适配文档。
- **新增 MiniCPM 系列适配 (#11560, #11551)**：完成 `MiniCPM3-4B` 和 `MiniCPM-2B-dpo-bf16` 在 Ascend NPU 的端到端测试配置与教程适配。

**📚 文档与 CI 优化**
- **DeepSeek-V4 部署修复 (#11554)**：在 DeepSeek-V4-Flash/Pro 的 Docker 部署教程中补充了 `--privileged=true` 必要参数。
- **CI 新增标签转换功能 (#11558)**：新增将 wait-feedback 评论自动转换为对应标签的 CI 功能。
- **中文文档更新 (#11549)**：同步更新了中文文档内容。

---

## 🐛 Issues

### #11561 — [[Bug]: qwen3_30b在A2、A3环境均出现性能劣化](https://github.com/vllm-project/vllm-ascend/issues/11561)
- **作者**: zhao-stack  **时间**: 2026-07-07 17:01 CST
- **标签**: bug, qwen3-dense, gqa-model
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Your output of above commands here ```  </details> A3环境， qwen3_30b在Commit 26437c1（main2main升级）前后存在明显性能差异 升级前：vllm-ascned: main/ada0817     +  vllm v0.20.1 升级后：vllm-ascned: main/26437c1     +  vl…

### #11557 — [[Bug]: 当用户请求tokens数超过系统模型上下文时，系统处理异常，但是返回的响应中，没有明确输入长度对应的tokens，不清晰，请优化下](https://github.com/vllm-project/vllm-ascend/issues/11557)
- **作者**: xuyucheng220  **时间**: 2026-07-07 16:38 CST
- **标签**: bug
- **摘要**: ### Your current environment  当前错误提示：  curl  -X POST http://172.16.0.49:9000/v1/chat/completions   -H "Content-Type: application/json"   -d @req.json    {"error":{"message":"This model's maximum context length is 4096 tokens. However, you requested 1 output tokens and your prompt contains at least 4…

### #11553 — [[Bug]: Missing static_forward_context cleanup on D2D netloader fallback & low MTP weight-loading efficiency](https://github.com/vllm-project/vllm-ascend/issues/11553)
- **作者**: yilunh998  **时间**: 2026-07-07 16:16 CST
- **标签**: bug, advanced-features, mtp/speculative-decode
- **摘要**: ### Your current environment  Collecting environment information... PyTorch version: 2.10.0+cpu Is debug build: False  OS: Ubuntu 24.04.4 LTS (aarch64) GCC version: (Ubuntu 13.3.0-6ubuntu2~24.04.1) 13.3.0 Clang version: Could not collect CMake version: version 4.3.2 Libc version: glibc-2.39  Python …

## 🔀 Pull Requests

### #11562 — [[BugFix] Revert pre-KV ACL graph profiling (#9865) to fix lmhead TP hang](https://github.com/vllm-project/vllm-ascend/pull/11562)
- **作者**: yiz-liu  **时间**: 2026-07-07 17:06 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This PR reverts #9865 / commit `9099b7f66d123ea704e329d7586333ad8b08db50`, which introduced pre-KV ACL graph memory profiling in `determine_available_memory()`.  After that change, the combined scenario below can hang during inference and eventually fail with…

### #11560 — [[Doc][Feature] Adapt MiniCPM3-4B to Ascend NPU](https://github.com/vllm-project/vllm-ascend/pull/11560)
- **作者**: zhangkx-777  **时间**: 2026-07-07 16:50 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it? This PR adapts the MiniCPM3-4B model to run on Ascend NPU via `vllm-ascend`. It adds the end-to-end test configuration, model adaptation tutorial, and a skill record documenting lessons learned and troubleshooting tips. It also registers the model in the night…

### #11559 — [[BugFix][KV Transfer] Fix Mooncake handshake port mapping](https://github.com/vllm-project/vllm-ascend/pull/11559)
- **作者**: Mango03111  **时间**: 2026-07-07 16:49 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Fixes Mooncake PD KV transfer when handshake listener ports cannot be safely inferred from `kv_port + logical_rank`.  This PR:  - lets Mooncake prefill workers bind available handshake ports and publish the actual endpoint in metadata; - keeps Mooncake worker…

### #11558 — [[CI] Add new function -- Convert wait-feedback comment into a label](https://github.com/vllm-project/vllm-ascend/pull/11558)
- **作者**: Tian-Fantasea  **时间**: 2026-07-07 16:45 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? Add new function in CI, convert wait-feedback comment into a label  ### Does this PR introduce _any_ user-facing change? No  ### How was this patch tested? Completed testing in the personal repository. File: https://github.com/Tian-Fantasea/Tian/blob/main/.git…

### #11556 — [[Doc][Model] Add QwQ-32B model tutorial](https://github.com/vllm-project/vllm-ascend/pull/11556)
- **作者**: WinterSun-ysws  **时间**: 2026-07-07 16:34 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Add QwQ-32B model adaptation for vLLM-Ascend. QwQ-32B is a 32B-parameter reasoning model based on the Qwen2ForCausalLM architecture, natively supported by upstream vLLM without additional code changes.  This PR adds: - Test configuration for QwQ-32B in `tests…

### #11555 — [[LoRA] Register Ascend dense-linear LoRA wrappers for MLP/o_proj under TP>1](https://github.com/vllm-project/vllm-ascend/pull/11555)
- **作者**: Liuchenbing-2026  **时间**: 2026-07-07 16:27 CST
- **摘要**: ## Summary  On the `v0.21.0rc` line, `refresh_all_lora_classes()` registers only the four QKV Ascend LoRA wrappers. The dense linear layers (`gate_up_proj` / `down_proj` / `o_proj`) are `AscendMergedColumnParallelLinear` / `AscendRowParallelLinear` / `AscendColumnParallelLinear` subclasses, but the …

### #11554 — [[Doc] Fix DeepSeek-V4 documentation for Docker deployment](https://github.com/vllm-project/vllm-ascend/pull/11554)
- **作者**: GDzhu01  **时间**: 2026-07-07 16:21 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  This PR adds the `--privileged=true` flag to the Docker run commands in the DeepSeek-V4-Flash and DeepSeek-V4-Pro deployment tutorials. This is necessary because the CANN driver on Ascend NPUs requires privileged container permissions to perform low-level ini…

### #11552 — [[wip][Feature] Make Dynamic SD comatible with Full Graphs](https://github.com/vllm-project/vllm-ascend/pull/11552)
- **作者**: HF-001  **时间**: 2026-07-07 16:13 CST
- **摘要**: ### What this PR does / why we need it? refer to: https://github.com/vllm-project/vllm/pull/45953 and https://github.com/vllm-project/vllm-ascend/pull/10894  ### How was this patch tested? todo - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be…

### #11551 — [[Doc][Feature] Adapt MiniCPM-2B-dpo-bf16 to Ascend NPU](https://github.com/vllm-project/vllm-ascend/pull/11551)
- **作者**: zhangkx-777  **时间**: 2026-07-07 16:08 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it? This PR adapts the MiniCPM-2B-dpo-bf16 model to run on Ascend NPU via `vllm-ascend`. It adds the end-to-end test configuration, model adaptation tutorial, and a skill record documenting lessons learned and troubleshooting tips. It also registers the model in t…

### #11549 — [[Doc] Update cn doc content](https://github.com/vllm-project/vllm-ascend/pull/11549)
- **作者**: wangxiyuan  **时间**: 2026-07-07 16:06 CST
- **标签**: documentation, ci/build, module:tools
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446
