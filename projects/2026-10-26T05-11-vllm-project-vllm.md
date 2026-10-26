# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-26 13:11 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要：

### 💡 Issue 动态
近期主要报告了 3 个 Bug 和 1 项性能优化提案：
1. **Rust 前端流式输出缺失字段**：Rust 前端在流式传输聊天块时，省略了 Python 端默认输出为 `null` 的字段（如 `finish_reason`），导致行为不一致（#63656）。
2. **KV Offload 相关缺陷**：
   - **对象键冲突**：OBJ 层的 keys 忽略了 `blocks_per_chunk` 参数，导致不同块大小的运行会错误地共享对象 keys（#63654）。
   - **加载无效数据**：当 offloaded chunks 跨越多个 GPU 块时，`OffloadingConnector` 会为滑动窗口层加载从未写入的 KV 数据（#63653）。
3. **MoE 路由性能优化提案**：提出在 SM90 架构上为 fp32 MoE router weights 增加 M-gated HPC-Ops dual-BF16 router GEMM 层，以提升计算效率（#63650）。

### 🔧 PR 动态
近期的 PR 集中在关键 Bug 修复、硬件生态支持及 CI 优化：
1. **核心 Bug 修复**：
   - **KV Offload 修复**：通过 `blocks_per_chunk` 对对象键进行命名空间隔离，修复键共享问题（#63655）；避免存储带有 null 块的滑动窗口 chunks，修复无效 KV 加载问题（#63652）。
   - **KV Connector (NIXL)**：在 pull 模式下支持 `P_TP > num_kv_heads` 且 `P_TP > D_TP` 的 GQA 模型（如 Llama-3.3-70B）（#63648）。
   - **前后端一致性修复**：Rust 前端现已像 Python 端一样，拒绝 `max_tokens` 超过 `max_model_len` 的请求，而非静默截断（#63649）；将客户端提供的 `remote_prefill_cached_tokens` 限制在提示长度内（#63659）。
2. **硬件与 CI 支持**：
   - **Intel XPU**：在 Intel GPU CI 中启用 Nightly PyTorch 测试（#63647），并在 XPU 上启用 `test_fusion_rmsnorm_quant` 测试（#63658）。
   - **CI 优化**：精简白名单，将 10 个 CPU 安全测试整合到两个专属 CPU 任务中，提升测试效率（#63645）。
3. **文档更新**：对齐并更新了 ROCm Kimi-K3 MI355X AgentX 的调度单元文档（#63657）及 Muse Spark Open 文档（#63651）。

### 🚀 Release 动态
近期暂无新版本发布信息。

---

## 🐛 Issues

### #63656 — [[Bug][Rust Frontend] Streaming chat chunks omit fields that Python emits as null, including finish_reason](https://github.com/vllm-project/vllm/issues/63656)
- **作者**: vito-lin-dev  **时间**: 2026-10-26 12:36 CST
- **标签**: rust
- **摘要**: ### Your current environment  - Rust frontend: `main` at `b3466364b6` (route tests against the mock engine); the relevant Rust files are unchanged on current `main`. - Python frontend: `vllm/vllm-openai:latest` (v0.32.0), `Qwen/Qwen3-0.6B`. The Python code cited below is from current `main` and matc…

### #63654 — [[Bug]: KV offload OBJ tier keys ignore blocks_per_chunk, so runs with different chunk sizes share object keys](https://github.com/vllm-project/vllm/issues/63654)
- **作者**: Vedant-Agarwal  **时间**: 2026-10-26 12:16 CST
- **摘要**: ### Your current environment  Not environment specific: the namespace is computed in Python and the bug reproduces in a CPU-only unit test on main (checked at d7595a767).  ### 🐛 Describe the bug  The OBJ secondary tier builds its object-key namespace without the chunk size. In `tiering/obj/manager.p…

### #63653 — [[Bug]: OffloadingConnector loads never-written KV for sliding-window layers when offloaded chunks span several GPU blocks](https://github.com/vllm-project/vllm/issues/63653)
- **作者**: pepmach  **时间**: 2026-10-26 12:15 CST
- **摘要**: ### Your current environment  <details> <summary>Environment (summary: <code>collect_env.py</code> asserts on macOS)</summary>  ```text macOS 27.0.1, Apple M5 Pro (arm64) vLLM main 24324ef0, built with VLLM_TARGET_DEVICE=cpu Python 3.12.14, torch 2.14.0, transformers 5.20.2 ```  The code involved is…

### #63650 — [[RFC] M-gated HPC-Ops dual-BF16 router GEMM for fp32 MoE router weights on SM90](https://github.com/vllm-project/vllm/issues/63650)
- **作者**: new-TonyWang  **时间**: 2026-10-26 12:05 CST
- **标签**: quantization
- **摘要**: ### 🚀 The feature, motivation and pitch  **RFC: Add an M-gated HPC-Ops dual-BF16 router GEMM tier for fp32 MoE router weights on SM90**  #### Prior work  - #49277 (feature request, stale) and PR #49312 by @lengrongfu (opt-in HPC BF16xFP32 router GEMM, currently stalled). This RFC proposes the same k…

## 🔀 Pull Requests

### #63659 — [[Bugfix] Clamp client-supplied remote_prefill_cached_tokens to the prompt length](https://github.com/vllm-project/vllm/pull/63659)
- **作者**: LeizhenZhang  **时间**: 2026-10-26 13:08 CST
- **标签**: bug
- **摘要**: ## Overview  Clamp the client-supplied `kv_transfer_params.remote_prefill_cached_tokens` override to `[0, prompt_len]` before it becomes `num_cached_tokens`. Fixes #58728.  This is the clamp @ptimizeroracle proposed in #58730, which robertgshaw2-redhat acked on the issue. That PR was closed by its a…

### #63658 — [[XPU][CI] Enable test_fusion_rmsnorm_quant on XPU](https://github.com/vllm-project/vllm/pull/63658)
- **作者**: Yejing-Lai  **时间**: 2026-10-26 12:40 CST
- **标签**: intel-gpu, torch.compile, quantization
- **摘要**: Add test_fusion_rmsnorm_quant support on XPU.  run cmd: pytest -s -v compile/passes --ignore compile/passes/distributed   --- <a href="https://app.blacksmith.sh/vllm-project/codesmith/vllm/pr/63658?autoLogin=true&ref=codesmith_pr_footer"><picture><source media="(prefers-color-scheme: dark)" srcset="…

### #63657 — [[Doc][ROCm] Update Kimi-K3 MI355X AgentX scheduling cells](https://github.com/vllm-project/vllm/pull/63657)
- **作者**: chunfangamd  **时间**: 2026-10-26 12:38 CST
- **标签**: rocm, kimi, k3, recipe
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Align the six c4–c32 MI355X Kimi-K3 AgentX scheduling cells with [InferenceX #4473](https://github.com/SemiAnalysisAI/InferenceX/pull/4473), building on the existing ten-point preset from [#63552](https://github.com/vllm-project/vllm/pull/63552).  ## Claim…

### #63655 — [[Bugfix][KV Offload][OBJ] Namespace object keys by blocks_per_chunk](https://github.com/vllm-project/vllm/pull/63655)
- **作者**: Vedant-Agarwal  **时间**: 2026-10-26 12:16 CST
- **标签**: bug
- **摘要**: ## Overview  Fixes #63654. The OBJ tier built its `FileMapper` without `blocks_per_file`, so its namespace digest ignored `blocks_per_chunk`. This passes `blocks_per_file=offloading_spec.blocks_per_chunk`, as the fs and p2p tiers already do.  ## Claims  - OBJ tiers with different `blocks_per_chunk` …

### #63652 — [[Bugfix][KV Offload] Do not store sliding-window chunks with null blocks](https://github.com/vllm-project/vllm/pull/63652)
- **作者**: pepmach  **时间**: 2026-10-26 12:15 CST
- **标签**: bug, kv-connector
- **摘要**: ## Overview  Fixes #63653.  With offloaded chunks of several GPU blocks, `OffloadingConnector` could store a sliding-window chunk that contained a null block. A later CPU hit then loaded the never-written slot as KV, and the request's output changed with no error.  ## Claims  - A sliding-window chun…

### #63651 — [[Doc] Update Muse Spark Open ](https://github.com/vllm-project/vllm/pull/63651)
- **作者**: jeejeelee  **时间**: 2026-10-26 12:11 CST
- **标签**: documentation, rust
- **摘要**: ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this PR.  - [ ] **Design Fit:** Minimizes…

### #63649 — [[Bugfix][Rust Frontend] Reject max_tokens greater than max_model_len in OpenAI routes like Python](https://github.com/vllm-project/vllm/pull/63649)
- **作者**: vito-lin-dev  **时间**: 2026-10-26 12:00 CST
- **标签**: bug, rust
- **摘要**: ## Overview  The Rust frontend accepts `max_tokens` greater than `max_model_len` on the OpenAI chat and completions routes and silently clamps it, returning 200. The Python frontend rejects these requests with 400. This PR adds the check to those two routes only, so other routes keep matching Python…

### #63648 — [[Bugfix][KV Connector][NIXL] Support P_TP > num_kv_heads with P_TP > D_TP in pull mode](https://github.com/vllm-project/vllm/pull/63648)
- **作者**: jimburtoft  **时间**: 2026-10-26 11:45 CST
- **标签**: bug, kv-connector
- **摘要**: ## Overview  The NIXL pull connector rejects a GQA model whose prefill TP exceeds its KV-head count when decode TP is smaller (for example, Llama-3.3-70B with 8 KV heads at P TP16 → D TP4). This PR supports that case: D reads one P replica per KV head and notifies the replicas it skips.  ## Claims  …

### #63647 — [[XPU][CI] Test nightly pytorch in Intel GPU CI](https://github.com/vllm-project/vllm/pull/63647)
- **作者**: zxd1997066  **时间**: 2026-10-26 11:32 CST
- **标签**: intel-gpu, ci/build
- **摘要**: ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this PR.  - [ ] **Design Fit:** Minimizes…

### #63645 — [[CI] Allowlist-shrink batch 3: wire 10 CPU-safe tests into two focused CPU jobs](https://github.com/vllm-project/vllm/pull/63645)
- **作者**: wjabbour  **时间**: 2026-10-26 11:17 CST
- **标签**: ci/build
- **摘要**: ## Summary - Follow-up to #54867 / #58055 / #59050 (test-tethering check): shrinks the allowlist by 10 more entries by wiring CPU-safe tests into one new CPU job in `misc.yaml`, `cpu-distributed-platforms-cli` (`:computer: (CPU) Distributed, Platforms, CLI`, 20 min). It installs nothing at runtime a…
