# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-11 10:37 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
本期共报告 3 个 Bug，主要集中在 Mooncake 传输与数据并行（DP）场景：
*   **Mooncake 内存注册失败** (#13966)：Worker 在运行时出现 `Mooncake memory registration failed` 的 Runtimeerror。
*   **DeepSeek-V4 Flash PD + Mooncake + MTP 场景下 Decode 停止生成** (#13964)：在 v0.23.0rc1 版本中，开启上述组合时，Decode DP ranks 进程保持运行但停止了文本生成。
*   **Embedding 模型开启数据并行报错** (#13961)：使用 `--data-parallel-size 2` 启动 Embedding 模型（如 bge-m3）时，触发 `ObservabilityConfig` 无法 pickle 序列化的错误。

### 🔧 PR 动态
本期 PR 重点修复了 P/D 分离与 MTP 场景下的关键 Bug，并带来了性能优化及新特性：

*   **Bug 修复**
    *   **修复 Mooncake CP transfer group block IDs** (#13965, #13968)：修复了 Mooncake 请求元数据按 KV cache 组存储 block ID，而 CP 路径按 transfer 组迭代导致的逻辑不匹配问题。#13968 为其向 `v0.26.0rc` 的反向移植。
    *   **修复 MTP replay 边界及陈旧事件** (#13963)：在 KV Pool 中保留了 MTP replay 边界，并清理了陈旧的逐层保存事件，解决 GVA 逐层寻址下的问题。

*   **新特性**
    *   **MRV2 Sfa 支持图模式** (#13958)：为 MRV2 Sfa 增加了图模式支持，提升执行效率。
    *   **适配 vLLM main 分支** (#13955)：将 vLLM-Ascend 适配至最新的 vLLM main 分支（v0.26.0, commit 11ba93f3）。

*   **性能优化**
    *   **C128 partial load 性能优化** (#13967)：针对 C128 场景的部分加载性能进行了优化。
    *   **GDN chunk state 与 KV-cache 布局统一** (#13957)：将 Qwen GDN chunk-prefill recurrent state 与 KV-cache 的 `[N, H, V, K]` 布局统一，移除了全量状态的 `transpose` 操作，减少开销。

*   **文档与 CI**
    *   **新增 Pipeline Parallelism 指南** (#13962)：添加了流水线并行（PP）的英文使用文档，涵盖单节点、多节点多进程及 Ray 启动示例。
    *   **补充 GLM-5.2 DSpark 验收率基准** (#13959, #13960)：为 `test_glm5_2.py` 补充了缺失的验收率 golden 数据（分别合入 v0.26 分支和主分支）。

### 🚀 Release 动态
*   本期暂无新版 Release 发布。

---

## 🐛 Issues

### #13966 — [[Bug]: Runtimeerror:Mooncake memory registration failed](https://github.com/vllm-project/vllm-ascend/issues/13966)
- **作者**: TZP20020330tzjly  **时间**: 2026-08-11 10:24 CST
- **标签**: bug
- **摘要**: ### Your current environment   Worker_D2_TP0_EP8 pid=80495 INFO 08-05 09:42:09 [mooncake_connector.py:1258] block shape: torch.Size([128, 1, 128]) I0805 09:42:09.763092 80495 ascend direct transport.cpp:444 AscendDirectTransport register mem addr:0x12da73a00000, length:183369728, location:, mem type…

### #13964 — [[Bug]: v0.23.0rc1 Decode DP ranks keep running requests but stop generation under DeepSeek-V4 Flash PD + Mooncake + MTP](https://github.com/vllm-project/vllm-ascend/issues/13964)
- **作者**: zmc1997  **时间**: 2026-08-11 10:11 CST
- **标签**: advanced-features, mtp/speculative-decode, llm-model, deepseek
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Collecting environment information... PyTorch version: 2.10.0+cpu Is debug build: False  OS: openEuler 24.03 (LTS-SP3) (aarch64) GCC version: (GCC) 12.3.1 (openEuler 12.3.1-105.oe2403sp3) Clang …

### #13961 — [[Bug]: Embedding model使用--data-parallel-size 2启动报错：_pickle.PicklingError: Can't pickle <class 'vllm.config.observability.ObservabilityConfig'>: it's not the same object as vllm.config.observability.ObservabilityConfig](https://github.com/vllm-project/vllm-ascend/issues/13961)
- **作者**: holyzhang828  **时间**: 2026-08-11 09:57 CST
- **标签**: bug
- **摘要**: ### Your current environment  开启DP双实例 python:312 硬件310P  ### 🐛 Describe the bug  启动命令：vllm serve  bge-m3  --host 127.0.0.1 --port 8042 --runner pooling --dtype float16 --data-parallel-size 2 --max-model-len 512 --max-num-seqs 32  报错： [INFO] 08-11 09:17:07.116 [PID:35876] utils.py:1104 Started DP Coo…

## 🔀 Pull Requests

### #13968 — [[v0.26.0rc][BugFix][P/D] Fix Mooncake CP transfer group block IDs](https://github.com/vllm-project/vllm-ascend/pull/13968)
- **作者**: Yuli-yx  **时间**: 2026-08-11 10:36 CST
- **摘要**: ### What this PR does / why we need it?  Backport of #13965 to releases/v0.26.0rc.  Fixes #13934.  Mooncake request metadata stores block IDs by KV cache group, while the CP path iterates transfer groups. GLM-5.2 splits one KV cache group into multiple transfer groups, so indexing remote_block_ids/l…

### #13967 — [opt C128 partial load perf](https://github.com/vllm-project/vllm-ascend/pull/13967)
- **作者**: UpDown9  **时间**: 2026-08-11 10:31 CST
- **摘要**: <!--  Thanks for sending a pull request!  BEFORE SUBMITTING, PLEASE READ https://docs.vllm.ai/en/latest/contributing/overview.html  --> ### What this PR does / why we need it? <!-- - Please clarify what changes you are proposing. The purpose of this section is to outline the changes and how this PR …

### #13965 — [[BugFix][P/D] Fix Mooncake CP transfer group block IDs](https://github.com/vllm-project/vllm-ascend/pull/13965)
- **作者**: Yuli-yx  **时间**: 2026-08-11 10:13 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Fixes #13934.  Mooncake request metadata stores block IDs by KV cache group, while _get_kv_split_metadata iterates Mooncake transfer groups. When one KV cache group is split into multiple transfer groups, the CP path incorrectly uses the transfer group index …

### #13963 — [[BugFix][KV Pool] Preserve MTP replay boundary and drain stale layerwise save events](https://github.com/vllm-project/vllm-ascend/pull/13963)
- **作者**: tyy0829  **时间**: 2026-08-11 10:04 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  main already has the GVA layerwise addressing for MTP (group_layer_cache_entry_offsets, prefix-byte-sum rank_layer_offset, MTP-first physical layer index, save-side kvpool_store_skip_tokens). Two pieces from the v0.23.0 layerwise-MTP fixes were still missing …

### #13962 — [[Doc] Add Pipeline Parallelism guide](https://github.com/vllm-project/vllm-ascend/pull/13962)
- **作者**: LostFox11  **时间**: 2026-08-11 10:04 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Adds an English Pipeline Parallelism feature guide that:  - provides single-node, two-node multiprocessing, and Ray startup examples near the beginning - explains TP/PP/DP topology planning and custom layer partitioning with `VLLM_PP_LAYER_PARTITION` - docume…

### #13960 — [[CI] Add GLM-5.2 DSpark acceptance rate golden](https://github.com/vllm-project/vllm-ascend/pull/13960)
- **作者**: wangbj127  **时间**: 2026-08-11 09:52 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it? `test_glm5_2.py` missed acceptance rate golden.  ### Does this PR introduce _any_ user-facing change? No.  ### How was this patch tested? Existing CI tests all pass.  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/0351e9aa1fdf…

### #13959 — [[CI][v0.26] Add GLM-5.2 DSpark acceptance rate golden](https://github.com/vllm-project/vllm-ascend/pull/13959)
- **作者**: wangbj127  **时间**: 2026-08-11 09:52 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it? `test_glm5_2.py` missed acceptance rate golden.  ### Does this PR introduce _any_ user-facing change? No.  ### How was this patch tested? Existing CI tests all pass.  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/d02df748bf9e…

### #13958 — [[Feature] MRV2 Sfa support graph mode](https://github.com/vllm-project/vllm-ascend/pull/13958)
- **作者**: SunnyLee151064  **时间**: 2026-08-11 09:24 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/0351e9aa1fdf1a51329d1906881528dfe61fc88e

### #13957 — [[Perf][GDN] Unify chunk state with KV-cache layout](https://github.com/vllm-project/vllm-ascend/pull/13957)
- **作者**: liuyao0  **时间**: 2026-08-11 09:22 CST
- **标签**: module:tests, module:ops
- **摘要**: ## What this PR does / why we need it?  Unifies Qwen GDN chunk-prefill recurrent state with the KV-cache `[N, H, V, K]` layout on the v0.23.0rc1 baseline.  - Removes the full state `transpose(...).contiguous()` operations before and after chunk prefill. - Maps cache `[V, K]` state to the internal `[…

### #13955 — [[Misc]feat: adapt to vLLM main (11ba93f3)](https://github.com/vllm-project/vllm-ascend/pull/13955)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-11 01:08 CST
- **标签**: module:tests, module:ops
- **摘要**: main2main adaptation failed — no steps completed.  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/0351e9aa1fdf1a51329d1906881528dfe61fc88e
