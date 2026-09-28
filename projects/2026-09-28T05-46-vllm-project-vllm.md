# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-28 13:46 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue
1. **Qwen4Exp CUDA Graph 捕获 OOM 问题** (#58960)：报告在 `--gpu-memory-utilization 0.90` 下，Qwen4Exp 的 QSA key-cache views 持续占用 profiling KV cache，导致 CUDA graph 捕获时显存溢出（OOM）。
2. **强化学习中 Logprobs 对齐提案** (#58959)：提议返回与采样掩码对齐的 logprobs，以满足 RL 中 Top-p score centering 对完整行为策略分布的需求。

### 🔧 Pull Request
**缺陷修复**
* **#58961 [Qwen4Exp]**：修复了 QSA key views 未及时释放 profiling KV cache 导致的 CUDA graph OOM 问题（对应 Issue #58960）。
* **#58968 [KVConnector]**：修复了 Kimi-K3 在分离式 Prefill/Decode 结合推测解码时，KV 连接器引发的传输生命周期与流终止错误。
* **#58958 [Frontend]**：修复了 `/v1/chat/completions/batch` 接口未正确应用 Harmony parser `adjust_request` 的问题，确保其行为与普通对话接口一致。
* **#58963 [HiSparse]**：修复了当主机内存池填满时，活跃请求可能保留无主机条目的纯 GPU 页面，导致后续释放错误的问题。
* **#58967 [ROCm]**：修复 ROCm 环境下 Mooncake bootstrap 端口在启动过程中未保持绑定的问题，并优化了错误反馈机制。

**性能优化**
* **#58957 [Qwen4Exp]**：针对 NVIDIA GPU，将 BF16 HC 降维投影与 SiLU 算子进行融合，有效提升了 48 token 以内 decode 批次的处理性能。
* **#58965 [HiSparse]**：重构代码以避免重复的前缀扫描，改由连接器直接驱动内存驻留，提升整体效率。

**基础设施与新特性**
* **#58966 [Docker]**：在系统镜像中内置 AWS EFA 用户空间及相关依赖库，解决了此前镜像无法开箱即用支持 AWS EFA 的问题（修复 NCCL 回退及 NIXL 依赖缺失）。
* **#58962 [KVConnector]**：为 MoRIIO 的 WRITE 模式消费者新增了超时机制与块回收功能。
* **#58964 [CPU]**：在 `DiffusionGemma` 中改用 PyTorch accelerator memory API 获取内存信息，修复了该模型在 CPU 环境下的执行报错。

### 🚀 Release
* *近期暂无新版本发布动态。*

---

## 🐛 Issues

### #58960 — [[Bug]: Qwen4Exp QSA key-cache views keep the CUDA-graph profiling KV cache alive; CUDA graph capture OOMs at --gpu-memory-utilization 0.90](https://github.com/vllm-project/vllm/issues/58960)
- **作者**: lucifer1004  **时间**: 2026-09-28 11:16 CST
- **摘要**: ### Your current environment  - vLLM `main` @ a4eb3f25d6 (V2 model runner, breakable CUDA graphs) - 2x NVIDIA RTX PRO 6000 Blackwell Server Edition (96 GB), driver 595.58.03, torch 2.13.0+cu130 - Model: `nvidia/Qwen3.8-Flash-Next-NVFP4` (`Qwen4ExpForConditionalGeneration`), TP2  (On this host the mo…

### #58959 — [[RFC]: Return logprobs aligned with sampling-mask token IDs for score centering](https://github.com/vllm-project/vllm/issues/58959)
- **作者**: aoshen02  **时间**: 2026-09-28 10:54 CST
- **标签**: RFC
- **摘要**: ### Motivation  Top-p score centering in RL needs the complete behavior-policy distribution on each generated token's sampling support: the retained token IDs and their corresponding normalized log probabilities. The sampled token's logprob alone is insufficient to compute the probability-weighted c…

## 🔀 Pull Requests

### #58968 — [[Bugfix][KVConnector] Fix K3 PD transfer lifetimes and stream finalization](https://github.com/vllm-project/vllm/pull/58968)
- **作者**: whx-sjtu  **时间**: 2026-09-28 13:37 CST
- **标签**: bug, tool-calling, kv-connector, mrv2, kimi, k3, scheduler
- **摘要**: ## Purpose  Fix native vLLM failures encountered while bringing up Kimi-K3 disaggregated prefill/decode with speculative decoding and multiple KV connectors. This PR is based on main and contains vLLM source and regression tests only.  ```text worker completions (child, rank)   -> deduplicate across…

### #58967 — [[Bugfix][ROCm] Keep Mooncake bootstrap ports bound during startup](https://github.com/vllm-project/vllm/pull/58967)
- **作者**: AndreasKaratzas  **时间**: 2026-09-28 13:28 CST
- **标签**: bug, documentation, rocm, kv-connector
- **摘要**: - Bind the bootstrap socket before starting Uvicorn and keep that listener open. - Report bind errors immediately and stop waiting when startup fails. - Let the ROCm test own a running bootstrap server and pass its endpoint to the workers and proxy. - Cover port ownership, cleanup, startup failures,…

### #58966 — [[Docker] Ship AWS EFA userspace alongside the system rdma-core](https://github.com/vllm-project/vllm/pull/58966)
- **作者**: whn09  **时间**: 2026-09-28 12:50 CST
- **标签**: ci/build
- **摘要**: ## Purpose Addresses #55635. The vLLM images can't use AWS EFA out of the box: NIXL's `LIBFABRIC` backend is missing both the EFA userspace and `libhwloc.so.15`, NCCL falls back to sockets, and Mooncake needs the EFA wheel plus the EFA userspace.  Running `aws-efa-installer` in the image isn't a goo…

### #58965 — [[Perf][HiSparse] Avoid repeated prefix scans; drive residency from the connector](https://github.com/vllm-project/vllm/pull/58965)
- **作者**: LucasWilkinson  **时间**: 2026-09-28 12:10 CST
- **标签**: kv-connector, kv-cache-manager
- **摘要**: Alternative to #57930, following the review thread at https://github.com/vllm-project/vllm/pull/57930#discussion_r4068549511. It keeps @S1ro1's prefix-scan cursor fix and folds in the restructure from S1ro1/vllm#9. That restructure removes the `resident_managers[0]` identity check that @NickLucche f…

### #58964 — [[CPU] Use accelerator memory API in DiffusionGemma](https://github.com/vllm-project/vllm/pull/58964)
- **作者**: zhejiangxiaomai  **时间**: 2026-09-28 11:59 CST
- **摘要**: ## Summary  Fix CPU execution in `DiffusionGemma` by using the supported PyTorch accelerator memory API when calculating the decode tile memory budget.  `current_platform.mem_get_info()` is not available on CPU, which caused inference to fail with:  ```text Current platform cpu does not have 'mem_ge…

### #58963 — [[Bugfix][HiSparse] Recover host backing for active GPU-only pages](https://github.com/vllm-project/vllm/pull/58963)
- **作者**: eopXD  **时间**: 2026-09-28 11:23 CST
- **标签**: bug, kv-connector, kv-cache-manager
- **摘要**: ## Description  When the HiSparse host pool fills, an active request can retain GPU-only pages with null host entries when no host page is available. Later allocations extend the host table, so freeing host capacity does not give those old pages backing and their GPU blocks stay pinned.  Retry one o…

### #58962 — [[KVConnector][MoRIIO] Add WRITE-mode consumer timeout and block reclaim](https://github.com/vllm-project/vllm/pull/58962)
- **作者**: amirakb89  **时间**: 2026-09-28 11:20 CST
- **标签**: needs-rebase, kv-connector
- **摘要**: In WRITE mode the MoRIIO decode side pushes-receives KV: the producer writes into decode blocks and signals completion out-of-band via a write_done notification. The decode consumer only ever reported completions that arrived, with no timeout. If that notification was lost (e.g. the producer QP hit …

### #58961 — [[Bugfix][Qwen4Exp] Release the profiling KV cache held by QSA key views](https://github.com/vllm-project/vllm/pull/58961)
- **作者**: lucifer1004  **时间**: 2026-09-28 11:17 CST
- **标签**: bug, qwen
- **摘要**: ## Purpose  Fixes #58960.  `QSAKeyStateCache.bind_kv_cache` stored the views `key_cache` and `rope_position_cache` next to `kv_cache`. `clear_layer_kv_caches` only resets `kv_cache`, so after CUDA graph memory profiling these views kept the profiling KV cache alive (9.76 GiB for Qwen3.8-Flash-Next-N…

### #58958 — [[Bugfix][Frontend] Apply Harmony adjust_request in batched chat completions](https://github.com/vllm-project/vllm/pull/58958)
- **作者**: sfeng33  **时间**: 2026-09-28 10:50 CST
- **标签**: bug, frontend, ready, gpt-oss
- **摘要**: ## Purpose  For gpt-oss (Harmony), `/v1/chat/completions/batch` skipped the Harmony parser's `adjust_request`, which `/v1/chat/completions` and `/v1/responses` both run. With a `json_schema` `response_format`, the raw schema then constrained the whole output rather than only the `final` channel. Bat…

### #58957 — [[Perf][Qwen4Exp] Fuse HC down projection and SiLU on NVIDIA](https://github.com/vllm-project/vllm/pull/58957)
- **作者**: gau-nernst  **时间**: 2026-09-28 10:29 CST
- **标签**: qwen, nvidia
- **摘要**: ## Purpose  Fuse Qwen4Exp's BF16 HC down projection and SiLU for NVIDIA decode batches up to 48 tokens, with the existing unfused path for larger batches. This is separate from #58706 (ROCm) and #53909 (other HC operations).  The CuTe DSL kernel adapts vLLM's `ll_bf16` dot-product and split-K GEMM p…
