# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-02 14:03 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期的动态摘要。

*(注：本次提供的数据仅包含 Pull Request 动态，未包含 Issue 和 Release 信息。)*

### 🛠️ Pull Request 动态
近期 PR 主要集中在**性能优化、Bug 修复和硬件/模型兼容性增强**：

**1. Bug 修复**
* **基准测试修复 (#59736)**：修复了 Rust `vllm-bench` 的 `openai-chat` 后端在未生成任何 token 但流正常结束时，错误标记为 `success` 的 Bug。
* **Kimi-K3 性能衰退修复 (#59733)**：修复了 Kimi-K3 配合 DSpark 使用时，因未声明 `max_tp_shards` 导致 MLA 层无法与目标合并而引发的性能衰退问题。

**2. 性能优化与内核调优**
* **算子融合与内核调优**：优化了 MoE 加权求和内核的启动配置以提升性能 (#59731)；针对 ROCm 平台，将 Qwen4Exp 的 QK-norm/RoPE/gate 和 KV-cache 写入操作融合进 AMD QSA prepare 启动中，与 NVIDIA 端实现对齐 (#59732)。
* **并发与流处理**：为 Nemotron-H 引入侧流并发机制，使 MoE 路由门控与 `fc1_latent` 投影能并发执行 (#59728)；为 Mamba SSU 后端引入侧流预取随机舍入种子机制 (#59729)。
* **分布式通信**：新增可选环境变量 `VLLM_CUSTOM_ALL_GATHER_SMALL=1`，将小型 TP all-gather 路由至自定义 all-reduce 通信器以优化效率 (#59730)。
* **解码路径优化**：在未配置投机解码时，将常规 GDN 解码保留在标准路径上，避免不必要的内核改动 (#59735)。

**3. 投机解码增强**
* **Nemotron-H MTP 优化 (#59727)**：为 Nemotron-H MTP 起草器支持了 `get_top_tokens()` 及局部 argmax 归约，并优化了 CUDA 上的词表并行 argmax，显著降低了计算开销。

**4. 文档更新**
* **硬件行为说明 (#59734)**：新增文档说明了在 Hopper NVSwitch 节点（H100、H20、H800）上，若驱动/Fabric Manager 版本过旧，NVLS all-reduce 可能会导致运行间结果非确定性的问题。

---
*注：由于本次输入数据未包含 Issue 和 Release 记录，故省略相关归纳。如后续提供，将重点提取版本发布亮点。*

---

## 🔀 Pull Requests

### #59736 — [[Bugfix][Rust Frontend] Fail token-less streams in vllm-bench openai-chat](https://github.com/vllm-project/vllm/pull/59736)
- **作者**: ZhenchengLin  **时间**: 2026-10-02 13:38 CST
- **标签**: bug, rust
- **摘要**: ## Purpose  Fixes #59722.  The Rust `vllm-bench` `openai-chat` backend set `output.success = true` whenever the stream ended cleanly, even if no token ever arrived. The `openai` (completions) backend and Python `vllm bench serve` only count a request as successful once a first token is received.  A …

### #59735 — [[Perf] Keep non-speculative GDN decode on the standard path](https://github.com/vllm-project/vllm/pull/59735)
- **作者**: DimensionSTP  **时间**: 2026-10-02 13:33 CST
- **摘要**: ## Overview  Keep ordinary GDN execution on the existing standard path when speculative decoding is not configured. Addresses #59520 without changing the speculative CUDA kernel or the backend default.  ## Claims  - Avoid the packed fused-normalization wrapper when the model cannot produce an MTP ba…

### #59734 — [[Doc] Document NVLS non-determinism on Hopper NVSwitch nodes](https://github.com/vllm-project/vllm/pull/59734)
- **作者**: www6v  **时间**: 2026-10-02 13:28 CST
- **标签**: documentation
- **摘要**: ## Summary  Add documentation about NVLS all-reduce being a source of run-to-run non-determinism on Hopper NVSwitch nodes (H100, H20, H800) when the kernel driver / Fabric Manager is older than 550.144.03.  Document that `NCCL_NVLS_ENABLE=0` alone fixes single-request reproducibility with ~1% latenc…

### #59733 — [[Bugfix][Kimi-K3] Declare max_tp_shards on the DSpark MLA KV cache spec](https://github.com/vllm-project/vllm/pull/59733)
- **作者**: hyukjlee  **时间**: 2026-10-02 12:56 CST
- **标签**: bug, dflash, kimi, k3
- **摘要**: ## Overview  Fixes a Kimi-K3 + DSpark perf regression from #57652: the K3 DSpark draft's `MLAAttentionSpec` does not declare `max_tp_shards`, so its MLA layers stop merging with the target's and the KV cache splits from **4 to 20 groups**. One-line fix: declare `max_tp_shards=1` on the draft spec, m…

### #59732 — [[ROCm][Perf] Fuse main QK-norm/RoPE/gate and KV-cache write into the AMD QSA prepare launch](https://github.com/vllm-project/vllm/pull/59732)
- **作者**: mjkvaak-amd  **时间**: 2026-10-02 12:41 CST
- **标签**: rocm, ci/build, qwen
- **摘要**: ## Overview  🚨 Depends on #57947 🚨  On ROCm, the Qwen4Exp QSA layer's main-attention QK-norm, RoPE, gate split and K/V cache write now run inside the fused QSA prepare launch, as #57097 did for NVIDIA.  ## Claims  - MI355X TP2 `amd/Qwen3.8-Flash-Next-Quark-MXFP4`: +0.9% to +2.9% output throughput, f…

### #59731 — [[Perf] Tune MoE weighted-sum kernel launch configuration](https://github.com/vllm-project/vllm/pull/59731)
- **作者**: jinzhen-lin  **时间**: 2026-10-02 12:15 CST
- **摘要**: ## Overview  Tune CUDA FP16/BF16 `moe_fused_mul_sum` launches: cap `BLOCK_K` at 4096 and use 8 warps on SM75, 16 on newer GPUs. FP32, small hidden sizes, and non-CUDA platforms keep their existing settings.  ## Claims  - Reduce serial hidden-tile iterations with a small launch-only change; retain th…

### #59730 — [[Distributed] One-shot IPC all-gather for small TP all-gathers](https://github.com/vllm-project/vllm/pull/59730)
- **作者**: pst2154  **时间**: 2026-10-02 11:13 CST
- **标签**: ci/build, nvidia
- **摘要**: ## Overview  Opt-in `VLLM_CUSTOM_ALL_GATHER_SMALL=1` (default off): route small TP all-gathers (default <= 64 KiB per rank, `VLLM_CUSTOM_ALL_GATHER_SMALL_MAX_BYTES`) through the custom all-reduce communicator's existing one-shot CUDA-IPC all-gather instead of an NCCL ring kernel. A plain copy, so bi…

### #59729 — [[Mamba] Prefetch stochastic-rounding seeds on a side stream](https://github.com/vllm-project/vllm/pull/59729)
- **作者**: pst2154  **时间**: 2026-10-02 11:12 CST
- **摘要**: ## Overview  Opt-in `VLLM_MAMBA_SR_SEED_PREFETCH=1` (default off): with the FlashInfer Mamba SSU backend and `--enable-mamba-cache-stochastic-rounding`, draw a forward pass's per-layer stochastic-rounding seeds up front on a side CUDA stream; each SSU call waits on an event for its seed. Seed values…

### #59728 — [[Model] NemotronH: overlap MoE router gate with fc1_latent on a side stream](https://github.com/vllm-project/vllm/pull/59728)
- **作者**: pst2154  **时间**: 2026-10-02 11:09 CST
- **摘要**: ## Overview  Opt-in `VLLM_NEMOTRON_H_MOE_ROUTER_OVERLAP=1` (default off): in Nemotron-H latent-MoE layers, run the router gate GEMM on a side CUDA stream concurrently with `fc1_latent_proj`, joined by a CUDA event before the MoE op reads the router logits. Same kernels on the same inputs, so outputs…

### #59727 — [[Spec Decode] Nemotron-H MTP get_top_tokens() and a fused vocab-parallel argmax](https://github.com/vllm-project/vllm/pull/59727)
- **作者**: pst2154  **时间**: 2026-10-02 11:07 CST
- **标签**: ci/build
- **摘要**: ## Overview  Make `speculative_config.use_local_argmax_reduction` usable for the Nemotron-H MTP drafter (add `get_top_tokens()`), and make `LogitsProcessor.get_top_tokens()` itself cheaper on CUDA at TP > 1: one Triton pass over the logits shard, a 64-byte-per-row candidate all-gather and a small pi…
