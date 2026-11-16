# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-16 15:08 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 📋 Issue 讨论
本期共有 2 个 RFC（意见征求），主要探讨性能优化与显存节省：
* **HiSparse 针对 Sparse-MLA 的优化方案 (#67874)**：探讨在 Sparse-MLA (DSA) Prefill 阶段实现按需 KV Gather 和分层双槽复用机制，以进一步优化 GPU/CPU 间的 KV 调度。
* **LM Head 在线量化机制 (#67865)**：提议引入通用的可选机制，在运行时对目标模型的 LM Head 进行在线量化，以缓解部分较小模型因词表过大带来的显存压力。

### 🔧 PR 变更
本期合并了 9 个 PR，主要集中在 **Rust 前端能力对齐**、**性能优化**及**硬件适配修复**：

**1. Rust 前端完善（向 Python 前端看齐）**
* **新增启动参数支持**：支持 `--default-mm-loras` (#67873)、`--hf-token` (#67871)。
* **修复 KV 共享模型**：支持 `--kv-sharing-fast-prefill`，修复 Gemma3n 等模型在 Rust 前端下提示词 token 错误的问题 (#67872)。
* **安全与健壮性提升**：多模态缓存键增加域标签以防冲突 (#67870)；非流式 `generate` 接口遇到引擎错误时正确返回 500 状态码而非 200 (#67869)。

**2. 性能与核心机制优化**
* **KDA Prefill 性能提升 (#67866)**：对 GLM-5.3-Flash 和 Kimi-K3 模型，在分页状态缓存上运行 Triton KDA prefill，并在 chunk recurrence 内部存储检查点状态。
* **KV Connector 布局优化 (#67863)**：MoRIIOConnector 为非 MLA 模型优先请求 `LBHNC` KV 缓存布局，解决异构 TP WRITE 传输问题。

**3. 模型修复与硬件 (XPU) 适配**
* **MiniCPM-V 4.7 修复 (#67868)**：修复该模型非流式输出被 `minicpmv` 解析器错误归类为推理内容的问题。
* **Intel XPU 调整**：因随机精度问题暂时禁用 BMG Rust Frontend Core Correctness CI 测试 (#67867)；移除不再维护的 `VLLM_XPU_FORCE_N_CONTIG_WEIGHT` 环境变量 (#67864)。

### 🚀 Release 版本发布
* 本期无新的 Release 版本发布。

---

## 🐛 Issues

### #67874 — [[RFC][HiSparse] On-Demand KV Gather and Layerwise Two-Slot Reuse for Sparse-MLA (DSA) Prefill](https://github.com/vllm-project/vllm/issues/67874)
- **作者**: ZD-euler  **时间**: 2026-11-16 15:07 CST
- **标签**: RFC
- **摘要**: ### Motivation.  HiSparse already offloads large sparse-MLA (DSA) main KV to CPU memory, keeps lightweight indexer KV on GPU, and resolves decode KV on demand from top-k indices. However, as noted in the [HiSparse design doc](https://github.com/vllm-project/vllm/blob/55aa0cab78dedbb018ad4f4ea649f2d3…

### #67865 — [[RFC]: Opt-in online quantization of the target model LM head](https://github.com/vllm-project/vllm/issues/67865)
- **作者**: sywang24  **时间**: 2026-11-16 14:47 CST
- **标签**: quantization
- **摘要**: ### Motivation.  Would the vLLM community be interested in a general, opt-in mechanism to quantize the **target model's output LM head** online? In some comparatively smaller models, the vocabulary projection can be a meaningful portion of decode work and memory traffic. We evaluated a 27B model on …

## 🔀 Pull Requests

### #67873 — [[Rust Frontend] Support --default-mm-loras](https://github.com/vllm-project/vllm/pull/67873)
- **作者**: BugenZhao  **时间**: 2026-11-16 15:04 CST
- **标签**: rust
- **摘要**: <!-- markdownlint-disable -->  ## Overview  The Rust frontend rejects `--default-mm-loras` at startup. This PR implements it like Python: each configured modality's adapter loads as a static LoRA named by the modality, and a chat completions request that names no adapter but carries that modality ru…

### #67872 — [[Rust Frontend] Support --kv-sharing-fast-prefill](https://github.com/vllm-project/vllm/pull/67872)
- **作者**: BugenZhao  **时间**: 2026-11-16 15:04 CST
- **标签**: rust
- **摘要**: <!-- markdownlint-disable -->  ## Overview  The Rust frontend rejects `--kv-sharing-fast-prefill` at startup, so KV-sharing models such as Gemma3n that set it cannot use the Rust frontend. Under the flag, prompt tokens get incorrect logits, and Python's frontend rejects requests for prompt logprobs …

### #67871 — [[Rust Frontend] Support --hf-token](https://github.com/vllm-project/vllm/pull/67871)
- **作者**: BugenZhao  **时间**: 2026-11-16 15:04 CST
- **标签**: rust
- **摘要**: <!-- markdownlint-disable -->  ## Overview  The Rust frontend rejects `--hf-token` at startup, so a deployment that passes its Hub token on the command line cannot run under `VLLM_USE_RUST_FRONTEND=1`. This PR supports it, and, in a second commit, keeps the token out of `/server_info` on both fronte…

### #67870 — [[Rust Frontend] Domain-tag multimodal cache keys like Python](https://github.com/vllm-project/vllm/pull/67870)
- **作者**: BugenZhao  **时间**: 2026-11-16 15:04 CST
- **标签**: rust
- **摘要**: <!-- markdownlint-disable -->  ## Overview  vllm-project/vllm#66473 tagged Python's multimodal cache keys with their source (`content:<digest>` for a computed hash, `uuid:<uuid>` for a client `uuid`), so a client can no longer set `uuid` to another item's content hash and hit that item's encoder-cac…

### #67869 — [[Rust Frontend] Return 500 for engine errors on non-streaming generate](https://github.com/vllm-project/vllm/pull/67869)
- **作者**: BugenZhao  **时间**: 2026-11-16 15:03 CST
- **标签**: rust
- **摘要**: ## Overview  Non-streaming `/inference/v1/generate` on the Rust frontend answers an engine error that carries no details (`finish_reason` `error`, no `RequestError`) with a 200 whose choice has `finish_reason: "error"`. Python returns a 500 there (vllm-project/vllm#49195), and every other Rust route…

### #67868 — [[Bugfix][Model] Fix MiniCPM-V 4.7 non-streaming output parsed as reasoning](https://github.com/vllm-project/vllm/pull/67868)
- **作者**: andrew-wen2  **时间**: 2026-11-16 15:00 CST
- **标签**: bug, tool-calling
- **摘要**: ## Overview  Fix `--reasoning-parser minicpmv` returning a default MiniCPM-V 4.7 answer as `reasoning` with `content: null` in non-streaming Chat Completions. The parser config now passes the ChatML turn boundaries, so the closed `<think></think>` that the 4.7 template puts in the prompt is detected…

### #67867 — [[XPU][CI]Disable BMG Rust Frontend Core Correctness due to random accuracy issue](https://github.com/vllm-project/vllm/pull/67867)
- **作者**: zxd1997066  **时间**: 2026-11-16 14:59 CST
- **标签**: intel-gpu, ci/build, rust
- **摘要**: ## Overview  Disable BMG Rust Frontend Core Correctness due to random accuracy issue  ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance wa…

### #67866 — [[Perf][KDA] Run the Triton KDA prefill on the paged state cache with in-kernel checkpoints](https://github.com/vllm-project/vllm/pull/67866)
- **作者**: JaredforReal  **时间**: 2026-11-16 14:55 CST
- **标签**: documentation, kimi
- **摘要**: ## Overview  Run the Triton KDA prefill of GLM-5.3-Flash and Kimi-K3 (NVIDIA) on the paged recurrent state, with the checkpoint state stored from inside the chunk recurrence. In mamba `align` mode the Triton backend then exports prefill checkpoints like FlashKDA already does, so a prompt is no longe…

### #67864 — [[XPU] Remove VLLM_XPU_FORCE_N_CONTIG_WEIGHT](https://github.com/vllm-project/vllm/pull/67864)
- **作者**: zufangzhu  **时间**: 2026-11-16 14:46 CST
- **标签**: intel-gpu
- **摘要**: ## Purpose Remove the `VLLM_XPU_FORCE_N_CONTIG_WEIGHT` env var added in #53536. It was opt-in (default `0`) and we no longer want to provide this interface.  ## Changes - `vllm/envs.py`: drop the declaration and the env lambda. - `vllm/model_executor/layers/linear.py`: drop the XPU branch in `Unquan…

### #67863 — [[KV Connector][MoRIIO] Prefer the LBHNC KV cache layout for non-MLA models](https://github.com/vllm-project/vllm/pull/67863)
- **作者**: ChengYM5685  **时间**: 2026-11-16 14:41 CST
- **标签**: kv-connector
- **摘要**: ## Overview  `MoRIIOConnector` now asks for the `LBHNC` KV cache layout (head-major blocks) on non-MLA models, as `NixlConnector` does. Without it, heterogeneous-TP WRITE transfers of an all-sparse-attention model (MiniMax-M3.1) fill the RDMA send queue and the decode request never gets its KV.  ## …
