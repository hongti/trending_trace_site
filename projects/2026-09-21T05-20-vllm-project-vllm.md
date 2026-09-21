# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-21 13:20 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要：

### Release (发布版本)
近期无新的版本发布动态。

### Issue (问题)
近期无公开的 Issue 动态。

### Pull Request (代码合并请求)
近期共有 10 个 PR，主要涉及 CI/CD 优化、性能改进、Bug 修复及新功能原型。重要变更如下：

**1. 核心功能与性能优化**
*   **性能分析器增强** (#57875)：为 Python 和 Rust 前端添加了基于会话的性能分析控制，`POST /start_profile` 接口现已支持三个可选的会话级参数。
*   **AMD Zen CPU 算子优化** (#57873)：针对 AMD Zen CPU 上的 zentorch，实现了量化权重在加载时的一次性预打包（支持 W8A8 和 DA8W4），避免了每次矩阵乘法时重复构建布局，显著提升效率。
*   **DSv4.1 P/D 交接原型** (#57872)：提交了一个实验性原型，验证仅编码器预填充的 P/D 交接设计。

**2. Bug 修复**
*   **V2 可中断 CUDA Graph 修复** (#57874)：修复了一个预热路径不匹配的 Bug。此前 eager 预热在准备 PIECEWISE 捕获时仍会广播 `NONE` 运行时模式，导致模型预热的内核路径与实际捕获的路径不一致。
*   **CI 构建修复** (#57871)：修复了 CPU 镜像构建中 triton-cpu sleef 子模块拉取失败的问题，该问题曾导致所有 CPU 构建任务失败。

**3. CI 与测试改进 (主要针对 ROCm/AMD)**
*   **AMD 测试矩阵扩充与去重** (#57876, #57877)：新增了十个可选的 AMD 对齐测试组（如 Mooncake EC TCP、DeepSeek 等），同时对剩余的可移植套件进行镜像处理，消除重复测试并明确归属。
*   **CI 失败阻塞机制** (#57870)：移除了 AMD 纯 Python 安装步骤的 `soft_fail` 覆盖，使其安装失败能够直接阻断流水线，保持硬失败的默认行为。
*   **测试内存优化** (#57868, #57869)：在 ROCm 环境下将 FP32 参考直接加载到设备以减少主机内存占用；并在 prompt 提取后使用 `torch.no_grad()` 释放 HF embedding 权重，防止退出后内存泄漏。

---

## 🔀 Pull Requests

### #57877 — [[CI][ROCm] Mirror remaining portable suites without duplicate coverage](https://github.com/vllm-project/vllm/pull/57877)
- **作者**: AndreasKaratzas  **时间**: 2026-09-21 13:12 CST
- **标签**: rocm, torch.compile, ci/build
- **摘要**: ## Purpose  Declare seven AMD mirrors while giving existing coverage a single owner:  | Group | AMD target | Coverage change | | --- | --- | --- | | Granite compatibility | MI355 | Remove Granite from the general hybrid suite. | | Multimodal beam search | MI300 | Exclude the selected cases from gene…

### #57876 — [[CI][ROCm] Add ten optional AMD parity groups](https://github.com/vllm-project/vllm/pull/57876)
- **作者**: AndreasKaratzas  **时间**: 2026-09-21 13:12 CST
- **标签**: rocm, ci/build
- **摘要**: ## Purpose  Add ten optional AMD mirrors with matching legacy `test-amd.yaml` jobs: Scale-out EC, Mooncake EC TCP, Kimi Linear disaggregation, Sharded RDT, single-GPU hidden-state extraction, DeepSeek V4 fused kernels, watermarking, watermark feature combinations, Qwen3.8 FP8 accuracy, and MoE refac…

### #57875 — [[Profiler] Add per-session profiling controls to Python and Rust frontends](https://github.com/vllm-project/vllm/pull/57875)
- **作者**: czhu-cohere  **时间**: 2026-09-21 13:10 CST
- **标签**: documentation, frontend, rust
- **摘要**: ## Purpose  Implement the Part D frontend/API split from #56542 on top of the merged profiler-core work in #57460.  `POST /start_profile` now accepts three optional per-session controls in both Python and Rust frontends:  - `profile_prefix` - `delay_iterations` - `max_iterations`  The same options a…

### #57874 — [[Bugfix][V2] Match breakable CUDA graph warmup dispatch to capture](https://github.com/vllm-project/vllm/pull/57874)
- **作者**: WoosukKwon  **时间**: 2026-09-21 13:00 CST
- **标签**: bug, ready, nvidia, mrv2
- **摘要**: ## Problem and fix  With V2 breakable CUDA graphs, eager warmup advertised runtime mode NONE even when preparing PIECEWISE capture. A model can therefore warm a different kernel path from the one captured on that stream.  On DeepSeek V4.1 TP4 with FULL_AND_PIECEWISE and capture sizes [1,2,4,8], warm…

### #57873 — [[CPU][Zen] Prepack quantized weights and add DA8W4 for zentorch on Zen CPUs](https://github.com/vllm-project/vllm/pull/57873)
- **作者**: charan-ponnada  **时间**: 2026-09-21 12:57 CST
- **标签**: quantization
- **摘要**: ## Purpose Prepack quantized weights once at load time for zentorch on AMD Zen CPUs, instead of rebuilding the blocked layout on every matmul. - W8A8: `zentorch_weight_prepack_for_dynamic_qlinear`, gated on `ZENDNNL_MATMUL_ALGO=1` - W4A16: `zentorch_woq_repack_weight(..., blocked_format=True)`, gate…

### #57872 — [[RFC prototype][DSv4.1] Encoder-only prefill P/D handoff](https://github.com/vllm-project/vllm/pull/57872)
- **作者**: wangyicong52  **时间**: 2026-09-21 12:36 CST
- **标签**: deepseek, kv-connector, mrv2, scheduler, kv-cache-manager, DSv4.1
- **摘要**: ## Purpose  This is an experimental validation prototype based on RFC #57738 / #57383. It is not a claim of ownership of either design or of the eventual production implementation.  The primary contribution is to make the proposed execution and handoff contract concrete enough to benchmark and valid…

### #57871 — [[CI][Build] Harden triton-cpu sleef submodule fetch in CPU image build](https://github.com/vllm-project/vllm/pull/57871)
- **作者**: vllm-agent  **时间**: 2026-09-21 12:24 CST
- **标签**: ci/build, cpu
- **摘要**: ## Problem  CPU lanes on main are failing at image build (builds [#90092](https://buildkite.com/vllm/ci/builds/90092), [#90127](https://buildkite.com/vllm/ci/builds/90127) — all 6 CPU shards red): triton-cpu's pinned clone (270e696d) occasionally leaves `third_party/sleef` empty, and CMake dies with…

### #57870 — [[CI][ROCm] Make Python-only installation failures blocking](https://github.com/vllm-project/vllm/pull/57870)
- **作者**: AndreasKaratzas  **时间**: 2026-09-21 12:11 CST
- **标签**: rocm, ci/build
- **摘要**: - Make AMD Python-only Installation failures block the pipeline. - Remove its explicit `soft_fail: true` override so the job inherits the hard-fail default. - Preserve the job's installation commands, scheduling, dependencies, and timeout.  The audit prompted by [AMD CI build 13265](https://buildkit…

### #57869 — [[ROCm][Tests] Reduce host memory when downcasting FP32 HF references](https://github.com/vllm-project/vllm/pull/57869)
- **作者**: AndreasKaratzas  **时间**: 2026-09-21 12:08 CST
- **标签**: rocm
- **摘要**: - Load eligible native FP32 causal HF references directly onto the current ROCm device when requesting FP16/BF16. - Preserve staged loading for same-dtype or unknown-dtype checkpoints and non-ROCm platforms. - Exclude explicit revisions, additional loader options, remote model code, quantized models…

### #57868 — [[Tests] Release HF embedding weights after prompt extraction](https://github.com/vllm-project/vllm/pull/57868)
- **作者**: AndreasKaratzas  **时间**: 2026-09-21 12:06 CST
- **标签**: ready
- **摘要**: - Compute reference prompt embeddings under `torch.no_grad()` in the language generation comparison test. - Prevent retained prompt values from keeping HF embedding weights alive after runner exit. - Preserve embedding values, MiniCPM scaling, and existing generation/logprob assertions.  The investi…
