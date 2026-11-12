# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-12 15:34 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issues
1. **数据并行启动崩溃回归 (#67183)**：使用 `--data-parallel-start-rank 0` 且 `data_parallel_size == 1` 时会导致启动崩溃（由 #47692 引入的回归）。
2. **Blackwell 架构启动失败 (#67177)**：在 SM120 (Blackwell) GPU 上，由于 FlashAttention 缺乏受支持的 PTX 工具链，导致 EngineCore 无法启动。
3. **UltraQuant 量化参数优化提案 (#67176)**：提出 UltraQuant 中的缩放常数 `c=0.156` 应在 UE8M0 snap 前保持 MSE 最优，建议改为 `c≈0.2`，可零成本将 KV 量化的 KL 散度降低 40-56%。

### 🚀 Pull Requests
1. **内核与算子修复及升级**：
   - **#67181**：升级 DeepGEMM，修复 CUDA graph replay 下的 MegaMoE combine-readiness 问题。
   - **#67178**：修复 Blackwell (SM120) 兼容性问题，使其优先使用 FlashInfer 而非 FlashAttention（对应 Issue #67177）。
   - **#67175**：升级 FlashMLA，为 SM90 启用 small-head ETAP sparse prefill 内核。
2. **系统与通信优化**：
   - **#67182**：优化 XPU 批处理不变集合通信，对大于 4MiB 的输入采用 all-to-all 和 Triton 求和结合 all-gather 的分片规约策略。
   - **#67174**：修复流水线并行下 MultiConnector 的握手元数据路由问题。
3. **前端与底层逻辑改进**：
   - **#67180**：为离线池化 API 新增聊天和多模态选项。
   - **#67173**：修复底层逻辑，不再将 slot-mapped token 缓存与 kernel blocks 对齐。
4. **文档与环境构建**：
   - **#67184**：同步 MiniMax-M3.1-Flash 的 GB300 Dynamo AgentX 预设。
   - **#67179**：修正 Hopper/Blackwell split-k 上关于批处理不变性的注释。
   - **#67172**：在环境收集脚本 (`collect_env`) 中加入 Intel oneAPI 编译器的具体构建日期。

### 📦 Release
*近期暂无版本发布动态。*

---

## 🐛 Issues

### #67183 — [[Bug]: Regression from #47692 - `--data-parallel-start-rank 0` with `data_parallel_size == 1` crashes at startup ("data_parallel_external_lb can only be set when data_parallel_size > 1")](https://github.com/vllm-project/vllm/issues/67183)
- **作者**: lanke-cy  **时间**: 2026-11-12 15:21 CST
- **摘要**: ## Your current environment  ```text vLLM: v0.30.0 (official upstream vLLM sources; the in-image vllm source is       unmodified on the affected path - the traceback file/line       vllm/engine/arg_utils.py:2342 matches upstream v0.30.0 exactly).       The regression also exists on current main (ver…

### #67177 — [[Bug]:  EngineCore fails to start on SM120 (Blackwell) due to unsupported PTX toolchain in FlashAttention](https://github.com/vllm-project/vllm/issues/67177)
- **作者**: seansie0830  **时间**: 2026-11-12 14:27 CST
- **标签**: bug
- **摘要**: ### 🐛 Describe the bug   ### Your current environment * **vLLM version:** `0.34.0rc1.dev245+ga7948445c` (V1 Engine) * **GPU Architecture:** NVIDIA Blackwell Consumer (`sm_120`) * **Model:** `HuggingFaceTB/SmolLM2-360M-Instruct` * **Command:** `vllm serve HuggingFaceTB/SmolLM2-360M-Instruct --gpu-mem…

### #67176 — [[RFC]: UltraQuant: the scale constant c = 0.156 is MSE-optimal before the UE8M0 snap, not after; c ≈ 0.2 cuts KV-quantization KL by 40–56% at no cost](https://github.com/vllm-project/vllm/issues/67176)
- **作者**: thomaszhang2661  **时间**: 2026-11-12 14:27 CST
- **标签**: quantization
- **摘要**: ### Motivation.  `ultraquant_4bit` picks each group's UE8M0 scale as  ``` s_raw = c * absmax(group)          # c = 0.156 (DEFAULT_CONSTANT_C, vllm/v1/attention/ops/ultraquant/format.py) s     = 2^round(log2(s_raw))       # UE8M0 snap ```  The constant is documented as "MSE-optimal; `s_raw = c * absm…

## 🔀 Pull Requests

### #67184 — [[Recipe] MiniMax-M3.1-Flash: sync the GB300 Dynamo AgentX presets to InferenceX #4998](https://github.com/vllm-project/vllm/pull/67184)
- **作者**: Juntian777  **时间**: 2026-11-12 15:23 CST
- **标签**: minimax, recipe
- **摘要**: ## Overview  Sync the MiniMax-M3.1-Flash GB300 Dynamo AgentX presets (`gb300_agentx_agg`, `gb300_agentx_pd`) to the current InferenceX lanes in [SemiAnalysisAI/InferenceX#4998](https://github.com/SemiAnalysisAI/InferenceX/pull/4998). They were last synced from #4631.  ## Claims  - Both presets pin t…

### #67182 — [[XPU][batch invariant] Optimize batch-invariant collectives with sharded reductions](https://github.com/vllm-project/vllm/pull/67182)
- **作者**: hlin99  **时间**: 2026-11-12 15:03 CST
- **标签**: intel-gpu, batch-invariance
- **摘要**: Use all-to-all and fused Triton summation for inputs above 4 MiB, followed by all-gather for all-reduce. Keep the existing all-gather path for smaller inputs and uneven reduce-scatter during graph capture.  Keep gather and sharded all-reduce implementations separate, with explicit dispatch at the en…

### #67181 — [[Bugfix][Kernel] Bump DeepGEMM for the MegaMoE combine-readiness fix under CUDA graph replay](https://github.com/vllm-project/vllm/pull/67181)
- **作者**: gau-nernst  **时间**: 2026-11-12 14:52 CST
- **标签**: bug, ready, ci/build, nvidia
- **摘要**: ## Overview  Bump DeepGEMM to vllm-project/DeepGEMM@196f338, which adds vllm-project/DeepGEMM#59: a cherry-pick of deepseek-ai/DeepGEMM#451, extended to the SiTU and NVFP4 kernels. It fixes stale MegaMoE outputs under CUDA graph replay at EP > 1. It is the only commit between the current pin (`0f769…

### #67180 — [[Frontend] Add chat and multimodal options to offline pooling APIs](https://github.com/vllm-project/vllm/pull/67180)
- **作者**: noooop  **时间**: 2026-11-12 14:47 CST
- **标签**: frontend, pooling
- **摘要**: ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this PR.  - [ ] **Design Fit:** Minimizes…

### #67179 — [[Doc] Correct the batch-invariance comment on Hopper/Blackwell split-k](https://github.com/vllm-project/vllm/pull/67179)
- **作者**: imthomp  **时间**: 2026-11-12 14:39 CST
- **标签**: batch-invariance
- **摘要**: ## Purpose  The comment in `enable_batch_invariant_mode` for SM90/SM100 says that split-k is the only source of batch variance on those GPUs and that the cuBLAS workspace config disables it. That is not accurate for every shape: with split-k disabled, some narrow shapes such as MoE router projection…

### #67178 — [[Bugfix][Attention] Prioritize FlashInfer over FlashAttention on SM120 (Blackwell)](https://github.com/vllm-project/vllm/pull/67178)
- **作者**: seansie0830  **时间**: 2026-11-12 14:27 CST
- **标签**: bug, nvidia
- **摘要**: fix  #67177  ### Essential Elements of an Effective PR Description Checklist - [x] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [x] The test plan, such as providing test command. - [x] The test results, such as pasting the results comparison.  ### Pu…

### #67175 — [[Build] Bump FlashMLA to enable SM90 small-head ETAP sparse prefill](https://github.com/vllm-project/vllm/pull/67175)
- **作者**: chaunceyjiang  **时间**: 2026-11-12 14:07 CST
- **标签**: ready, ci/build
- **摘要**: <!-- markdownlint-disable --> Bump FlashMLA from `847ef308` to `b18ed744` to integrate the SM90 small-head ETAP sparse prefill kernel from [FlashMLA#37](https://github.com/vllm-project/FlashMLA/pull/37).  ## Claims  - Enable the upstream ETAP sparse prefill path for 8, 16, and 32 query heads on SM90…

### #67174 — [[Bugfix][KV Connector] Fix handshake metadata routing in MultiConnector with pipeline parallelism](https://github.com/vllm-project/vllm/pull/67174)
- **作者**: xsxszab  **时间**: 2026-11-12 13:58 CST
- **标签**: bug, kv-connector
- **摘要**: ## Overview  `MultiConnector` reports the handshake metadata of the first sub-connector that has any, and passes the engine core's merged `{(pp_rank, tp_rank): metadata}` to every sub-connector. With pipeline parallelism the merged dict has `pp_rank > 0` entries, and a sub-connector without its own …

### #67173 — [[Bugfix] Don't align slot-mapped token caches to kernel blocks](https://github.com/vllm-project/vllm/pull/67173)
- **作者**: DuFanYin  **时间**: 2026-11-12 13:56 CST
- **标签**: bug
- **摘要**: ## Overview  `customize_attention_spec` aligns the block stride of every spec on a fixed-kernel-block backend to whole kernel blocks, so packed blocks can be split. A packed slot-mapped token cache is never split: `map_kv_cache_to_kernel_blocks` rejects it at binding. Its alignment only inflates the…

### #67172 — [Include oneAPI compiler build date in collect_env](https://github.com/vllm-project/vllm/pull/67172)
- **作者**: xinyu-intel  **时间**: 2026-11-12 13:51 CST
- **摘要**: Report the Intel oneAPI DPC++/C++ Compiler version together with its 8-digit build date (e.g. 2026.1.0(20260101)) instead of the version alone, falling back to version-only when the date is unavailable.    ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Reque…
