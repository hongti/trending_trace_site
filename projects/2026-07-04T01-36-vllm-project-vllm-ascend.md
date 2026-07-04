# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-04 01:36 UTC

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期的动态摘要：

### 🚀 版本发布
* 近期无版本发布动态。

### ❓ Issue 动态
* 本次统计周期内无新增独立 Issue 记录，部分已知问题已通过对应的 PR 直接修复（如 Qwen3 310P 量化支持 #10666、DeepSeek-OCR2 310P 适配 #10667）。

### 🛠 PR 动态
PR 活动主要集中在**新特性支持、性能优化、Python 3.12 兼容性修复及 CI 维护**上，重要变更如下：

**1. 新特性与模型支持**
* **AscendStore 回移至 v0.22.1** (#11428)：将 AscendStore retention/load-mask 流程及 DeepSeek V4 hybrid KV Pool 修复从 v0.20.2rc 回移至 v0.22.1rc 分支。
* **310P 支持 Qwen3 W8A8 量化** (#11427)：新增 310P 路径的 `compressed-tensors` W8A8 量化配置，支持加载 LLM-Compressor 格式的 Qwen3 检查点。
* **新增 Minimax m3 模型开发** (#11420)：初步引入 Minimax m3 模型适配支持。

**2. 核心性能优化**
* **310P 随机采样大幅提速** (#11419)：将 310P 上的 exponential noise 生成从 CPU 侧移至 NPU 端侧计算，消除了每步 decode 约 **36ms** 的 Host-Device 往返开销，显著提升推理速度。

**3. 关键 Bug 修复与兼容性**
* **修复 Python 3.12 兼容性** (#11421, #11422)：修复 NPU 运行时在 Python 3.12 下的兼容问题，包括映射 `torch.accelerator` 内存接口以支持通用 vLLM 显存分析，解决 `ops` 模块循环导入，以及修正 spec-decode rejection sampler。
* **修复 CP hybrid 组 Hash 粒度** (#11425)：在启用 KV transfer 的 Ascend CP hybrid KV cache 组中保留细粒度的 request block hashes，避免本地 prefix caching 关闭时 Hash 粒度退化。
* **DeepSeek-OCR2 适配 310P** (#11424)：适配 DeepSeek-OCR2 在 Atlas 300I DUO (310P) 单卡 FP16 推理，修复了 ND 算子、相对位置编码及 RoPE 等后端兼容问题。

**4. CI 与工程化改进**
* **修复 mooncake 安装 HTTP 错误** (#11426)：解决 CI 流程中安装 mooncake 依赖时的网络报错。
* **规范 shellcheck 严重性级别** (#11423)：修复 pre-commit hook 忽略 `SHELLCHECK_OPTS` 的问题，防止仓库全局的 shellcheck 警告误阻断无关 PR。

---

## 🔀 Pull Requests

### #11428 — [[Feature] Support AscendStore retention masks for v0.22.1](https://github.com/vllm-project/vllm-ascend/pull/11428)
- **作者**: Pz1116  **时间**: 2026-07-04 01:35 UTC
- **摘要**: ### What this PR does / why we need it?  This PR rebases #10439 onto the `releases/v0.22.1rc` branch.  It backports the AscendStore retention/load-mask flow and DeepSeek V4 hybrid KV Pool fixes from the v0.20.2rc PR to v0.22.1rc, while preserving the release branch's existing hybrid/Mamba coordinato…

### #11427 — [[Feat][Quantization] Support Qwen3 W8A8 compressed-tensors on 310P](https://github.com/vllm-project/vllm-ascend/pull/11427)
- **作者**: treason258  **时间**: 2026-07-03 18:24 UTC
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Fixes #10666.  This PR adds the 310P path needed to serve LLM-Compressor `compressed-tensors` W8A8 checkpoints such as `ramblingpolymath/Qwen3-32B-W8A8`:  - Registers a 310P `compressed-tensors` quantization config so the 310P scheme registry is used. - Route…

### #11426 — [[CI] fix http error when install mooncake](https://github.com/vllm-project/vllm-ascend/pull/11426)
- **作者**: MrZ20  **时间**: 2026-07-03 12:52 UTC
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? fix error when install mooncake.  https://github.com/vllm-project/vllm-ascend/actions/runs/28659357797/job/84996715240#step:13:91  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https:…

### #11425 — [[BugFix] Keep connector hash granularity with CP hybrid groups](https://github.com/vllm-project/vllm-ascend/pull/11425)
- **作者**: Pz1116  **时间**: 2026-07-03 12:51 UTC
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This PR keeps fine-grained request block hashes when KV transfer is enabled for Ascend CP hybrid KV cache groups, even if local prefix caching is disabled.  Without this, the Ascend CP resolver returns `hash_block_size == scheduler_block_size` whenever `enabl…

### #11424 — [[BugFix][Model] Support DeepSeek-OCR2 on 310P](https://github.com/vllm-project/vllm-ascend/pull/11424)
- **作者**: treason258  **时间**: 2026-07-03 12:37 UTC
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Fixes #10667. Ref #9079.  This PR adapts DeepSeek-OCR-2 for Atlas 300I DUO (310P) single-card serving:  - Adds 310P-specific backend fixes for DeepSeek-OCR2 FP16 paths, including ND linear/LM head handling, rel-pos attention fallback, RoPE edge cases, tokeniz…

### #11423 — [[CI] Make shellcheck hook honor configured severity](https://github.com/vllm-project/vllm-ascend/pull/11423)
- **作者**: MengqingCao  **时间**: 2026-07-03 11:59 UTC
- **标签**: module:tools, merge-conflicts
- **摘要**: ## What this PR does / why we need it?  The shellcheck pre-commit hook currently ignores `SHELLCHECK_OPTS` even though CI exports it in `pr_test.yaml`. As a result, repo-wide shellcheck warnings can fail unrelated PRs during `pre-commit run --all-files`. This PR makes `tools/shellcheck.sh` pass the …

### #11422 — [[BugFix] Fix NPU runtime compatibility for Python 3.12](https://github.com/vllm-project/vllm-ascend/pull/11422)
- **作者**: ShuhaoZhangTony  **时间**: 2026-07-03 11:50 UTC
- **标签**: module:ops, module:core
- **摘要**: ## Summary  - Map `torch.accelerator.get_memory_info` and `memory_allocated` to the NPU implementations so generic vLLM memory profiling works on Ascend. - Avoid eager `vllm_ascend.ops` imports while `vllm_ascend.device.device_op` is still initializing. - Make the spec-decode rejection sampler helpe…

### #11421 — [Fix NPU runtime compatibility for Python 3.12](https://github.com/vllm-project/vllm-ascend/pull/11421)
- **作者**: ShuhaoZhangTony  **时间**: 2026-07-03 10:40 UTC
- **标签**: module:ops, module:core
- **摘要**: ## Summary  - Map `torch.accelerator.get_memory_info` and `memory_allocated` to the NPU implementations so generic vLLM memory profiling works on Ascend. - Avoid eager `vllm_ascend.ops` imports while `vllm_ascend.device.device_op` is still initializing. - Make the spec-decode rejection sampler helpe…

### #11420 — [[Feature]Minimax m3 dev](https://github.com/vllm-project/vllm-ascend/pull/11420)
- **作者**: AuroraEmiya  **时间**: 2026-07-03 10:38 UTC
- **标签**: documentation, ci/build, module:tests, module:ops, module:quantization, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/a30addc7548a9a8b9b3323a7bc3eb7d7c4895d1c

### #11419 — [[Ops][Feature] Optimize 310P random sampling with on-device exponential noise](https://github.com/vllm-project/vllm-ascend/pull/11419)
- **作者**: vladimirevmenoff  **时间**: 2026-07-03 10:36 UTC
- **摘要**: ### What this PR does / why we need it?  This PR replaces the CPU-side exponential noise generation in `_random_sample_310p` with an on-device Uniform + inverse-CDF implementation. This eliminates a per-decode-step host-device round trip, which previously cost ~36 ms per decode step on 310P.  ### Do…
