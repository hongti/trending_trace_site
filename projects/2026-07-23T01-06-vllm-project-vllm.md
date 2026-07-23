# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-23 09:06 CST

## AI 总结

以下是 **vllm-project/vllm** 近期动态的中文简洁摘要：

### 🐛 Issue 动态
1. **FlashInfer sampler 启动崩溃** (#49497)：在无系统 CUDA toolkit（无 nvcc）的默认预编译/wheel 安装环境下，FlashInfer sampler 的 JIT 编译会导致引擎启动崩溃，且无法回退至原生 sampler。
2. **DiffusionGemma 解码误判崩溃** (#49495)：当单个 token 的 prefill 余量被误判为 decode 阶段时，会导致 DiffusionGemma 模型运行崩溃。
3. **Harmony 模型拒绝 namespace 工具类型** (#49493)：在 Responses API (`/v1/responses`) 中，Harmony 模型（如 `openai/gpt-oss-120b`）会拒绝 `namespace` 工具类型并报错不支持。

---

### 🔧 PR 动态
**▶ 前端与 API 更新**
- **Anthropic API 缓存隔离** (#49498)：为 Anthropic Messages API 添加了 `cache_salt` 支持，实现多用户部署下的显式前缀缓存隔离。
- **Rust 前端 gRPC 服务发现** (#49491)：在 Rust 前端的 gRPC 控制 API 中新增 `GetServerInfo` 和 `GetModelInfo` 接口，支持服务器与模型发现。
- **Rust 前端 finish_reason 修复** (#49496)：修复了 Rust 聊天补全路径中，强制命名函数选择（forced named function choice）时 `finish_reason` 错误返回 `"tool_calls"` 的问题，现已正确返回 `"stop"`。

**▶ 核心 Bug 修复**
- **CUDA graph 捕获修复** (#49494)：在分段（piecewise）CUDA graph 损获期间跳过通用 attention，保留 MRV2 的 attention 元数据，并将其路由至 profiling 路径。
- **Speculative Decode 配置解析** (#49490)：修复了 Step3p5 MTP drafter 针对 VLM 目标模型（Step3p7）的 draft config 解析逻辑，改为从 text config 正确读取。
- **NVFP4 MoE 权重写入修复** (#49489)：修复了 FlashInfer NVFP4 后端因重叠的 stride-zero 存储导致共享 input scales 无法在逐层重载时写入的问题。
- **移除 SciPy 依赖** (#49485)：从 Inkling scale planning 中移除了非核心依赖 SciPy，避免因可选依赖未安装而报错。

**▶ 性能优化**
- **DeepSeek-V4 Decode 加速** (#49486)：在不需要时跳过 topk 和 router 计算，为 DeepSeek-V4-Flash 的 Decode 场景带来 **3.4% 的端到端 TTFT（首字延迟）提升**。
- **Inkling 内存与性能优化** (#49487)：避免 Inkling 模型中不必要的临时张量分配，提升了性能并防止在较小内存配置下出现 OOM。

**▶ 维护与自动化**
- **量化标签自动化** (#49492)：为仓库添加自动化标签逻辑，针对涉及核心量化目录的 PR 或包含 `quant` 字段的标题/内容自动打上标签，改善 Issue/PR 分类。

---

### 🚀 Release 动态
近期无新版本发布记录。

---

## 🐛 Issues

### #49497 — [[Bug]: FlashInfer sampler JIT crashes engine startup when nvcc isn't discoverable (default precompiled/wheel install) — no fallback to native sampler](https://github.com/vllm-project/vllm/issues/49497)
- **作者**: Dhruv235  **时间**: 2026-07-23 08:08 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text ==============================         System Info ============================== OS                           : Ubuntu 26.04 LTS (x86_64) GCC version                  : (Ubuntu 15.2.…

### #49495 — [[Bug]: DiffusionGemma crashes when a one-token prefill remainder is mistaken for decode](https://github.com/vllm-project/vllm/issues/49495)
- **作者**: shubhamprshr27  **时间**: 2026-07-23 08:01 CST
- **标签**: bug
- **摘要**: ### Your current environment  OS                           : Ubuntu 24.04.4 LTS (x86_64) GCC version                  : 13.3.0 Libc version                 : glibc-2.39  PyTorch version              : 2.11.0+cu130 Is debug build               : False CUDA used to build PyTorch   : 13.0  Python versi…

### #49493 — [[Bug]: Harmony models reject namespace tool type in Responses API](https://github.com/vllm-project/vllm/issues/49493)
- **作者**: hmoghani  **时间**: 2026-07-23 07:48 CST
- **摘要**: ## Description  Harmony models (e.g., `openai/gpt-oss-120b`) reject `namespace` tool types in the Responses API (`/v1/responses`) with the error `"tool type namespace not supported"`. This occurs on all vLLM versions (tested v0.21.0+rhaiv.10 and upstream v0.25.1) and all parsers (`openai`, `hermes`)…

## 🔀 Pull Requests

### #49498 — [[Frontend] Add cache_salt support to Anthropic Messages API](https://github.com/vllm-project/vllm/pull/49498)
- **作者**: aeon-x  **时间**: 2026-07-23 08:57 CST
- **标签**: frontend
- **摘要**: ## Purpose  Closes #46688.  The OpenAI-compatible Chat Completions / Completions APIs accept a `cache_salt` for explicit prefix-cache isolation in multi-user deployments. The Anthropic Messages API (`/v1/messages`) did not expose it, so clients built on the Anthropic schema had to switch API formats…

### #49496 — [[Rust Frontend] Fix finish reason for named tool choices](https://github.com/vllm-project/vllm/pull/49496)
- **作者**: reidliu41  **时间**: 2026-07-23 08:04 CST
- **标签**: rust
- **摘要**: ## Purpose    The Rust chat completions path returned `finish_reason: "tool_calls"`   whenever a response contained a tool call.    For a forced named function choice, the expected finish reason is `"stop"`.   The previous behavior caused the Rust and Python frontends to return different   terminal …

### #49494 — [[Bugfix] Skip generic attention during piecewise CUDA graph capture](https://github.com/vllm-project/vllm/pull/49494)
- **作者**: mgoin  **时间**: 2026-07-23 07:56 CST
- **标签**: bug, ready, v1, nvidia
- **摘要**: ## Summary  - Preserve attention metadata during MRV2 CUDA graph capture for model-specific consumers. - Route generic MHA/MLA attention through its profiling path during non-breakable PIECEWISE capture. - Propagate the selective skip through microbatch and autoregressive speculator contexts.  ## Ro…

### #49492 — [Add quantization label automation](https://github.com/vllm-project/vllm/pull/49492)
- **作者**: mgoin  **时间**: 2026-07-23 07:34 CST
- **标签**: ready, ci/build, quantization
- **摘要**: ## Summary  - label PRs that touch the core quantization directory or contain `quant` in the title - label issues containing the whole words `quantization` or `quantized`  ## Why  The existing `quantization` label had no automatic PR or issue application, so relevant work was under-labeled. Searches…

### #49491 — [[Rust Frontend][gRPC] Add server and model discovery](https://github.com/vllm-project/vllm/pull/49491)
- **作者**: connorcarpenter15  **时间**: 2026-07-23 07:29 CST
- **标签**: v1, rust
- **摘要**: ## Purpose  Add `GetServerInfo` and `GetModelInfo` to the Rust frontend's gRPC control API.  This PR includes the two service-separation commits from #49466 because that dependency is not yet in `main`:  - Rename the wire inference service from `vllm.Generate` to `vllm.Inference`. - Split inference …

### #49490 — [[Bugfix][Spec Decode] Step3p5 MTP: resolve draft config from the text config (Step3p7 on Model Runner V2)](https://github.com/vllm-project/vllm/pull/49490)
- **作者**: chanh  **时间**: 2026-07-23 07:13 CST
- **标签**: bug
- **摘要**: ## Summary  The Step3p5 MTP drafter — used for **Step-3.7-Flash (`Step3p7`, a VLM)** — resolves its config via `vllm_config.model_config.hf_config`, which for a VLM target is the **top-level `Step3p7Config`**. The text-model fields it reads (`vocab_size`, `hidden_size`, `num_hidden_layers`, `num_nex…

### #49489 — [[Bugfix] Make shared NVFP4 MoE scales writable](https://github.com/vllm-project/vllm/pull/49489)
- **作者**: S1ro1  **时间**: 2026-07-23 06:17 CST
- **标签**: bug, ready, nvidia, quantization
- **摘要**: FlashInfer NVFP4 backends broadcast shared input scales with expand, leaving registered parameters backed by overlapping stride-zero storage. Layerwise reload cannot copy regenerated values into those parameters.  Materialize independent per-expert scale storage and cover the copy-back operation wit…

### #49487 — [[Performance][Model] Avoid transient Inkling result allocations (performance, and OOM prevention on smaller memory configurations)](https://github.com/vllm-project/vllm/pull/49487)
- **作者**: mikekg  **时间**: 2026-07-23 05:32 CST
- **标签**: performance, ready
- **摘要**: ## Purpose  Inkling currently performs two required output operations using out-of-place expressions. In the tested H100 configuration, each allocates an unnecessary temporary tensor:  - Applying `global_scale` to the dense-MLP output allocates an 86 MiB   temporary. - Adding the sink-expert output …

### #49486 — [[DSv4 Perf] Skip topk and router when not needed, 3.4% E2E TTFT improvement for Decode case](https://github.com/vllm-project/vllm/pull/49486)
- **作者**: yewentao256  **时间**: 2026-07-23 05:09 CST
- **标签**: ready
- **摘要**: ## Purpose  Skip topk and router when not needed  Part of https://github.com/vllm-project/vllm/issues/45861  ## Test  `vllm serve deepseek-ai/DeepSeek-V4-Flash   --tensor-parallel-size 4   --enable-expert-parallel   --attention-backend FLASHMLA_SPARSE_DSV4   --attention-config '{"use_fp4_indexer_cac…

### #49485 — [[Bugfix][Model] Remove SciPy dependency from Inkling scale planning](https://github.com/vllm-project/vllm/pull/49485)
- **作者**: mikekg  **时间**: 2026-07-23 05:02 CST
- **标签**: bug, ready
- **摘要**: ## Purpose  Inkling’s HMLP vision scale planning calls scipy.optimize.linear_sum_assignment, but SciPy is not a core vLLM dependency; it is installed only through optional extras such as audio and bench. Consequently, constructing the released Inkling vision tower in a standard vLLM environment fail…
