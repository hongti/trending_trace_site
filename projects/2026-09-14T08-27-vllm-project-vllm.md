# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-14 16:27 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
近期主要报告了两个底层硬件与算子相关的 Bug：
- **DFlash2 在 XPU 编译失败** (#56787)：在 Intel Arc Pro B70 (Xe2) 上使用 DFlash2 跑_draft_模型时，`torch.compile` 会因自定义算子伪内核中的动态形状步长断言而失败。临时解决方法是禁用 draft 类的编译。
- **HiSparse MLA 索引器 CUDA 崩溃** (#56785)：在 NVIDIA GB200 NVL 环境下进行分段 cudagraph 捕获时，HiSparse MLA 索引器因同步事件等待时机不当（在捕获段外等待段内记录的事件）而引发 `invalid argument` CUDA 错误。

### 🔧 PR 动态
近期合并/提交的 PR 集中在跨硬件适配、性能优化及架构重构上：
- **新特性与性能优化**：
  - **采样快速路径** (#56782)：为 V1 runner 引入了精简版的目标 token 评分快速路径，显著提升重排器等仅需计算少量候选 token 场景的效率。
  - **ROCm MoE 优化** (#56789)：为 Qwen MoE 共享专家门控引入 Tiny-dot kernel，优化单 token 解码性能。
  - **注意力机制灵活性** (#56784)：允许用户在使用 `FLASHINFER_MLA` 后端时，显式选择 FlashInfer 的 CuTe DSL MLA 解码内核。
  - **Rust 前端 MoE 支持** (#56778, #56779)：支持在 Python worker 中捕获路由专家选择，并通过 gRPC 将其作为不透明字节传输给 Rust 前端。
- **硬件适配与调优**：
  - **Intel XPU** (#56788)：为 Intel Arc Pro B70 上的 Qwen3-30B-A3B-GPTQ-Int4 添加了调优后的 `int4_w4a16` Triton fused-MoE 配置。
- **Bug 修复与安全**：
  - **XPU 溢出修复** (#56781)：修复了 XPU 上 USM 设备指针可能超出有符号 64 位整数正数范围导致的 `OverflowError`。
  - **EPD 媒体选项修复** (#56786)：修复了 EPD 代理在构造编码器请求时丢失特定媒体处理选项的 Bug。
  - **安全加固** (#56780)：处理了 LMCache MP 请求被提前中止时追踪器不存在的问题（原代码使用了不安全的 `assert`）。
  - **CI 依赖固定** (#56783)：在 XPU 环境将 `gpt-oss` 固定为 0.0.8，规避新版 0.0.9 的精度问题。

### 🚀 Release 动态
近期无新的 Release 版本发布。

---

## 🐛 Issues

### #56787 — [[Bug]: DFlash2 draft model fails torch.compile on XPU — dynamic-shape stride assert in custom-op fake kernel (workaround: disable compile on draft class)](https://github.com/vllm-project/vllm/issues/56787)
- **作者**: SergiioB  **时间**: 2026-09-14 16:15 CST
- **摘要**: ### Reproduction steps  On XPU (Intel Arc Pro B70, Xe2), serve any target with the DFlash2 drafter (vLLM PR #52816 lineage, `method: dflash`, e.g. incoai/Qwen3.8-27B-DFlash2) **without** `--enforce-eager` on the vllm/vllm-openai-xpu image (0.27.2rc1.dev77+gac7509e2b). Engine init dies during the dra…

### #56785 — [[Bug]: HiSparse MLA indexer crashes with CUDA error: invalid argument during piecewise cudagraph capture - logical_topk_ready is recorded inside the capture segment but waited on after eager_break_during_capture ends capture](https://github.com/vllm-project/vllm/issues/56785)
- **作者**: janbernloehr  **时间**: 2026-09-14 16:14 CST
- **摘要**: ### Your current environment  Captured on the failing run (TP rank 0) inside the official `vllm/vllm-openai:nightly` container, NVIDIA GB200 NVL, 4 GPUs, aarch64. Long constant blobs (`NVIDIA_REQUIRE_CUDA`, CPU flag/vulnerability rows, empty NUMA rows, and the remaining pinned `nvidia-*` CUDA 13.0 c…

## 🔀 Pull Requests

### #56789 — [[ROCm][Perf] Tiny-dot kernel for the Qwen MoE shared-expert gate](https://github.com/vllm-project/vllm/pull/56789)
- **作者**: roberteg16  **时间**: 2026-09-14 16:16 CST
- **标签**: performance, rocm, qwen
- **摘要**: <!-- markdownlint-disable --> ## Summary  A Qwen MoE block gates its shared expert with `sigmoid(shared_expert_gate(x))`, where `shared_expert_gate` is a `ReplicatedLinear(hidden_size, 1)`. During decode with a single token that is a 1x1xK GEMM, and hipBLASLt answers it with a 64x96x32 macro tile pl…

### #56788 — [[XPU] Add tuned int4_w4a16 fused-MoE config for Qwen3-30B-A3B-GPTQ-Int4 on Intel Arc Pro B70](https://github.com/vllm-project/vllm/pull/56788)
- **作者**: pmanczak  **时间**: 2026-09-14 16:15 CST
- **标签**: intel-gpu, qwen
- **摘要**: ## Summary  Adds a tuned Triton fused-MoE (`int4_w4a16`, GPTQ) config for **Intel(R) Arc(TM) Pro B70 Graphics**, for the `E=128, N=768` shape (`moe_intermediate_size=768`, 128 experts, `group_size=128`) used by the real, published **`Qwen/Qwen3-30B-A3B-GPTQ-Int4`** checkpoint (and shared by `Qwen3-C…

### #56786 — [[Bugfix][EPD] Preserve media processing options in encoder requests](https://github.com/vllm-project/vllm/pull/56786)
- **作者**: jiangkuaixue123  **时间**: 2026-09-14 16:15 CST
- **标签**: bug, documentation, kv-connector
- **摘要**: ## Purpose  The EPD proxy drops per-request media processing options when constructing encoder primer requests. An encoder can therefore use the default image size while PD applies the requested override, producing incompatible embeddings and placeholders. With Qwen3.5-35B-A3B, the pre-fix proxy rep…

### #56784 — [[Attention] Allow explicit FlashInfer CuTe DSL MLA decode selection](https://github.com/vllm-project/vllm/pull/56784)
- **作者**: bzantium  **时间**: 2026-09-14 16:08 CST
- **标签**: documentation, nvidia
- **摘要**: ## Purpose  Allow users of `FLASHINFER_MLA` to explicitly select FlashInfer's existing CuTe DSL decode kernel:  ```bash vllm serve MODEL --attention-backend FLASHINFER_MLA \     --attention-config.flashinfer_mla_decode_backend cute-dsl ```  The default `auto` preserves existing dispatch, including t…

### #56783 — [[XPU][CI] pin gpt-oss to 0.0.8](https://github.com/vllm-project/vllm/pull/56783)
- **作者**: zhenwei-intel  **时间**: 2026-09-14 16:08 CST
- **标签**: intel-gpu, ci/build, gpt-oss
- **摘要**: ## Purpose `pip compile` resolved `gpt-oss` to the latest 0.0.9, but that version has accuracy issues, so manually pin it back to `0.0.8` to align with `cuda.txt`  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The …

### #56782 — [[Feature][Sampling] Compact target-token-scoring fast path (V1 runner)](https://github.com/vllm-project/vllm/pull/56782)
- **作者**: demondfdfjg  **时间**: 2026-09-14 16:05 CST
- **标签**: documentation, qwen
- **摘要**: Releated issue: #56767  Rerankers and relevance scorers finish prefill and only need the sore for a small fixed set of K candidate tokens(often K=2, yes/no),but the runner still projects the last hidden state through the full-vocab LM Head([B,H] @ [V, H]^T ->[B,V], V≈151k for Qwen2.5) before the sam…

### #56781 — [[XPU][UT] Fix OverflowError when storing embeds.data_ptr()](https://github.com/vllm-project/vllm/pull/56781)
- **作者**: RyanMa29  **时间**: 2026-09-14 15:48 CST
- **标签**: intel-gpu, mrv2
- **摘要**: ## Purpose  On XPU, USM device pointers returned by `embeds.data_ptr()` can exceed the positive range of a signed 64-bit integer.  `PromptEmbedsState.add_request()` stores the prompt embedding pointer in a NumPy-backed `torch.int64` buffer. Assigning a high-bit XPU pointer directly to this buffer ra…

### #56780 — [[Security] Handle absent tracker in LMCache MP request_finished](https://github.com/vllm-project/vllm/pull/56780)
- **作者**: jperezdealgaba  **时间**: 2026-09-14 15:45 CST
- **标签**: kv-connector
- **摘要**: ## Summary  `LMCacheMPConnectorUpstream.request_finished()` calls `_get_request_tracker()` which uses a bare `assert` to check tracker existence. When a request is aborted before scheduling (via the `abort_immediately` path in `v1/engine/core.py:484-488`), no tracker is ever created because `get_num…

### #56779 — [[Rust Frontend] Transport terminal routed-expert payloads over gRPC](https://github.com/vllm-project/vllm/pull/56779)
- **作者**: biswapanda  **时间**: 2026-09-14 15:45 CST
- **标签**: gpt-oss, rust
- **摘要**: ## Purpose  Transport routed-expert output as opaque bytes through the Rust frontend. The request carries the prompt offset used by the backend, engine-core multipart output is resolved without interpreting the payload schema, aggregation retains the terminal snapshot, and gRPC emits it only on the …

### #56778 — [[MoE] Emit terminal routed-expert payloads for the Rust frontend](https://github.com/vllm-project/vllm/pull/56778)
- **作者**: biswapanda  **时间**: 2026-09-14 15:45 CST
- **标签**: rust, scheduler
- **摘要**: ## Purpose  Capture routed-expert selections in the Python vLLM worker, retain the latest per-sequence snapshot in scheduler output, serialize it through the existing engine-core auxiliary-frame mechanism, and attach it only to the terminal model-runner output. The Rust transport remains schema-opaq…
