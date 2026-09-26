# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-26 13:21 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文简洁摘要：

### 🐛 Issues (问题与讨论)
本期共有 3 个重要 Issue，主要涉及大模型推理结果损坏、KV-cache 生命周期管理及底层算子映射错误：
*   **GLM-5.3 输出损坏 (#17541)**：在 A3 P/D 架构下，8192 长度的 prefill 批次导致输出损坏。根因分析（RCA）指出，这是由于 fused-MoE 接收端容量设置过小（`mega_moe_max_tokens` 设为 8192，而默认应为 65536）导致 6,816 token 损坏。
*   **僵尸 KV-cache 释放 RFC (#17537)**：提出“感知会话的僵尸 KV-cache 释放（KVGC）”方案。在多轮对话中，若客户端重写或压缩历史，原有保留的 prefix cache 会变成“僵尸”缓存，需探讨更合理的回收机制。
*   **GLM-5-Next 尾槽映射失效 (#17532)**：#16925 引入的循环短路机制导致 GLM-5-Next KPool 尾槽映射在 fused path（MTP 起草器）外失效，导致 nightly 测试全部失败。

---

### 🛠 Pull Requests (代码提交)
本期 PR 主要集中在 Bug 修复、性能优化及架构重构上：

**重要 Bug 修复**
*   **MRV2 K/V cache 视图注册 (#17539)**：修复了在 Model Runner V2 下，Ascend 分离的 K 和 V 视图未能全部注册到 KV connectors 导致的 cache offloading/reloading 异常。
*   **Sparse MLA 计划重建 (#17535)**：修复 MTP drafts 中的内核索引越界问题，强制 `sparse_mla` 必须基于实际接收的张量来生成算子计划。
*   **Fused Slot Mapping 修复 (#17530, #17529)**：修复负位置未在所有缓存组中填充的问题；同时限制了 JIT 特化数量并预热，解决 `(num_reqs, tile)` 组合过多导致的性能下降。

**性能优化**
*   **MiniMax-M3 DP 空闲冗余跳过 (#17538)**：在 V2 model runner 的数据并行推理中，跳过空闲 rank 重复执行的 idle-DP indexer 工作，提升服务性能。

**重构与新特性**
*   **KV cache 机制重构 (#17533)**：弃用 vllm-ascend 内部针对 fp8/int8 KV cache 的三个 monkey-patch，改用 vLLM 原生的可插拔 `kv-cache-dtype` 机制。
*   **算子重命名 (#17540)**：将 AscendC 的 `chunk_fwd_o` 算子重命名为 `chunk_fwd_o_vllm`，避免与上游 Triton 实现产生命名冲突。
*   **Dockerfile 更新 (#17534)**：在 Docker 镜像中安装 `flash-linear-attention-npu`。

**CI 与测试**
*   **CI 解耦与测试 (#17531, #17536)**：为 `v0.27.1rc` 版本移除镜像合并任务中对 docker daemon 的依赖；并添加了相关插件测试。

---

### 🚀 Release (版本发布)
*   **暂无正式 Release 发布**。
*   *注*：从 PR #17531 可以看出，团队目前正在进行 **v0.27.1rc（候选发布版）** 的 CI/CD 流程优化与筹备工作。

---

## 🐛 Issues

### #17541 — [[Bug][GLM-5.3] A3 P/D: 8192 prefill batch corrupts output; 919K middle retrieval still wrong at 4096](https://github.com/vllm-project/vllm-ascend/issues/17541)
- **作者**: ZhengDeL  **时间**: 2026-09-26 11:20 CST
- **摘要**: > **RCA update (2026-09-26):** The 6,816-token corruption is caused by an undersized fused-MoE receiver capacity: we explicitly set `mega_moe_max_tokens=8192` where the running build defaults to `65536`. Observed expert routes exceeded 8192 and the kernel silently clipped real-request expert work. W…

### #17537 — [[RFC]: Session-aware zombie KV-cache release (KVGC)](https://github.com/vllm-project/vllm-ascend/issues/17537)
- **作者**: gobest  **时间**: 2026-09-25 17:40 CST
- **标签**: RFC
- **摘要**: ### Motivation.  In multi-turn chat services, vLLM's prefix cache deliberately keeps the KV cache of finished requests as eviction candidates. When a client rewrites or compresses the conversation between turns (or the previous turn's output is superseded), the blocks from the previous turn can no l…

### #17532 — [[Bug]: #16925's circular short-circuit drops GLM-5-Next KPool tail slot mapping outside the fused path (MTP drafter); nightly test_glm5next_tail_slot_mapping_triton fails 4/4](https://github.com/vllm-project/vllm-ascend/issues/17532)
- **作者**: mashuiping  **时间**: 2026-09-25 13:58 CST
- **标签**: glm5, advanced-features, mtp/speculative-decode, llm-model
- **摘要**: ### Your current environment   <details> <summary>The output of `python collect_env.py`</summary>  ```text Collecting environment information... PyTorch version: 2.10.0+cpu Is debug build: False OS: Ubuntu 22.04.5 LTS (aarch64) GCC version: 11.4.0 Python version: 3.12.13 (64-bit runtime) CPU: Kunpen…

## 🔀 Pull Requests

### #17540 — [[Refactor][Misc] Rename chunk_fwd_o operator to chunk_fwd_o_vllm](https://github.com/vllm-project/vllm-ascend/pull/17540)
- **作者**: ZT-AIA  **时间**: 2026-09-26 10:59 CST
- **标签**: ci/build, module:tests, module:ops, ready-precise
- **摘要**: ### What this PR does / why we need it?  Rename the AscendC `chunk_fwd_o` operator to `chunk_fwd_o_vllm` to avoid naming conflicts with the upstream Triton implementation, following the same naming convention as `causal_conv1d_v310`.  - Rename operator directory `csrc/moe/chunk_fwd_o` -> `csrc/moe/c…

### #17539 — [[BugFix][MRV2] Register every K/V cache view with KV connectors](https://github.com/vllm-project/vllm-ascend/pull/17539)
- **作者**: Liuchenbing-2026  **时间**: 2026-09-25 22:27 CST
- **标签**: module:tests, merge-conflicts, ready-precise
- **摘要**: ### What this PR does / why we need it?  KV cache offloading stores and reloads each KV cache view separately. Ascend keeps K and V in separate views, but under Model Runner V2 the KV cache map that reaches `get_kv_connector` was flattened to a single tensor per layer, because the runner filters tha…

### #17538 — [[Performance][MiniMax-M3] Skip repeated MRV2 idle-DP indexer work](https://github.com/vllm-project/vllm-ascend/pull/17538)
- **作者**: HaoxinZong  **时间**: 2026-09-25 19:26 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  In data-parallel MiniMax-M3 serving, the V1 and V2 model runners represent an idle DP rank differently.  With the V2 model runner, an idle rank can execute an eager, padding-only, one-token decode forward while another DP rank is processing a real prefill. Th…

### #17536 — [[Test]add plugin](https://github.com/vllm-project/vllm-ascend/pull/17536)
- **作者**: jiangyunfan1  **时间**: 2026-09-25 16:49 CST
- **标签**: ci/build, module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/ced6857afa0ea7b2e3f0846a62e1394e90f15607

### #17535 — [[BugFix][Attention] Rebuild the sparse MLA plan from the passed indices for MTP drafts](https://github.com/vllm-project/vllm-ascend/pull/17535)
- **作者**: Drosrin  **时间**: 2026-09-25 16:39 CST
- **标签**: module:tests, ready-a5, ready-precise
- **摘要**: ### What this PR does / why we need it?  `sparse_mla` must generate the operator plan from the very tensors the operator call receives — a plan built over different ones makes the kernel index past what it was handed. It only regenerated the plan when the query was trimmed to the unpadded row count,…

### #17534 — [[CI][Feature] Install flash-linear-attention-npu in Dockerfiles](https://github.com/vllm-project/vllm-ascend/pull/17534)
- **作者**: underfituu  **时间**: 2026-09-25 16:36 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it?  Resubmits the changes from #17503 with the same file content and diff, using the contributor's signed-off commit identity. The Dockerfiles install flash-linear-attention-npu for the supported image variants, and the scheduled image workflow is updated accordi…

### #17533 — [[Refactor]Adopt vLLM pluggable kv-cache-dtype mechanism for fp8/int8 KV cache](https://github.com/vllm-project/vllm-ascend/pull/17533)
- **作者**: lcfenglinwan  **时间**: 2026-09-25 15:09 CST
- **标签**: module:core
- **摘要**: ### What this PR does / why we need it? Replaces vllm-ascend's three monkey-patches for the fp8/int8 KV-cache dtypes with vLLM's pluggable kv-cache-dtype mechanism  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?   - vLLM main: https://github.com/vllm-project/vll…

### #17531 — [[releases/v0.27.1rc][CI] remove docker daemon dependency from image merge jobs](https://github.com/vllm-project/vllm-ascend/pull/17531)
- **作者**: zhangxinyuehfad  **时间**: 2026-09-25 13:36 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? remove docker daemon dependency from image merge jobs  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/ba07e4a48fc951300d97eb506217dd530583dea3

### #17530 — [[BugFix] Pad negative positions in every group of the fused slot mapping](https://github.com/vllm-project/vllm-ascend/pull/17530)
- **作者**: mashuiping  **时间**: 2026-09-25 13:19 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Fixes #17528.  `positions` is shared by every KV-cache group of a fused slot-mapping launch, but since #16537 the fused kernels pad negative positions only in circular groups. In a mixed circular/non-circular layout (DSV4.1's compressor state, GLM-5-Next's in…

### #17529 — [[BugFix][Performance] Bound fused slot-mapping JIT specializations and prewarm them](https://github.com/vllm-project/vllm-ascend/pull/17529)
- **作者**: mashuiping  **时间**: 2026-09-25 13:19 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Fixes #17527.  The fused multi-group slot-mapping kernels from #15289 take `NUM_REQS` and `PARALLEL_TILES` as `tl.constexpr`. Every new `(num_reqs, tile)` pair seen at runtime therefore compiles a new Triton specialization on the hot path (~5.5 s bisheng comp…
