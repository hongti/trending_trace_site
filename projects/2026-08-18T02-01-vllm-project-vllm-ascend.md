# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-18 10:01 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库最近动态的中文摘要：

### 🐛 Issue (问题反馈)
1. **NPU 算子运行报错 (#14456)**：在运行 `test_colqwen3_5.py` 测试时，触发上游 NPU 算子 `aclnnChunkGatedDeltaRule` 调用失败（错误码 161002）。
2. **M-RoPE 模型投机解码失败 (#14453)**：使用 draft model 进行投机解码时，M-RoPE 架构模型（如 GLM-4.6V / GLM-4.6V-Flash）报错 `NotImplementedError`，提示当前不支持 M-RoPE。

---

### 🔧 Pull Request (代码合并)

**🚀 新特性与模型支持**
* **新增 Kimi K3 模型支持 (#14454)**：将 Kimi K3 模型支持从 v0.26.0rc 分支迁移适配至基于 vLLM 0.27 的 main 分支。
* **新增 A5 MegaMoE 融合后端 (#14449)**：为 A5 MXFP MoE 请求引入逻辑 FUSED_MC2 路径，并添加带有专用 prepare/finalize 处理的 MegaMoE 封装器。
* **Mooncake Store 消费端 Decode 卸载 (#14455)**：[WIP] 为 Mooncake Store consumers 添加 decode offloading 功能。

**⚡ 性能优化**
* **融合 SFA DCP output all-to-all (#14457)**：使用自定义 Ascend 算子替换 SFA DCP 输出后处理的热点代码路径，包含支持 stride 感知的 Triton pack 和单个 HCCL all-to-all 通信，提升 attention 处理性能。

**🐛 Bug 修复**
* **修复 GLM5.2 MTP Bug (#14451)**：修复 GLM5.2 在多 token 预测（MTP）中未使用共享索引的问题。
* **修复 MXFP Shared-Expert 崩溃 (#14450)**：`aclnnSwigluGroupQuant` 算子不支持 `glu_alpha/glu_bias` 参数，导致 MXFP shared-expert 路径崩溃。此 PR 移除了这两个不匹配的 kwargs 以修复该错误。

**🛠 重构与工程维护**
* **废弃 Hybrid Mamba 补丁 (#14447)**：清理并移除了 hybrid Mamba 配置相关的猴子补丁（monkey patches）。
* **新增代码所有者 (#14452)**：为 CMake 和 C 源文件添加新的 Code Owner。
* **CI 与构建更新 (#14458, #14446)**：CI 流水线更新（Hjp 818）；支持使用 CPUrunner 构建镜像。

---

### 📦 Release (版本发布)
* **近期无新版本发布**。

---

## 🐛 Issues

### #14456 — [[Bug][Upstream]: RuntimeError: npu_chunk_gated_delta_rule:../third_party/op-plugin/op_plugin/ops/opapi/ChunkGatedDeltaRuleKernelNpuOpApi.cpp:57 NPU function error: call aclnnChunkGatedDeltaRule failed, error code is 161002](https://github.com/vllm-project/vllm-ascend/issues/14456)
- **作者**: jiangyunfan1  **时间**: 2026-08-18 09:07 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Your output of above commands here ```  </details>   ### 🐛 Describe the bug  tests/models/multimodal/pooling/test_colqwen3_5.py https://github.com/vllm-project/vllm-ascend/actions/runs/320153663…

### #14453 — [[Bug]: Speculative decoding with draft model fails for M-RoPE models (e.g., GLM-4.6V): "does not support M-RoPE yet"](https://github.com/vllm-project/vllm-ascend/issues/14453)
- **作者**: pcb1990  **时间**: 2026-08-17 20:37 CST
- **标签**: bug, glm-4v, multimodal-understanding
- **摘要**: ### Your current environment  When using `--spec-method draft_model` with a multimodal model that uses M-RoPE (e.g., GLM-4.6V / GLM-4.6V-Flash), vLLM fails with `NotImplementedError: Speculative Decoding with draft models or parallel drafting does not support M-RoPE yet`.  This blocks speculative de…

## 🔀 Pull Requests

### #14458 — [[CI] Hjp 818](https://github.com/vllm-project/vllm-ascend/pull/14458)
- **作者**: xqchen7  **时间**: 2026-08-18 09:50 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14457 — [perf(attention): fuse SFA DCP output all-to-all](https://github.com/vllm-project/vllm-ascend/pull/14457)
- **作者**: pisceskkk  **时间**: 2026-08-18 09:26 CST
- **标签**: module:tests, module:ops
- **摘要**: ## What this PR changes  This PR replaces the SFA DCP output post-processing hot path with a custom Ascend operator:  - stride-aware Triton pack for the local attention output and LSE - one HCCL all-to-all instead of separate output/LSE collectives - fused numerically stable LSE combine and weighted…

### #14455 — [[wip][Feature] Add decode offloading to Mooncake Store consumers](https://github.com/vllm-project/vllm-ascend/pull/14455)
- **作者**: HF-001  **时间**: 2026-08-18 08:53 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it? todo  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested? todo  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14454 — [[Feature][Model] Add Kimi K3 support for vLLM 0.27](https://github.com/vllm-project/vllm-ascend/pull/14454)
- **作者**: maoxx241  **时间**: 2026-08-17 20:45 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  This is a stacked draft that depends on #14426.  It brings Kimi K3 model support from `releases/v0.26.0rc` to the vLLM 0.27-based `main` branch. The old release implementation patched Kimi configuration, rendering, processing, parsers, scheduling, and model c…

### #14452 — [[Misc]Add new code owners for CMake and C source files](https://github.com/vllm-project/vllm-ascend/pull/14452)
- **作者**: ZT-AIA  **时间**: 2026-08-17 20:32 CST
- **摘要**: ### What this PR does / why we need it? Add new code owners for CMake and C source files  ### Does this PR introduce _any_ user-facing change? No.  ### How was this patch tested? No. - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c5…

### #14451 — [[BugFix] Fix GLM5.2 MTP bug that dose not utilize shared index](https://github.com/vllm-project/vllm-ascend/pull/14451)
- **作者**: lijiahang226  **时间**: 2026-08-17 20:29 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/d02df748bf9efd99022f1a062597dc3cb3808485

### #14450 — [[BugFix] Fix bug that aclnnSwigluGroupQuant uses a unaccepted argument](https://github.com/vllm-project/vllm-ascend/pull/14450)
- **作者**: lijiahang226  **时间**: 2026-08-17 20:26 CST
- **标签**: module:ops
- **摘要**: …roup_quant  aclnnSwigluGroupQuant does not accept glu_alpha/glu_bias, so the MXFP shared-expert path crashed with "Unknown keyword argument 'glu_alpha'". Remove the two kwargs to match the W4A8MXFP main path in device_op.py, and fail loudly when a non-default alpha/beta is configured instead of sil…

### #14449 — [feat(moe): add A5 MegaMoE fused backend](https://github.com/vllm-project/vllm-ascend/pull/14449)
- **作者**: wt0671  **时间**: 2026-08-17 20:17 CST
- **标签**: module:tests, module:ops, module:core, module:quantization
- **摘要**: Route supported A5 MXFP MoE requests through the logical FUSED_MC2 path and add a MegaMoE wrapper with dedicated prepare/finalize handling. Add info logs for path selection and operator argument validation.  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change…

### #14447 — [[Refactor][Platform] Retire hybrid Mamba config patches](https://github.com/vllm-project/vllm-ascend/pull/14447)
- **作者**: kokomicx  **时间**: 2026-08-17 19:56 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  This PR retires the following hybrid Mamba configuration monkey patches:  - `vllm_ascend/patch/platform/patch_mamba_config.py` - `vllm_ascend/patch/platform/patch_mamba_config_310.py`  Hybrid Mamba block-size selection, attention/Mamba page-size alignment, an…

### #14446 — [build image with CPUrunner](https://github.com/vllm-project/vllm-ascend/pull/14446)
- **作者**: wenshun88  **时间**: 2026-08-17 19:45 CST
- **标签**: ci/build, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f
