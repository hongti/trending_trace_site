# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-08 09:06 CST

## AI 总结

# vllm-project/vllm-ascend 近期动态摘要

---

## 🐛 Issue

| # | 标题 | 要点 |
|---|------|------|
| #11588 | **V1 structured outputs 在 mixed auto backends 下崩溃** | 当 `structured_outputs_config.backend=auto` 时，同一 V1 引擎生命周期内不同请求可能解析到不同后端，导致崩溃 |
| #11587 | **main2main 自动化流程中断** | main2main 同步自动化未完成所有步骤，状态为 `failed`，需人工介入审核 |
| #11583 | **GLM 5.2 0day镜像 cached_tokens 命中异常偏高** | 启用 `--enable-prompt-tokens-details` 后测试 random 数据集，KV cache 命中率异常偏高（疑似统计/缓存逻辑 bug） |

---

## 🔀 Pull Request

### 🔧 Bug 修复

- **#11589** Guard mixed structured output backends — 修复 #11588，记录首次请求解析的后端类型，后续请求若解析为不同后端则抛出明确错误，避免引擎崩溃
- **#11584** mega_moe_max_tokens 默认值修正 — 从 65536 → 131072，防止 MoE 场景下 token 限制过早截断
- **#11580** **Revert all2all quantization** — 回退此前 all2all 量化提交，因 `npu_moe_init_routing_v2` 在 MXFP4 量化场景下无法获取正确数据类型，导致行为异常
- **#11577** chunk_scaled_dot_kkt_fwd_kernel 重编译问题修复 — Triton kernel 中 `bh_step`、`task_num`、`num_core` 参数此前未正确处理，导致重复编译
- **#11581** 修复 #11053 引入的 UT 错误

### ⚡ 性能优化

- **#11586** PCP FA restore & output merge 优化 — PCP 混合注意力路径中构建 `pcp_fa_padding_restore_idx`，用 `index_select` 替代原有恢复逻辑（零填充方式），提升 FA QKV 恢复效率

### 🆕 新特性 / 功能增强

- **#11585** Layerwise KV Pooling + Memcache Backend（v0.23.0 backport）— 为逐层 KV cache 传输引入 Memcache 后端，per-rank 存储键格式 `@{model_name}:{rank}`
- **#11582** **TP-mismatch PD 分离式 KV Pooling（GQA 支持）** — 支持 Prefill/Decode TP 不等（如 prefill tp4 / decode tp2）的 GQA 模型，Decode 用 strided put 存储 KV，Prefill 用 strided get 加载
- **#11579** ec_cache_manager vllm-ascend 适配 — 将 ec_cache_manager 适配至 vllm-ascend

### 🤖 CI / Misc

- **#11578** 自动更新 `test_config.yaml` 中的预估测试时间（CI workflow 自动生成）
- **#11587**（Issue 同步）main2main 流程需人工审核

---

## 📦 Release

> 本次窗口内 **无新版本发布**。#11585 为 `releases/v0.23.0` 的 backport，说明 v0.23.0 仍在持续补丁维护中。

---

### 📌 总结重点

1. **最关键修复**：#11589/#11588 解决了 V1 structured output `auto` 后端混合导致的引擎崩溃问题
2. **重要回退**：#11580 回退了 all2all 量化，MXFP4 场景受影响的用户需关注后续修复进展
3. **KV Cache 生态持续扩展**：Memcache 后端、TP-mismatch PD 分离式 Pooling、ec_cache_manager 适配，表明 vllm-ascend 在 KV cache 分布式/分离式调度方向投入大量精力
4. **性能方向**：PCP 注意力路径优化（#11586）是当前 attention 性能改进的抓手

---

## 🐛 Issues

### #11588 — [V1 structured outputs can crash on mixed auto backends](https://github.com/vllm-project/vllm-ascend/issues/11588)
- **作者**: QwertyJack  **时间**: 2026-07-08 04:48 CST
- **摘要**: ### Problem  When `structured_outputs_config.backend` is `auto`, two valid structured-output requests can resolve to different backends in the same V1 engine lifetime.  One reproduced sequence is:  1. Send an object JSON schema request. It resolves to `xgrammar` and returns HTTP 200. 2. Send a tuple…

### #11587 — [main2main manual review required ()](https://github.com/vllm-project/vllm-ascend/issues/11587)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-08 01:58 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR:  - Commit range: `1f486d96a17303ce8db8e02be39545b2be338446`...`` - Status: `failed`  ## Final Summary  main2main adaptation failed — no steps completed.

### #11583 — [[Bug]: GLM 5.2 0day镜像中，使能-enable-prompt-tokens-details，测试random数据集，cached_tokens命中异常偏高](https://github.com/vllm-project/vllm-ascend/issues/11583)
- **作者**: Sfeching  **时间**: 2026-07-07 21:02 CST
- **标签**: bug, glm5, triaged, llm-model
- **摘要**: ### Your current environment  torch                                    2.10.0+cpu torch_npu                                2.10.0 torchaudio                               2.10.0+cpu torchvision                              0.25.0+cpu transformers                             5.5.4 triton             …

## 🔀 Pull Requests

### #11589 — [Guard mixed structured output backends](https://github.com/vllm-project/vllm-ascend/pull/11589)
- **作者**: QwertyJack  **时间**: 2026-07-08 04:48 CST
- **标签**: module:tests
- **摘要**: ### Summary  Fixes #11588.  This patch adds a vllm-ascend guard for V1 structured outputs when `backend=auto` resolves different requests to different backends in the same engine lifetime.  - Record the first resolved structured-output backend on `StructuredOutputsConfig`. - Reject later requests th…

### #11586 — [[Performance][Attention] Optimize PCP FA restore and output merge](https://github.com/vllm-project/vllm-ascend/pull/11586)
- **作者**: li1how  **时间**: 2026-07-07 23:24 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  This PR optimizes the PCP hybrid attention path:  - Builds and passes `pcp_fa_padding_restore_idx` so the FA QKV restore path can use `index_select` with a zero-padding row instead of boolean scatter into a workspace. - Uses `torch_npu.npu_attention_update` f…

### #11585 — [[KV Cache][Feature] Support Layerwise KV Pooling with Memcache Backend (v0.23.0)](https://github.com/vllm-project/vllm-ascend/pull/11585)
- **作者**: tyy0829  **时间**: 2026-07-07 22:55 CST
- **标签**: documentation, module:tests, module:core, ready
- **摘要**: ## What this PR does / why we need it?  Backport of PR #11444 to `releases/v0.23.0`.  Support memcache backend for layerwise KV cache transfer in vllm-ascend.  ### Key design - Per-rank store key: `@{model}@{block_hash}@{head_or_tp_rank}` where `head_or_tp_rank = tp_rank // put_step` (shared within …

### #11584 — [[BugFix] mega moe max token default value fix](https://github.com/vllm-project/vllm-ascend/pull/11584)
- **作者**: justice-dance  **时间**: 2026-07-07 21:25 CST
- **标签**: module:core
- **摘要**: ### What this PR does / why we need it? Change the default value of mega_moe_max_tokens from 65536 to 131072.  ### Does this PR introduce _any_ user-facing change? NA  ### How was this patch tested? Configurable parameter, no extra testing needed.  - vLLM version: v0.23.0 - vLLM main: https://github…

### #11582 — [[Feature][KV Pool] Support TP-mismatch PD disaggregated KV pooling fo…](https://github.com/vllm-project/vllm-ascend/pull/11582)
- **作者**: DreamerLeader  **时间**: 2026-07-07 20:48 CST
- **标签**: module:tests
- **摘要**: …r GQA  Add AscendStore pooling adaptation for TP-unequal prefill/decode (e.g. prefill tp4 / decode tp2) on GQA dense-attention models. Decode stores KV via strided put; prefill loads via strided get, sharing a unified effective_tp namespace so cached blocks are mutually reachable.  All new paths ar…

### #11581 — [[Misc] Fix UT error.](https://github.com/vllm-project/vllm-ascend/pull/11581)
- **作者**: menogrey  **时间**: 2026-07-07 20:33 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it?   #11053 introduce a UT error. This PR fix it.  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #11580 — [[BugFix] Revert all2all quantization](https://github.com/vllm-project/vllm-ascend/pull/11580)
- **作者**: lijiahang226  **时间**: 2026-07-07 20:30 CST
- **标签**: module:ops
- **摘要**: ### What this PR does / why we need it? This commit caused incorrect behavior in the MXFP4 quantization scenario, since operation npu_moe_init_routing_v2 can not get the correct data type. ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 -…

### #11579 — [ec_cache_manager vllm-ascend适配](https://github.com/vllm-project/vllm-ascend/pull/11579)
- **作者**: hanxi-java  **时间**: 2026-07-07 20:15 CST
- **标签**: module:core
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version:  - vLLM main: https://github.com/vllm-project/vllm/commit/

### #11578 — [[CI] Auto-update estimated test times in test_config.yaml](https://github.com/vllm-project/vllm-ascend/pull/11578)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-07 19:59 CST
- **摘要**: ## Summary  This PR was auto-generated by the **Update estimated test times** [workflow](https://github.com/vllm-project/vllm-ascend/actions/runs/28860528956).  It updates the `estimated_times` values in `.github/workflows/scripts/test_config.yaml` based on actual elapsed times collected from CI wor…

### #11577 — [[Ops][BugFix] chunk_scaled_dot_kkt_fwd_kernel recompile issues](https://github.com/vllm-project/vllm-ascend/pull/11577)
- **作者**: OsirisDuan  **时间**: 2026-07-07 19:56 CST
- **标签**: module:ops
- **摘要**: ### What this PR does / why we need it?  This PR fixes recompilation issues in the `chunk_scaled_dot_kkt_fwd_kernel` Triton kernel. The parameters `bh_step`, `task_num`, and `num_core` were previously defined as `tl.constexpr`, which caused Triton to recompile the kernel whenever these values change…
