# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-26 10:11 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📌 Issue 动态
* **文档修补反馈**：指出 MiniMax-M3 中文安装手册存在遗漏，在容器内编译 Rust 前端前，需先执行 `pip install setuptools-rust`。(#14962)

### 🚀 PR 动态
**🔧 新特性**
* **推测解码支持**：支持基于 PCP 的 MTP 和 Eagle3 推测解码（目前为堆叠 Draft PR，依赖于前置 PR 合并）。(#14960)
* **非对称 DCP 支持**：支持 Prefill 和 Decode 之间使用非对称 DCP 大小（如 Prefill=2 / Decode=1），修复了 SFA 复制索引计算的差异。(#14958)
* **融合算子**：[WIP] 添加融合算子支持。(#14957)

**⚡ 性能与重构**
* **MoE 架构重构**：将 MegaMoe 设为唯一的 FusedMC2 路径，移除了旧版运行时能力检查及 `dispatch_ffn_combine` 回退逻辑（因 CANN 主分支已全面支持 MegaMoe）。(#14963)
* **MoE 性能优化**：避免冗余的 MC2 top-k 权重类型转换，减少无谓开销。(#14961)

**🐛 Bug 修复**
* **MRV2 稳定性**：稳定 PCP full-decode 图填充逻辑（依赖前置 PR）。(#14959)
* **DeepSeek V4 修复**：修复 DeepSeek V4 MTP V2 图投影形状错误，解决 Ascend 非量化 GEMM 伪造实现修改形状的问题。(#14954)
* **KV Pool 修复**：修复混合层式索引器缓冲区分配问题，优化 GVA 层式复用下的内存分配逻辑。(#14953)

**🛠️ CI 与工程化**
* **CI 流程优化**：将 main2main 流程从 A2 GPU runner 迁移至纯 CPU runner，并将 E2E 测试分发至 A2/A3/310P runner，移除了 NPU 特有步骤。(#14956)
* **环境变量配置**：为非交互式入口点配置环境变量，修复了在 Ascend 910B 上部署 TTS 工作负载时的环境问题。(#14955)

### 📦 Release 动态
* 近期无新版本发布。

---

## 🐛 Issues

### #14962 — [[Doc]: Feedback for `/zh-cn/latest/tutorials/models/MiniMax-M3.html`](https://github.com/vllm-project/vllm-ascend/issues/14962)
- **作者**: libotao514  **时间**: 2026-08-26 10:04 CST
- **标签**: documentation, minimax, gqa-model
- **摘要**: ### 📚 The doc issue  https://docs.vllm.ai/projects/ascend/zh-cn/latest/tutorials/models/MiniMax-M3.html#41-docker vLLM Ascend (中文) MiniMax-M3 安装手册的步骤3，编译 Rust 前端，需要在容器内执行 pip install setuptools-rust ./build_rust.sh 这在不联网环境下依赖无法正常下载，导致编译rust前端失败。   ### Suggest a potential alternative/fix  将镜像内的依赖配置完整…

## 🔀 Pull Requests

### #14963 — [[Refactor][MoE] Make MegaMoe the only FusedMC2 path](https://github.com/vllm-project/vllm-ascend/pull/14963)
- **作者**: willi4869  **时间**: 2026-08-26 10:06 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  CANN now provides the MegaMoe operator for the versions supported by the main branch, so the legacy runtime capability check and `dispatch_ffn_combine` fallback are no longer needed.  This PR:  * Removes `_MEGA_MOE_SUPPORTED` and the `cann_ops_transformer` av…

### #14961 — [[Perf][MoE] Avoid redundant MC2 top-k weight casts](https://github.com/vllm-project/vllm-ascend/pull/14961)
- **作者**: dragondream-chen  **时间**: 2026-08-26 09:15 CST
- **标签**: module:tests, module:quantization
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/d02df748bf9efd99022f1a062597dc3cb3808485

### #14960 — [[Feature][MRV2] Support MTP and Eagle3 speculative decoding with PCP](https://github.com/vllm-project/vllm-ascend/pull/14960)
- **作者**: wzx0726  **时间**: 2026-08-26 01:32 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  > [!IMPORTANT] > This is a stacked Draft PR that depends on #14026, #14878, and #14959. Until those PRs are merged, GitHub's `main`-based diff also contains prerequisite commits. This branch will be rebased after the dependencies merge.  This PR enables MTP a…

### #14959 — [[BugFix][MRV2] Stabilize PCP full-decode graph padding](https://github.com/vllm-project/vllm-ascend/pull/14959)
- **作者**: wzx0726  **时间**: 2026-08-26 01:31 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  > [!IMPORTANT] > This is a stacked Draft PR that depends on #14026. Until #14026 is merged, GitHub's `main`-based diff also contains prerequisite commits. Please review the logical delta after #14026; this branch will be rebased after the dependency merges.  …

### #14958 — [[Feature][P/D] Support unsymmetrical DCP between prefill and decode](https://github.com/vllm-project/vllm-ascend/pull/14958)
- **作者**: lsjfy-open-com  **时间**: 2026-08-26 01:23 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? v0.26.0rc already supports unequal DCP sizes between prefill and decode (e.g. Prefill DCP=2 / Decode DCP=1). This PR fixes a different gap: **SFA replicated-indexer KV transfer was gated only on the local decode DCP switch**.  `enable_sfa_dcp_replicated_indexe…

### #14957 — [[FEAT] [WIP] add fused op](https://github.com/vllm-project/vllm-ascend/pull/14957)
- **作者**: Bourn3z  **时间**: 2026-08-26 01:03 CST
- **标签**: module:core
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/ba07e4a48fc951300d97eb506217dd530583dea3

### #14956 — [main2main: run flow on CPU runner, dispatch E2E to A2/A3/310P runners](https://github.com/vllm-project/vllm-ascend/pull/14956)
- **作者**: wjunLu  **时间**: 2026-08-26 00:31 CST
- **标签**: ci/build
- **摘要**: ## Summary - **schedule_main2main.yaml**: move the main2main flow from the A2 GPU runner to a pure CPU runner (`linux-amd64-cpu-8-hk`); remove NPU-specific steps (npu-smi check, Mooncake wheel, triton, device install) now owned by the E2E workflows; add the "Pre-start E2E runner envs" step which dis…

### #14955 — [[CI][Misc] Configure environment variables for non-interactive entrypoints](https://github.com/vllm-project/vllm-ascend/pull/14955)
- **作者**: yadongtan  **时间**: 2026-08-26 00:23 CST
- **摘要**: ### What this PR does / why we need it?  We first encountered this issue while deploying a TTS workload using a `vllm-omni:v0.26`-based image on an Ascend 910B host.  The environment was configured correctly when entering the container through an interactive Bash shell, but processes launched direct…

### #14954 — [[BugFix][Model] Fix DeepSeek V4 MTP V2 graph projection shape](https://github.com/vllm-project/vllm-ascend/pull/14954)
- **作者**: jiajinzhu2  **时间**: 2026-08-25 23:19 CST
- **摘要**: ### What this PR does / why we need it?  DeepSeek V4 MTP passes `previous_hidden_states` to `h_proj` with shape `[num_tokens, hc_mult, hidden_size]`. The Ascend unquantized GEMM fake implementation models the output as a 2-D tensor using only `x.shape[0]`, so MRV2 graph tracing loses the `hc_mult` d…

### #14953 — [[BugFix][KV Pool] Fix mixed layerwise indexer buffer allocation](https://github.com/vllm-project/vllm-ascend/pull/14953)
- **作者**: ader47  **时间**: 2026-08-25 22:46 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This PR fixes mixed LI C8 and unquantized SFA indexer cache allocation for GVA layerwise reuse.  - Share one raw indexer buffer per layerwise slot only when layer reuse is actually applied. - Preserve independent per-layer allocation when the concrete KV cach…
