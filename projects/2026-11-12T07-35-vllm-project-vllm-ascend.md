# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-12 15:35 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
- **#21684 [Bug]: BF16 target logits change DeepSeek V4.1 DSpark verification on A3**
  报告了一个在 Ascend A3 上运行 DeepSeek V4.1-Flash (W4A8) 的 Bug：目标模型在 DSpark 投机解码验证前，将其 logits 舍入为 BF16，导致错误拒绝了 DSpark 候选 Token。

---

### 🚀 PR 动态
本轮合并的 PR 涵盖了新特性支持、多项关键 Bug 修复及 CI 优化：

**✨ 新特性与模型支持**
- **#21690 [Mooncake V2]**: 使 `MooncakeConnectorV2` 支持 layerwise push（分层推送），并保留已分配的块。
- **#21687 [Kimi]**: 在 Pipeline 并行的最后一个阶段为目标模型提供共享的 Draft 输入嵌入。
- **#21682 [Liquid d1]**: 开始将 Liquid d1-3B 模型（包含 ShortConv 层）集成到 Ascend 平台。

**🛠️ Bug 修复**
- **#21691 [DSpark]**: 修复 DSpark 静态 kernel 编译白名单不兼容问题（因 Draft 和 Target 模型在 CUDA Graph 捕获时使用了不同的查询长度）。
- **#21688 [PCP]**: 修复 PCP (Prefix Caching Prompt) 场景下，当仅剩一个新 prompt token 时生命周期标记错误导致的短 prefill 问题，适用于所有后端。
- **#21686 [RFork]**: 修复 RFork 注册模型时无法正确读取量化 checkpoint 索引的问题，以正确恢复 DSpark/MTP 头的归属状态。
- **#21683 [KV Transfer]**: 允许 C8_MXFP KV 传输识别使用 Memcache 后端的 AscendStoreConnector，修复了由此导致的启动失败问题。

**⚙️ CI 与基础设施**
- **#21689**: 按分支隔离 lint 镜像，避免主分支与发布分支依赖版本不同引发的冲突，并添加了轻量级工具镜像。
- **#21685**: 新增独立的 `presmoke` CI 流程，可通过给 PR 打上 `presmoke` 标签触发。

**📝 文档与杂项**
- **#21681**: 修复 po 文件。

---

### 📦 Release 动态
- 本期动态内无新的版本发布。

---

## 🐛 Issues

### #21684 — [[Bug]: BF16 target logits change DeepSeek V4.1 DSpark verification on A3](https://github.com/vllm-project/vllm-ascend/issues/21684)
- **作者**: KeyanHu-git  **时间**: 2026-11-12 14:56 CST
- **标签**: llm-model, deepseek
- **摘要**: ### 🐛 Describe the bug  DeepSeek-V4.1-Flash-1017 W4A8 on Ascend A3 can reject a DSpark candidate because the target head rounds its logits to BF16 before verification. With the target hidden state, head weights and proposed token fixed, preserving FP32 logits changes the target's selected token and …

## 🔀 Pull Requests

### #21691 — [[BugFix] Fix incompatibility of Dspark's static kernel compliation allow list](https://github.com/vllm-project/vllm-ascend/pull/21691)
- **作者**: RuishenYang  **时间**: 2026-11-12 15:23 CST
- **摘要**: ### What this PR does / why we need it? This PR fixes StaticKernel compilation for DSpark speculative decoding in vLLM Ascend.  DSpark uses different query lengths for CUDA Graph capture:  Draft model: num_spec_tokens tokens per request.  Target model: num_spec_tokens + 1 tokens per request.  Previo…

### #21690 — [[BugFix][Feature] Enable Mooncake V2 layerwise push and preserve allocation blocks](https://github.com/vllm-project/vllm-ascend/pull/21690)
- **作者**: nwpu-zxr  **时间**: 2026-11-12 15:18 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Allow `MooncakeConnectorV2` to select `MooncakePushConnector` with `kv_connector_extra_config.use_layerwise=true`. Omitting the option or setting it to `false` keeps the existing pull behavior. The V2 entry point constructs the concrete push/pull connector, p…

### #21689 — [[CI] Isolate lint images by branch and add a lightweight tools image](https://github.com/vllm-project/vllm-ascend/pull/21689)
- **作者**: MrZ20  **时间**: 2026-11-12 15:15 CST
- **标签**: documentation, ci/build
- **摘要**: ### What this PR does / why we need it? All branches currently share one lint image. When main and release branches require different dependency versions, rebuilding that image can break pre-commit checks, including mypy, on another branch.  This PR separates images by branch and purpose:  - Use `li…

### #21688 — [Pcp short prefill all backends](https://github.com/vllm-project/vllm-ascend/pull/21688)
- **作者**: Sunny66888  **时间**: 2026-11-12 15:07 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  A cached prompt tail can still be marked as prefilling even when only one new prompt token remains. Under PCP, this lifecycle flag previously caused the request to use prefill partitioning while attention could classify it as decode, leaving request ownership…

### #21687 — [[Feature][Kimi] Provision the shared draft embedding on the last PP stage](https://github.com/vllm-project/vllm-ascend/pull/21687)
- **作者**: tanjiangshan  **时间**: 2026-11-12 15:05 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  [RFC #21537](https://github.com/vllm-project/vllm-ascend/issues/21537) PR 1 of 3: make the target responsible for providing the shared input embedding on its last pipeline-parallel stage.  Under PP, a DSpark draft that reuses the target input embedding runs o…

### #21686 — [[BugFix] Read quantized checkpoint indexes for RFork model state](https://github.com/vllm-project/vllm-ascend/pull/21686)
- **作者**: yangsonglin13  **时间**: 2026-11-12 15:02 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Before RFork registers a model, `_get_checkpoint_weight_names` reads the checkpoint weight names to restore model state such as MTP/DSpark head ownership. It only looked for the index file that vLLM's `DefaultModelLoader` reports (`model.safetensors.index.jso…

### #21685 — [[CI] add presmoke for CI](https://github.com/vllm-project/vllm-ascend/pull/21685)
- **作者**: xqchen7  **时间**: 2026-11-12 14:58 CST
- **标签**: documentation, ci/build
- **摘要**: ### What this PR does / why we need it?  Adds a standalone **presmoke** CI capability, triggered by applying the `presmoke` label to a PR (`.github/workflows/pr_presmoke_trigger.yaml`).  **Why**: when CANN-related content in the root Dockerfiles changes (base image version, torch_npu / CANN alignmen…

### #21683 — [[BugFix] Allow C8 KV transfer with AscendStore Memcache](https://github.com/vllm-project/vllm-ascend/pull/21683)
- **作者**: kitaikuyo421esfj-hub  **时间**: 2026-11-12 14:56 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it? Allow the C8_MXFP KV transfer guard to recognize AscendStoreConnector when its backend is Memcache. This addresses the startup failure that currently says C8_MXFP v1 PD disaggregation only supports MooncakeConnectorV1. The existing Mooncake connector predicate…

### #21682 — [[Model][Feature] Investigate Liquid d1 ShortConv support on Ascend](https://github.com/vllm-project/vllm-ascend/pull/21682)
- **作者**: do420  **时间**: 2026-11-12 14:54 CST
- **摘要**: ### What this PR does / why we need it?  Start the Ascend integration of Liquid d1-3B, which uses the upstream LFM2-VL model and ShortConv layers. Upstream ShortConv imports convolution functions directly; if it is imported before Ascend replaces those functions, it can retain CUDA references. Refre…

### #21681 — [[doc][Misc] Fix po files](https://github.com/vllm-project/vllm-ascend/pull/21681)
- **作者**: ella1107  **时间**: 2026-11-12 14:47 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? Fix po files  ### Does this PR introduce _any_ user-facing change? No  ### How was this patch tested? No  - vLLM main: https://github.com/vllm-project/vllm/commit/544f182d9f0c9bfe8bd7645525951d5e3692a542
