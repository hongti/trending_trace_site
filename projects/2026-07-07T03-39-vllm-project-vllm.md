# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-07 11:39 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文简洁摘要：

### 📌 Issue (1项)
- **#47802 [Feature]**：提出优先级调度应影响 KV/Prefix 缓存保留策略。作者指出，当前 vLLM 的 `priority` 字段仅作用于请求调度，但在缓存压力下，并未优先保留高优先级请求的 KV/Prefix 缓存，期望此特性得到增强。

### 🔧 Pull Request (10项)
**🚀 新特性与架构改进**
- **#47810 [Rust Frontend]**：为 Rust 前端引导启动 OpenTelemetry 追踪导出器。解决了当前 Rust 前端启动时拒绝 `--otlp-traces-endpoint` 的问题，这是落实 RFC #44757 的关键步骤。
- **#47809 [PostTraining]**：增加 rollout artifacts 的传输配置与 Schema 支持，并实现了传输队列连接器（transfer-queue connector）。
- **#47806 [Core/NVIDIA]**：为自定义 All-Reduce 操作引入 suspend/resume 生命周期钩子，支持 TP 组静默并关闭 CUDA IPC 句柄，增强了后端控制力。
- **#47808 [Spec Decode]**：（WIP 状态）实现无 sampler padding 的 DSpark 容量重分配，目前为草稿，需进一步迭代审查。

**⚡ DeepSeek V4 (DSv4) 专项优化与修复**
- **#47807**：重构 DeepSeek V4 mHC（multi-head-compression）预热流程并移除 token-size 上限。解决了因 TileLang JIT 编译未覆盖的 token size 导致的多秒延迟飙升问题。
- **#47805**：修复 FlashInfer 后端在 DSv4 上的兼容性问题，通过检测环境变量自动禁用 packed KV cache layout（修复 #47783）。

**🐛 Bug 修复**
- **#47801 [DCP]**：修复 MLA 模型在 DCP a2a combine 后端启动时的 bf16 bitcast 崩溃问题，通过将 LSE 转为 fp32 解决（修复 #47800）。
- **#47803 [Spec Decode]**：修复异步调度期间滑动窗口注意力（SWA）块释放的错误。
- **#47804 [CI/LoRA]**：修复近期引入的 LoRA 回归问题，确保 QKV fuser 的 split sizes 在 LoRA wrappers 中正确保留，修复了 CI 构建失败。

**🔄 依赖更新**
- **#47811**：Dependabot 自动升级依赖，跨 1 个目录进行了 160 项小版本更新。

### 🎉 Release (无)
- 近期暂无新版本发布信息。

---

## 🐛 Issues

### #47802 — [[Feature]: Priority scheduling is not reflected in KV/prefix cache retention under cache pressure](https://github.com/vllm-project/vllm/issues/47802)
- **作者**: lcc-star  **时间**: 2026-07-07 10:13 CST
- **标签**: feature request
- **摘要**: ### 🚀 The feature, motivation and pitch  I ran a minimal experiment to check whether the `priority` field in vLLM affects only request scheduling, or whether it is also considered by KV/prefix cache management.  The result suggests that `priority` works for request scheduling, but does not appear to…

## 🔀 Pull Requests

### #47811 — [Bump the minor-update group across 1 directory with 160 updates](https://github.com/vllm-project/vllm/pull/47811)
- **作者**: dependabot[bot]  **时间**: 2026-07-07 11:32 CST
- **标签**: rocm, ci/build, cpu, nvidia, dependencies
- **摘要**: Dependabot will resolve any conflicts with this PR as long as you don't alter it yourself. You can also trigger a rebase manually by commenting `@dependabot rebase`.  [//]: # (dependabot-automerge-start) [//]: # (dependabot-automerge-end)  ---  <details> <summary>Dependabot commands and options</sum…

### #47810 — [[Rust Frontend] Bootstrap the OpenTelemetry trace exporter](https://github.com/vllm-project/vllm/pull/47810)
- **作者**: tahsintunan  **时间**: 2026-07-07 11:31 CST
- **标签**: rust
- **摘要**: ### Summary  - The Rust frontend has no OpenTelemetry tracing and rejects `--otlp-traces-endpoint` at startup. This is the exporter-bootstrap step of RFC #44757: it stands up the OTLP trace provider so a later PR can emit the `llm_request` span. No span is emitted yet. - Graduates `--otlp-traces-end…

### #47809 — [[PostTraining] Add artifact transfer connector for rollout artifacts](https://github.com/vllm-project/vllm/pull/47809)
- **作者**: aoshen02  **时间**: 2026-07-07 11:28 CST
- **标签**: frontend, needs-rebase, ci/build, v1
- **摘要**: ## Summary - Add artifact transfer configuration and schema support. - Add artifact transfer connector implementations, including a transfer-queue connector.

### #47808 — [[WIP][Spec Decode] DSpark capacity reallocation without sampler padding](https://github.com/vllm-project/vllm/pull/47808)
- **作者**: LucasWilkinson  **时间**: 2026-07-07 11:21 CST
- **标签**: speculative-decoding, v1, qwen, nvidia
- **摘要**: ## Status  Draft/WIP. This branch is purely vibecoded and needs further iteration before it should be considered ready for review or merge. A human should line-review and be able to defend every changed line before this leaves draft.  This replaces #47694, which was accidentally opened from `vllm-pr…

### #47807 — [refactor: streamline DeepSeek V4 mHC warmup and remove token-size cap](https://github.com/vllm-project/vllm/pull/47807)
- **作者**: leihuang-sketch  **时间**: 2026-07-07 11:18 CST
- **标签**: deepseek
- **摘要**: ## Purpose DeepSeek-V4 inference suffers from multi-second latency spikes caused by TileLang JIT compilation of mHC (multi-head-compression) kernels when the scheduler encounters token sizes that were not warmed up. The existing warmup only covers a fixed set of power-of-two token sizes up to 16,384…

### #47806 — [[Core][Hardware][NVIDIA] Add custom all-reduce suspend hooks](https://github.com/vllm-project/vllm/pull/47806)
- **作者**: galletas1712  **时间**: 2026-07-07 11:13 CST
- **标签**: nvidia
- **摘要**: ## Purpose  Add backend-owned suspend/resume lifecycle hooks for vLLM custom all-reduce:  - `CustomAllreduce.prepare_for_suspend()` collectively quiesces the TP group and closes every imported CUDA IPC peer mapping. - `CustomAllreduce.reinit_after_resume()` exchanges fresh IPC handles, reopens all p…

### #47805 — [[DSv4] Disable packed KV cache layout for FlashInfer backend](https://github.com/vllm-project/vllm/pull/47805)
- **作者**: tlrmchlsmth  **时间**: 2026-07-07 10:57 CST
- **标签**: v1
- **摘要**: Small fix for https://github.com/vllm-project/vllm/issues/47783 by detecting `FLASHINFER_MLA_SPARSE_DSV4` and disabling the packed layout in that case

### #47804 — [[CI][LoRA] Preserve QKV fuser split sizes through LoRA wrappers](https://github.com/vllm-project/vllm/pull/47804)
- **作者**: AndreasKaratzas  **时间**: 2026-07-07 10:30 CST
- **标签**: ready
- **摘要**: This PR fixes the LoRA regression in CI: - https://buildkite.com/vllm/ci/builds/76671/list?jid=019f3973-c928-4da0-997d-33258ddbe1c5&tab=output  The regression was probably introduced by: - #47187   After transformers QKV fuser it rewrites separate `q_proj`, `k_proj`, and `v_proj` calls into one fuse…

### #47803 — [[Bugfix][Spec Decode] Fix SWA block freeing during async scheduling](https://github.com/vllm-project/vllm/pull/47803)
- **作者**: TheEpicDolphin  **时间**: 2026-07-07 10:14 CST
- **标签**: bug, needs-rebase, v1

### #47801 — [[Bugfix][DCP] Cast LSE to fp32 in a2a combine to fix bf16 bitcast crash](https://github.com/vllm-project/vllm/pull/47801)
- **作者**: shawntsai  **时间**: 2026-07-07 10:08 CST
- **标签**: bug, v1
- **摘要**: # [Bugfix][DCP] Cast LSE to fp32 in a2a combine to fix bf16 bitcast crash  ## Purpose  Fixes #47800.  The DCP all-to-all combine backend (`--dcp-comm-backend a2a`) crashes at startup on MLA models whose decode backend returns a non-fp32 softmax LSE:  ``` ValueError: Cannot bitcast data-type of size …
