# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-06 14:45 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要：

### 🚀 Release
**v0.31.0 版本发布**
*   **版本亮点**：本次发布汇聚了 307 位贡献者的 717 次提交（其中包含 96 位新贡献者）。
*   **核心特性**：重点提升了 **DeepSeek-V4.1-Flash** 的性能，特别是引入了 FlashMLA mega attention 相关的优化。

---

### 🐛 Issue（问题反馈）
近期主要报告了 3 个影响特定硬件或配置的 Bug：
1.  **数据并行权重加载失败**：非 MoE（稠密）模型在使用 `--data-parallel-size > 1` 时，`vllm preload` 守护进程的 `dp_size` 键值不匹配，导致无法使用 IPC 缓存加载。
2.  **前缀缓存导致输出损坏**：在 vLLM 0.30/0.31 版本中，Qwen3.8-27B NVFP4 模型使用 DFlash2/DSpark 及前缀缓存时，缓存命中后输出会损坏（0.29 版本正常）。
3.  **Blackwell 显卡共享内存溢出**：在 sm_120 架构 GPU（如 RTX PRO 6000 / RTX 5090）上，TRITON_MLA 解码阶段所需的共享内存超出了硬件限制，导致解码失败。

---

### 🔧 Pull Request（代码合并）
本次 PR 主要围绕上述 Bug 修复、性能优化及文档完善展开：

**1. 重要 Bug 修复**
*   **#60179**：修复非 MoE 模型在数据并行下的权重缓存键值问题，将守护进程的键值统一标记为 `DP=1`（对应 Issue #60178）。
*   **#60173**：针对共享内存 <128 KiB 的 GPU（如 sm_120），调整 Triton MLA 解码仅使用单阶段，解决显存溢出问题（对应 Issue #60172）。
*   **#60181**：修复 NIXL 在处理具有多个 KV 缓存组或 KV 共享层的模型时，主机到设备（H2D）同步覆盖接收到的 KV 缓存的问题。
*   **#60180**：修复前端 `--reasoning-parser hf` 始终将推理 token 计数报告为 0 的问题。
*   **#60177**：修复 Rust 基准测试客户端在跨越 HTTP 数据块和注释时丢失 SSE 负载的问题。

**2. 性能优化**
*   **#60169**：针对 gfx950 架构的 ROCm 平台，在 DeepSeek V4.1 的 32x32 MXFP8 线性层中启用 AITER FlyDSL GEMM 以提升性能。

**3. 功能与 CI 改进**
*   **#60170**：CPU Recipes 工具现在能结合模型 Attention 和 KV head 数量自动推荐合适的张量并行（TP）大小，避免不兼容配置。
*   **#60171**：在搭载 NVIDIA SM89（Ada 架构，如 L4, L40S, RTX 4090）的 GPU 上启用 Block FP8 量化测试。

**4. 文档更新**
*   **#60175**：补充了前缀缓存命中率（`vllm:prefix_cache_hits` / `queries`）的监控文档，并添加了 Grafana 仪表盘示例。
*   **#60176**：增加了关于 NIXL 节点之间非对称连接器配置导致有效 HMA 状态不匹配的文档说明。

---

## 🐛 Issues

### #60178 — [[Bug]: `vllm preload` daemon never matches a dense (non-MoE) engine with `--data-parallel-size > 1` (WeightCacheKey `dp_size` mismatch)](https://github.com/vllm-project/vllm/issues/60178)
- **作者**: nien-hui  **时间**: 2026-10-06 14:24 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text OS             : Ubuntu 22.04.3 LTS (x86_64) Python         : 3.12.14 PyTorch        : 2.13.0+cu130 (CUDA 13.0) transformers   : 5.17.0 GPUs           : 2x NVIDIA RTX A6000 Nvidia dri…

### #60174 — [[Bug] DFlash2/DSpark + prefix caching corrupt output after a cache hit on Qwen3.8-27B NVFP4 (compressed-tensors) on 0.30/0.31; 0.29, FP8 target and MTP are fine](https://github.com/vllm-project/vllm/issues/60174)
- **作者**: sinmkd  **时间**: 2026-10-06 13:54 CST
- **标签**: kv-cache-manager
- **摘要**: ### Your current environment  - vLLM 0.31.0 (also reproduced on 0.30.0). vLLM 0.29.0 passes the same test. - torch 2.13.0+cu130, flashinfer-python/cubin 0.7.0.post1 (0.6.18.post1 on 0.30.0), CUDA 13.3, driver 610.57.04 - 1× RTX PRO 5000 Blackwell 48 GB (sm_120), Ubuntu 24.04, kernel 7.0.0-34 - Targe…

### #60172 — [[Bug]: TRITON_MLA decode exceeds shared memory on sm_120 (RTX PRO 6000 / RTX 5090): "Required: 102400, Hardware limit: 101376"](https://github.com/vllm-project/vllm/issues/60172)
- **作者**: shiv-oxmiq  **时间**: 2026-10-06 13:45 CST
- **摘要**: ### Your current environment  - vLLM 0.31.0 (official `vllm/vllm-openai` image), Triton 3.7.1 - 2x NVIDIA RTX PRO 6000 Blackwell Workstation Edition (compute capability 12.0), TP=2, PCIe - Model: `sarvamai/sarvam-105b-fp8` (`SarvamMLAForCausalLM`, `kv_lora_rank` 512, `qk_rope_head_dim` 64), `--kv-ca…

## 🔀 Pull Requests

### #60181 — [[Bugfix][NIXL] Fix host-buffer h2d sync overwriting received KV](https://github.com/vllm-project/vllm/pull/60181)
- **作者**: lucamotz  **时间**: 2026-10-06 14:35 CST
- **标签**: bug, kv-connector
- **摘要**: ## Overview  With `kv_buffer_device="cpu"`, `sync_recved_kv_to_device` overwrites D's received KV with host pages NIXL never filled, for models with several KV cache groups or KV-sharing layers. Each group now copies only its own layers' host buffers; the multi-group case regressed in #53780.  ## Cl…

### #60180 — [[Bugfix][Frontend] Count reasoning tokens for the hf response-template parser](https://github.com/vllm-project/vllm/pull/60180)
- **作者**: kotwal-itpro  **时间**: 2026-10-06 14:33 CST
- **标签**: bug, tool-calling
- **摘要**: ## Overview  `--reasoning-parser hf` always reported `completion_tokens_details.reasoning_tokens: 0`, while returning the reasoning text. `ResponseTemplateParser` inherits `Parser.count_reasoning_tokens`, which returns 0. This PR counts the generated tokens in the response template's thinking region…

### #60179 — [[Bugfix][Weight Cache] Key non-MoE DP daemons as DP=1 like the engine](https://github.com/vllm-project/vllm/pull/60179)
- **作者**: nien-hui  **时间**: 2026-10-06 14:32 CST
- **标签**: bug
- **摘要**: ## Overview  Fixes #60178 . A dense (non-MoE) engine with `--data-parallel-size N` sends `dp_size=1, dp_rank=0` to the `vllm preload` daemon, which keys each rank as `dp_size=N, dp_rank=r`, so `--load-format ipc_cache` never maps the cached weights. The daemon now keys dense DP ranks as `(1, 0)` lik…

### #60177 — [[Bugfix][Benchmark] Preserve Rust SSE payloads across chunks and comments](https://github.com/vllm-project/vllm/pull/60177)
- **作者**: Jaeyeong-CHOI  **时间**: 2026-10-06 13:57 CST
- **标签**: bug, rust
- **摘要**: ## Overview  Preserve SSE text and usage in the Rust benchmark clients across HTTP chunk boundaries, LF/CRLF/CR event framing, and leading comment lines. This combines the related response-data-loss fixes in one small production change and one shared HTTP test fixture.  ## Claims  - Preserve transpo…

### #60176 — [[Docs][KV Connector] Explain effective HMA mismatches for NIXL peers](https://github.com/vllm-project/vllm/pull/60176)
- **作者**: Jaeyeong-CHOI  **时间**: 2026-10-06 13:57 CST
- **标签**: documentation, kv-connector
- **摘要**: ## Overview  Document how asymmetric connector configurations can cause an effective HMA-state mismatch during the NIXL handshake. Addresses the documentation request in #60124; does not add field-level mismatch diagnostics.  ## Claims  - Explain why omitting the HMA flag on both instances need not …

### #60175 — [[Docs][Metrics] Document prefix cache hit rate monitoring and add a G…](https://github.com/vllm-project/vllm/pull/60175)
- **作者**: vinitrbl  **时间**: 2026-10-06 13:55 CST
- **标签**: documentation
- **摘要**: ## Overview  The prefix caching guide never mentions `vllm:prefix_cache_hits` or `vllm:prefix_cache_queries`, and none of the example dashboards chart the hit rate. Working out what the number actually counts currently means reading the scheduler. This adds a short Monitoring section to `docs/featur…

### #60173 — [[Bugfix] Use one stage in Triton MLA decode on GPUs with <128 KiB shared memory](https://github.com/vllm-project/vllm/pull/60173)
- **作者**: shiv-oxmiq  **时间**: 2026-10-06 13:45 CST
- **标签**: bug
- **摘要**: ## Purpose  Fixes #60172.  On sm_120 GPUs (RTX PRO 6000, RTX 5090), `TRITON_MLA` is the only MLA backend, and the first decode fails with:  ```text out of resource: shared memory, Required: 102400, Hardware limit: 101376 ```  `_decode_grouped_att_m_fwd` launches the MLA shape (`BLOCK_DMODEL=512`, `B…

### #60171 — [[CI/Build] Enable block FP8 quantization tests on NVIDIA SM89 (Ada)](https://github.com/vllm-project/vllm/pull/60171)
- **作者**: amankarki151  **时间**: 2026-10-06 13:28 CST
- **标签**: nvidia, quantization
- **摘要**: ## Overview  Run the block-FP8 kernel tests on SM89 (Ada: L4, L40S, RTX 4090, RTX 6000 Ada). Today `tests/kernels/quantization/test_block_fp8.py` skips the whole module below SM90, so these GPUs get no block-FP8 test coverage even though they have FP8 tensor cores and the Triton block-FP8 path runs …

### #60170 — [[CPU][Recipes] Detect model head constraints for automatic tensor parallel selection in vLLM Recipes Tool](https://github.com/vllm-project/vllm/pull/60170)
- **作者**: louie-tsai  **时间**: 2026-10-06 13:09 CST
- **摘要**: ## Overview  Make automatic CPU recipe TP selection consider model attention and KV head counts alongside NUMA topology. This prevents incompatible suggestions such as TP=4 for `microsoft/Phi-4-reasoning`, which requires TP=2 under the current power-of-two policy.  ## Claims  - Detect head constrain…

### #60169 — [[ROCm][DSv4.1][Perf] Use AITER FlyDSL GEMM for DeepSeek V4.1's 32x32 MXFP8 linears on gfx950](https://github.com/vllm-project/vllm/pull/60169)
- **作者**: ahmed-bsod  **时间**: 2026-10-06 12:36 CST
- **标签**: rocm, deepseek, DSv4.1
- **摘要**: ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this PR.  - [ ] **Design Fit:** Minimizes…

## 🚀 Releases

### [v0.31.0](https://github.com/vllm-project/vllm/releases/tag/v0.31.0)
- **作者**: khluu  **时间**: 2026-10-05 14:44 CST
- **摘要**: # v0.31.0  ## Highlights  This release features 717 commits from 307 contributors (96 new)!  * **DeepSeek-V4.1-Flash performance**: FlashMLA mega attention with the V4.1 NVFP4 compressed KV cache is now the SM100 default (#56935); DeepGEMM sparse MQA logits for the indexer (#56254) and Mega-Gate fus…
