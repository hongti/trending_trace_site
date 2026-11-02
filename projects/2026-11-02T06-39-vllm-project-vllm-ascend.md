# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-02 14:39 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📋 Issue 动态
- **#20620 [RFC]: 为 Ascend 950 上的 MegaMoE 构建、打包和加载 DeepGEMM-Ascend**
  - 作者: linfeng-yuan
  - 内容: 提出方案探讨如何为 Ascend 950 上的 MegaMoE 实验性集成构建和加载 DeepGEMM-Ascend，旨在解决当前依赖外部准备环境的问题。

### 🔧 Pull Request 动态

**✨ 新特性与功能增强**
- **#20621 为所有稠密解码器系列启用稠密序列并行**: 移除了此前仅限 Qwen3.5 系列的白名单限制，将序列并行布局推广到通用的解码器层，扩大了适配范围。
- **#20615 将 GLM RecoverSSM 验证记录移出 KV cache**: 改为按 `max_num_seqs` 分配每层的暂存缓冲区，优化存储结构并改进索引验证逻辑。
- **#20612 通过 HIXL 在 P/D 实例间共享 A2 主机表**: 为独立的 DeepSeek V4.1 prefill 和 decode 实例提供了一种可选的 A2 后端，保持单一不可变表，节省资源。
- **#20619 / #20618 添加引擎与 Worker 监控的 Watchdog**: 同步上游 vLLM 的 PR #55700，引入 Watchdog 机制解决引擎核心或 Worker 进程挂起的问题。

**🐛 Bug 修复**
- **#20617 修复推测解码 V2 EAGLE 加载问题**: 加载 draft 运行时配置，解决 Kimi-K2.6 搭配 EAGLE3 在 Model Runner V2 上因 `He=14336` 报错的问题。
- **#20616 修复 CI 密钥扫描器失效问题**: 修复了 gitleaks 脚本在 CI 模式下提前退出导致未实际扫描 PR 变更的漏洞。
- **#20614 回溯打包张量修复至 v0.30.0**: 将修复内存别名和未对齐混合数据类型问题的 PR #19164 向后移植到 `releases/v0.30.0` 分支。

**🚀 性能优化**
- **#20613 跨步骤缓存解码 GDN Attention 元数据**: 针对混合（线性注意力）模型，避免在每次解码步骤中为每个 KV-cache 组重复构建元数据，从而提升性能。

**🤖 CI 与构建**
- **#20611 更新 main 分支依赖**: 更新了 `triton-ascend`、`torch-npu`、`memfabric` 及 `memcache` 组件。

### 🚀 Release 动态
近期无新的 Release 版本发布。

---

## 🐛 Issues

### #20620 — [[RFC]: Build, package and load DeepGEMM-Ascend for MegaMoE on Ascend 950](https://github.com/vllm-project/vllm-ascend/issues/20620)
- **作者**: linfeng-yuan  **时间**: 2026-11-02 14:28 CST
- **标签**: RFC
- **摘要**: ### Motivation.  The experimental DeepInfer MegaMoE integrations (#20299 and #20395 on main, #20519 and #20520 on releases/v0.32.0rc) currently depend on an externally prepared DeepGEMM-Ascend environment. The framework changes alone do not reproduce the installation used for hardware validation.  T…

## 🔀 Pull Requests

### #20621 — [[Feature][Model] Enable dense sequence parallelism for all dense decoder families](https://github.com/vllm-project/vllm-ascend/pull/20621)
- **作者**: chenxi-hh  **时间**: 2026-11-02 14:36 CST
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: The FlashComm switch previously enabled dense sequence parallelism only for the Qwen3.5 family, through a model-type whitelist and a per-family decoder-layer patch. This moves the layout to the generic layer primitives every dense family shares, so any dense checkpoint following the standard norm ->…

### #20619 — [[Misc]: Add Watchdog for Engine and Worker Monitoring](https://github.com/vllm-project/vllm-ascend/pull/20619)
- **作者**: wenjinhust  **时间**: 2026-11-02 14:27 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Fixes #16631   Sync with the upstream PR [#55700](https://github.com/vllm-project/vllm/pull/55700).  **Why:** When the vLLM engine core or worker process hangs on Ascend (stuck kernel call, deadlock, stalled IPC), there is currently no visibility into where t…

### #20618 — [Add Watchdog for Engine and Worker Monitoring](https://github.com/vllm-project/vllm-ascend/pull/20618)
- **作者**: wenjinhust  **时间**: 2026-11-02 14:25 CST
- **标签**: documentation, ci/build, module:tests, module:ops, module:core, module:quantization, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  Fixes #16631   Sync with the upstream PR [#55700](https://github.com/vllm-project/vllm/pull/55700).  **Why:** When the vLLM engine core or worker process hangs on Ascend (stuck kernel call, deadlock, stalled IPC), there is currently no visibility into where t…

### #20617 — [[BugFix][Spec Decode] Load V2 EAGLE with its draft runtime config](https://github.com/vllm-project/vllm-ascend/pull/20617)
- **作者**: sugm0521  **时间**: 2026-11-02 14:11 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Kimi-K2.6 with EAGLE3 on Model Runner V2 can fail with `MlaPrologV3` rejecting `He=14336`. The draft's first layer legitimately concatenates two 7168-wide inputs.  Ascend prepares `draft_vllm_config`, but the inherited EAGLE loader passes the target's `self.v…

### #20616 — [[CI] Make the secret scanner actually scan PR changes](https://github.com/vllm-project/vllm-ascend/pull/20616)
- **作者**: Mr-qiji  **时间**: 2026-11-02 14:11 CST
- **摘要**: ### What this PR does / why we need it?  The CI (pull_request) mode of `.github/workflows/scripts/gitleaks.sh` early-exited, so no secret scanning ran at all in CI: any PR containing a credential passed the secret scan step (see the PR #20440 probe linked in #20452).  This PR makes the CI mode scan …

### #20615 — [[Feature] Move GLM RecoverSSM verify records out of KV cache](https://github.com/vllm-project/vllm-ascend/pull/20615)
- **作者**: lijiahang226  **时间**: 2026-11-02 14:09 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  GLM RecoverSSM currently stores temporary verify records in every KDA KV page. Allocate per-layer scratch buffers by `max_num_seqs` instead, index verify/commit records by speculative batch row, and retain only convolution/SSM state in KV pages. The Ascend me…

### #20614 — [[v0.30.0][BugFix][WeightTransfer] Backport packed tensor fixes (#19164)](https://github.com/vllm-project/vllm-ascend/pull/20614)
- **作者**: yeyunu  **时间**: 2026-11-02 14:03 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Backport #19164 to `releases/v0.30.0`.  Packed weight transfer on this branch can return tensors that alias a reusable receive buffer, reject unaligned mixed-dtype slices or scalar tensors, and mishandle asynchronous transfer cleanup. This patch gives unpacke…

### #20613 — [[Performance][Ops] Cache decode GDN attention metadata across steps](https://github.com/vllm-project/vllm-ascend/pull/20613)
- **作者**: weifulun  **时间**: 2026-11-02 13:43 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Ref #20607.  `AscendGDNAttentionMetadataBuilder.build()` is invoked once per GDN KV-cache group on every decode step. On hybrid (linear attention) models, rebuilding the same `GDNAttentionMetadata` for every group is a significant part of the host-side step t…

### #20612 — [[Feature][Engram] Share A2 host tables across P/D instances via HIXL](https://github.com/vllm-project/vllm-ascend/pull/20612)
- **作者**: foraxe  **时间**: 2026-11-02 13:13 CST
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Independent DeepSeek V4.1 prefill and decode instances otherwise retain separate host Engram tables. This opt-in A2 backend keeps one immutable table in an owner process and fetches selected rows over HIXL. The owner gathers quantized rows into registered hos…

### #20611 — [[CI] update triton-ascend torch-npu memfabric memcache for main](https://github.com/vllm-project/vllm-ascend/pull/20611)
- **作者**: xqchen7  **时间**: 2026-11-02 13:07 CST
- **标签**: ci/build, ready-precise
- **摘要**: ### What this PR does / why we need it? update triton-ascend torch-npu memfabric memcache for main  ### Does this PR introduce _any_ user-facing change? NA  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change invol…
