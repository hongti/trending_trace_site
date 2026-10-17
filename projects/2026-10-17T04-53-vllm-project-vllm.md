# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-17 12:53 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的简洁摘要：

### 🐛 Issue 动态
近期主要报告了 4 个问题，集中在新特性组合与错误提示上的缺陷：
1. **MXFP8 与 Humming 后端冲突**（#62136）：使用 `mxfp8` 量化并指定 `moe_backend="humming"` 时，权重转换因缺失 `weight_scale` 键而崩溃。
2. **CUTLASS FP8 MoE 错误提示不友好**（#62135）：在线 FP8 搭配 `cutlass` 后端时，仅提示“被禁用”而无具体原因，未像其他后端那样给出具体的拒绝理由。
3. **Ngram GPU 推测解码与编译模式冲突**（#62134）：`ngram_gpu` 推测解码配置与 `compilation_config.mode="NONE"` 同开时，会导致引擎初始化失败。
4. **特性请求：语法约束与 EAGLE 推测解码结合**（#62130）：希望解决当前 vLLM 中语法约束采样与 EAGLE 推测解码无法有效组合使用的痛点。

### 🔧 Pull Request 动态
PR 主要聚焦于新硬件 Kernel 适配、性能优化及多项关键 Bug 修复：
**新特性与性能优化：**
- **新模型 Kernel 支持**：为 Step-5 Preview 和 Muse Spark Open 添加了融合 Norm 算子（#62132）和 FP32 Router GEMM 支持（#62126）。
- **ROCm 性能优化**：在 DSv4.1-Flash 单解码路径中，将 DSpark draft 层切换至 mono K1/K2 kernel 运行以提升性能（#62131）。
- **INT8 GEMM 性能修复**：针对 SM89 (Ada) 架构，修复了 int8 W8A16 CUTLASS GEMM 在特定矩阵尺寸下性能骤降的问题，改用 16 行 tile 提升 ~1.5x 速度（#62128）。

**关键 Bug 修复：**
- **死锁与崩溃修复**：修复流水线并行下 MRV2 推测解码预热时的死锁问题（#62129）；修复 SM110 上 OAI Triton MXFP4 MoE 的运行错误（#62133）。
- **精度溢出修复**：修复 GDN chunk/prefill 路径下因大 fp32 SSM 状态导致的 fp16 溢出问题，保留 fp32 精度（#62119）。
- **前端与解析修复**：修复 HTTP 错误响应头（如 `Retry-After`）丢失的问题（#62127）；修复推理结束后首个内容带有前导空格的解析 Bug（#62125）。

**CI/CD：**
- 将 B200 Kernels 和 MoE 重构的评估测试从手动分片迁移至自动分片，提升 CI 效率（#62120）。

### 🚀 Release 动态
*近期暂无新版本发布信息。*

---

## 🐛 Issues

### #62136 — [[Bug]:  quantization="mxfp8" + moe_backend="humming" crashes weight conversion with a bare KeyError: 'weight_scale' — the MXFP8 oracle selects humming, then a third-party schema converter raises](https://github.com/vllm-project/vllm/issues/62136)
- **作者**: jyfct356  **时间**: 2026-10-17 12:43 CST
- **标签**: bug, quantization
- **摘要**: ### Your current environment  ``` ==============================         System Info ============================== OS                           : Ubuntu 22.04.4 LTS (x86_64) GCC version                  : (Ubuntu 12.3.0-1ubuntu1~22.04.3) 12.3.0 Clang version                : 17.0.6 (https://github.…

### #62135 — [[Bug]: moe_backend="cutlass" with online FP8 raises a reason-free "vLLM CUTLASS FP8 MoE backend is disabled for this configuration." — every sibling backend reports the actual rejection reason via _make_log_unsupported](https://github.com/vllm-project/vllm/issues/62135)
- **作者**: jyfct356  **时间**: 2026-10-17 12:42 CST
- **标签**: bug, nvidia, quantization
- **摘要**: ### Your current environment  ``` ==============================         System Info ============================== OS                           : Ubuntu 22.04.4 LTS (x86_64) GCC version                  : (Ubuntu 12.3.0-1ubuntu1~22.04.3) 12.3.0 Clang version                : 17.0.6 (https://github.…

### #62134 — [[Bug]: speculative_config.method="ngram_gpu" with compilation_config.mode="NONE" kills engine init — ValueError: No compilation mode is set.](https://github.com/vllm-project/vllm/issues/62134)
- **作者**: jyfct356  **时间**: 2026-10-17 12:41 CST
- **标签**: bug
- **摘要**: ### Your current environment  ``` ==============================         System Info ============================== OS                           : Ubuntu 22.04.4 LTS (x86_64) GCC version                  : (Ubuntu 12.3.0-1ubuntu1~22.04.3) 12.3.0 Clang version                : 17.0.6 (https://github.…

### #62130 — [[Spec Decode] Compose grammar-constrained sampling with EAGLE speculative decoding](https://github.com/vllm-project/vllm/issues/62130)
- **作者**: YuvrajSinghBhadoria2  **时间**: 2026-10-17 11:57 CST
- **标签**: structured-output, speculative-decoding
- **摘要**: ## Problem  Speculative decoding and grammar-constrained sampling (`structured_outputs`, guided decoding) do not compose in vLLM today. Under structured outputs the target verifier only accepts grammar-valid tokens, so *unconstrained* draft models waste most of their draft budget on tokens the FSM w…

## 🔀 Pull Requests

### #62133 — [[Bugfix][Kernel] Fix OAI Triton MXFP4 MoE on SM110](https://github.com/vllm-project/vllm/pull/62133)
- **作者**: wtdcode  **时间**: 2026-10-17 12:39 CST
- **标签**: bug, gpt-oss, quantization
- **摘要**: ## Overview  Follow-up to #58877 to fix Triton kernels for sm110. Note this does not change the default kernel selection for sm110.  ## Claims  Fixes https://github.com/vllm-project/vllm/issues/61374 and supersedes #61387  ## Validation  Verified by the original author @janbernloehr  ## Details  App…

### #62132 — [[Kernel]Add norm fused kernel for Step-5 Preview and Muse Spark Open](https://github.com/vllm-project/vllm/pull/62132)
- **作者**: jeejeelee  **时间**: 2026-10-17 12:37 CST
- **标签**: performance, ci/build
- **摘要**: ## Overview Part of https://github.com/vllm-project/vllm/pull/61727 and https://github.com/vllm-project/vllm/pull/61879   ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional…

### #62131 — [[ROCm][Perf][DSv4.1] Mono decode for the DSpark draft layers](https://github.com/vllm-project/vllm/pull/62131)
- **作者**: Fangzhou-Ai  **时间**: 2026-10-17 11:57 CST
- **标签**: rocm, deepseek, dflash, DSv4.1
- **摘要**: ## Overview  DSv4.1-Flash mono decode (MI355X, `--monokernel`): the DSpark draft's three layers now run on the mono K1 / K2 kernels too, as ATOM's `DraftMonoRunner` does, instead of vLLM's per-op path. Follow-up to #60397 and #61082; they follow `--monokernel` like the target's layers, with no new s…

### #62129 — [[Bugfix][MRV2] Fix speculative warmup deadlock with pipeline parallelism](https://github.com/vllm-project/vllm/pull/62129)
- **作者**: chenjr97  **时间**: 2026-10-17 11:44 CST
- **标签**: bug, mrv2
- **摘要**: ## Overview  Fix a pipeline-parallel startup deadlock in MRV2 speculative decoding introduced by [#59260](https://github.com/vllm-project/vllm/pull/59260).  ## Claims  - Ensure all PP stages participate in the additional greedy warmup pass. - Add regression coverage for matching warmup rounds across…

### #62128 — [[Kernel] Use 16-row tiles for SM89 int8 CUTLASS GEMM at M in (16, 64] for small N](https://github.com/vllm-project/vllm/pull/62128)
- **作者**: amankarki151  **时间**: 2026-10-17 11:30 CST
- **标签**: nvidia
- **摘要**: ## Overview  Fixes #56924. On SM89 (Ada), the int8 W8A8 CUTLASS GEMM gets ~1.5x slower when M goes from 16 to 17, and again from 32 to 33. The dispatch moves from 16-row tiles (`sm89_int8_config_M16`) to 32-row tiles at M=17 and 64-row tiles at M=33. For the small-N layers of models like Qwen3-1.7B …

### #62127 — [[Bugfix][Frontend] Preserve HTTP error response headers](https://github.com/vllm-project/vllm/pull/62127)
- **作者**: KuanJia  **时间**: 2026-10-17 11:27 CST
- **标签**: bug, frontend, cohere
- **摘要**: ## Overview  Preserve HTTP error headers through vLLM's shared exception handler and Cohere error-envelope conversion. A `429` raised with `Retry-After: 30` now keeps its retry delay; `401` responses keep their authentication challenge.  ## Claims  - Forward `HTTPException.headers` when building the…

### #62126 — [[Kernel] Add FP32 router GEMM support for Step-5 Preview and Muse Spark Open](https://github.com/vllm-project/vllm/pull/62126)
- **作者**: jeejeelee  **时间**: 2026-10-17 11:19 CST
- **标签**: ready
- **摘要**: <!-- markdownlint-disable --> Part of https://github.com/vllm-project/vllm/pull/61727 and https://github.com/vllm-project/vllm/pull/61879 ## Overview  <!-- Summarize the change in 1-2 lines. Link relevant issues and PRs. -->  ## Claims  <!-- State the accomplishments in a few concrete bullets: speed…

### #62125 — [[Bugfix][Parser] Strip leading whitespace from first content delta af…](https://github.com/vllm-project/vllm/pull/62125)
- **作者**: Laurent-Zhang  **时间**: 2026-10-17 10:58 CST
- **标签**: bug, tool-calling, qwen
- **摘要**: …ter reasoning end  Fixes #61972    ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this …

### #62120 — [[CI] Move B200 Kernels and MoE Refactor evals from manual to automatic shards](https://github.com/vllm-project/vllm/pull/62120)
- **作者**: Thangnguyenvn98  **时间**: 2026-10-17 10:16 CST
- **标签**: ci/build
- **摘要**: ## Overview  Moves three steps that split their tests by hand onto `automatic_shard`, following #62077 and #62084. `kernels-b200` ran a hand-written `case "$$BUILDKITE_PARALLEL_JOB"` block per shard. The two MoE Refactor integration steps read a different config list in each shard (`config-*-shard-$…

### #62119 — [[Bugfix][Kernel] GDN chunk/prefill path: honor fp32 SSM state precision](https://github.com/vllm-project/vllm/pull/62119)
- **作者**: miggle711  **时间**: 2026-10-17 10:03 CST
- **标签**: bug
- **摘要**: ## Purpose  Closes #62116.  #54146 fixed the GDN decode-update kernel overflowing fp16 with a large fp32 SSM state, but its own description flagged that the same bug exists on the chunk/prefill path:  > The same materialize-in-activation-dtype pattern exists in the chunk/prefill kernels (`chunk_o`, …
