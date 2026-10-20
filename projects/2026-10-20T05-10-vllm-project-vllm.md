# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-20 13:10 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的简明摘要：

### 🐛 Issue 动态
*   **设备兼容性报错 (#62609)**：有开发者反馈，在非 Blackwell 架构的 A100 上运行 Bolt 模型（NVIDIA-Nemotron-3.5-Bolt）时，触发了 “B12X NVFP4 kernels require a Blackwell 12x device” 硬件兼容性错误，问题指向了量化模块（`modelopt_block_quant.py`）的设备校验逻辑。

### 🔧 Pull Request 动态
本次 PR 主要集中在 **KV Offload 竞态修复**、**ROCm 性能优化**以及 **CI/构建完善**：

**1. 重要 Bug 修复**
*   **KV Offload 竞态与原子化 (#62615, #62611)**：修复了分层路径中批量查找的竞态条件。新逻辑使批量查找解析变为原子操作（无副作用探测所有键 -> 一次性锁定所有主命中项 -> 再启动次要层提升），防止早期未锁定的命中项被驱逐。
*   **Model Runner V2 采样修复 (#62612)**：修复了即使启用了 `--use-fp64-gumbel`，Model Runner V2 仍向 FlashInfer 发送未设种子的批次导致该参数被静默忽略的问题。

**2. 性能优化 (ROCm)**
*   **解码批次 GEMM 优化 (#62614)**：针对 ROCm BF16x3 router（如 MiniMax-M2/M3 使用的 fp32 门控权重），引入 Split-K BF16x3 router GEMM，解决了小批量（M ≤ 32）解码批次性能不佳的问题。
*   **Prefill 混合批次优化 (#62605)**：优化 `prefix_prefill`，使其能在所有 XCD 上运行混合批次并跳过空查询块，大幅改善了长 prefill 与短请求共存时的性能瓶颈。

**3. 构建、CI 与文档**
*   **CPU Triton 依赖修复 (#62608)**：发布带索引的 Triton CPU wheel 依赖，解决了 uv 拒绝 GitHub URL 依赖导致离线 wheelhouse 安装失败的问题。
*   **CI 修复 (#62616, #62607)**：为 XPU 测试添加 `kaldi-native-fbank` 依赖修复 Intel CI 失败；修复 DockerHub 清理任务中因过滤页面导致分页信号丢失、旧 nightly 版本未被正确保留/清理的问题。
*   **特性增强与文档 (#62613, #62610)**：为 KV Offload 的循环状态组（如 MambaSpec）发射自描述事件；更新 Kimi-K3 GB300 食谱文档，采用概率性 DSpark 采样。

### 🚀 Release 动态
*   本次提供的数据中**未包含版本发布（Release）**信息。

---

## 🐛 Issues

### #62609 — [[Bug]: A100 run Bolt (nv-community/NVIDIA-Nemotron-3.5-Bolt-74B-A9B-24GB) throw :B12X NVFP4 kernels require a Blackwell 12x device](https://github.com/vllm-project/vllm/issues/62609)
- **作者**: jack197906  **时间**: 2026-10-20 12:22 CST
- **标签**: bug, nvidia, quantization
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Your output of `python collect_env.py` here ```  </details>   ### 🐛 Describe the bug  please help check below issue, the version is:nightly-77154ca6116ae34cbfe314beef1016bde9b63989 s/…

## 🔀 Pull Requests

### #62616 — [[XPU][CI] Add kaldi-native-fbank to XPU test requirements](https://github.com/vllm-project/vllm/pull/62616)
- **作者**: chaojun-zhang  **时间**: 2026-10-20 13:09 CST
- **摘要**: ## Overview  Add `kaldi-native-fbank` to the XPU test requirements, which `FireRedASR2Processor` needs. It is already in the cuda/rocm test requirements. Fixes Intel CI failure: https://buildkite.com/vllm/intel-ci/builds/14740#01a156e9-87c0-4a3e-8d41-4d921d8fb0a5  ## Claims  - `test_model_tensor_sch…

### #62615 — [[Bugfix][KV Offload] Pin batch lookup HITs before secondary-tier promotions](https://github.com/vllm-project/vllm/pull/62615)
- **作者**: Alex-ai-future  **时间**: 2026-10-20 13:05 CST
- **标签**: bug
- **摘要**: ## Overview  Fix a batch-lookup race in the tiering path: probing one key can start a secondary-tier promotion that evicts an earlier, not-yet-pinned primary HIT of the same `LookupMsg`, so the HIT reported by the probe later fails in `create_store_job()`. The fix probes the whole batch first, pins …

### #62614 — [[ROCm][Perf] Split-K BF16x3 router GEMM for decode batches](https://github.com/vllm-project/vllm/pull/62614)
- **作者**: jiacao-amd  **时间**: 2026-10-20 13:04 CST
- **标签**: rocm
- **摘要**: ## Purpose  The ROCm BF16x3 router tier (`bf16x3_router_gemm_rocm`, fp32 gate weights as in MiniMax-M2/M3) only serves batches of 2048+ tokens. Decode batches above the gfx950 fp32 low-M kernel (M ≤ 32) and below 2048 fall back to `F.linear(x.float(), w)` on the fp32 pipe, which is both slower and l…

### #62613 — [[KV Offload] Emit self-describing events for recurrent state groups](https://github.com/vllm-project/vllm/pull/62613)
- **作者**: luckyq  **时间**: 2026-10-20 12:56 CST
- **标签**: documentation, kv-connector
- **摘要**: ## Purpose  `OffloadingEventsTracker` emits a `block_size=0` placeholder `BlockStored` for every group that has a load window. `MambaSpec` groups get a one-chunk window because they depend on a single state, not because their hashes are meaningless: the offloaded chunk is the recurrent state checkpo…

### #62612 — [[Bugfix][Model Runner V2] Honor use_fp64_gumbel when FlashInfer sampling is available](https://github.com/vllm-project/vllm/pull/62612)
- **作者**: cycancode  **时间**: 2026-10-20 12:45 CST
- **标签**: bug, mrv2
- **摘要**: ## Overview  With `--use-fp64-gumbel`, Model Runner V2 still sends unseeded top-k/top-p batches to FlashInfer's fused sampler, which draws fp32 noise, so the flag is silently ignored for those steps. This skips the FlashInfer sampler when fp64 Gumbel is requested, matching the V1 sampler and MRV2's …

### #62611 — [[Bugfix][KV Offload] Make batch lookup resolution atomic](https://github.com/vllm-project/vllm/pull/62611)
- **作者**: Alex-ai-future  **时间**: 2026-10-20 12:35 CST
- **标签**: bug
- **摘要**: ## Overview  Make batch lookup resolution atomic: probe every key without side effects, pin all primary HITs in one all-or-nothing call, then start promotions. Four commits: atomic `_prepare_load`, the new `ParentManager` batch contract, the KVCR migration, and per-lookup failure containment.  Relat…

### #62610 — [[Docs] Use probabilistic DSpark sampling in Kimi-K3 GB300 recipes](https://github.com/vllm-project/vllm/pull/62610)
- **作者**: RohitNagraj  **时间**: 2026-10-20 12:34 CST
- **标签**: ready, verified, dflash, kimi, k3
- **摘要**: ## Overview  Use `draft_sample_method: probabilistic` in the Kimi-K3 GB300 TP8/TP16 low-latency guide script, matching the aggregated configurations in [InferenceX #4047](https://github.com/SemiAnalysisAI/InferenceX/pull/4047).  ## Claims  - Corrects the documented draft sampling method for both agg…

### #62608 — [[Build][CPU] Publish indexed Triton CPU wheel dependencies](https://github.com/vllm-project/vllm/pull/62608)
- **作者**: ztysth  **时间**: 2026-10-20 12:13 CST
- **标签**: documentation, ci/build, cpu
- **摘要**: ## Overview  CPU wheels expose Triton as a GitHub URL dependency, which uv rejects when resolving vLLM from an index and which prevents offline wheelhouse installation. Publish the same CPU Triton build as an indexed dependency and retain its exact version and platform conditions. Fixes #62322.  ## …

### #62607 — [[CI] Preserve DockerHub cleanup pagination across filtered pages](https://github.com/vllm-project/vllm/pull/62607)
- **作者**: ztysth  **时间**: 2026-10-20 11:50 CST
- **标签**: ci/build
- **摘要**: ## Overview  DockerHub cleanup stopped whenever a page contained no selected nightly tags, even if later pages remained. Follow the server pagination signal so old nightlies are included in the retention calculation, and abort before deletion if any page cannot be fetched or parsed.  ## Claims  - Em…

### #62605 — [[ROCm][Perf] prefix_prefill: run mixed batches on all XCDs and skip empty query blocks](https://github.com/vllm-project/vllm/pull/62605)
- **作者**: zhentaocc  **时间**: 2026-10-20 11:23 CST
- **标签**: rocm
- **摘要**: ## Overview  `context_attention_fwd` in `prefix_prefill.py` runs `ROCM_ATTN`'s prefills and spec-decode verifies. It gets much slower when a long prefill shares a batch with short requests, which is the normal mixed step of chunked prefill. This PR fixes two causes:  1. **Grid order.** The grid was …
