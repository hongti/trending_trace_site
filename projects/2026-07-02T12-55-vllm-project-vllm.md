# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-02 12:55 UTC

## AI 总结

## vllm-project/vllm 近期动态摘要

---

### 🐛 Issue

1. **#47436 — Blackwell SM120 上 Block-scaled FP8 加载崩溃**
   - v0.24.0 在 NVIDIA RTX PRO 6000（SM120 / compute capability 12.0）加载 compressed-tensors W8A8 模型时，DeepGEMM 抛出 "Unknown SF transformation" 断言错误，无法正常启动。

2. **#47430 — Qwen3.6-27B 前缀缓存失效**
   - v0.23.0 下多请求共享相同 system prompt 时，prefix cache 命中率始终为 0.0%，预期应命中共享前缀。

---

### 🔧 Pull Request

**新特性 / 增强：**

- **#47434 — 支持 AutoRound 格式 Block-Wise FP8 量化**
  - 新增 AutoRound 量化格式（W8G128×128 block-wise FP8）的加载与推理支持，扩展了 vLLM 的量化生态。

- **#47433 — HPC_ATTN 后端支持 MTP 与动态调度注意力**
  - 为高性能注意力后端新增 multi-token prediction（MTP）支持和动态调度注意力机制。

- **#47423 — CPU Offloading EC Connector（编码器缓存卸载到 CPU）**
  - 新增 `ECCPUConnector`，将 encoder 输出卸载至共享 CPU mmap 区域（`/dev/shm`），使单个 vLLM 实例可在后续请求中复用编码器缓存，降低显存占用。

- **#47435 — Rust 前端调度器日志补齐**
  - 补充 Rust 前端周期性日志与 Python 端的对齐：包括 deferred requests、preemptions、prefix cache 命中率、speculative decoding 吞吐量等统计项。

**Bug 修复：**

- **#47429 — Speculative Decoding：DeepSeek-V4-Flash 缺少 `draft_id_to_target_id` 映射**
  - 修复 `DSparkDeepseekV4ForCausalLM` 在推测解码时因未定义 `draft_id_to_target_id` 导致的 `AttributeError` 崩溃。

- **#47428 — Mamba2 在非推测解码模式下崩溃**
  - 修复混合 Mamba 模型中 `num_accepted_tokens` 形状错误传递导致的 `selective_state_update` 断言失败。

- **#47426 — XPU 平台 Gemma4 性能回退**
  - Gemma4Config 在 XPU 平台强制使用 `TRITON_ATTN` 后端（不支持），改为允许平台原生 `FLASH_ATTN` 后端，修复性能回退。

- **#47427 — MoE FlashInfer TRT-LLM 调优参数低估**
  - 将 `tune_max_num_tokens` 设为 `max_num_tokens × dp_size`（最低 8192），修复 DP>1 及大 batch 场景下的参数低估问题。

**回滚（因 CI 失败）：**

- **#47432 — 回滚 FlashInfer CUTLASS MoE tuning token bound 修复（#46838）**
  - B200 夜间 CI 出现 MoE 集成测试失败，触发自动回滚。

- **#47431 — 回滚 TRT-LLM FP8 MoE gemm1_alpha/beta/clamp_limit（#45723）**
  - H100 夜间 CI 出现 2 项 LM Eval 测试失败，触发自动回滚。

---

### 📦 Release

> 本期数据中**无新版本发布**记录。

---

**总结要点：** 近期活动集中在 **Blackwell 新硬件兼容性**（FP8 加载崩溃）、**前缀缓存可靠性**、**量化格式扩展**（AutoRound FP8）以及多项**核心内核/后端稳定性修复**（Mamba2、Spec Decoding、MoE 调优、XPU 注意力后端）。两项 MoE 相关 PR 因夜间 CI 失败被自动回滚，需关注后续修复进展。

---

## 🐛 Issues

### #47436 — [[Bug]: Block-scaled FP8 (compressed-tensors W8A8) crashes on load on SM120 Blackwell (RTX PRO 6000), v0.24.0 — DeepGEMM "Unknown SF transformation" assertion](https://github.com/vllm-project/vllm/issues/47436)
- **作者**: Odrec  **时间**: 2026-07-02 12:51 UTC
- **摘要**: ### Your current environment  <details> <summary>Environment</summary>  - **vLLM**: v0.24.0 (official `vllm/vllm-openai:v0.24.0` Docker image) - **GPU**: NVIDIA RTX PRO 6000 Blackwell Server Edition — **compute capability 12.0 (SM120)** - **Driver**: 580.126.20 - **CUDA (image)**: 13.0, **torch** 2.…

### #47430 — [[Bug]: vllm==0.23.0 Qwen3.6-27B, system prompt same in multi requests, But prefix cache failure.](https://github.com/vllm-project/vllm/issues/47430)
- **作者**: qingkongby  **时间**: 2026-07-02 11:52 UTC
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Your output of `python collect_env.py` here ```  </details>   ### 🐛 Describe the bug  vllm==0.23.0 Qwen3.6-27B, system prompt same in multi requests, But prefix cache always 0.0%  ###…

## 🔀 Pull Requests

### #47435 — [[Rust Frontend] Improve scheduler stats logging parity](https://github.com/vllm-project/vllm/pull/47435)
- **作者**: BugenZhao  **时间**: 2026-07-02 12:14 UTC
- **标签**: ready, rust
- **摘要**: Signed-off-by: Bugen Zhao <i@bugenzhao.com><!-- markdownlint-disable -->   ## Purpose  This PR fills several straightforward Rust frontend periodic log-stats parity gaps with Python:  - deferred requests, preemptions, and external prefix cache hit rate - spec decoding: accepted/draft throughput, acc…

### #47434 — [[AutoRound] Support AutoRound Format Block-Wise FP8 in vLLM](https://github.com/vllm-project/vllm/pull/47434)
- **作者**: Zhenzhong1  **时间**: 2026-07-02 12:10 UTC
- **摘要**: Test:  ```bash CUDA_VISIBLE_DEVICES=1 python examples/basic/offline_inference/generate.py --model ../tmp_autoround/Llama-3.1-8B-fp-w8g128x128 --enforce-eager --gpu_memory_utilization 0.8  -------------------------------------------------- Prompt: 'Hello, my name is' Generated text: ' Adelina. I am a…

### #47433 — [[Attention Backend] HPC_ATTN backend support mtp and dynamic scheduled attention](https://github.com/vllm-project/vllm/pull/47433)
- **作者**: thisjiang  **时间**: 2026-07-02 12:09 UTC
- **标签**: documentation, v1
- **摘要**: ## Purpose  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such as providing test command. - [ ] The …

### #47432 — [Revert "[Bugfix][Kernel] Correct FlashInfer CUTLASS MoE tuning token bound" (#46838)](https://github.com/vllm-project/vllm/pull/47432)
- **作者**: vllm-agent  **时间**: 2026-07-02 12:08 UTC
- **标签**: bug, nvidia
- **摘要**: ## Revert of https://github.com/vllm-project/vllm/pull/46838  This reverts the merge commit 2665ed704b04219dd67a6bb82636cd51bbe98183.  ### Reason  This PR is linked to **1 new CI failure** in nightly build [#75919](https://buildkite.com/vllm/ci/builds/75919): - **MoE Refactor Integration Test (B200 …

### #47431 — [Revert "[MoE] Plumb gemm1_alpha/beta/clamp_limit into TRT-LLM FP8 MoE" (#45723)](https://github.com/vllm-project/vllm/pull/47431)
- **作者**: vllm-agent  **时间**: 2026-07-02 12:08 UTC
- **标签**: nvidia
- **摘要**: ## Revert of https://github.com/vllm-project/vllm/pull/45723  This reverts the merge commit fa248139a0206b3e39780296e3f056a978957f63.  ### Reason  This PR is linked to **2 new CI failures** in nightly build [#75919](https://buildkite.com/vllm/ci/builds/75919): - **LM Eval Humming (H100 - TEMPORARY)*…

### #47429 — [[Bugfix][Spec Decode] Add missing draft_id_to_target_id to DSparkDeepseekV4ForCausalLM](https://github.com/vllm-project/vllm/pull/47429)
- **作者**: Laurent-Zhang  **时间**: 2026-07-02 11:38 UTC
- **标签**: bug, deepseek
- **摘要**: ## What this PR does  Fixes #47418  `DSparkDeepseekV4ForCausalLM` is a full-vocab draft model (draft ids are target ids, no d2t remapping). However, `DSparkSpeculator.load_draft_model` accesses `model.draft_id_to_target_id` unconditionally, causing an `AttributeError` when running DeepSeek-V4-Flash …

### #47428 — [[ModelRunner V2] Fix Mamba2 crash on non-spec-decode](https://github.com/vllm-project/vllm/pull/47428)
- **作者**: njhill  **时间**: 2026-07-02 11:36 UTC
- **标签**: ready, v1
- **摘要**: `MambaHybridModelState.build_metadata` populated num_accepted_tokens on every forward pass, so the Mamba2 attention builder passed a wrongly shaped tensor `(num_reqs, not num_decodes)` straight into `selective_state_update`, tripping `assert num_accepted_tokens.shape == (N,)`. This crashed hybrid Ma…

### #47427 — [[MoE] Set FI TRTLLM `tune_max_num_tokens=self.max_num_tokens * self.dp_size`](https://github.com/vllm-project/vllm/pull/47427)
- **作者**: netanel-haber  **时间**: 2026-07-02 11:30 UTC
- **标签**: nvidia
- **摘要**: FLashInfer TRTLLM MoE kernels: - Current behavior underestimates: When `max_num_batched_tokens > 8192`, and when DP>1 even more, as we need max_num_batched_tokens * DP. - Pass a DP-aware value: max_num_tokens * dp_size for all-gather and max_num_tokens for routed dispatch, with a minimum of 8192 [pr…

### #47426 — [[Bugfix][Hardware][XPU] Fix Gemma4 performance regression by allowing platform-native attention](https://github.com/vllm-project/vllm/pull/47426)
- **作者**: Vinay12345-neutron  **时间**: 2026-07-02 11:08 UTC
- **标签**: bug, intel-gpu
- **摘要**: Co-authored-by: Gemini Signed-off-by: Vinay12345-neutron vinayrjumani@gmail.com     ## Purpose `Gemma4Config` forces the `TRITON_ATTN` backend when `is_fa_version_supported(4)` is false. However, on XPU platforms, standard CUDA/ROCm FA4 is not available, yet the platform's default `FLASH_ATTN` backe…

### #47423 — [[EC Connector] CPU Offloading EC Connector](https://github.com/vllm-project/vllm/pull/47423)
- **作者**: omerpaz95  **时间**: 2026-07-02 10:32 UTC
- **标签**: v1, cpu
- **摘要**: # Add `ECCPUConnector` — CPU encoder-cache offloading connector  ## Purpose  This PR adds `ECCPUConnector`, a self-contained encoder-cache (EC) connector that offloads encoder outputs to a shared CPU `mmap` region on `/dev/shm` so a single `ec_role=both` vLLM instance can reuse them on a later reque…
