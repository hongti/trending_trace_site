# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-13 11:11 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期的动态摘要：

### 📌 Issue 动态

近期 Issue 主要集中在 **MRV2（Model Runner V2）算子性能优化** 和 **运行时/Bug 反馈**：

*   **MRV2 性能优化提案（作者: chengduxiaowu）**：集中提交了 5 项针对 MRV2 路径的算子优化提案，旨在提升 Ascend NPU 上的执行效率：
    *   `_apply_grammar_bitmask_kernel`：优化结构化输出中 grammar bitmask 的应用。
    *   `bincount` 与 `apply_penalties`：优化采样路径中的二进制掩码构建与频率/重复惩罚计算。
    *   `compute_token_logprobs`：优化采样中 token 的 log-softmax 概率计算。
    *   *注意*：针对投机解码 DFlash 路径的 `_prepare_dflash_inputs_kernel` 优化提案已被撤销（#14159）。
*   **Bug 报告**：
    *   **Triton 编译错误**（#14154）：`wy_fast.py` 在等长序列（`cu_seqlens=None`）分支下触发 `NameError: i_t is not defined`。
    *   **多流重叠性能瓶颈**（#14153）：在 TP8+EP W8A8 MoE 场景下，开启 FlashComm1 与 shared-expert multistream overlap 会导致 ReduceScatter 产生严重的局部延迟。

---

### 🔀 Pull Request 动态

PR 活动主要涵盖**新特性引入、关键 Bug 修复及上游适配**：

*   **🚀 新特性**：
    *   **AscendC Categorical Sampling**（#14141）：为 MRV2 引入原生 AscendC 分类采样算子，替代原本基于 Triton 的 Gumbel 采样，避免了生成全词表随机值的内存开销。
*   **🛠️ 重要 Bug 修复**：
    *   **动态 W8A8 mm_reduce_scatter 融合支持**（#14162）：修复了行并行线性算子中动态 W8A8 场景下的融合问题。
    *   **CausalConv1d 崩溃修复**（#14161）：修复了在混合 Mamba-Transformer 模型（如 Qwen3.6）+ MTP/EAGLE 投机解码时，`numAcceptedTokens` 超过变长段长度导致的算子崩溃。
    *   **Disaggregated Prefill (P/D) 请求清理**（#14142）：修复了分离式预填路径中被拒绝请求的清理逻辑，并对非 400 错误增加重试。
    *   **KV Transfer 队列冲突**（#14132）：移植上游修复，解决 Ascend KV Store 中 save 与 load 共队列导致的异常。
*   **🔧 维护与重构**：
    *   **上游适配**（#14135, #14133）：持续适配 vLLM 主分支最新代码（截至8月11日），其中 #14133 适配失败。
    *   **CI 回滚**（#14137）：撤销了破坏端到端 CI 的多模态输入测试框架更新。
    *   **重构**（#14139）：fia tiling sink 逻辑重构。
    *   **文档**（#14155）：自动翻译更新了 14 个文档文件。

---

### 🚀 Release 动态

近期**无**新版本 Release 发布。

---

## 🐛 Issues

### #14160 — [[Contribution] [Perf][MRV2] _apply_grammar_bitmask_kernel 算子性能优化](https://github.com/vllm-project/vllm-ascend/issues/14160)
- **作者**: chengduxiaowu  **时间**: 2026-08-13 10:48 CST
- **摘要**: ## 背景  MRV2 结构化输出（structured output / grammar）路径中，`_apply_grammar_bitmask_kernel` 将 grammar bitmask 应用到 logits 上，把被屏蔽位置置为 -inf。Ascend 侧实现位于 `vllm_ascend/worker/v2/structured_outputs.py`，经 `patch/worker/patch_v2/patch_triton.py` 替换 vllm-core GPU 版本。  当前实现注释明确指出：上游 GPU 版 `BLOCK_SIZE=8192` 在 Ascend NPU…

### #14159 — [[已撤销] [Contribution] [Perf][MRV2] _prepare_dflash_inputs_kernel 算子性能优化](https://github.com/vllm-project/vllm-ascend/issues/14159)
- **作者**: chengduxiaowu  **时间**: 2026-08-13 10:48 CST
- **摘要**: ## 背景  MRV2 投机解码 DFlash 路径中，`_prepare_dflash_inputs_kernel` 为 draft 模型准备输入：context/query positions、slot mapping、input_ids、sample indices 等多组输出。Ascend 侧实现 `_prepare_dflash_inputs_kernel_ascend` 位于 `vllm_ascend/worker/v2/spec_decode/dflash/speculator.py`，经 `patch/worker/patch_v2/patch_triton.py` 替换 vl…

### #14158 — [[Contribution] [Perf][MRV2] bincount 算子性能优化](https://github.com/vllm-project/vllm-ascend/issues/14158)
- **作者**: chengduxiaowu  **时间**: 2026-08-13 10:48 CST
- **摘要**: ## 背景  MRV2 采样路径中，`bincount` 构建 prompt 的二进制 bitmask（`prompt_bin_mask`，标记 prompt 中出现过的 token）与输出 token 计数（`output_bin_counts`），供 `apply_penalties` 使用。Ascend 侧实现位于 `vllm_ascend/worker/v2/sample/penalties.py`。  当前实现为 Triton kernel `_bincount_kernel`，grid `(num_tokens, num_blocks)`，`BLOCK_SIZE=1024`。ker…

### #14157 — [[Contribution] [Perf][MRV2] apply_penalties 算子性能优化](https://github.com/vllm-project/vllm-ascend/issues/14157)
- **作者**: chengduxiaowu  **时间**: 2026-08-13 10:48 CST
- **摘要**: ## 背景  MRV2 采样路径中，`apply_penalties` 对 logits 施加 repetition / frequency / presence 惩罚。Ascend 侧实现位于 `vllm_ascend/worker/v2/sample/penalties.py`，经 `patch/worker/patch_v2/patch_triton.py` 替换 vllm-core GPU 版本。  当前实现为 Triton kernel `_penalties_kernel`，grid `(num_tokens, num_blocks)`，`BLOCK_SIZE=4096` 分块遍历…

### #14156 — [[Contribution] [Perf][MRV2] compute_token_logprobs 算子性能优化](https://github.com/vllm-project/vllm-ascend/issues/14156)
- **作者**: chengduxiaowu  **时间**: 2026-08-13 10:48 CST
- **摘要**: ## 背景  MRV2（Model Runner V2）采样路径中，`compute_token_logprobs` 负责对采样 token 计算 log-softmax 概率。Ascend 侧实现位于 `vllm_ascend/worker/v2/sample/logprob.py`，通过 `vllm_ascend/patch/worker/patch_v2/patch_triton.py` 替换 vllm-core 的 GPU 版本（`vllm/v1/worker/gpu/sample/logprob.py`）。  当前实现为 Triton kernel `_topk_log_softma…

### #14154 — [[Bug]: wy_fast.py: NameError: i_t is not defined in equal-length (cu_seqlens=None) path](https://github.com/vllm-project/vllm-ascend/issues/14154)
- **作者**: duan927  **时间**: 2026-08-13 10:19 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  Collecting environment information... PyTorch version: 2.10.0+cpu Is debug build: False  OS: Ubuntu 22.04.5 LTS (aarch64) GCC version: (Ubuntu 11.4.0-1ubuntu1~22.04.3) 11.4.0 Clang version: 15.0.7 CMake…

### #14153 — [[Performance][FlashComm1][TP8+EP] shared-expert multistream overlap causes large rank-local ReduceScatter producer skew](https://github.com/vllm-project/vllm-ascend/issues/14153)
- **作者**: luentong  **时间**: 2026-08-13 10:04 CST
- **标签**: performance
- **摘要**: ## Summary  When `enable_flashcomm1=true` and `multistream_overlap_shared_expert=true`, the TP8+EP W8A8 MoE workload shows a large rank-local delay before some FlashComm1 `ReduceScatter` operations.  The delay is not primarily inside HCCL after the collective has started. In the CANN trace, the dela…

## 🔀 Pull Requests

### #14162 — [[BugFix][Ops] Support dynamic W8A8 mm_reduce_scatter fusion](https://github.com/vllm-project/vllm-ascend/pull/14162)
- **作者**: Bill845514379  **时间**: 2026-08-13 11:09 CST
- **摘要**: ### What this PR does / why we need it?    This PR adds and fixes dynamic W8A8 support for `mm_reduce_scatter` fusion in row-parallel linear ops.    Main changes:    - Route dynamic W8A8 row-parallel linear through `npu_dynamic_quant` and `npu_mm_reduce_scatter_base` when fusion is enabled.   - Resh…

### #14161 — [[BugFix] Fix CausalConv1d crash when numAcceptedTokens exceeds varlen segment length](https://github.com/vllm-project/vllm-ascend/pull/14161)
- **作者**: scyiwei1986  **时间**: 2026-08-13 10:52 CST
- **标签**: module:ops
- **摘要**: ## What this PR does / why we need it Fixes a crash in the `aclnnCausalConv1d` operator when using MTP/EAGLE speculative decoding with hybrid Mamba-Transformer models (e.g., Qwen3.6) on Ascend NPU.  ### Problem The service crashes after running for a period of time with the following error: [ERROR] …

### #14155 — [[Doc] Translated Doc files 2026-08-13](https://github.com/vllm-project/vllm-ascend/pull/14155)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-13 10:46 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **14** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents/sfa_remote_d2h_connector.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES…

### #14142 — [[Bugfix][P/D] Clean up rejected requests and retry non-400 failures](https://github.com/vllm-project/vllm-ascend/pull/14142)
- **作者**: moonseeker1  **时间**: 2026-08-13 09:57 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This replaces #13929 with a branch that can be updated from the `moonseeker1` fork.  It fixes rejected-request cleanup in the disaggregated prefill path and applies the requested backend retry policy:  - release prefill KV and roll back scheduler accounting w…

### #14141 — [[Feature][Sampling] Add AscendC categorical sampling for MRV2](https://github.com/vllm-project/vllm-ascend/pull/14141)
- **作者**: freyfwt  **时间**: 2026-08-13 09:56 CST
- **标签**: documentation, ci/build, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  MRV2 currently relies on a Triton implementation for Gumbel sampling, which materializes full-vocabulary random values and does not provide one native sampling path shared by ordinary sampling and speculative rejection on Ascend.  This PR adds an AscendC cate…

### #14139 — [[Refactor] fia tiling sink](https://github.com/vllm-project/vllm-ascend/pull/14139)
- **作者**: hw-zhoutianyang  **时间**: 2026-08-13 09:38 CST
- **标签**: documentation, module:tests, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14137 — [Revert "[CI] Updating the multi-modal input testing feature in the nightly and weekly framework"](https://github.com/vllm-project/vllm-ascend/pull/14137)
- **作者**: wangxiyuan  **时间**: 2026-08-13 08:54 CST
- **标签**: module:tests, module:tools
- **摘要**: Reverts vllm-project/vllm-ascend#13751  This PR break e2e CI - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14135 — [[Misc]feat: adapt to vLLM main (5426311d)](https://github.com/vllm-project/vllm-ascend/pull/14135)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-13 05:16 CST
- **标签**: module:tests, module:ops, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to August 11.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | `tests/e2e/conftest.py` | [751f2ccd](https://github.com/vllm-project/vll…

### #14133 — [[Misc]feat: adapt to vLLM main (b1b75204)](https://github.com/vllm-project/vllm-ascend/pull/14133)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-13 00:29 CST
- **标签**: module:tests, module:ops
- **摘要**: main2main adaptation failed — no steps completed.  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14132 — [[BugFix][KVTransfer] Avoid co-queueing save with load in ascend store…](https://github.com/vllm-project/vllm-ascend/pull/14132)
- **作者**: Wfd567  **时间**: 2026-08-12 23:17 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This PR ports the root-cause fix from vllm-project/vllm#43371 to the Ascend KV store (`vllm_ascend/distributed/kv_transfer/kv_pool/ascend_store`).  **Root cause**  In `kv_both` mode, when a request can load its KV from the pool (`load_spec.can_load == True`),…
