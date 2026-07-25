# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-25 09:05 CST

## AI 总结

以下是对 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🚀 Release（版本发布）
本期暂无新的 Release 版本发布信息。

---

### 🐛 Issue（问题与提议）
**1. CI 失效问题**
*   **PyTorch 编译单元测试失败 (#49772)**：`test_dynamic_shapes_compilation` 测试中，在同一进程启动 compiled 和 eager LLM 后，删除 compiled 模型并执行 `gc.collect()` 等清理操作时出现问题。
*   **语音转文本 WER 测试失败 (#49771)**：由于 `evaluate==0.4.3` 调用了新版 `huggingface_hub`（1.22.0）中已移除的 `HfFolder` API，导致三项 WER 正确性测试全部报错。

**2. 功能提议**
*   **引入 REFUTE 科研评估基准 (#49769)**：建议在 vLLM 的评估/研究工具文档中加入 **REFUTE**（一款科学批判与校准基准工具），以丰富 LLM 评估和 RAG 质量测量生态。

---

### 🔧 Pull Request（代码变更与修复）
**1. 重要修复**
*   **保留投机解码的 Draft Buffer (#49774)**：修复了 Speculative Decoding 在 Level-2 sleep/wake（休眠/唤醒，会释放模型内存池）时，draft-model buffers 被错误丢弃的问题，确保仅恢复 target buffers。
*   **修复在线权重量化在 TP 中的 Scale 不一致 (#49764)**：此前在线量化从 rank-local shards 推导 scale，导致张量并行（TP）的打包方式与未分片/专家并行不一致；本 PR 实现了在 TP 中共享 online weight scales，确保一致性。

**2. CI 修复与优化**
*   **改用 jiwer 计算 WER (#49773)**：直接使用 `jiwer` 替代有依赖冲突的 `evaluate` 库计算 WER，修复了 Issue #49771。
*   **修复编译测试 Flaky 失败 (#49770)**：重构了 `VLLM_DISABLE_COMPILE_CACHE`，解决持久化 CI agent 因热缓存从磁盘加载图而跳过自定义 post-grad passes 导致的随机失败。
*   **ROCm CI 本地磁盘缓存 (#49763)**：强制 ROCm 的 native compile caches 写入本地磁盘，提升 CI 稳定性。

**3. 新特性与性能优化**
*   **FlashInfer CuTe-DSL NVFP4 量化 (#49775)**：新增可选的 FlashInfer CuTe-DSL 后端用于 NVFP4 激活量化（通过 `KernelConfig.nvfp4_input_quant_backend` 开启），默认仍使用 vLLM 内置内核。
*   **ReplaySSM 投机解码 ucache 后端 (#49766)**：为 GDN ReplaySSM 投机解码引入可选的融合 CuTeDSL "ucache" 后端（fp16 state/cache），提升性能。
*   **支持混合 MLA+SSM 模型的 NIXL P/D (#49762)**：基于重构后的 NIXL connector，重新实现了对混合 MLA+SSM 模型的 KV Connector 及 Prefill/Decode (P/D) 分离的支持。

**4. 回滚与暂存**
*   **回滚 GLM-5.2 Blackwell 解码优化 (#49768)**：回滚了此前提交的 PR #48597。
*   **暂存 Helion 生成代码 (#49767)**：标记为 [DO NOT REVIEW]，仅作暂存用。

---

## 🐛 Issues

### #49772 — [[CI Failure]: PyTorch Compilation Unit Tests - test_dynamic_shapes_compilation](https://github.com/vllm-project/vllm/issues/49772)
- **作者**: Change72  **时间**: 2026-07-25 08:04 CST
- **摘要**: ### What failed  `test_dynamic_shapes_compilation` starts a compiled `LLM` and then an eager `LLM` in the same process. It deletes the compiled model and runs `gc.collect()`, `empty_cache()`, and `synchronize()`, but the compiled EngineCore sometimes remains alive.  The compiled core initializes wit…

### #49771 — [[CI Failure]: entrypoints-integration-speech_to_text - WER tests fail through evaluate HfFolder](https://github.com/vllm-project/vllm/issues/49771)
- **作者**: Change72  **时间**: 2026-07-25 07:59 CST
- **摘要**: ## Summary  The Speech-to-Text CI job fails in all three WER correctness tests because `evaluate==0.4.3` calls `huggingface_hub.hf_api.HfFolder`, which is not available in `huggingface_hub==1.22.0`.  The incompatible dependency combination is still present on `main` at `213f681f8`.  ## Failures  - `…

### #49769 — [Add REFUTE scientific critique + calibration benchmark](https://github.com/vllm-project/vllm/issues/49769)
- **作者**: connerlambden  **时间**: 2026-07-25 07:28 CST
- **摘要**: ## Proposal  Add **REFUTE** to related / evaluation / research tooling docs if this project surfaces LLM evaluation, RAG quality, or scientific agent workflows.  REFUTE is a scientific critique + calibration benchmark (paper-grounded claims → predictions → judge scores → Brier/ECE).  - Product: http…

## 🔀 Pull Requests

### #49775 — [[Kernel] FlashInfer CuTe-DSL NVFP4 Quantization](https://github.com/vllm-project/vllm/pull/49775)
- **作者**: philipphack  **时间**: 2026-07-25 08:56 CST
- **标签**: performance, nvidia, quantization
- **摘要**: ## Purpose  Adds an opt-in FlashInfer CuTe-DSL backend for NVFP4 activation quantization through `KernelConfig.nvfp4_input_quant_backend`. The default `auto` setting continues to use vLLM's built-in kernel.  The change:  - Supports linear, 128x4, and TRTLLM small-M 8x4 scale layouts. - Routes activa…

### #49774 — [[Bugfix][Spec Decode] Preserve draft buffers across level-2 sleep](https://github.com/vllm-project/vllm/pull/49774)
- **作者**: aoshen02  **时间**: 2026-07-25 08:29 CST
- **标签**: bug, v1
- **摘要**: ## Summary  Preserve speculative decoding draft-model buffers across level-2 sleep/wake.  Level-2 sleep releases the model memory pool and can discard registered draft buffers. Restoring only the target model buffers leaves draft-side runtime metadata at cleared values after wake. This change snapsh…

### #49773 — [[CI] Compute speech WER directly with jiwer](https://github.com/vllm-project/vllm/pull/49773)
- **作者**: Change72  **时间**: 2026-07-25 08:06 CST
- **摘要**: ## Purpose  Closes #49771.  The speech correctness tests use `evaluate.load("wer")`. With the current test dependencies, `evaluate==0.4.3` calls the removed `huggingface_hub.hf_api.HfFolder` API and fails after inference completes.  This change computes WER directly with `jiwer`, which is already a …

### #49770 — [[CI] fix compile test | refactor VLLM_DISABLE_COMPILE_CACHE for tests](https://github.com/vllm-project/vllm/pull/49770)
- **作者**: divakar-amd  **时间**: 2026-07-25 07:52 CST
- **标签**: ready
- **摘要**: Fixes `tests/compile/test_compile_ranges.py` tests that flakily failed on persistent CI agents when a warm vLLM compile cache caused graphs to load from disk, skipping the custom post-grad passes the tests count.   Adds a new `disable_vllm_compile_cache` fixture (fresh Inductor cache + `VLLM_DISABLE…

### #49768 — [Revert "[Perf][GLM-5.2] Blackwell decode optimizations"](https://github.com/vllm-project/vllm/pull/49768)
- **作者**: WoosukKwon  **时间**: 2026-07-25 07:01 CST
- **标签**: new-model, speculative-decoding, ci/build, v1, deepseek, nvidia
- **摘要**: Reverts vllm-project/vllm#48597

### #49767 — [[DO NOT REVIEW]Checked in helion generated code](https://github.com/vllm-project/vllm/pull/49767)
- **作者**: yushangdi  **时间**: 2026-07-25 06:45 CST
- **标签**: quantization
- **摘要**: ## Purpose  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such as providing test command. - [ ] The …

### #49766 — [Gdn ucache backend](https://github.com/vllm-project/vllm/pull/49766)
- **作者**: ameynaik-hub  **时间**: 2026-07-25 06:30 CST
- **标签**: performance, needs-rebase, v1, qwen, quantization
- **摘要**: # [Perf] GDN ReplaySSM spec decode: opt-in fused CuTeDSL "ucache" backend (fp16 state/cache)  > **Stacked PR — review only the top 2 commits.** This PR sits on top of ReplaySSM > PR **#47576**; until that merges, GitHub's diff view includes both. The new work is > the last 2 commits (core backend + …

### #49764 — [[Quantization] Share online weight scales across TP](https://github.com/vllm-project/vllm/pull/49764)
- **作者**: S1ro1  **时间**: 2026-07-25 06:09 CST
- **标签**: quantization
- **摘要**: ## Purpose  Online weight quantization currently derives scales from rank-local shards. Consequently, tensor-parallel packing can use a different recipe from unsharded or expert-parallel packing of the same BF16 checkpoint.  This PR:  - reduces the scalar dense-weight `amax` across the TP group befo…

### #49763 — [[ROCm][CI] Force native compile caches onto local disk](https://github.com/vllm-project/vllm/pull/49763)
- **作者**: aarushjain29  **时间**: 2026-07-25 05:31 CST
- **标签**: rocm, ready, ci/build

### #49762 — [[KV Connector] Support NIXL P/D for hybrid MLA+SSM models ](https://github.com/vllm-project/vllm/pull/49762)
- **作者**: njhill  **时间**: 2026-07-25 04:46 CST
- **标签**: v1, kv-connector
- **摘要**: Re-implements the intent of https://github.com/vllm-project/vllm/pull/44848 on top of the reworked NIXL connector, from the current architecture rather than the original patch. KimiLinear pools its KDA (GDN-typed MambaSpec) and MLA layers into shared HMA tensors, making every region dual-purpose. Si…
