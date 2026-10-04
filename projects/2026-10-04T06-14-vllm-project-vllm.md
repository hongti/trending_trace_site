# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-04 14:14 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期（截至 2026 年 10 月 4 日）的动态摘要：

### 🛠️ Pull Requests (PR)
本次汇总的 PR 主要涵盖新特性、核心优化、Bug 修复以及构建/测试改进：

**1. 新特性与核心架构优化**
*   **支持基于 Gather 的 DCP (#59916)**：为无法原生运行 DCP 的 MLA（Multi-Head Latent Attention）模型引入了与后端无关的 gather-based DCP。这提供了一种不同于 layer-sharded KV 的新方案，并包含对 DSv4 的支持。
*   **优化休眠时的 CUDA graph 内存管理 (#59909)**：核心改进。休眠模式下不再将 CUDA graph 池备份到主机内存，而是直接释放。因为捕获后无需再次捕获，此举大幅节省了显存和唤醒时的拷贝时间。
*   **Inkling PEFT 适配器加载支持 (#59915)**：允许在 vLLM 服务端直接加载由 Transformers 保存的 Inkling PEFT 适配器，解决了参数命名和布局不匹配的问题。

**2. Bug 修复**
*   **修复 CLI 弃用警告误报 (#59917)**：解决了在 Python 3.10–3.12 环境下，运行 `vllm chat` 和 `vllm complete` 时每次都会错误打印 `url is deprecated` 警告的问题。
*   **修复 Inkling 推理 token 统计错误 (#59914)**：修复了当思考块由 prompt 触发而非生成 token 触发时，`usage.reasoning_tokens` 错误返回 0 的前端 Bug。
*   **FlashInfer MoE-EP 相关修复 (#59912, #59911)**：
    *   在缺少 `nvshmem4py` 时提前拒绝 `flashinfer_moe_ep_cutedsl` 后端，并在 CUDA 13 上自动安装该依赖，避免加载整个 checkpoint 后才报错。
    *   为 FlashInfer MoE-EP 别名化内置的 DeepGEMM 子模块，修复了默认镜像中的 `ImportError` 崩溃问题。

**3. 构建、测试与 CI**
*   **传递 CXXFLAGS 给 DeepGEMM (#59918)**：修复了内置 DeepGEMM `_C` 扩展在编译时忽略环境变量 `CXXFLAGS` 的构建问题。
*   **修复 Blackwell GPU 上的内核测试 (#59910)**：修复了两个仅在 SM100 系列（如 NVIDIA GB300）上才会暴露的测试端 Bug。
*   **CI 自动打标签 (#59913)**：为与 Pooling（池化）相关的 PR 和 Issue 自动添加标签，方便相关审核者及时介入。

---

### 🐛 Issues
在本次提供的数据中，未包含独立的 Issue 记录（部分 PR 中关联并修复了如 #48094, #59903, #59904 等历史 Issue）。

---

### 🚀 Release
在本次提供的数据中，未包含近期的新版本（Release）发布记录。

---

## 🔀 Pull Requests

### #59918 — [[Build] Pass CXXFLAGS to the bundled DeepGEMM _C build](https://github.com/vllm-project/vllm/pull/59918)
- **作者**: lucifer1004  **时间**: 2026-10-04 13:35 CST
- **摘要**: ## Overview  `tools/build_deepgemm_C.py` compiles the bundled DeepGEMM `_C` extension with `$CXX` and a fixed argument list, so it is the one native target in the wheel that ignores `CXXFLAGS`. This PR passes `CXXFLAGS` through, ahead of the script's own flags.  ## Claims  - The bundled DeepGEMM `_C…

### #59917 — [[Bugfix] Scope deprecated CLI argument warnings to the parser that defines them](https://github.com/vllm-project/vllm/pull/59917)
- **作者**: TylerMa-debugg  **时间**: 2026-10-04 13:28 CST
- **标签**: bug
- **摘要**: ## Overview  Fixes #48094 (also reported earlier as #42852). On Python 3.10–3.12, `vllm chat` and `vllm complete` print a spurious `argument 'url' is deprecated` warning on every run, because `run-batch`'s deprecated `--url` leaks into sibling subcommands.  cc @hmellor  ## Claims  - `vllm chat` / `v…

### #59916 — [[Feature][DCP] Gather-based DCP for MLA attention without native DCP (+ DSv4)](https://github.com/vllm-project/vllm/pull/59916)
- **作者**: LucasWilkinson  **时间**: 2026-10-04 13:09 CST
- **标签**: deepseek, mrv2, DSv4
- **摘要**: ## Overview  Draft: a backend-agnostic, gather-based DCP for MLA models that can't run DCP natively. It is an alternative to layer-sharded KV (KVPP, #59059). It also includes two bug fixes found while validating it: DCP group rank order under PCP, and the DSv3.2/GLM top-k across PP stages.  ## Claim…

### #59915 — [[Model][LoRA] Load Inkling PEFT adapters in the serving layout](https://github.com/vllm-project/vllm/pull/59915)
- **作者**: aoshen02  **时间**: 2026-10-04 12:16 CST
- **标签**: inkling
- **摘要**: ## Problem  An Inkling PEFT adapter saved from the Transformers model cannot be loaded directly by the vLLM Inkling model. PEFT uses `self_attn.{q,k,v,r,o}_proj`, separate dense gate/up factors, and flattened factors for routed and sink experts. The serving model instead uses fused attention project…

### #59914 — [[Bugfix][Frontend] Count prompt-opened Inkling reasoning tokens](https://github.com/vllm-project/vllm/pull/59914)
- **作者**: harshaa765  **时间**: 2026-10-04 11:42 CST
- **标签**: bug, tool-calling, inkling
- **摘要**: ## Overview  `InklingParser.count_reasoning_tokens` returns `0` whenever the thinking block was opened by the prompt rather than by a generated token, so `usage.reasoning_tokens` is wrong for prefilled-thinking Inkling requests. The override is unnecessary and the inherited `ParserEngine` implementa…

### #59913 — [[CI] Auto-label pooling PRs and issues](https://github.com/vllm-project/vllm/pull/59913)
- **作者**: taneem-ibrahim  **时间**: 2026-10-04 11:37 CST
- **标签**: ci/build
- **摘要**: ### Purpose  Add automatic `pooling` labels so pooling reviewers can find relevant PRs and issues.  ### AI assistance disclosure  Codex (GPT-6) drafted the code change and ran local validation.

### #59912 — [[Bugfix] flashinfer_moe_ep_cutedsl: reject it without nvshmem4py and install nvshmem4py on CUDA 13](https://github.com/vllm-project/vllm/pull/59912)
- **作者**: wangshangsam  **时间**: 2026-10-04 11:36 CST
- **标签**: bug, ci/build, deepseek, nvidia
- **摘要**: ## Overview  Fixes #59903. `--moe-backend flashinfer_moe_ep_cutedsl` loaded the whole checkpoint and then failed with `ModuleNotFoundError: No module named 'nvshmem'`, because `nvshmem4py` isn't installed. This PR rejects the backend at selection when `nvshmem.core` isn't usable, and installs `nvshm…

### #59911 — [[Bugfix] Alias vendored DeepGEMM submodules for FlashInfer MoE-EP](https://github.com/vllm-project/vllm/pull/59911)
- **作者**: wangshangsam  **时间**: 2026-10-04 11:36 CST
- **标签**: bug, nvidia
- **摘要**: ## Overview  Fixes #59904. With only vLLM's vendored DeepGEMM installed (the default in vLLM's images), `--moe-backend flashinfer_moe_ep_mega_deep_gemm` crashes with `ImportError: generic_type: type "Runtime" is already registered!` wherever FlashInfer does `from deep_gemm.utils import ...`. This PR…

### #59910 — [[Test] Fix two kernel tests on Blackwell GPUs](https://github.com/vllm-project/vllm/pull/59910)
- **作者**: yifanFengg  **时间**: 2026-10-04 11:03 CST
- **摘要**: ## Overview  Fix two test-side bugs that only surface on SM100-family (Blackwell) GPUs. I found both by running the kernel tests on NVIDIA GB300 (SM 10.3); neither test is exercised on Blackwell by CI today. Test-only change, no product code is touched.  ## Claims  - `test_silu_mul_fp8_quant_deep_ge…

### #59909 — [[Core] Release the CUDA graph pool on sleep instead of backing it up](https://github.com/vllm-project/vllm/pull/59909)
- **作者**: aoshen02  **时间**: 2026-10-04 10:40 CST
- **标签**: documentation, nvidia
- **摘要**: ## Purpose  Follow-up to #59160. With `sleep_mode_offload_cudagraph=True`, sleep copied the CUDA graph pool to pinned host memory and copied it back on wake. The copy is unnecessary: after capture, no tensor is allocated in the pool. It only holds scratch (activations, kernel workspaces, indexer log…
