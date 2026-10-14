# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-14 13:16 CST

## AI 总结

# vLLM 仓库动态摘要（2026-10-14）

---

## 🐛 Issue

- **#61561** — 当 `--api-server-count > 1` 时，`--shutdown-timeout` 无法正常排空在途请求。原因是启动器先停止 API 服务器再停止引擎，导致所有在途请求直接报 `EngineDeadError` 失败，引擎的排空逻辑形同虚设。

---

## 🔧 Pull Request

### 新特性 / 模型支持
- **#61566** — 为 GLM-5.3-Flash 的推测解码新增 `--use-replayssm` 选项，让 KDA 层走 Kimi-K3 RecoverSSM 验证/提交路径，每请求仅持有一组 KDA 状态块，降低显存开销。
- **#61565** — 新增 Decision 3.0 决策模型支持（`vllm-sr/d3`、`d3-mini`、`d3-edge`），可直接从 Hub checkpoint 加载服务。
- **#61556** — 更新 Humming 依赖，带来 **Rubin 架构初始支持**、Blackwell 架构完整支持及现有架构的显著优化，并移除了 NVFP4 scale 的临时 workaround。

### Bug 修复
- **#61562** — 修复上述 #61561 问题：调整为**先停引擎再停 API 服务器**，使 `--shutdown-timeout` 在多 API 服务器场景下也能正确排空在途请求。
- **#61563** — 修复 Model Runner V2 启动时未正确处理 EAGLE3 单状态草稿配置的问题（影响 GPT-OSS-120B 的短上下文和高吞吐变体）。
- **#61557** — 推测解码中，保留语法校验失败的 draft token 用于拒绝采样，避免错误丢弃草稿，提升生成正确性。
- **#61558** — 多进程模式下，远程 worker 断连时快速失败传播，并取消阻塞的 socket 读取，改善关闭流程的可靠性。
- **#61559** — ROCm 环境将 AITER 回退锁定到 `v0.1.24.post1`，因为 v0.1.25 修改了 gfx950 MXFP4 融合 MoE 数值计算，确定性地破坏了 DSv4.1 mono decode 数值测试。

### 性能 / 内核优化
- **#61560** — 优化 `swigluoai_and_mul`（GPT-OSS）和 `situ_and_mul`（Kimi）内核的启动配置，根据设备能力动态调整 grid 规模，替代原先固定每 token 一个 block 的低效策略。

### 平台兼容
- **#61564** — 修复同时存在 NVIDIA + Intel GPU 的主机上 XPU wheel 启动失败的问题（两个平台插件同时激活导致冲突），增加平台选择优先级逻辑。

---

## 📦 Release

本次动态中无新版本发布。

---

**总体趋势**：本次更新集中在**推测解码增强**（GLM-5.3-Flash KDA、EAGLE3 修复、语法采样修复）、**多进程/关闭流程健壮性**、**新硬件架构支持**（Rubin/Blackwell）以及**多平台共存**（CUDA + XPU）等方面。

---

## 🐛 Issues

### #61561 — [[Bug]: `--shutdown-timeout` never drains in-flight requests when `--api-server-count > 1`: the launcher stops the API servers before the engines, so every in-flight request fails with EngineDeadError and the engines then drain for nobody](https://github.com/vllm-project/vllm/issues/61561)
- **作者**: kuatovakamila  **时间**: 2026-10-14 12:10 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information... ==============================         System Info ============================== OS                           : Rocky Linux 9.8 (Blue Onyx) (x86…

## 🔀 Pull Requests

### #61566 — [[Spec Decode][GLM-5.3-Flash] Support KDA RecoverSSM (--use-replayssm)](https://github.com/vllm-project/vllm/pull/61566)
- **作者**: JaredforReal  **时间**: 2026-10-14 13:14 CST
- **标签**: kimi, k3, glm
- **摘要**: ## Overview  Let GLM-5.3-Flash's KDA layers use the Kimi-K3 RecoverSSM verify/commit path (`--use-replayssm`) during speculative decoding, so each request holds one KDA state block per group instead of `2 + num_speculative_tokens`. Follows #59147 (which made `--mamba-ssm-cache-dtype` reach the GLM K…

### #61565 — [[Model] Support Decision 3.0 decision models (vllm-sr/d3)](https://github.com/vllm-project/vllm/pull/61565)
- **作者**: Xunzhuo  **时间**: 2026-10-14 13:04 CST
- **标签**: documentation, new-model, frontend, multi-modality, pooling
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Serve the Decision 3.0 decision models ([`vllm-sr/d3`](https://huggingface.co/vllm-sr/d3) and its smaller variants such as `vllm-sr/d3-mini` and `vllm-sr/d3-edge`) from their Hub checkpoints as they are: `vllm serve vllm-sr/d3`, then ask typed questions on…

### #61564 — [[XPU][Platform] Handle nodes with both CUDA and XPU drivers active](https://github.com/vllm-project/vllm/pull/61564)
- **作者**: Sundaresan-G  **时间**: 2026-10-14 12:59 CST
- **标签**: intel-gpu, nvidia
- **摘要**: ## Overview  On a host exposing both NVIDIA and Intel GPUs, an XPU wheel currently fails to start: the CUDA and XPU platform plugins both activate (raising "Only one platform plugin can be activated"), and Triton disables itself on seeing multiple active drivers. This makes an XPU build behave corre…

### #61563 — [[Bugfix][MRV2][Spec Decode] Honor EAGLE3 single-state draft configuration](https://github.com/vllm-project/vllm/pull/61563)
- **作者**: sragrawal  **时间**: 2026-10-14 12:56 CST
- **标签**: bug, speculative-decoding, mrv2
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Fix Model Runner V2 startup for the GPT-OSS-120B EAGLE3 [short-context](https://huggingface.co/nvidia/gpt-oss-120b-Eagle3-short-context) and [throughput](https://huggingface.co/nvidia/gpt-oss-120b-Eagle3-throughput) drafts, which set `use_aux_hidden_state=…

### #61562 — [[Bugfix] Stop engines before API servers so --shutdown-timeout drains with --api-server-count > 1](https://github.com/vllm-project/vllm/pull/61562)
- **作者**: kuatovakamila  **时间**: 2026-10-14 12:15 CST
- **标签**: bug, frontend
- **摘要**: ## Overview  Fixes #61561. With `--api-server-count > 1` (the default whenever `--data-parallel-size > 1`), `--shutdown-timeout N` never drained in-flight requests: the launcher stopped the API servers before the engines, so every in-flight request failed with `EngineDeadError` (HTTP 500) and the en…

### #61560 — [[Kernel] Scale swigluoai_and_mul and situ_and_mul launches with the device](https://github.com/vllm-project/vllm/pull/61560)
- **作者**: XiaobingSuper  **时间**: 2026-10-14 12:03 CST
- **摘要**: ## Overview  `swigluoai_and_mul` (GPT-OSS) and `situ_and_mul` (Kimi) launch `grid(num_tokens)` with one block per token and stride that block over the hidden dim, so one token is one block whatever the hidden size. They have the same shape as the `act_and_mul` family, so they can reuse the launch he…

### #61559 — [[ROCm][Bugfix] Pin AITER back to v0.1.24.post1 for DSv4.1 mono decode numerics](https://github.com/vllm-project/vllm/pull/61559)
- **作者**: Ronald1995  **时间**: 2026-10-14 11:52 CST
- **标签**: bug, rocm, ready, ci/build, DSv4.1
- **摘要**: ## Overview  Pin the ROCm base image back to AITER `v0.1.24.post1`. AITER `v0.1.25` (#61230) changes the gfx950 MXFP4 fused-MoE numerics and deterministically breaks `tests/models/test_dsv41_mono_numerics.py::test_mono_decode_layer_matches_vllm[4]`. Because that step's `source_file_dependencies` do …

### #61558 — [[Bugfix][Multiproc] Fail fast on remote worker disconnects](https://github.com/vllm-project/vllm/pull/61558)
- **作者**: olivierzach  **时间**: 2026-10-14 11:50 CST
- **标签**: bug
- **摘要**: ## Overview  Extend #53597 to propagate remote-worker/coordinator disconnects and cancel blocked socket-backed response reads during shutdown. This fixes failure propagation, not the underlying CUDA fault that originally exposed the hang.  **Dependent draft — do not merge before #53597.** The head i…

### #61557 — [[Bugfix][Spec Decode] Preserve grammar-invalid drafts for rejection sampling](https://github.com/vllm-project/vllm/pull/61557)
- **作者**: yzong-rh  **时间**: 2026-10-14 11:50 CST
- **标签**: bug, structured-output, scheduler
- **摘要**: Fixes https://github.com/vllm-project/vllm/issues/60451  Rework of https://github.com/tomasruizt/vllm/pull/18: - Remove `num_acceptable_drafts` related changes and rejection sampling changes - Remove grammar backend changes, limiting changes to scheduler and structured output manager only  Passes un…

### #61556 — [[Dependency] Update Humming and remove NVFP4 scale workaround](https://github.com/vllm-project/vllm/pull/61556)
- **作者**: jinzhen-lin  **时间**: 2026-10-14 11:22 CST
- **标签**: ci/build, quantization
- **摘要**: This PR updates the Humming version used by vLLM, introducing initial support for the Rubin architecture, full support for the Blackwell architecture, and significant optimizations for existing architectures.  This PR also reverts the changes introduced in https://github.com/vllm-project/vllm/pull/6…
