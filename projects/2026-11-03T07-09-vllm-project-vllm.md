# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-03 15:09 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🚀 Release
**v0.33.0 版本发布**
* **版本亮点**：本版本包含了来自 389 位贡献者（其中 122 位为新增贡献者）的 927 次提交。
* **新模型支持**：引入了 Reflection Beam-1.0 模型（501B 参数 MoE 架构，23B 激活参数）。

### 💻 Pull Requests (PR)
本期 PR 涵盖了性能优化、新特性、跨平台修复及 CI 升级，重点如下：

**新特性与性能优化**
* **[ROCm][Perf] W4A16 性能调优** (#65322)：针对 gfx115x 等硬件重新调整了 int4 skinny GEMM 的解码调度规则，提升了批次 1-5 的处理性能。
* **[Feature][MoE] 共享路由专家行恢复器** (#65321)：允许 MoE 后端为整个模型捕获过程提供单一的路由专家行恢复回调，优化了缓存管理。
* **[Core][KV Connector] 异步 KV 加载优化** (#65319)：允许在运行批次满载时启动受限的异步 KV 加载（通过 `--max-num-inflight-kv-loads` 控制），使释放的插槽能立即用于新请求，减少等待时间。

**Bug 修复**
* **[CPU] macOS 加载失败修复** (#65320)：修复了 macOS arm64 平台上因缺少 C++ 因果卷积内核导致 Gated DeltaNet（如 Qwen3.5）bf16 模型无法加载的问题，改用 PyTorch GDN 路径。
* **[MRV2] MoE 工作空间分配** (#65318)：在图捕获前为 Model Runner V2 预留 MoE 工作空间，修复了默认路径下的运行错误。
* **[Metrics] 抢占统计修正** (#65317)：将请求抢占次数的统计点移至调度器内部，防止因请求在恢复前被中止导致统计遗漏。
* **[Frontend] 工具语法顺序修复** (#65316)：修复了严格工具语法中必选属性必须前置的问题，避免模型生成可选参数后无法再生成必选参数。
* **[CPU] 支持 INT8 WNA16 MoE** (#65314)：为 CPU 端补充了分组 INT8 仅权重层的专家实现，修复了混合 INT4/INT8 压缩检查点加载失败的问题。

**CI/构建与同步**
* **[CI/Build] 默认 Python 版本升级** (#65313)：将所有 Docker 镜像和 CI 环境的默认 Python 版本从 3.12 升级至 3.14。
* **[Sync] 分支同步** (#65315)：将 `main` 分支的最新代码合并至 `rl/aux-output` 分支。

*(注：本期数据中未包含 Issue 动态)*

---

## 🔀 Pull Requests

### #65322 — [[ROCm][Perf] W4A16: retune int4 skinny GEMM decode tuples for gfx115x](https://github.com/vllm-project/vllm/pull/65322)
- **作者**: roberteg16  **时间**: 2026-11-03 14:55 CST
- **标签**: rocm
- **摘要**: ## Summary  Retunes the `wvSplitK_int4_g` dispatch for batch 1–5 on gfx1151 and gfx1150; gfx1152/gfx1153 reuse the gfx1150 rules (not measured there). Rules apply to grouped weights whose activations fit LDS; every other case and every other target keep their current tuple.  | Target | Batch | Appli…

### #65321 — [[Feature][MoE] Allow a shared routed-expert row restorer](https://github.com/vllm-project/vllm/pull/65321)
- **作者**: Liamup777  **时间**: 2026-11-03 14:51 CST
- **标签**: documentation
- **摘要**: ## Overview  Allow a MoE backend to supply one routed-expert row-restoration callback for the whole captured model. The capturer defaults to the existing recovery algorithm and keeps ownership of buffer writes.  Draft alternative for #61539, compared with #64689 and #62441. These are competing desig…

### #65320 — [[Bugfix][CPU] Use the PyTorch GDN conv path on macOS](https://github.com/vllm-project/vllm/pull/65320)
- **作者**: MeKaustubh07  **时间**: 2026-11-03 14:40 CST
- **标签**: bug, cpu
- **摘要**: ## Overview  On macOS arm64 CPU, Gated DeltaNet models such as Qwen3.5 fail to load in bf16 since #54093. The C++ causal conv1d kernels are not built on Apple, but the Python checks still select them. This makes the checks look at the compiled ops, so Apple uses the PyTorch conv path again, as it di…

### #65319 — [[Core][KV Connector] Start capped async KV loads while the batch is full](https://github.com/vllm-project/vllm/pull/65319)
- **作者**: njhill  **时间**: 2026-11-03 14:39 CST
- **标签**: kv-connector, scheduler
- **摘要**: ## Overview  Lets async KV loads (e.g. P/D NIXL pulls on a decode instance) start while the running batch is full, capped by a new `--max-num-inflight-kv-loads`, so a freed slot can go straight to a request whose KV has already arrived. Alternative to #64628 and #61725, which address the same proble…

### #65318 — [[Bugfix][MRV2] Reserve MoE workspace before graph capture](https://github.com/vllm-project/vllm/pull/65318)
- **作者**: RyanMa29  **时间**: 2026-11-03 14:38 CST
- **标签**: bug, mrv2
- **摘要**: ## Overview  Follow up on #48179 by reserving MoE workspace in the default Model Runner V2 path.   As the pooling models use MRV2 by default (#48290), existing fix in #48179 cannot resolve the issue although the idea still works. So this PR includes the rebased changes from #48179 and adds MRV2 inte…

### #65317 — [[Bugfix][Metrics] Count preemptions in the scheduler, not on resume](https://github.com/vllm-project/vllm/pull/65317)
- **作者**: shiv4289  **时间**: 2026-11-03 14:27 CST
- **标签**: bug, scheduler
- **摘要**: ## Overview  `vllm:num_preemptions` is counted from the request's `PREEMPTED` event, which only reaches the frontend with that request's next output. So a request that is preempted and then aborted before it resumes is never counted, and the rest are counted late. This counts preemptions in the sche…

### #65316 — [[Bugfix][Frontend] Put required properties first in strict tool grammars](https://github.com/vllm-project/vllm/pull/65316)
- **作者**: Shawnsuun  **时间**: 2026-11-03 14:16 CST
- **标签**: bug, documentation, tool-calling
- **摘要**: ## Overview  Fixes #65173. Strict tool grammars pin object properties to schema declaration order, so an optional parameter declared before a required one cannot be generated after the model has emitted the required one. This PR reorders each object schema's `properties` so required properties come …

### #65315 — [[Sync] Merge main into rl/aux-output](https://github.com/vllm-project/vllm/pull/65315)
- **作者**: Ronald1995  **时间**: 2026-11-03 14:13 CST
- **标签**: documentation, performance, rocm, frontend, intel-gpu, speculative-decoding, torch.compile, ci/build, qwen, deepseek, kv-connector, nvidia, quantization, mrv2, DSv4, rust, kimi, k3, kv-cache-manager, minimax, DSv4.1, mooncake, pooling, nixl
- **摘要**: Syncs `rl/aux-output` with the latest remote `main` (`9a519c0585`) so the branch is no longer behind.  - Two merge commits bring in the 15 `main` commits the branch was missing (`40941f2c4b` first sync, then `9a519c0585` for the 3 newer ones). - Preserves the branch-only design docs commit: `docs/de…

### #65314 — [[Bugfix][CPU]-Support-grouped-INT8-WNA16-MoE](https://github.com/vllm-project/vllm/pull/65314)
- **作者**: sudhanshu112233shukla  **时间**: 2026-11-03 14:10 CST
- **标签**: bug, cpu, quantization
- **摘要**: ## Motivation  Fixes #65070. CPU WNA16 MoE supported grouped INT4 weights but had no expert implementation for grouped INT8 weight-only layers, so mixed INT4/INT8 compressed-tensors checkpoints failed during backend selection.  ## Modifications  - Add a CPU grouped-INT8 WNA16 fallback that dequantiz…

### #65313 — [[CI/Build] Make Python 3.14 the default for images and CI](https://github.com/vllm-project/vllm/pull/65313)
- **作者**: njhill  **时间**: 2026-11-03 14:09 CST
- **标签**: documentation, rocm, intel-gpu, ci/build, cpu, agents, mooncake
- **摘要**: ## Overview  Make Python 3.14 the default: every Docker image and the CI that runs in them move from 3.12 to 3.14, the test lock files are resolved for 3.14, and the nightly smoke test that covered 3.14 now covers 3.12. Draft until Mooncake and LMCache publish 3.14 wheels (see Blockers). Follows #34…

## 🚀 Releases

### [v0.33.0](https://github.com/vllm-project/vllm/releases/tag/v0.33.0rc3)
- **作者**: khluu  **时间**: 2026-11-02 18:10 CST
- **摘要**: # v0.33.0  ## Highlights  This release features 927 commits from 389 contributors (122 new)!  * **New models**: Reflection Beam-1.0 (501B MoE, 23B active) with its DFlash2 drafter (#62074), Yandex AliceAI-Foundation-80B-A3B (#57963), Recursive Flint (40B MoE, 2.5B active) (#62938), Muse Spark Open (…
