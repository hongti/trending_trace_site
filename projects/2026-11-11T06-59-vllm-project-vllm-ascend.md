# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-11 14:59 CST

## AI 总结

# vllm-ascend 仓库近期动态摘要

---

## 🚀 Release

### v0.32.0rc1（2026-11-10）
vLLM Ascend **v0.32.0rc1** 发布，对齐上游 vLLM v0.32.0，为该版本的首个候选发布版本（RC1）。

---

## 🔧 Pull Request 摘要

共 **10 个 PR**，集中在**性能优化、新特性、Bug 修复与 CI 维护**：

### 新特性 / 功能增强
- **#21549** — 为 Kimi K3/K3.1 推测解码新增实验性的 **Ascend KDA RecoverSSM** 路径（opt-in），保持递归 checkpoint 不变。
- **#21545** — KV Pool 支持 **Step-5 Memcache 池化**与逐层 eager 传输（已被精简版替代）。
- **#21551** — 输入准备阶段集成 **PCP-MROPE 位置编码**支持。

### 性能优化
- **#21550** — 支持 **SwiGLUStep MXFP8 量化**（fused），使用 `npu_swiglu_mx_quant` mode 3，保留 SiLU 后 clamp 限制 7.0。
- **#21543** — 新增 **SwiGLU 图融合 pass**，覆盖直接使用 aten ops 而非 `AscendSiluAndMul` 的模型路径（如 DeepseekV4VisionMLP）。
- **#21542** — 为 Kimi K3.1 启用 **DSpark FlashMLA 异步解码**，避免不必要的 worker 主线程序列长度同步。
- **#21544** — GLM5Next 新增 `VLLM_ASCEND_GLM_REPLICATED_EMBED` 开关，保持输入 embedding 表不分片，跳过 mask/all-reduce。

### Bug 修复
- **#21548** — Mooncake PD 握手端口改为**动态分配**，绑定空闲端口并发布到 worker 握手元数据中（修复 #10653）。
- **#21547** — 当 `cpu_offload=False` 且 DP>1 时，对 `dp_shared_memory` 自动启用增加**保护检查**，避免错误覆盖配置。

### CI 维护
- **#21546** — 更新 CANN 版本至 **9.3.0_20261110170400**。

---

## 📋 Issue
本轮数据中**无独立 Issue 活动**。

---

**总体趋势**：本期活动围绕 **Kimi K3.1 推测解码生态**（KDA RecoverSSM、FlashMLA 异步解码）和**量化/融合性能优化**（SwiGLUStep MXFP8、SwiGLU 图融合）两条主线展开，同时修复了 Mooncake PD 端口分配和 DP 共享内存配置的稳定性问题。v0.32.0rc1 标志着下一个正式版本的临近。

---

## 🔀 Pull Requests

### #21551 — [Integrate PCP-MROPE position preparation in inputs](https://github.com/vllm-project/vllm-ascend/pull/21551)
- **作者**: baolongsun  **时间**: 2026-11-11 14:56 CST
- **标签**: module:ops
- **摘要**: Add support for PCP-MROPE positions in input preparation.  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves…

### #21550 — [[Performance][Model] Support fused SwiGLUStep MXFP8 quantization](https://github.com/vllm-project/vllm-ascend/pull/21550)
- **作者**: yihua-fan  **时间**: 2026-11-11 14:55 CST
- **标签**: module:tests, module:quantization
- **摘要**: Use npu_swiglu_mx_quant mode 3 for SWIGLUSTEP with per-expert token counts. Preserve the post-SiLU clamp limit of 7.0 and the existing SwiGLU-OAI implementation.  Add unit coverage for activation dispatch, both scale algorithms, and count/cumulative group lists. Syntax and diff checks pass; format.s…

### #21549 — [[Feature] Add experimental Kimi KDA RecoverSSM support](https://github.com/vllm-project/vllm-ascend/pull/21549)
- **作者**: MAyang38  **时间**: 2026-11-11 14:51 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Add an experimental, opt-in Ascend KDA RecoverSSM path for Kimi K3 and K3.1 speculative decoding. Verification keeps the recurrent checkpoint unchanged and writes request-indexed records; after sampling, the accepted state and convolution history are committe…

### #21548 — [[BugFix] Allocate Mooncake PD handshake ports dynamically](https://github.com/vllm-project/vllm-ascend/pull/21548)
- **作者**: HBnothave  **时间**: 2026-11-11 14:38 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Fixes #10653. Mooncake PD workers now bind the ZMQ handshake listener to a free port and publish that endpoint in worker handshake metadata. The scheduler retains stable logical rank ports for shard routing and completion counts, while decode workers connect …

### #21547 — [[Fix] Guard dp_shared_memory auto-enable with cpu_offload check](https://github.com/vllm-project/vllm-ascend/pull/21547)
- **作者**: yeatschou0-ui  **时间**: 2026-11-11 14:32 CST
- **标签**: module:tests, module:core
- **摘要**: ## What this PR does / why we need it?  When `cpu_offload=False` (e.g. ETP path with `embedding_across_dp=True`) and DP>1 on Model Runner V2, `AscendConfig.derive_and_validate` auto-set `dp_shared_memory=True`. This conflicts with the upstream constraint `_validate_shared_memory` which requires `dp_…

### #21546 — [[CI] update cann=9.3.0_20261110170400](https://github.com/vllm-project/vllm-ascend/pull/21546)
- **作者**: xqchen7  **时间**: 2026-11-11 14:27 CST
- **摘要**: ### What this PR does / why we need it? update cann=9.3.0_20261110170400  ### Does this PR introduce _any_ user-facing change? NA  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions …

### #21545 — [[Feature][KV Pool] Support Step-5 Memcache pooling and eager layerwise transfers](https://github.com/vllm-project/vllm-ascend/pull/21545)
- **作者**: zouyida2052  **时间**: 2026-11-11 14:24 CST
- **标签**: documentation, module:tests, module:quantization
- **摘要**: Superseded by the focused stacked draft: https://github.com/zouyida2052/vllm-ascend/pull/2  The replacement compares directly against #20946 and contains only pooling changes and tests. This broad draft included the model prerequisite and is closed to avoid duplicate review.  ---  ### What this PR d…

### #21544 — [perf(glm5next): replicate the input embedding to skip its mask and al…](https://github.com/vllm-project/vllm-ascend/pull/21544)
- **作者**: mengguangxin1818  **时间**: 2026-11-11 14:12 CST
- **摘要**: Add a VLLM_ASCEND_GLM_REPLICATED_EMBED switch that keeps the GLM input embedding table unsharded.  The input embedding currently always uses VocabParallelEmbedding, which shards the table over TP and so does three things on every lookup: - gather the N requested rows: 104 us at chunk 15232, and inde…

### #21543 — [[Performance][Ops] Add SwiGLU graph fusion pass](https://github.com/vllm-project/vllm-ascend/pull/21543)
- **作者**: Cold-E  **时间**: 2026-11-11 14:09 CST
- **标签**: module:tests, module:core
- **摘要**: ## Why  Some model paths write the SwiGLU activation with raw aten ops instead of going through `AscendSiluAndMul` / `npu_swiglu`. For example `DeepseekV4VisionMLP.forward`:  ```python gate, up = self.w1(x).chunk(2, dim=-1) return self.w2(F.silu(gate) * up) ```  This keeps the `chunk -> silu -> mul`…

### #21542 — [[Performance][SpecDecode] Enable Kimi K3.1 DSpark FlashMLA async decode](https://github.com/vllm-project/vllm-ascend/pull/21542)
- **作者**: hust17yixuan  **时间**: 2026-11-11 14:08 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Enable the existing A5 FlashMLA draft path for Kimi K3.1 with a `K3DSparkModel` draft, and avoid an unnecessary worker-main-thread sequence-length synchronization between the draft and the next target step.  - Remove the unconditional `kimi_k31` FlashMLA excl…

## 🚀 Releases

### [v0.32.0rc1](https://github.com/vllm-project/vllm-ascend/releases/tag/v0.32.0rc1)
- **作者**: linfeng-yuan  **时间**: 2026-11-11 00:45 CST
- **摘要**: ## v0.32.0rc1 - 2026.11.10  This is the first release candidate of vLLM Ascend v0.32.0, aligned with upstream vLLM v0.32.0. This release introduces support for **MiniMax-M3.1-Flash** and **Qwen3.8-Flash-Next**, and adds optimizations for **DeepSeek V4.1-Flash** and **GLM-5.3-Flash**. It also improve…
