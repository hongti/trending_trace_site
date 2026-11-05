# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-05 15:14 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的简洁摘要：

### 📋 Issue 动态
*   **#21053 [RFC]: 标准化 KV cache 分配流程**
    *   **摘要**: 提出标准化 KV cache 分配机制的讨论。当前 MRV2 在处理 KV cache 布局时需要跨越多个阶段（如平台块大小选择、Ascend/上游分组等），该 RFC 旨在梳理并规范这一底层流程。

### 🔧 PR 动态
**新特性与性能优化**
*   **#21059 [Feature] GLM RecoverSSM 支持 PCP 前缀复用**: 回传至 `v0.32.0rc`，使 GLM-5.3-Flash RecoverSSM 模型能够通过 PCP 复用对齐的 prefill 检查点，提升效率。
*   **#21058 [Ops] 优化 Ascend PLE 短对话**: 采用 SD layout 和专用算子，进一步优化了 Ascend PLE 的短对话（short-conv）性能，这是对前期 prefill、decode 及 block-size 调优工作的延续。
*   **#21056 [SpecDecode] 草稿模型支持滑动窗口 KV cache**: 为 EAGLE/EAGLE3/MTP 投机解码的草稿模型引入 `draft_kv_window` 配置，允许使用滑动窗口 KV cache。
*   **#21052 [Perf] 融合 DFlash2 候选边评分**: 同步上游优化，融合 DFlash2 的候选边评分过程以提升性能。

**重要 Bug 修复**
*   **#21057 [BugFix] 修复 AscendRMSNorm 残差路径**: 在 batch-invariant 模式下，将 AscendRMSNorm 的残差路径重新路由至原生 `npu_add_rms_norm` 算子，修复了此前版本中误删条件门控导致的算子调用问题。
*   **#21054 / #21055 [BugFix] 恢复 GLM5Next 设备侧归一化**: 修复了 GLM5Next 继承父类配置导致 vLLM 直接传输原始像素（跳过 CPU 重缩放/归一化）的问题，恢复其在设备侧的归一化处理（#21055 为回传至 `v0.32.0rc` 的补丁）。
*   **#21051 [BugFix] 移植 Kimi K3 MoE 及编译修复**: 将 Kimi K3 的 MoE 运行时调度、依赖声明及 MegaMoE 适配修复回传至 `v0.30.0` 版本。

**开发与调试工具**
*   **#21050 [Debug] 新增编译调试开关**: 引入 `VLLM_DUMP_NGE_FX` 和 `VLLM_ASCEND_DISABLE_MMRS_PASS` 开关，方便开发者进行通信融合（如 Qwen4Exp MM+RS / AG+MM）的编译级调试。
*   **#21049 [DOC] Rex-Omni 模型优化总结**: 更新文档，总结了 `NPUModelRunner._sample()` 覆盖上游采样逻辑等模型优化经验。

### 🚀 Release 动态
*   本次更新周期内，**无新的 Release 发布**。

---

## 🐛 Issues

### #21053 — [[RFC]: standardize KV cache allocation process](https://github.com/vllm-project/vllm-ascend/issues/21053)
- **作者**: Angazenn  **时间**: 2026-11-05 14:31 CST
- **标签**: RFC
- **摘要**: ### Motivation.  MRV2 currently handles KV cache layout across several stages:  ```text platform block-size selection     -> get_kv_cache_spec()     -> Ascend / upstream grouping     -> model-owned or Ascend-specific planning     -> allocate_kv_cache()     -> reshape / backend bind ```  The implemen…

## 🔀 Pull Requests

### #21059 — [[Feature][KV Pool][v0.32.0rc] Support GLM RecoverSSM prefix reuse with PCP](https://github.com/vllm-project/vllm-ascend/pull/21059)
- **作者**: slipfre  **时间**: 2026-11-05 15:01 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Backport #20835 (source head `aa06c1318`) to `releases/v0.32.0rc`, based on `14b1da771`. Enable GLM-5.3-Flash RecoverSSM to reuse aligned prefill checkpoints through AscendStore and make KDA pooling follow the model's actual state partitions under PCP.  - Exp…

### #21058 — [[Ops][Feature] Optimize Ascend PLE short conv with SD layout and spec kernel](https://github.com/vllm-project/vllm-ascend/pull/21058)
- **作者**: like-0517  **时间**: 2026-11-05 15:00 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  This PR is the follow-up to the Ascend PLE short-convolution work already merged for prefill (#20344), decode (#20643) and block-size tuning (#20697). Those changes kept the conv-state cache in the backend (DS) layout and still routed speculative (MTP) decode…

### #21057 — [[BugFix][NPU] Route AscendRMSNorm residual path through native npu_add_rms_norm under batch-invariant mode](https://github.com/vllm-project/vllm-ascend/pull/21057)
- **作者**: SeaAsters  **时间**: 2026-11-05 14:43 CST
- **标签**: module:ops
- **摘要**: ### What this PR does / why we need it?  #16139 made `AscendRMSNorm.forward_oot` unconditionally call the `npu_add_rms_norm_bias` custom op and dropped the `enable_custom_op()` gate together with the native `torch_npu.npu_add_rms_norm` fallback.  That broke batch-invariant mode. With `VLLM_BATCH_INV…

### #21056 — [[Feature][SpecDecode] Support sliding-window KV cache for draft models](https://github.com/vllm-project/vllm-ascend/pull/21056)
- **作者**: diaochengxi  **时间**: 2026-11-05 14:39 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  This PR adds `additional_config["draft_kv_window"]` for Model Runner V2 target-dependent speculative decoding. It bounds supported EAGLE/EAGLE3/MTP draft attention KV caches to the most recent `W` tokens instead of allocating draft KV for `max_model_len`. The…

### #21055 — [[BugFix][Model][v0.32.0rc] Restore GLM5Next device-side normalization](https://github.com/vllm-project/vllm-ascend/pull/21055)
- **作者**: Toneymiller  **时间**: 2026-11-05 14:38 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Apply the GLM5Next input-normalization fix to `releases/v0.32.0rc`.  The model inherits `supports_mm_device_do_normalize=True`, so vLLM skips CPU rescaling/normalization and supplies raw pixels. Its Ascend-specific vision tower only cast those pixels before p…

### #21054 — [[BugFix][Model] Restore GLM5Next device-side normalization](https://github.com/vllm-project/vllm-ascend/pull/21054)
- **作者**: Toneymiller  **时间**: 2026-11-05 14:38 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  GLM5Next inherits `supports_mm_device_do_normalize=True` from its parent. vLLM therefore supplies raw pixels instead of rescaling and normalizing them on the CPU. The Ascend-specific vision tower did not install or call the input normalizer: it only cast the …

### #21052 — [[Perf]Fuse DFlash2 candidate edge scoring](https://github.com/vllm-project/vllm-ascend/pull/21052)
- **作者**: HF-001  **时间**: 2026-11-05 14:15 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it? refer to: https://github.com/vllm-project/vllm/pull/64916  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves …

### #21051 — [[BugFix][Model] Port Kimi K3 MoE and compile fixes to v0.30.0](https://github.com/vllm-project/vllm-ascend/pull/21051)
- **作者**: wt0671  **时间**: 2026-11-05 13:06 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  Kimi K3 needs runtime KDA metadata dispatch, explicit shared/routed Vector-resource dependencies, and a common prefill/decode decision before using MegaMoE across real and idle ranks. This ports the relevant behavior from GitCode PRs 35, 42, 48, 50, 51, 54, 6…

### #21050 — [[Debug][Comp] Add VLLM_DUMP_NGE_FX and VLLM_ASCEND_DISABLE_MMRS_PASS toggles](https://github.com/vllm-project/vllm-ascend/pull/21050)
- **作者**: wxhhuawei  **时间**: 2026-11-05 12:46 CST
- **标签**: module:ops
- **摘要**: ## What this PR does  Adds two opt-in development toggles for compiler-level bring-up and debugging of communication-fusion work (used while building the Qwen4Exp MM+RS / AG+MM fusions; independent of any fusion code):  - `VLLM_DUMP_NGE_FX=1` dumps pre-compile FX graph source to `/tmp/nge_fx_dump/g_…

### #21049 — [[DOC] Rex-Omni Model Optimization Summary](https://github.com/vllm-project/vllm-ascend/pull/21049)
- **作者**: crazy-git-helu  **时间**: 2026-11-05 12:34 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  `NPUModelRunner._sample()` overrides the upstream `GPUModelRunner._sample()`. The override already ported the `update_async_output_token_ids()` call, but it is still missing the equivalent update for speculative decoding:  ```python if self.use_async_scheduli…
