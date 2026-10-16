# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-16 12:53 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 摘要
- **工具解析器解析中断问题 (#61954)**：有用户反馈，当字符串参数中包含原始换行符时，`PythonicToolParser` 和 `Llama4PythonicToolParser` 会发生解析中断报错。

---

### 🚀 PR 摘要

**✨ 新特性**
- **支持 logprobs 辅助输出 (#61953)**：引入了对 logprobs 辅助输出的支持。
- **Rust 前端返回路由专家信息 (#61949)**：Rust 的 `/generate` 路由现已支持返回 `routed_experts` 数据（需开启相应标志），修复了此前该数据被静默丢弃的问题。

**⚡ 性能优化**
- **GDN 核心 CUDA 图捕获优化 (#61952)**：在混合批次（prefill + decode）的可中断 CUDA Graphs 中捕获 Qwen GDN 核心，取代原本在每层 GDN 中运行的 eager break，提升执行效率。
- **GLM-5.3-Flash 性能提升 (#61951)**：在 ROCm 的 Triton 后端为 GLM-5.3-Flash 启用了 Mamba prefill 检查点，消除了额外的向前传递，优化了处理流程。

**🐛 Bug 修复**
- **ROCm Kimi K3 在线量化修复 (#61950)**：修复了 ROCm 上 Kimi K3 在线权重加载时，由于 padded merged KDA projection 导致的问题，确保检查片到达后能顺利完成加载。
- **Cohere Chat v2 流式错误修复 (#61947)**：修复了 Cohere Chat v2 在流式传输时丢弃错误信息的问题，现在会正确保留原始错误消息并以错误事件返回。
- **Unlimited-OCR 图像 Token 预估修复 (#61946)**：修正了 Unlimited-OCR 模型最大图像 Token 边界预估不足的问题，避免因预算估小而拒绝有效图像。

**🔧 CI / 基础设施**
- **AMD CI 迁移与调整 (#61955)**：将 17 个测试组从 MI300 迁移至 MI355，并在 AITER 上测试大型 DeepSeek/Nemotron 评估。
- **XPU 内核升级 (#61956)**：将 xpu-kernels 升级至 v0.1.16.1。
- **Intel GPU CI 安全修复 (#61948)**：移除了 Intel GPU CI 中 Docker 的特权模式，防范潜在的安全风险。

---

### 📦 Release 摘要
- 本次提供的动态中暂无版本发布信息。

---

## 🐛 Issues

### #61954 — [[Bug]: pythonic and llama4_pythonic tool parsers break on a raw newline inside a string argument](https://github.com/vllm-project/vllm/issues/61954)
- **作者**: shoichiromiyamoto  **时间**: 2026-10-16 11:42 CST
- **标签**: tool-calling
- **摘要**: ### Your current environment  vLLM main @ d6f0b7a, CPU build on macOS arm64. The affected code is pure-Python parsing.  ### 🐛 Describe the bug  `PythonicToolParser` and `Llama4PythonicToolParser` fail when a string argument contains a raw newline, such as multi-line code written with real line break…

## 🔀 Pull Requests

### #61956 — [[XPU] bump xpu-kernels to v0.1.16.1](https://github.com/vllm-project/vllm/pull/61956)
- **作者**: yma11  **时间**: 2026-10-16 12:33 CST
- **标签**: intel-gpu, ci/build
- **摘要**: ## Overview  Bump vllm-xpu-kernels to v0.1.16.1.  ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this P…

### #61955 — [[CI][AMD] Move 17 test groups from MI300 to MI355 and test large DeepSeek/Nemotron evals on AITER](https://github.com/vllm-project/vllm/pull/61955)
- **作者**: AndreasKaratzas  **时间**: 2026-10-16 11:58 CST
- **标签**: rocm, ci/build, deepseek
- **摘要**: - Move 14 single-GPU groups from `mi300_1` to `mi355_dpx`, and LM Eval Large Models / ROCm Harness from `mi300_8` to `mi355_8` (`test-amd.yaml` + `test_areas` mirrors) - Move LM Eval KV-Offload Large (DeepSeek-V4-Flash) from `mi300_4` to `mi355_4` - Drop the MI300 copies whose MI355 twin already exi…

### #61953 — [feat: support logprobs auxoutput](https://github.com/vllm-project/vllm/pull/61953)
- **作者**: floatlibai  **时间**: 2026-10-16 11:40 CST
- **标签**: performance, mrv2, scheduler, kv-cache-manager
- **摘要**: support logprobs auxoutput split from #60789

### #61952 — [[Perf][GDN] Capture the GDN core in breakable CUDA graphs of mixed batches (CUDA graph 2/2)](https://github.com/vllm-project/vllm/pull/61952)
- **作者**: kimseunghyun-kr  **时间**: 2026-10-16 11:40 CST
- **标签**: torch.compile, nvidia, kimi, k3
- **摘要**: ## Overview  Capture the Qwen GDN core in the breakable CUDA graphs of mixed (prefill + decode) steps, instead of running it as an eager break in every GDN layer. This is the second of two PRs and is stacked on #61779 (persistent padded mixed-batch metadata and static-grid GDN prefill kernels); only…

### #61951 — [[ROCm][GLM-5.3-Flash] Enable KDA prefill checkpoints on the Triton backend](https://github.com/vllm-project/vllm/pull/61951)
- **作者**: xxraincandyxx  **时间**: 2026-10-16 11:39 CST
- **标签**: rocm, glm
- **摘要**: ## Overview  Enables Mamba prefill checkpointing for GLM-5.3-Flash on ROCm (the Triton `chunk_kda_with_fused_gate` path), eliminating the extra forward pass over the last chunk's remainder in `align` cache mode. Resolves #59755; builds on the Kimi-K3 ROCm enablement (#58344) and reuses the host-side…

### #61950 — [[Bugfix][ROCm] Fix Kimi K3 online quantization with padded projections](https://github.com/vllm-project/vllm/pull/61950)
- **作者**: whx-sjtu  **时间**: 2026-10-16 11:34 CST
- **标签**: bug, rocm, quantization, kimi, k3
- **摘要**: ## Overview  Fix ROCm Kimi K3 online weight loading so its padded merged KDA projection can finish as soon as all checkpoint shards arrive. Declare the checkpoint-backed element count and zero-initialize declared padding when materializing meta weights.  ### Summary  ```text six KDA checkpoint shard…

### #61949 — [[Rust][Frontend] Return routed experts from the generate route](https://github.com/vllm-project/vllm/pull/61949)
- **作者**: aoshen02  **时间**: 2026-10-16 11:28 CST
- **标签**: rust
- **摘要**: ## Purpose  The Rust `/inference/v1/generate` route silently drops routed-expert (R3) data. With `--enable-return-routed-experts`, the Python route returns `choices[].routed_experts` (base64 `.npy`, shape `[num_tokens - 1, num_layers, top_k]`) on the terminal response; the Rust route has no such fie…

### #61948 — [[XPU][CI] Remove Docker's privileged mode to prevent security risks in intel GPU CI](https://github.com/vllm-project/vllm/pull/61948)
- **作者**: zxd1997066  **时间**: 2026-10-16 11:14 CST
- **标签**: intel-gpu, ci/build
- **摘要**: ## Overview Remove Docker's privileged mode to prevent security risks in intel GPU CI  ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance w…

### #61947 — [[Bugfix][Frontend] Preserve Cohere Chat v2 streaming errors](https://github.com/vllm-project/vllm/pull/61947)
- **作者**: KuanJia  **时间**: 2026-10-16 11:02 CST
- **标签**: bug, frontend, cohere
- **摘要**: ## Overview  Return parser and generation failures as Cohere Chat v2 error events, including failures before the first normal chunk. Preserve the original error message instead of discarding it or replacing it with a chunk-validation error.  ## Claims  - Emit `message-end` with `finish_reason="ERROR…

### #61946 — [[Bugfix][Model] Fix maximum image token profiling for Unlimited-OCR](https://github.com/vllm-project/vllm/pull/61946)
- **作者**: Rapisurazurite  **时间**: 2026-10-16 10:59 CST
- **标签**: bug, ready, multi-modality
- **摘要**: ## Overview  Fix Unlimited-OCR's underestimated maximum image-token bound, which can reject valid images with a small encoder budget.  ## Claims  - Use a 1×32 profiling crop grid to cover the maximum number of newline features. - Correct the default per-image bound from 3553 to 3793 tokens.  ## Vali…
