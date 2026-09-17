# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-17 13:13 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要。本期数据主要包含拉取请求（PR）活动，未涉及 Issue 和 Release 更新。

### 📝 Pull Request (PR) 动态

**1. 新模型与硬件后端支持**
* **新增 Qwen3.8-Flash-Next CPU 后端** (#57294)：为 `Qwen3.8-Flash-Next` 模型添加了原生的 x86 CPU 后端支持。
* **升级 XPU 内核** (#57291)：将 XPU 内核版本升级至 0.1.15.1。

**2. 重要 Bug 修复**
* **DeepSeek-V4.1 相关修复**：
  * 修复了在 SM120 硬件上使用 64-token sparse-MLA 页面时的声明问题 (#57292)。
  * 修复了在 ROCm gfx942 显卡上进行图捕获时的段错误（segfault）问题 (#57288)。
* **推测解码修复** (#57290)：修复了在 PP1（流水线并行度为1）下隐藏状态提取器的草稿并行配置问题。
* **AMD/ROCm 修复** (#57289)：修复了嵌套 RoPE（Nested RoPE）验证补丁无法自动运行，导致特定模型加载失败的问题。
* **FlashInfer 修复** (#57285)：隔离了 FlashInfer BF16 的补充自动调优过程，避免其对其他算子（如 MXFP8 drafter GEMM）造成异常影响。

**3. 性能优化**
* **ROCm gfx1201 性能优化** (#57286)：针对 gfx1201 的 M8 BF16 head 交错了 M4 分组，在单次启动中写入连续输出以提升性能。

**4. CI 与工程维护**
* **修复 Lint 错误** (#57293)：修复了主分支上由于先前合并导致 pre-commit CI 失败的问题。
* **调整 CI 超时时间** (#57287)：将 Elastic EP Scaling 测试步骤的超时时间从 30 分钟延长至 40 分钟，以避免测试被意外中断。

---
*注：本次提供的数据未包含 Issue 讨论与 Release 发版信息。*

---

## 🔀 Pull Requests

### #57294 — [[CPU] Add Qwen3.8-Flash-Next CPU backend](https://github.com/vllm-project/vllm/pull/57294)
- **作者**: tianmu-li  **时间**: 2026-09-17 13:03 CST
- **标签**: ci/build, qwen, cpu
- **摘要**: ## Purpose  This PR adds a native x86 CPU backend for Qwen3.8-Flash-Next (`Qwen4ExpForCausalLM` and `Qwen4ExpForConditionalGeneration`) using Model Runner V2.  The CPU backend:  - implements the Qwen4Exp model, model state, HyperConnection, QSA, PLE, and N-gram embedding paths under the CPU-owned mo…

### #57293 — [[Lint] Fix pre-commit error on main](https://github.com/vllm-project/vllm/pull/57293)
- **作者**: shen-shanshan  **时间**: 2026-09-17 12:58 CST
- **摘要**: ## Purpose  It looks last second commit in main branch fa3622a4a [EC Connector] Add Metrics Collection (#54960) failed the pre-commit CI  [Commits · vllm-project/vllm](https://github.com/vllm-project/vllm/commits/main/).  The ruff rules are changed by [[Docs] Add `pydocstyle` to the `ruff` rules by …

### #57292 — [[Bugfix][DSv4.1] Use 64-token sparse-MLA pages on SM120](https://github.com/vllm-project/vllm/pull/57292)
- **作者**: luoyuctl  **时间**: 2026-09-17 12:54 CST
- **标签**: bug, deepseek, nvidia, DSv4.1
- **摘要**: Draft: partially addresses #56702 / #56461.  ### Problem  DeepSeek-V4.1 takes the SM12x path on compute capability 12.x (`DeepseekV4FlashInferSM120Attention`), but three sites that declare the sparse-MLA kernel block size still treat SM120 like SM100 and return 128:  | file | current | | --- | --- |…

### #57291 — [[XPU] Bump kernels to 0.1.15.1](https://github.com/vllm-project/vllm/pull/57291)
- **作者**: jikunshang  **时间**: 2026-09-17 12:42 CST
- **标签**: intel-gpu, ci/build
- **摘要**: ## Purpose  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such as providing test command. - [ ] The …

### #57290 — [[Bugfix][Spec Decode] Keep hidden-state extractor drafts at PP1](https://github.com/vllm-project/vllm/pull/57290)
- **作者**: venkywonka  **时间**: 2026-09-17 12:32 CST
- **标签**: bug
- **摘要**: ## Purpose  Fix only the `extract_hidden_states` configuration branch: its last-stage-only extractor must use a draft parallel configuration with pipeline-parallel size one, rather than aliasing the target's parallel configuration. The target keeps its original pipeline-parallel size and object iden…

### #57289 — [[AMD][Bugfix] Make the nested-RoPE validation patch reach automatic validation](https://github.com/vllm-project/vllm/pull/57289)
- **作者**: okorzh-amd  **时间**: 2026-09-17 12:31 CST
- **标签**: bug, rocm
- **摘要**: ## Summary  `_patch_hf_transformers_nested_rope_validation` never runs on the path it was written for, so `poolside/Laguna-XS-2.1-NVFP4` still fails to load:  ``` AttributeError: 'float' object has no attribute 'get'   transformers/modeling_rope_utils.py:850, in validate_rope ```  `huggingface_hub`'…

### #57288 — [[ROCm][Bugfix] Fix DeepSeek V4.1 graph capture on gfx942](https://github.com/vllm-project/vllm/pull/57288)
- **作者**: akii96  **时间**: 2026-09-17 12:23 CST
- **标签**: bug, rocm, deepseek, DSv4, DSv4.1
- **摘要**: ## Motivation  On 4× MI325X (gfx942), DeepSeek-V4.1-Flash consistently segfaults during the second FULL_AND_PIECEWISE graph-capture pass on the latest vllm nightly image -> `vllm/vllm-openai-rocm:nightly-af1c01499b289be555c475669ba50a88e96d846e`  FULL_DECODE_ONLY and --enforce-eager start successful…

### #57287 — [[CI] Raise Elastic EP Scaling step timeout 30m -> 40m](https://github.com/vllm-project/vllm/pull/57287)
- **作者**: vllm-agent  **时间**: 2026-09-17 12:20 CST
- **标签**: ci/build
- **摘要**: ## Problem  Nightly main [#89515](https://buildkite.com/vllm/ci/builds/89515) (dashboard alert #1476): `elastic-ep-scaling-test` passed all 5 tests in 28m01s, then was killed by the 30-minute step timeout during interpreter teardown, ~2s after the pytest summary.  ## Fix  `timeout_in_minutes: 30 -> …

### #57286 — [[ROCm][Perf] Interleave M4 groups for the gfx1201 M8 BF16 head](https://github.com/vllm-project/vllm/pull/57286)
- **作者**: Terrydaktal  **时间**: 2026-09-17 12:12 CST
- **标签**: rocm
- **摘要**: ## Purpose  Add a narrowly admitted eight-row BF16 `wvSplitK` path on gfx1201. It interleaves two four-row groups in one launch and writes their outputs contiguously, retaining the M4 arithmetic topology. Existing specializations use the default group count of one. The default Python serving dispatc…

### #57285 — [[Bugfix] Isolate supplemental FlashInfer BF16 autotuning](https://github.com/vllm-project/vllm/pull/57285)
- **作者**: jiahanc  **时间**: 2026-09-17 12:02 CST
- **标签**: bug, nvidia
- **摘要**: ## Purpose  The supplemental 32-token FlashInfer BF16 warmup currently applies its bounded `tuning_buckets` to every operation in the dummy run. An MXFP8 drafter GEMM with M=40 can consequently inherit the BF16 bucket cap and reuse a low-M tactic that does not support its input.  Run this supplement…
