# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-20 10:52 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 📌 Issues (2 项)
1. **量化检查点加载崩溃 (#49137)**：auto_round (INC) 路径硬编码了 `lm_head_quantized=False`，导致通过 Intel auto_round 合法生成的、包含量化 `lm_head` 的检查点（如 Qwen 系列微调模型）在 vLLM 中崩溃。
2. **延迟统计直方图数值异常 (#49135)**：当请求在未达到某个统计里程碑前就已完成时，`IterationStats` 通过直接相减原始时间戳来计算延迟，缺乏防护逻辑，导致延迟直方图出现相当于主机运行时长级别的异常大值。

---

### 🛠 Pull Requests (10 项)
**核心与 API 修复**
- **修复延迟统计异常 (#49136)**：针对 Issue #49135，为未到达里程碑的请求增加防护，避免计算出无意义的延迟区间。
- **拒绝矛盾的自定义操作指令 (#49134)**：修复 `CompilationConfig.custom_ops` 可同时启用和禁用同一操作的漏洞，将矛盾指令的报错从运行期 `assert` 提前到解析阶段直接拒绝。
- **修复 Responses API 空内容引发的 500 错误 (#49130)**：处理客户端重放先前助手轮次时内容为空或为拒绝（refusal）的情况，防止 HTTP 500 崩溃。

**推测解码与量化修复**
- **修复 DSpark 草稿模型量化数值损坏 (#49133)**：当目标模型（如 NVFP4 DeepSeek-V4）与草稿模型量化配置不一致（如 MXFP4）时，会导致草稿模型静默数值损坏。此 PR 让 DSpark 使用自身的模型/量化配置构建草稿模型。
- **支持经典混合草稿模型 RFC (#49138)**：提出 RFC（草案），计划支持 LFM2/LFM2.5 short_conv 等经典混合草稿模型。目前叠加在其他未合并 PR 之上，仅供讨论，尚不可用。

**内核与底层组件修复**
- **修复 Persistent Top-K 内核状态重用 bug (#49139)**：修复 `persistent_topk` 在处理完长行（seq_len > 32768）后紧接处理短/中等行时，直方图错误重用导致的正确性问题。
- **修复 KVConnector 资源泄漏 (#49132)**：修复 `ExampleHiddenStatesConnector` 中异步写入基础设施未正确释放 `ThreadPoolExecutor` 及文件描述符（锁文件）的问题。

**性能优化**
- **SM80 批次不变矩阵乘法性能提升 (#49131)**：为 SM80（Ampere 架构）上的 batch-invariant 模式引入形状自适应的 tile 配置，优化 `mm/addmm/matmul` 性能。

**CI 与基础设施修复（ROCm/AMD）**
- **隔离 NIXL 并发引擎内部端口 (#49129)**：修复 AMD CI 中第四个 NIXL 配置启动失败的问题，隔离并发预填充引擎的端口。
- **修复 ROCm CI 稀疏 MLA 测试 (#49128)**：修复 AMD CI 中稀疏 MLA 元数据同步 fixture 报错 `_num_compute_units` 属性缺失的问题。

---

### 🚀 Releases
- 本次追踪周期内**无新版本发布**。

---

## 🐛 Issues

### #49137 — [[Bug]: auto_round (INC) path hardcodes lm_head_quantized=False, crashing on checkpoints with quantized lm_head that auto-round legally produces](https://github.com/vllm-project/vllm/issues/49137)
- **作者**: noonghunna  **时间**: 2026-07-20 09:48 CST
- **摘要**: ### Your current environment  - vLLM v0.25.1 (docker `vllm/vllm-openai:v0.25.1`), 2× RTX 3090 (Ampere, TP=2) - Model: Qwen3.5/3.6-family (Tess-4-27B finetune), quantized with Intel auto-round 0.14.2, `format="auto_round"`  ### 🐛 Describe the bug  The auto_round (INC) quantization path **hardcodes `l…

### #49135 — [[Bug]: Latency histograms get host-uptime-sized values when a request finishes before reaching a milestone](https://github.com/vllm-project/vllm/issues/49135)
- **作者**: hyunnnchoi  **时间**: 2026-07-20 09:21 CST
- **摘要**: ### Your current environment  <details> <summary>Environment</summary>  `vllm/collect_env.py` cannot run on this platform — `get_intel_graphics_compiler_version()` calls `get_pkg_version()`, which begins with `assert get_platform() == "linux"` (`vllm/collect_env.py:304`, called from `:371`), so the …

## 🔀 Pull Requests

### #49139 — [[Bugfix][Kernel] Fix persistent top-k histogram reuse after short rows](https://github.com/vllm-project/vllm/pull/49139)
- **作者**: fxfxfxfxfxfxfxfx  **时间**: 2026-07-20 10:07 CST
- **标签**: bug
- **摘要**: <!-- markdownlint-disable -->  ## Purpose  Fix a correctness bug in `persistent_topk` when one persistent CTA group processes a radix row (`seq_len > 32768`), followed by a short or medium row (`seq_len <= 32768`), and then another radix row.  The kernel previously used the outer row iteration count…

### #49138 — [[Spec Decode][RFC] Support classical hybrid draft models (LFM2/LFM2.5 short_conv) — stacked on #44296 + #35062](https://github.com/vllm-project/vllm/pull/49138)
- **作者**: qtris123  **时间**: 2026-07-20 09:51 CST
- **标签**: speculative-decoding, v1
- **摘要**: ## ⚠️ Draft / RFC — do not merge  Opening this as a **draft for visibility and discussion of #49112**, not as a merge-ready change. It is **stacked on two other open PRs** and is **not yet functional end-to-end** (see "Status" below).  ### What this enables Speculative decoding with a **classical, s…

### #49136 — [[Bugfix] Avoid nonsense latency intervals for unreached milestones](https://github.com/vllm-project/vllm/pull/49136)
- **作者**: hyunnnchoi  **时间**: 2026-07-20 09:22 CST
- **标签**: bug, v1
- **摘要**: ## Purpose  Fixes #49135.  `IterationStats.update_from_finished_request()` and `OutputProcessor.do_tracing()` both compute latency intervals by subtracting raw engine-core timestamps, with no guard for a milestone that was never reached:  ```python queued_time = req_stats.scheduled_ts - req_stats.qu…

### #49134 — [[Bugfix] Reject contradictory custom-op directives](https://github.com/vllm-project/vllm/pull/49134)
- **作者**: taneem-ibrahim  **时间**: 2026-07-20 09:21 CST
- **标签**: bug
- **摘要**: ## Purpose  `CompilationConfig.custom_ops` accepted an operation that was both explicitly enabled and disabled, deferring the conflict to a runtime `assert` in `CustomOp.enabled()`. The existing `all`/`none` conflict check was also an `assert`; under `python -O`, both checks disappeared and the inva…

### #49133 — [[Bugfix][Spec Decode] DSpark: build draft under its own model/quant config (NVFP4 target corrupts MXFP4 draft experts)](https://github.com/vllm-project/vllm/pull/49133)
- **作者**: h-guo18  **时间**: 2026-07-20 09:14 CST
- **标签**: bug, v1
- **摘要**: ## Purpose  Fix silent numerical corruption of the DSpark drafter when the target model is quantized differently from the draft checkpoint — most visibly, a modelopt-NVFP4 DeepSeek-V4 target (`nvidia/DeepSeek-V4-{Flash,Pro}-NVFP4`) paired with the official DSpark drafter (`deepseek-ai/DeepSeek-V4-{F…

### #49132 — [[Bugfix][KVConnector] Release executor and lock fds in ExampleHiddenS…](https://github.com/vllm-project/vllm/pull/49132)
- **作者**: Henry-Cheng  **时间**: 2026-07-20 09:10 CST
- **标签**: bug, v1, kv-connector
- **摘要**: *Purpose*  `ExampleHiddenStatesConnector` spins up worker-side async-write infrastructure in `__init__` -- a `ThreadPoolExecutor` (`self._executor`) and per-request `.lock` file descriptors held with `LOCK_EX` (`self._lock_fds`) -- but never overrides `KVConnectorBase_V1.shutdown()`. The base implem…

### #49131 — [[Perf] Shape-adaptive tile config for batch-invariant matmul on SM80](https://github.com/vllm-project/vllm/pull/49131)
- **作者**: shivasathishs-rp  **时间**: 2026-07-20 08:54 CST
- **标签**: v1
- **摘要**: ## Purpose  Part of the batch-invariant performance work tracked in #27433 ("Optimize the batch invariant performance").  On SM80 (Ampere), `enable_batch_invariant_mode()` routes every `mm`/`addmm`/`matmul`/ `linear` through the Triton `matmul_persistent` kernel, because cuBLASLt-only determinism is…

### #49130 — [[Bugfix] Handle empty or refusal content when reconstructing Responses input messages](https://github.com/vllm-project/vllm/pull/49130)
- **作者**: TimurRakhmatullin86  **时间**: 2026-07-20 08:31 CST
- **标签**: bug, frontend
- **摘要**: ## Purpose  The Responses API returns an unhandled **HTTP 500** when a client replays a prior assistant turn whose content is a refusal or is empty.  `_construct_message_from_response_item` (`vllm/entrypoints/openai/responses/utils.py`) reads `item.content[0].text` unguarded for a `ResponseOutputMes…

### #49129 — [[CI][NIXL] Isolate concurrent engine internal ports](https://github.com/vllm-project/vllm/pull/49129)
- **作者**: AndreasKaratzas  **时间**: 2026-07-20 08:09 CST
- **标签**: ready, v1, kv-connector
- **摘要**: The [failing AMD CI run](https://buildkite.com/vllm/amd-ci/builds/11018/list?sid=019f799a-f5f8-41e1-acf6-52e248ccd3ac&tab=output) failed while starting the fourth NIXL configuration: concurrent prefiller and decoder EngineCores both selected internal TCPStore port `43991`, producing `EADDRINUSE` and…

### #49128 — [[ROCm][CI] Fix sparse MLA metadata sync fixture](https://github.com/vllm-project/vllm/pull/49128)
- **作者**: AndreasKaratzas  **时间**: 2026-07-20 08:08 CST
- **标签**: rocm, ready
- **摘要**: The [failing AMD CI run](https://buildkite.com/vllm/amd-ci/builds/11018/list?jid=019f799a-f6b0-4579-9197-1c495ef92071&tab=output) raised `AttributeError: _num_compute_units` in the synthetic sparse-MLA builder fixture. `git bisect` from `c233d90aa826df072872df47b201450059be8e71` to `b6ff8a2f509cc7ac…
