# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-21 09:04 CST

## AI 总结

以下是对 **vllm-project/vllm** 近期动态的中文摘要：

### 🐛 Issue 动态
近期主要报告了几个影响构建、性能及缓存机制的 Bug：
1. **性能严重回退 (#49259)**：Qwen3-VL-32B-FP8 模型在 v0.25.1 版本上吞吐量严重崩溃，回退至 v0.24.0 后恢复正常。
2. **PyTorch Nightly 构建失败 (#49260)**：由于 PyTorch 最新版 ATen 强制要求 C++20，导致 vLLM 内置的 `vllm-flash-attn`（基于 C++17 编译）构建失败。
3. **持久 KV 缓存缺少命名空间 (#49261)**：持久化 KV offload 缓存的路径未按 model revision 进行命名空间隔离，可能导致不同版本模型间的缓存冲突。
4. **WSL 循环导入 (#49265)**：在 WSL 环境下裸代码检出（未安装）时触发循环导入（与 #48397 同源）。

---

### 🔧 PR 动态
近期 PR 集中在核心 Bug 修复、ROCm/AMD 适配优化及新特性支持：

**🛠 核心修复**
- **KV 缓存命名空间隔离 (#49266)**：对应修复 #49261，为持久 KV offload 缓存指纹加入 model revision，解决缓存冲突。
- **KV Connector 重计算逻辑修复 (#49252)**：修复了在同步/异步 KV 加载失败触发 `recompute` 策略时，产生错误重计算的问题。
- **Gemma4 Speculative Decode 修复 (#49262)**：修复 Gemma4 MTP draft layers 未正确继承目标层 KV cache scales（缩放比例滞留在 1.0）的问题。
- **OpenAI API 响应体限制 (#49256)**：限制畸形请求 validation-error 的响应体积，防止恶意或错误输入通过 Pydantic 错误暴露超长内容导致 400 响应体爆炸。

**🚀 新特性与性能优化**
- **ROCm Sparse MLA 性能提升 (#49263)**：为 ROCm AITER Sparse MLA 后端新增 `densemha` 支持，专门优化短序列 prefill 性能。
- **支持 NVFP4 权重加载 (#49258)**：支持加载 llm-compressor Inkling 格式的 NVFP4 权重，规范化了 MoE 专家参数名映射与全局缩放加载。
- **gRPC Abort 控制 (#49255)**：新增幂等的 `vllm.Control.Abort` RPC，支持按 Request ID 终止 Rust 前端的活跃请求。

**⚙️ 硬件与 CI 适配 (AMD/ROCm)**
- **AMD AITER LDS 溢出修复 (#49264)**：针对 ROCm Wave32 架构下 AITER unified-attention 请求超出 GPU LDS 限制的问题，增加 Triton fallback 机制。
- **MI355 CI 弃用 DinD (#49257)**：MI355 的 CI 测试不再使用 Docker-in-Docker (DinD)，改为直接在 AMD K8s Pod 中运行。
- **MI325 集群 Shadow 测试 (#49253)**：为 MI325 集群路由配置 Shadow 测试（标记为 DO NOT MERGE，仅用于验证）。

---

### 📦 Release 动态
**本期暂无新版本发布记录。**（注：Issue 中提及的 v0.25.1 为既有版本，引发了性能回退问题，需关注后续修复版本）。

---

## 🐛 Issues

### #49265 — [[Bug]: Circular import on WSL, bare checkout (no install) — dup of #48397](https://github.com/vllm-project/vllm/issues/49265)
- **作者**: lcheng321  **时间**: 2026-07-21 08:53 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text WSL2 (Ubuntu) on Windows 11 Python 3.14 torch 2.13.0+cpu vLLM: git clone of main, no install (no pip install -e ., no build) — PYTHONPATH=. only ```  </details>  ### 🐛 Describe the bu…

### #49261 — [[Bug]: Persistent KV offload cache is not namespaced by model revision](https://github.com/vllm-project/vllm/issues/49261)
- **作者**: mindungil  **时间**: 2026-07-21 08:19 CST
- **摘要**: ### Your current environment  <details> <summary>The relevant output of <code>python vllm/collect_env.py</code></summary>  ```text ==============================         System Info ============================== OS                           : Ubuntu 24.04.3 LTS (x86_64) GCC version                 …

### #49260 — [[Bug]: torch-nightly build fails — vllm-flash-attn (_vllm_fa2_C) built with C++17 but ATen now requires C++20](https://github.com/vllm-project/vllm/issues/49260)
- **作者**: atalman  **时间**: 2026-07-21 07:51 CST
- **摘要**: ### Summary  Building vLLM against **torch nightly** fails compiling the vendored flash-attention extension (`_vllm_fa2_C`) because PyTorch now requires **C++20**, while `vllm-flash-attn`'s host C++ sources are still compiled with C++17.  ### Where  - `Full CI run torch nightly` (branch `main`, comm…

### #49259 — [[Bug]: [Performance]: Qwen3-VL-32B-FP8 throughput collapses on v0.25.1 (v0.24.0 fine)](https://github.com/vllm-project/vllm/issues/49259)
- **作者**: csttsn  **时间**: 2026-07-21 07:32 CST
- **标签**: bug
- **摘要**: ### Your current environment  - vLLM Docker image: `vllm/vllm-openai:v0.25.1`; confirmed fixed by reverting to `vllm/vllm-openai:v0.24.0` with no other changes - GPU: 1x NVIDIA RTX 6000 Ada Generation, 48 GB VRAM - Model: `Qwen/Qwen3-VL-32B-Instruct-FP8` (pre-quantized FP8, block_shape [128,128]), a…

## 🔀 Pull Requests

### #49266 — [[Bugfix][KV Offload] Namespace persistent cache by model revision](https://github.com/vllm-project/vllm/pull/49266)
- **作者**: mindungil  **时间**: 2026-07-21 08:57 CST
- **标签**: bug, v1, kv-connector
- **摘要**: ## Purpose  Closes #49261.  Persistent native KV offloading tiers derive their cache namespace from `FileMapper.fields`. The fingerprint includes the model name and cache layout, but not the model revision. Deployments using different revisions of the same model repository can therefore resolve to t…

### #49264 — [[Bugfix][Hardware][AMD] Handle AITER unified-attention LDS overflow with Triton fallback](https://github.com/vllm-project/vllm/pull/49264)
- **作者**: cofuente  **时间**: 2026-07-21 08:52 CST
- **标签**: bug, rocm, v1
- **摘要**: ## Purpose   On ROCm, AITER's `kernel_unified_attention_3d` can request more LDS than the GPU provides. On affected Wave32 archs, `select_3d_config` chooses a tile size and pipeline depth **without checking the LDS budget**, so once Triton double-buffers the K/V tiles, the kernel asks for **65792** …

### #49263 — [[Perf] [Feat] [ROCm] Add densemha support to ROCm AITER Sparse MLA](https://github.com/vllm-project/vllm/pull/49263)
- **作者**: tjtanaa  **时间**: 2026-07-21 08:46 CST
- **标签**: rocm, v1
- **摘要**: ## Purpose  This PR follows https://github.com/vllm-project/vllm/pull/47327 in optimizing the performance for short sequence prefill of ROCm AITER Sparse MLA backend.  ## Test Plan  GSM8K with 30 num-shot  Performance gain on DeepSeek V3.2  ## Test Result  Server command:  ``` #!/bin/bash  export VL…

### #49262 — [[Bugfix][Spec Decode] Inherit target KV cache scales in Gemma4 MTP draft layers](https://github.com/vllm-project/vllm/pull/49262)
- **作者**: philipshurpik  **时间**: 2026-07-21 08:21 CST
- **标签**: bug, v1
- **摘要**: Gemma4 MTP draft layers share the target's physical KV cache (kv_sharing_target_layer_name), but their Attention objects keep their own _k_scale/_v_scale, which stay at 1.0 because assistant checkpoints carry no KV scales. On targets with calibrated FP8 KV scales (e.g. unsloth/gemma-4-31b-it-nvfp4),…

### #49258 — [[Model] Support llm-compressor Inkling NVFP4 weights](https://github.com/vllm-project/vllm/pull/49258)
- **作者**: mgoin  **时间**: 2026-07-21 07:28 CST
- **摘要**: ## Summary  - Normalize nested llm-compressor Inkling expert names to the existing fused-MoE parameter names while retaining flattened-name support. - Load compressed-tensors per-expert global scales into the existing w13/w2 layouts.  This is not a duplicate of #48876: it is a minimal alternative bu…

### #49257 — [[CI][AMD] Deprecate DinD for MI355 tests](https://github.com/vllm-project/vllm/pull/49257)
- **作者**: AndreasKaratzas  **时间**: 2026-07-21 07:28 CST
- **标签**: rocm, ci/build
- **摘要**: This deprecates DinD for all MI355 CI jobs and runs them directly in AMD Kubernetes pods; no open PR duplicates this change. Validation: `pre-commit run --files .buildkite/test-amd.yaml` and rendered-pipeline checks passed; AI assistance was used.

### #49256 — [fix(openai): bound validation-error response size for malformed requests](https://github.com/vllm-project/vllm/pull/49256)
- **作者**: hclsys  **时间**: 2026-07-21 07:21 CST
- **标签**: frontend
- **摘要**: `validation_exception_handler` renders every `RequestValidationError` entry into the 400 body, and each pydantic error embeds the full offending `input` value. So a single malformed request that fails validation many times echoes the payload once per error.  Reported on `/v1/responses` (#49239): one…

### #49255 — [feat(grpc): add abort control RPC](https://github.com/vllm-project/vllm/pull/49255)
- **作者**: connorcarpenter15  **时间**: 2026-07-21 07:10 CST
- **标签**: rust
- **摘要**: ## Purpose  Add an idempotent gRPC control RPC for aborting active Rust frontend requests by ID.  - Add `vllm.Control.Abort` with repeated request IDs and an empty response. - Register `vllm.Control` on the existing gRPC listener. - Resolve caller-facing request IDs through the frontend's external-t…

### #49253 — [DO NOT MERGE](https://github.com/vllm-project/vllm/pull/49253)
- **作者**: AndreasKaratzas  **时间**: 2026-07-21 06:58 CST
- **标签**: rocm, ready, ci/build
- **摘要**: Shadow testing MI325 cluster, analogous to #46653 for MI300.  - Route the compatible AMD suite and AMD test-area mirrors to `mi325_{1,2,4,8}` queues through the automatic `amdproduction` selector while retaining `amdexperimental` and `amdshadow` support. - Run 431 surge definitions, expanding to 497…

### #49252 — [[Bugfix][KV Connector] Fix incorrect recompute on KV-load-failure recovery (sync + async)](https://github.com/vllm-project/vllm/pull/49252)
- **作者**: ayoub-ibm  **时间**: 2026-07-21 06:50 CST
- **标签**: bug, v1, kv-connector
- **摘要**: ## Summary  Fixes #49250. `kv_load_failure_policy="recompute"` did not produce a correct recompute when a KV connector rejected a promised synchronous load, on the V2 GPU model runner (the default for non-MoE generative models).  Two independent bugs:  1. V2 GPU model runner (`update_requests`): a r…
