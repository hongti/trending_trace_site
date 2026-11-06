# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-06 15:13 CST

## AI 总结

# vLLM 仓库动态摘要（截至 2026-11-06）

---

## 🐛 Issue
- **#66055** [Bug] `thinking_token_budget` 在早期消息提及推理起始标记时，错误地把后续全部 prompt token 都计入推理预算，导致推理功能被关掉（Qwen3 等模型受影响）。

---

## 🔧 Pull Requests

### Bugfix
- **#66056** 修复 #66055：早期消息中引用的 `<think>` 起始标记不再被计入 `thinking_token_budget`，避免推理被误关闭。
- **#66057** [Gemma4] 修复视频帧忽略 `videos_kwargs.max_soft_tokens` 的问题，此前会静默损坏视频输入。
- **#66052** [DSA] 修复 ShardQ 索引器每步复用同一块 pinned host buffer 导致的宿主缓冲竞态，改为每步使用新缓冲。
- **#66062** [Doc] 修复因 mock torch 子类导致的严格文档构建失败。

### 新特性 / 模型支持
- **#66061** 新增 **NCP-OLMo** 纯 HF checkpoint 后端支持，覆盖 MRV2 层级化模型状态与离线示例。
- **#66053** Rust 前端新增 `--tool-call-parser poolside_v1`，支持 `poolside/Laguna-XS-2.1` 的工具调用解析。

### 性能优化
- **#66058** [GDN] 将 prefill state 的 gather/scatter 融合进共享 Triton kernel，提升 Qwen3-Next/Qwen3.5 linear attention 与 Kimi KDA 的性能。
- **#66054** [MiniMax-M3] 在张量并行下按 token 对大 batch 分片，并将 MoE finalize 融合进 reduce-scatter，显著降低每层 100MB 级 NCCL all-reduce 开销。
- **#66059** 支持多节点 serving 的 D2D scaling，助力快速启动。

### 基础设施
- **#66060** [RL/Model Loader] 在真实模型家族上运行 `test_load_plan.py` 不变量测试，确保 planned 加载与冷启动逐字节一致。

---

## 📦 Release
本次摘要时间窗口内无新版本发布。

---

## 🐛 Issues

### #66055 — [[Bug]: thinking_token_budget turns off reasoning when an earlier message mentions <think>](https://github.com/vllm-project/vllm/issues/66055)
- **作者**: Arsh624  **时间**: 2026-11-06 14:03 CST
- **标签**: tool-calling
- **摘要**: ### Your current environment  ```text vllm 0.32.0 from PyPI, and main at d60c4404e (Model Runner V2) RTX 4070 Laptop GPU (8 GB), WSL2 Ubuntu 24.04, Python 3.12 ```  ### 🐛 Describe the bug  If a message before the assistant turn contains the literal text `<think>`, `thinking_token_budget` can switch …

## 🔀 Pull Requests

### #66062 — [[Doc][Bugfix] Fix strict docs build broken by subclassing mocked torch classes](https://github.com/vllm-project/vllm/pull/66062)
- **作者**: subhashpolisetti  **时间**: 2026-11-06 15:07 CST
- **标签**: bug, documentation
- **摘要**: ## Overview  The strict docs build on main has failed since #65235 with `Failed to import vllm.entrypoints.cli.preload.PreloadSubcommand` and `TypeError: expected string or bytes-like object, got 'PydanticMagicMock'` ([main build at #65235](https://app.readthedocs.org/projects/vllm/builds/35689810/)…

### #66061 — [[Feature][Model] Add NCP-OLMo support for pure HF checkpoints](https://github.com/vllm-project/vllm/pull/66061)
- **作者**: LuckySJTU  **时间**: 2026-11-06 14:48 CST
- **标签**: documentation, new-model, ci/build
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Add the pure-HF NCP-OLMo backend, request-owned hierarchical model state for MRV2, focused correctness coverage, and an offline example. Related to #56880. DFlash/speculative decoding is outside this PR; the RFC remains open for that work.  ## Claims  - Lo…

### #66060 — [[RL][Model Loader] Load plan: per-family plan == load_weights tests](https://github.com/vllm-project/vllm/pull/66060)
- **作者**: zzhisyyds2  **时间**: 2026-11-06 14:42 CST
- **摘要**: ## Overview  Part of #64418 (P2, per-family part), on top of #65235. Runs the `test_load_plan.py` invariant (a planned update equals a cold start, byte for byte, at the same addresses) on real model classes built from tiny random configs, one per RL-relevant family. Tests only; no change under `vllm…

### #66059 — [[Fast start] Support D2D scaling for multi-node serving](https://github.com/vllm-project/vllm/pull/66059)
- **作者**: Isotr0py  **时间**: 2026-11-06 14:19 CST
- **标签**: documentation, frontend, needs-rebase, ci/build
- **摘要**: ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this PR.  - [ ] **Design Fit:** Minimizes…

### #66058 — [[Perf][GDN] Fuse prefill state gather/scatter with the shared Triton kernels](https://github.com/vllm-project/vllm/pull/66058)
- **作者**: gau-nernst  **时间**: 2026-11-06 14:15 CST
- **标签**: kimi
- **摘要**: ## Overview  Replace eager GDN prefill state gather/scatter with the existing shared Triton `gather_initial_states` / `scatter_states` kernels for Qwen3-Next/Qwen3.5 linear attention and Kimi KDA. Supersedes [#64498](https://github.com/vllm-project/vllm/pull/64498); see comparison below with [#50372…

### #66057 — [[Bugfix][Gemma4] Honor videos_kwargs.max_soft_tokens for video frames](https://github.com/vllm-project/vllm/pull/66057)
- **作者**: Alex-Wengg  **时间**: 2026-11-06 14:11 CST
- **标签**: bug, documentation, multi-modality
- **摘要**: ## Overview  Fixes #50788. Gemma 4 now honors `videos_kwargs.max_soft_tokens` for video frames, as Transformers does.  On main this setting silently corrupts video inputs. The video loop passes a flat `max_soft_tokens=70`, but vLLM's kwarg scoping drops a flat key whenever `videos_kwargs` also sets …

### #66056 — [[Bugfix][Reasoning] Don't count a start marker quoted in an earlier message against thinking_token_budget](https://github.com/vllm-project/vllm/pull/66056)
- **作者**: Arsh624  **时间**: 2026-11-06 14:03 CST
- **标签**: bug, documentation, cpu, mrv2
- **摘要**: ## Overview  Fixes #66055. When a system, user or tool message quotes the reasoning start marker (`<think>` is a special token for Qwen3), `thinking_token_budget` counted every later prompt token as reasoning and could force `</think>` as the first output token. Now an end-of-sequence token in the p…

### #66054 — [[Perf][MiniMax-M3] Shard large batches by token and fuse the MoE finalize into the reduce-scatter](https://github.com/vllm-project/vllm/pull/66054)
- **作者**: MrGeva  **时间**: 2026-11-06 13:58 CST
- **标签**: ci/build, minimax
- **摘要**: ## Overview  With tensor parallelism and no expert parallelism, every MiniMax-M3 layer all-reduces two TP-partial activations. At an 8192-token step that is two 100 MB NCCL all-reduces per layer (121 per step, about 28% of the GPU time on GB300 TP4). The residual add, Gemma RMSNorm and MXFP8 quantiz…

### #66053 — [[Rust Frontend] Add the Poolside tool parser](https://github.com/vllm-project/vllm/pull/66053)
- **作者**: jiangLLM  **时间**: 2026-11-06 13:46 CST
- **标签**: documentation, tool-calling, rust
- **摘要**: ## Overview  The Rust frontend does not recognize `--tool-call-parser poolside_v1`. This PR adds it for `poolside/Laguna-XS-2.1` and covers the Poolside tool-parser item in #44280.  ## Claims  - Parse Laguna's XML tool calls in streamed and complete responses. - Return long string arguments as they …

### #66052 — [[Bugfix][DSA] Stage ShardQ indexer indices in fresh host memory each step](https://github.com/vllm-project/vllm/pull/66052)
- **作者**: AMD-yanfeiwang  **时间**: 2026-11-06 13:46 CST
- **标签**: bug
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Fix a host-buffer race in the DSA indexer metadata builder's ShardQ path (#63956): each step's gather indices and renumbered query ranges were uploaded from single reused pinned buffers, which async scheduling can overwrite before the step's copy runs.  ##…
