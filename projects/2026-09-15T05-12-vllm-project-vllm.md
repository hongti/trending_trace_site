# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-15 13:12 CST

## AI 总结

# vLLM 仓库动态摘要（2026-09-15）

---

## 📋 Issue（5 条）

1. **#56951 / #56948 — CSOAI 治理互操作性**
   - 由 CSOAI-ORG 提交，涉及 vLLM 与外部治理框架（gspc-harness / MCP server）的互操作，包括高吞吐量治理和推理治理，属功能提议性质。

2. **#56949 — Dense DP 权重传输选择错误 IPC 载荷**
   - 在稠密数据并行权重传输中，完整的 `update_info` 列表被发送到每个 EngineCore，导致各 GPU worker 选取了错误的本地载荷。

3. **#56945 — ROCm 镜像未设置 `VLLM_ROCM_USE_AITER`**
   - 导致 Qwen3.5-122B-A10B-FP8 在 MI300X 上回退到 Triton MoE + ROCM_ATTN，短 prompt 慢 1.7×、12k 长度慢 3.7×，应使用 AITER + 统一注意力。

4. **#56943 — `EngineDeadError` 堆栈帧累积、日志洪水**
   - Worker 崩溃后，每个在途流式请求都打印完整 traceback，导致日志爆炸。

---

## 🔀 Pull Request（9 条）

| PR | 标签 | 摘要 |
|---|---|---|
| #56952 | Bugfix / Kimi K3 | 将 DSpark draft 从目标 MLA KV group 中拆分，修复 `MLAAttentionSpec.merge` 中 `non_causal_multi_token_decode` 标志 OR 运算导致的错误 |
| #56950 | Bugfix / Dense DP | 使用 DP index 进行 rank-local 权重更新，修复独立重配置的 dense DP engine 将 `data_parallel_rank` 重置为 0 的问题（对应 #56949） |
| #56947 | Bugfix / Core | 为每个请求抛出**新**的 `EngineDeadError` 实例，而非复用共享异常，避免堆栈帧累积与日志洪水（对应 #56943） |
| #56946 | Bugfix / HiSparse / NIXL | 对未对齐的 host-import 尾部进行本地预填充重算，修复 HiSparse resident manager 与 NIXL pull 广播全 prompt 的不兼容 |
| #56944 | ROCm / Perf | 将 DSA prefill indexer 行分片到 TP ranks 上，提升 ROCm 推理性能 |
| #56942 | ROCm / DSpark | 为 DSpark 启用基于置信度的自适应调度器（adaptive scheduler），继承并 rebase #52362 |
| #56941 | CI | 移除 OTel GPU 采样器的 root 权限，修复 Buildkite CI 中以 root 运行导致的 CDI/MIG 路径问题 |
| #56939 | KV Connector / Mooncake | 通过 Prometheus 暴露 Mooncake 传输统计指标，覆盖 `build_prom_metrics` 方法 |
| #56938 | CI | 对 CPU 镜像构建中截断的 Git/HTTP 网络流进行重试，覆盖 GitHub clone 断连和 LLVM 下载中断 |
| #56937 | Refactor / Multimodal | 重构并统一多模态前缀范围（prefix range）计算逻辑 |

**关键修复亮点：**
- **Kimi K3 投机解码**：修复 DSpark draft 与 target 的 MLA 合并冲突
- **Dense DP 权重传输**：修复 rank 选取错误的根因
- **日志洪水**：从根本上消除异常实例复用导致的堆栈累积
- **ROCm 性能**：DSA 分片 + AITER 适配持续优化 MI300X 等卡

---

## 🚀 Release

本次时间段内 **无新版本发布**。

---

## 🐛 Issues

### #56951 — [CSOAI — vLLM interop: high-throughput governance](https://github.com/vllm-project/vllm/issues/56951)
- **作者**: CSOAI-ORG  **时间**: 2026-09-15 12:48 CST
- **摘要**: https://github.com/CSOAI-ORG/gspc-harness | MCP server | x402 | nicholas@csoai.org

### #56949 — [[Bug]: Dense DP weight transfer selects the wrong IPC payload](https://github.com/vllm-project/vllm/issues/56949)
- **作者**: lxy-alexander  **时间**: 2026-09-15 12:45 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information...   ==============================         System Info ============================== OS                           : Rocky Linux 9.8 (Blue Onyx) (x…

### #56948 — [CSOAI — vLLM interop: inference governance](https://github.com/vllm-project/vllm/issues/56948)
- **作者**: CSOAI-ORG  **时间**: 2026-09-15 12:45 CST
- **摘要**: https://github.com/CSOAI-ORG/gspc-harness | 22 axes | pip: gspc-harness | nicholas@csoai.org

### #56945 — [[Bug]: rocm/vllm image never sets VLLM_ROCM_USE_AITER — Qwen3.5-122B-A10B-FP8 runs Triton MoE + ROCM_ATTN, 1.7x slower at short prompts and 3.7x at 12k than AITER + ROCM_AITER_UNIFIED_ATTN (MI300X/MI325X/MI355X)](https://github.com/vllm-project/vllm/issues/56945)
- **作者**: jolpindo96  **时间**: 2026-09-15 12:22 CST
- **标签**: rocm
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code> — <code>rocm/vllm:rocm10.0.0_ubuntu24.04_py3.14_pytorch_2.12.0_vllm_0.27.0</code>, unmodified, on an MI300X</summary>  ```text Collecting environment information... ==============================       …

### #56943 — [[Bug]: Per-request EngineDeadError tracebacks accumulate stack frames and flood logs after a worker crash](https://github.com/vllm-project/vllm/issues/56943)
- **作者**: linnea-lin-00638949  **时间**: 2026-09-15 12:07 CST
- **摘要**: ### 🐛 Describe the bug  When an engine-core worker dies during serving, every in-flight streaming request logs a full `EngineDeadError` traceback via `logger.exception("Error in chat completion stream generator.")` (`vllm/entrypoints/openai/chat_completion/serving.py`, line 908 on current main). Wor…

## 🔀 Pull Requests

### #56952 — [[Bugfix][Kimi K3] Split DSpark draft out of the target MLA KV group](https://github.com/vllm-project/vllm/pull/56952)
- **作者**: lucifer1004  **时间**: 2026-09-15 12:58 CST
- **标签**: bug, dflash, kimi, k3
- **摘要**: ## What  The K3-native DSpark draft marks its MLA layers `non_causal_multi_token_decode=True`. `MLAAttentionSpec.merge` ORs that flag, so when the draft's spec merges with the target's identically-shaped MLA spec, the causal target group inherits it too. That raises the group's TritonMLA reorder thr…

### #56950 — [[Bugfix] Use DP index for rank-local weight updates](https://github.com/vllm-project/vllm/pull/56950)
- **作者**: lxy-alexander  **时间**: 2026-09-15 12:45 CST
- **标签**: bug
- **摘要**: ## Purpose  Fix #56949  This change is about the wrong rank-local payload selection in vLLM dense DP weight transfer. Because an independently reconfigured dense DP engine resets data_parallel_rank to 0 but keeps its original DP identity in data_parallel_index, using data_parallel_rank makes all DP …

### #56947 — [[Bugfix][Core] Raise a fresh EngineDeadError per request instead of re-raising the shared instance](https://github.com/vllm-project/vllm/pull/56947)
- **作者**: linnea-lin-00638949  **时间**: 2026-09-15 12:42 CST
- **标签**: bug
- **摘要**: ### Purpose  Fixes #56943.  When the engine dies, `AsyncLLM.output_handler` logs the root cause once and then `OutputProcessor.propagate_error()` hands the **same** exception instance to every in-flight request's `RequestOutputCollector`. Each consumer's `raise output` prepends its own frames onto t…

### #56946 — [[Bugfix][HiSparse][NIXL] Recompute unaligned host-import tails locally](https://github.com/vllm-project/vllm/pull/56946)
- **作者**: LucasWilkinson  **时间**: 2026-09-15 12:24 CST
- **标签**: bug, kv-connector, kv-cache-manager
- **摘要**: # [Bugfix][HiSparse][NIXL] Leave unaligned host-import tails for local prefill  ## Problem  NIXL pull advertises the whole prompt as externally computed. The host-only HiSparse resident manager requires imported prefixes to end on a cache-block boundary. A 32,019-token prompt with block size 64 ther…

### #56944 — [[ROCm][Perf] Shard DSA prefill indexer rows across TP ranks](https://github.com/vllm-project/vllm/pull/56944)
- **作者**: sumin-hong  **时间**: 2026-09-15 12:16 CST
- **标签**: rocm
- **摘要**: > Draft rebased onto upstream main `a72c046463cecc35737f353b2b6cc0f44bad3f29`. The GPU/model results below are previously reported measurements on base `8c34a372323b3becc92c183257d195562882ef1a`; they have not been rerun on this rebased head.  ## Purpose  This PR splits the DSA sparse-attention inde…

### #56942 — [[ROCm][DSv4] Enable confidence-based adaptive scheduler for DSpark](https://github.com/vllm-project/vllm/pull/56942)
- **作者**: larryli2-amd  **时间**: 2026-09-15 12:03 CST
- **标签**: rocm, deepseek, DSv4, dflash
- **摘要**: ## Inherits [#52362](https://github.com/vllm-project/vllm/pull/52362)  This work inherits and rebases the ROCm DSpark adaptive-verification implementation from #52362. It is not an independent implementation of the feature.  This branch should only be submitted after explicit coordination with the a…

### #56941 — [[CI] Drop root privileges for the OTel GPU sampler](https://github.com/vllm-project/vllm/pull/56941)
- **作者**: khluu  **时间**: 2026-09-15 12:02 CST
- **标签**: ci/build
- **摘要**: ## Why  Buildkite #88719 H200 CUDAGraph job `01a09ded-1725-4be8-ba4e-e3781183453f` ran the background `ci_gpu.py` OTel sampler as root. At that revision (`df31f348e4e0`), its CDI/MIG path imported `vllm.third_party.pynvml` through the package and left 34 root-owned bytecode/cache entries in the moun…

### #56939 — [[KV Connector][Mooncake] Expose transfer stats via Prometheus](https://github.com/vllm-project/vllm/pull/56939)
- **作者**: CAICAIIs  **时间**: 2026-09-15 11:40 CST
- **标签**: documentation, kv-connector
- **摘要**: ## Purpose  `MooncakeConnector` collects per-transfer stats but never exports them: it does not override `KVConnectorBase_V1.build_prom_metrics`, which returns `None`, so the stats only reach the periodic `KV Transfer metrics: ...` log line. `mooncake/stats.py` carried a `TODO(mooncake-stats)` for e…

### #56938 — [[CI] Retry CPU image builds after truncated network streams](https://github.com/vllm-project/vllm/pull/56938)
- **作者**: khluu  **时间**: 2026-09-15 11:34 CST
- **标签**: ci/build, cpu
- **摘要**: ## Summary  - classify truncated Git/HTTP stream failures as transient in the CPU image build retry loop - cover GitHub clone disconnect/early-EOF/index-pack failures and interrupted LLVM downloads - preserve the existing four-attempt limit and 10/20/40-second backoff  ## Why  Main builds [#88996](h…

### #56937 — [[Refactor][Multimodal] Refactor and unify mm prefix range computation](https://github.com/vllm-project/vllm/pull/56937)
- **作者**: Isotr0py  **时间**: 2026-09-15 11:30 CST
- **标签**: mrv2
- **摘要**: ## Purpose  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such as providing test command. - [ ] The …
