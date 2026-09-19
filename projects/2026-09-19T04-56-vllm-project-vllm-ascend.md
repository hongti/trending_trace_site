# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-19 12:56 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### Issue 动态
近期主要收到 2 个高优先级 Bug 报告：
*   **混合 GDN 模型序列化问题** (#16916)：在 Atlas 800 A3 上服务 Qwen3.6-35B-A3B（INT4 W4A16）时，开启 `--enable-prefix-caching` 及 mamba cache 模式下，默认的 `max_num_batched_tokens=2048` 导致长 Prefills 被序列化，影响性能。
*   **MegaMoe 崩溃与乱码** (#16914)：在 A3 上使用 PD 分离架构（4P4D）部署 Kimi K3 并开启 MegaMoe 相关算子融合时，产生乱码输出并引发 vector-core 崩溃（acl error 507035）。

### PR 动态
近期 PR 主要集中在关键 Bug 修复、性能优化以及 CI/测试维护：

**1. 重要 Bug 修复**
*   **兼容性与算子修复**：修复在 pre-950 NPUs 上加载原生 block FP8 权重时与 CANN 的兼容问题 (#16922)。
*   **Attention 与 SFA 修复**：修复 GLM5Next 在 Ascend A5 上的 NoPE sparse MLA 路径的 `sinks` 张量和 mask/window 参数问题 (#16918)；拆分混合 KDA/MLA 缓存对齐时产生的超限 A5 Sparse MLA 缓存页 (#16913)。
*   **调度与投机解码修复**：修复 `SchedulerDynamicBatch` 使用已废弃的 `KVCacheManager` API 的问题 (#16911)；修复 MiniMax-M2.5 + Eagle3 MRV2 夜间测试中投机解码的重采样 RNG 耦合 Bug (#16920)。

**2. 性能优化**
*   **Triton 算子优化**：通过引入 CANN extract/insert_slice 实现单 tile load/store，大幅减少 `_triton_rope_siso` 内核的全局内存访问开销 (#16917)。
*   **Attention 开销降低**：减少大型 DSA-CP prefill 过程中冗余的缓存收集和隐藏状态聚合操作，降低 prefill 开销 (#16915)。

**3. CI 与测试**
*   为 Qwen3-32B-QuaRot 夜间测试启用 Model Runner V2 (#16919)。
*   更新 DeepSeek V4 Pro 和 GLM 5.1 的外部 DP 夜间性能基线 (#16912)。
*   自动翻译并合入 90 个中文文档文件 (#16921)。

### Release 动态
*   本期暂无 Release 发布。

---

## 🐛 Issues

### #16916 — [[Bug]: Hybrid GDN models (Qwen3.6-35B-A3B): long prefills serialize with default max_num_batched_tokens=2048 (<70GB branch) + mamba align mode](https://github.com/vllm-project/vllm-ascend/issues/16916)
- **作者**: Phoveran  **时间**: 2026-09-19 10:02 CST
- **摘要**: ### What happened?  Serving a hybrid GDN model (**Qwen3.6-35B-A3B**, INT4 W4A16, 40 layers: linear-attention/GDN + full attention) on Atlas 800 A3 with `--enable-prefix-caching` (mamba cache mode resolves to `align`): **concurrent long prefills are scheduled strictly one at a time.**  During a burst…

### #16914 — [[Bug] MegaMoe on Kimi K3 (A3, PD-disaggregated 4P4D) produces garbled output and vector-core crash (acl error 507035)](https://github.com/vllm-project/vllm-ascend/issues/16914)
- **作者**: linnea-lin-00638949  **时间**: 2026-09-19 00:51 CST
- **摘要**: # [Bug] MegaMoe on Kimi K3 (A3, PD-disaggregated 4P4D) produces garbled output and vector-core crash (acl error 507035)  ## Summary  When enabling MegaMoe (`enable_fused_mc2: 2` + `enable_prefill_mc2: true`) for Kimi K3 W4A8 on Atlas 800I A3, **every request returns garbled tokens**, and the prefill…

## 🔀 Pull Requests

### #16922 — [[BugFix][Quantization] Load native block FP8 weights on pre-950 NPUs](https://github.com/vllm-project/vllm-ascend/pull/16922)
- **作者**: U1stRsouland  **时间**: 2026-09-19 12:47 CST
- **标签**: module:tests, module:quantization
- **摘要**: ### What this PR does / why we need it?  Native block-wise FP8 checkpoints (`quant_method: "fp8"`) store E4M3 weights together with FP32 `weight_scale_inv` tensors. On pre-950 Ascend NPUs, some CANN builds cannot cast FP8 E4M3 weights to FP32 on device, so real checkpoint loading fails in `resolve_b…

### #16921 — [[Doc] Translated Doc files 2026-09-19](https://github.com/vllm-project/vllm-ascend/pull/16921)
- **作者**: vllm-ascend-ci  **时间**: 2026-09-19 12:35 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **90** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/slash-commands.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/user_stories/index.po<…

### #16920 — [[BigFIx][CI] Fix a resample bug and enable mrv2 in Mimimax M2.5](https://github.com/vllm-project/vllm-ascend/pull/16920)
- **作者**: AuroraEmiya  **时间**: 2026-09-19 12:23 CST
- **标签**: module:tests, module:ops
- **摘要**: <!--  Thanks for sending a pull request!  BEFORE SUBMITTING, PLEASE READ https://docs.vllm.ai/en/latest/contributing/overview.html  --> ### What this PR does / why we need it?  This PR contains two related fixes for the MiniMax-M2.5 + Eagle3 MRV2 nightly path:  1. **Fix residual resampling RNG coupl…

### #16919 — [[CI] Enable model runner V2 for Qwen3-32B-QuaRot nightly](https://github.com/vllm-project/vllm-ascend/pull/16919)
- **作者**: CXY-Katrina  **时间**: 2026-09-19 11:34 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Add `VLLM_USE_V2_MODEL_RUNNER: "1"` to the `envs` of `Qwen3-32B-QuaRot-eagle3.yaml` so the Qwen3-32B-QuaRot single-node nightly case on `linux-aarch64-nightly-a3-2` exercises model runner V2. This is a one-line addition. Other jobs sharing this YAML also inhe…

### #16918 — [[BugFix][SFA] Fix GLM5Next sparse MLA sinks and mask/window parameters](https://github.com/vllm-project/vllm-ascend/pull/16918)
- **作者**: zzzzzz198  **时间**: 2026-09-19 11:21 CST
- **摘要**: ### What this PR does / why we need it? This PR fixes the GLM5Next NoPE sparse MLA path on Ascend A5.  Two issues are addressed:  1. `SparseFlashMla` requires a per-head `sinks` tensor, but the GLM5Next SFA caller does not provide one, causing the host-side error  `sinks must be provided`.     Add a…

### #16917 — [[Optimize][Triton][Ascend] Reduce global memory traffic in _triton_rope_siso via single-tile load/store with CANN extract/insert_sliceTrition rope siso opt](https://github.com/vllm-project/vllm-ascend/pull/16917)
- **作者**: like-0517  **时间**: 2026-09-19 10:08 CST
- **标签**: documentation, module:tests, module:ops, merge-conflicts
- **摘要**: ### What this PR does / why we need it? The `_triton_rope_siso` kernel currently issues two separate global loads (qk left half + right half) followed by two separate stores, and computes address offsets/masks on every token loop iteration. This PR reduces global memory traffic and eliminates redund…

### #16915 — [[Performance][Attention] Reduce DSA-CP prefill overhead](https://github.com/vllm-project/vllm-ascend/pull/16915)
- **作者**: lrf-vm  **时间**: 2026-09-19 01:54 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Large DSA-CP prefill performs separate main/indexer cache gathers, gathers hidden states that attention immediately shards again, and reduce-scatters a full output containing each rank's complete local projection. Generic scatter cache writes add substantial …

### #16913 — [[BugFix][Attention] Split oversized A5 Sparse MLA cache pages](https://github.com/vllm-project/vllm-ascend/pull/16913)
- **作者**: YanpengDing  **时间**: 2026-09-18 23:53 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Hybrid KDA/MLA cache alignment can produce attention storage pages larger than the sparse attention operator limit. For GLM5Next with DP8/TP1, the storage page is 4352 tokens, while SparseFlashMla accepts an ori_kv block size of at most 1024.  The A2/A3 path …

### #16912 — [[Test]Update nightly performance baselines](https://github.com/vllm-project/vllm-ascend/pull/16912)
- **作者**: czydyy  **时间**: 2026-09-18 23:43 CST
- **标签**: module:tests
- **摘要**: Update the DeepSeek V4 Pro and GLM 5.1 external DP performance baselines to match the latest measured results.  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/8403…

### #16911 — [[Scheduler][BugFix] Fix stale KVCacheManager API in SchedulerDynamicBatch](https://github.com/vllm-project/vllm-ascend/pull/16911)
- **作者**: AlekseiPol  **时间**: 2026-09-18 23:20 CST
- **摘要**: ## What this PR does / why we need it  Fixes `SchedulerDynamicBatch` using the removed `KVCacheManager.create_empty_block_list()` API.  When scheduling a waiting request with `num_computed_tokens > 0`, for example after an asynchronous KV receive in P/D disaggregated inference, `SchedulerDynamicBatc…
