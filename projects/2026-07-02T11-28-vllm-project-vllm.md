# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-02 11:28 UTC

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文简洁摘要：

### 📌 Issue 动态
近期 Issue 主要聚焦于**模型输出稳定性与一致性**问题：
* **精度/对数概率偏差**：微调的 Qwen3.5-VL mobile-use 模型在 vLLM 与 HuggingFace transformers 之间存在 logprob（对数概率）差异，影响生成准确性 (#47425)。
* **输出不稳定 Bug**：DiffusionGemma (v0.24.0) 在多次推理相同输入时，前后输出结果不一致 (#47424)。
* **流式输出 Bug**：Qwen3-ASR 模型在流式生成时出现分块异常 (#47421)。
* **特定模型运行失败**：DeepSeek-v4-flash-DSpark 模型的 `spe`（推测执行）函数无法正常运行 (#47418)。

### 🚀 PR 动态
近期 PR 主要围绕**硬件平台扩展、核心调度优化及 Bug 修复**：

**1. 硬件与平台支持扩展**
* **ROCm 支持 DeepSeek-V4 推测解码**：在 AMD GPU（MI350X/MI355X, gfx950）上启用了 DeepSeek-V4-Pro 的 DSpark 推测解码，打破了此前该功能仅限 NVIDIA 的限制 (#47419, #47414)。
* **XPU Gemma4 性能修复**：修复 Gemma4 在 XPU 平台的性能回退，允许其使用平台原生的 `FLASH_ATTN`，而非被强制指定不支持的 `TRITON_ATTN` (#47426)。
* **CephFS 加载优化**：将 CephFS 识别为网络文件系统，启用 safetensors 自动预取，优化大模型检查点的随机读取延迟 (#47412)。

**2. 核心架构与调度优化**
* **Encoder 缓存 CPU 卸载**：新增 `ECCPUConnector`，可将 encoder 输出卸载到 CPU `/dev/shm` 的共享 mmap 区域，支持单实例跨请求复用，降低开销 (#47423)。
* **数据并行 (DP) 负载均衡防偏**：修复 DP 负载均衡器在打分相同时总是优先选择首个引擎的系统性偏差，通过旋转打破平局以避免偏载 (#47420)。
* **KV 缓存分层监控**：为 `SecondaryTierManager` 增加 `tier_idx`，支持多层级 KV 缓存部署下每层独立指标的采集与区分 (#47413)。

**3. 稳定性与 Bug 修复**
* **Worker 错误溯源**：修复 Worker 初始化失败（如 OOM、KV cache 错误等）时，父进程只抛出模糊报错的问题，现在会将真实的异常根因传递给父进程 (#47422)。
* **推测解码形状对齐**：对混合了“正常 1-token 解码”与“草稿 token 验证”的批次进行 Padding，解决因形状不一致导致的后端报错 (#47417)。

**4. 模型特性支持**
* **Kimi K2.5 图像预处理融合**：新增基于 numba 的可选 CPU 融合预处理路径，将 padding、归一化与 patchification 融合，提升视觉模型处理效率 (#47416)。

### 🎉 Release 动态
* 本次提供的数据中 **暂无新版本 Release 发布信息**。

---

## 🐛 Issues

### #47425 — [[Performance]: Logprob divergence between vLLM and transformers on a fine-tuned Qwen3.5-VL mobile-use model](https://github.com/vllm-project/vllm/issues/47425)
- **作者**: WRWA  **时间**: 2026-07-02 10:50 UTC
- **标签**: performance
- **摘要**: ### Proposal to improve performance  # Proposal to improve performance  I am not proposing a performance optimization or a new performance-improvement design in this issue.  This report is about an observed accuracy / logprob discrepancy between vLLM and HuggingFace transformers when serving a fine-…

### #47424 — [[Bug]: DiffusionGemma output unstable (v0.24.0)](https://github.com/vllm-project/vllm/issues/47424)
- **作者**: catsled  **时间**: 2026-07-02 10:37 UTC
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Your output of `python collect_env.py` here ```  </details>   ### 🐛 Describe the bug  tp =4 pp = 1  when i ask a question: "你叫什么" about 10 times 1、the outputs are correct at 1~5th, bu…

### #47421 — [[Bug]: Streaming output segmentation （Qwen3-ASR）](https://github.com/vllm-project/vllm/issues/47421)
- **作者**: yangyyt  **时间**: 2026-07-02 09:59 UTC
- **标签**: bug
- **摘要**: ### Your current environment  <details>  ```text Collecting environment information... ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (x86_64) GCC version                  : (Ubuntu 11.4.0-1ubuntu1~22.04.3) 11.4.0 C…

### #47418 — [[Bug]: The spe function of the DeepSeek-v4-flash-DSpark model fails to run](https://github.com/vllm-project/vllm/issues/47418)
- **作者**: Rozwel-dx  **时间**: 2026-07-02 09:41 UTC
- **标签**: bug
- **摘要**: ### Your current environment  Environment Details vLLM Version: main torch: 2.11.0 torchvision: 0.26.0 cu13  GPU: H20 CPU: Intel(R) Xeon(R) Platinum 8480+  CPU @ 2.0GHz   ### 🐛 Describe the bug  Run successfully with the following command： `vllm serve /data/Weights/DeepSeek-V4-Flash-DSpark/ --tensor…

## 🔀 Pull Requests

### #47426 — [[Bugfix][Hardware][XPU] Fix Gemma4 performance regression by allowing platform-native attention](https://github.com/vllm-project/vllm/pull/47426)
- **作者**: Vinay12345-neutron  **时间**: 2026-07-02 11:08 UTC
- **标签**: bug, intel-gpu
- **摘要**: Co-authored-by: Gemini Signed-off-by: Vinay12345-neutron vinayrjumani@gmail.com     ## Purpose `Gemma4Config` forces the `TRITON_ATTN` backend when `is_fa_version_supported(4)` is false. However, on XPU platforms, standard CUDA/ROCm FA4 is not available, yet the platform's default `FLASH_ATTN` backe…

### #47423 — [[EC Connector] CPU Offloading EC Connector](https://github.com/vllm-project/vllm/pull/47423)
- **作者**: omerpaz95  **时间**: 2026-07-02 10:32 UTC
- **标签**: v1, cpu
- **摘要**: # Add `ECCPUConnector` — CPU encoder-cache offloading connector  ## Purpose  This PR adds `ECCPUConnector`, a self-contained encoder-cache (EC) connector that offloads encoder outputs to a shared CPU `mmap` region on `/dev/shm` so a single `ec_role=both` vLLM instance can reuse them on a later reque…

### #47422 — [fix(v1): surface worker init failure root cause to the parent](https://github.com/vllm-project/vllm/pull/47422)
- **作者**: hclsys  **时间**: 2026-07-02 10:12 UTC
- **标签**: v1
- **摘要**: Fixes #47415  On a WorkerProc init failure (KV-cache ValueError, OOM, arch mismatch, …) the worker only logged the real error to its own stderr. The parent then hit EOF on the ready pipe and raised a generic `WorkerProc initialization failed ... See stack trace for root cause`, so the actual excepti…

### #47420 — [[Core][DP] Rotate load-balancer tie-break to avoid systematic engine bias](https://github.com/vllm-project/vllm/pull/47420)
- **作者**: mayuyuace  **时间**: 2026-07-02 09:59 UTC
- **标签**: v1
- **摘要**: **Motivation** In DPLBAsyncMPClient.get_core_engine_for_request, the internal DP load balancer selects the engine with the lowest score (waiting * 4 + running), scanning from a fixed eng_start_index and using a strict < comparison. When engines are tied, the winner is always the first one scanned — …

### #47419 — [[ROCm] Enable DeepSeek-V4 DSpark speculative decoding on AMD (MI350X / MI355X, gfx950)](https://github.com/vllm-project/vllm/pull/47419)
- **作者**: larryli2-amd  **时间**: 2026-07-02 09:44 UTC
- **标签**: rocm, deepseek
- **摘要**: ## Summary  DSpark speculative decoding for **DeepSeek-V4-Pro (1.6T)** was previously **NVIDIA-only** in vLLM: `vllm/models/deepseek_v4/__init__.py` sets `DSparkDeepseekV4ForCausalLM = None` on ROCm, so `--speculative-config '{"method":"dspark",...}'` fails at model-load time on AMD GPUs.  This PR e…

### #47417 — [[Bugfix][Spec Decode] Pad mixed running decode batches](https://github.com/vllm-project/vllm/pull/47417)
- **作者**: hubunt  **时间**: 2026-07-02 09:36 UTC
- **标签**: bug, v1
- **摘要**: ## Purpose  Related to #36122.  When speculative decoding is enabled, one running request may verify draft tokens while another running request in the same scheduler step is still a normal 1-token decode. That can produce a mixed normal/spec decode batch shape and trigger backends that require unifo…

### #47416 — [Add fused Kimi K2.5 image preprocessing](https://github.com/vllm-project/vllm/pull/47416)
- **作者**: Kevin-XiongC  **时间**: 2026-07-02 09:36 UTC
- **摘要**: ## Summary  This PR adds an opt-in fused CPU image preprocessing path for Kimi-K2.5/K2.6 vision chunks.  - Adds `KimiK25FusedVisionProcessor`, a vLLM-local processor that preserves the Kimi NaViT resize/token semantics while fusing padding, normalization, and patchification through numba. - Adds `VL…

### #47414 — [[ROCm] [DSpark] Enable dspark for deepseek v4](https://github.com/vllm-project/vllm/pull/47414)
- **作者**: tjtanaa  **时间**: 2026-07-02 08:55 UTC
- **标签**: rocm, deepseek
- **摘要**: ## Purpose  Enable DSpark on ROCm.  ## Test Plan  Server command: ```bash #!/bin/bash  rm -rf ~/.cache/vllm   # export VLLM_ROCM_USE_AITER_CUSTOM_AR=1 export VLLM_ROCM_USE_AITER=1  vllm serve deepseek-ai/DeepSeek-V4-Pro-DSpark \   --host localhost \   --port 8001 \   --tensor-parallel-size 8 \   --d…

### #47413 — [[KV Offload] Add tier_idx to SecondaryTierManager for per-tier metrics](https://github.com/vllm-project/vllm/pull/47413)
- **作者**: Alex-ai-future  **时间**: 2026-07-02 08:32 UTC
- **标签**: v1
- **摘要**: ## Purpose  Add `tier_idx` to `SecondaryTierManager` so that secondary tiers can identify their position in the tier list. This is needed for per-tier metrics (e.g. `TIERING_BLOCK_QUERIES`, `TIERING_BLOCK_HITS`) to distinguish between multiple tiers in a multi-tier deployment.  ## Changes  - Add `ti…

### #47412 — [[Model Loader] Enable safetensors auto-prefetch on CephFS](https://github.com/vllm-project/vllm/pull/47412)
- **作者**: seadog007  **时间**: 2026-07-02 07:59 UTC
- **摘要**: ## Purpose A simple PR that enables safetensors auto-prefetch on CephFS.  CephFS is a network filesystem and has similar random-read latency concerns as NFS/Lustre when loading large safetensors checkpoints through mmap. This PR treats `ceph` as a recognized network filesystem for safetensors auto-p…
