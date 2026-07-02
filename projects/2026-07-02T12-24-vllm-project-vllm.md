# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-02 12:24 UTC

## AI 总结

# vllm-project/vllm 近期动态摘要

---

## 🐛 Issue

- **#47430 — Prefix Cache 失效问题**：在 vllm `0.23.0` 版本中，使用 Qwen3.6-27B 模型时，多个请求共享相同 system prompt，但 prefix cache 命中率始终为 0.0%，无法正常复用前缀缓存。这可能影响长系统提示场景下的推理效率与吞吐。

---

## 🔀 Pull Request

### 🆕 新特性 / 增强

- **#47434 — 支持 AutoRound Block-Wise FP8 格式**：为 vLLM 引入对 AutoRound 量化格式的 Block-Wise FP8 支持，扩展了低精度推理的模型兼容性。
- **#47433 — HPC_ATTN 后端支持 MTP 与动态调度 Attention**：为高性能注意力后端新增多 token 预测（MTP）及动态调度能力。
- **#47435 — Rust Frontend 日志对齐 Python**：补齐 Rust 前端周期性统计日志与 Python 端的差异，新增 deferred requests、preemptions、外部 prefix cache 命中率及 speculative decoding 吞吐等指标。
- **#47423 — CPU Offloading EC Connector**：新增 `ECCPUConnector`，将 encoder 输出卸载至共享 CPU mmap 区域（`/dev/shm`），使单一 `ec_role=both` 实例可在后续请求中复用 encoder 缓存，降低显存占用。

### 🔧 Bug 修复

- **#47429 — Speculative Decode 修复**：为 `DSparkDeepseekV4ForCausalLM` 补充缺失的 `draft_id_to_target_id` 属性，修复 DeepSeek-V4-Flash 运行时的 `AttributeError` 崩溃。
- **#47428 — Mamba2 非推测解码崩溃修复**：修正 `MambaHybridModelState.build_metadata` 在每次 forward pass 中错误填充 `num_accepted_tokens` 导致张量形状不匹配的 crash。
- **#47426 — XPU 平台 Gemma4 性能回退修复**：在 XPU 平台上允许使用平台原生 `FLASH_ATTN` 后端而非强制 `TRITON_ATTN`，解决 Gemma4 因 FA4 不可用而造成的性能下降。

### 🔄 MoE 相关修复与优化

- **#47427 — TRT-LLM MoE tuning 范围修正**：修正 FlashInfer TRT-LLM MoE kernel 在 `max_num_batched_tokens > 8192` 及 DP>1 场景下的容量低估问题，传入 DP-aware 的 token 数值。
- **#47431 — 回退 TRT-LLM FP8 MoE gemm1_alpha/beta/clamp_limit PR**：因导致 2 项 CI 失败（LM Eval Humming 等）而回退 #45723。
- **#47432 — 回退 FlashInfer CUTLASS MoE tuning token bound 修复**：因导致 MoE Refactor Integration Test (B200) CI 失败而回退 #46838。

---

## 📦 Release

本次数据中无新版本发布信息。

---

> **整体趋势**：近期活动集中在 **MoE kernel 稳定性修复（两项回退 + 一项调优）**、**Prefix Cache 问题暴露与日志增强**、**混合架构（Mamba2/Spec Decode）崩溃修复**，以及 **新量化格式（AutoRound FP8）和新卸载机制（CPU EC Connector）的引入**。MoE 相关变更波动较大，需关注后续 CI 修复进展。

---

## 🐛 Issues

### #47430 — [[Bug]: vllm==0.23.0 Qwen3.6-27B, system prompt same in multi requests, But prefix cache failure.](https://github.com/vllm-project/vllm/issues/47430)
- **作者**: qingkongby  **时间**: 2026-07-02 11:52 UTC
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Your output of `python collect_env.py` here ```  </details>   ### 🐛 Describe the bug  vllm==0.23.0 Qwen3.6-27B, system prompt same in multi requests, But prefix cache always 0.0%  ###…

## 🔀 Pull Requests

### #47435 — [[Rust Frontend] Improve scheduler stats logging parity](https://github.com/vllm-project/vllm/pull/47435)
- **作者**: BugenZhao  **时间**: 2026-07-02 12:14 UTC
- **标签**: rust
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

### #47427 — [[MoE] Set TRT-LLM tuning range from token capacity](https://github.com/vllm-project/vllm/pull/47427)
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
