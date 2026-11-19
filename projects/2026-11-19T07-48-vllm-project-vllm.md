# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-19 15:48 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要：

### 📋 Issue
*   **#68616 [RFC] 混合 RL rollouts 的独立采样掩码选择退出机制**
    *   作者: Ronald1995
    *   摘要: 提出在一个 RL rollout 引擎服务不同采样策略的训练源时，支持独立的采样掩码配置。截断的 rollouts 需要精确的采样支持以进行重放，而非截断的 rollouts 也需要相应的处理策略。
*   **#68610 [Bug] TP>1 时 `vllm:num_requests_waiting` 指标统计异常**
    *   作者: srkim-business
    *   摘要: 指出在张量并行（TP>1）环境下，自 #65466 合并后，`num_requests_waiting` 指标会短暂包含调度器尚未真正处理的请求，导致监控数据不准确。

### 🔧 Pull Request
*   **#68615 [Tracing] 追踪功能增强**
    *   新增缓存读取 token 数、完成原因以及标准的 OTel GenAI 追踪属性。
*   **#68614 [Core][PP] 流水线并行调度优化**
    *   针对异步调度的流水线并行（PP > 1），允许越过空闲的空步骤提前接收新请求，并根据积压请求调整预填充微批大小，提升吞吐。
*   **#68613 [Kernel][MoE] 支持 Kimi-K3.1 SiTU 激活函数**
    *   修复了在 Hopper 架构（SM90）上使用 DeepGEMM Mega MoE 后端时，无法处理 Kimi-K3.1 SiTU 激活函数的 `NotImplementedError` 问题。
*   **#68612 [Rust Frontend] 支持 /generative_scoring 路由**
    *   Rust 前端不再对 `/generative_scoring` 返回 501 错误，而是复用 Python 路由语义，将每个 item 转换为 `max_tokens=1` 的请求来读取标签 token。
*   **#68611 [Spec Decode] 拒绝 token 特征提取**
    *   为 MRv2 新增 `gather_rejected_tokens()` 方法，用于在验证步骤中提取被拒绝的草稿后缀特征（包含隐藏状态、目标位置和请求状态等）。
*   **#68609 [Model] HY4-Flash 模型支持 NIXL P/D 分离**
    *   为 HY4-Flash 模型启用 NIXL 拉取模式的 Prefill/Decode 分离部署。
*   **#68608 [Refactor] 权重传输重构**
    *   为打包 IPC 添加后端扩展点，将原本与 CUDA 绑定的缓冲区分配、句柄查找等解耦，使平台插件能够复用传输和导入器生命周期。
*   **#68607 [Bugfix] 修复 AutoRound 检查点加载报错**
    *   在加载 AutoRound FP8-attention 检查点时，忽略 q/k/v 的 max/min 统计信息，修复了 `ValueError` 加载失败的问题。
*   **#68606 [Feature] EPLB 异步策略子进程**
    *   新增实验性功能：启用可选的持久化 CPU 子进程来执行异步 EPLB（专家负载均衡）策略计算，使其脱离推理进程的 Python GIL 限制。
*   **#68605 [Bugfix] 修复预填充中止时的 logprobs 报错**
    *   修复了带有 `echo` 和 `logprobs` 的 `/v1/completions` 请求在预填充阶段被中止时，引发 HTTP 500（`IndexError`）的问题，现通过 padding 修复。

### 🚀 Release
*   本周期内无新的 Release 版本发布。

---

## 🐛 Issues

### #68616 — [[RFC]: Independent sampling-mask opt-out for mixed RL rollouts](https://github.com/vllm-project/vllm/issues/68616)
- **作者**: Ronald1995  **时间**: 2026-11-19 15:45 CST
- **标签**: RFC
- **摘要**: ### Motivation  A single RL rollout engine can serve training sources with different sampling policies. Truncated rollouts need the exact sampling support for replay. Untruncated rollouts still need sampled-token logprobs for the training loss, and an MoE training run may also need routed experts fo…

### #68610 — [[Bug]: With TP>1, `vllm:num_requests_waiting` briefly includes requests the scheduler hasn't seen yet (since #65466)](https://github.com/vllm-project/vllm/issues/68610)
- **作者**: srkim-business  **时间**: 2026-11-19 15:22 CST
- **标签**: bug, scheduler
- **摘要**: ### Your current environment  Not the full `collect_env.py` output: the behavior follows from the engine-core code path described below and does not depend on the hardware. We can add it if needed.  - vLLM v0.34.0, official `vllm/vllm-openai:v0.34.0` image (amd64) - 4x NVIDIA A100, `--tensor-paralle…

## 🔀 Pull Requests

### #68615 — [[Tracing] Add cache read tokens, finish reasons, and new standard OTel GenAI attributes](https://github.com/vllm-project/vllm/pull/68615)
- **作者**: sugix  **时间**: 2026-11-19 15:45 CST
- **摘要**: This picks up vllm-project/vllm#40799 by @Csrayz, which Csrayz closed and asked me to open fresh. It's their two commits, unchanged and credited, plus two from me, all on current main.  ## What it does  The request span gets the current OpenTelemetry GenAI attribute names:  - `gen_ai.usage.input_tok…

### #68614 — [[Core][PP] Admit new requests past idle empty steps, and size prefill microbatches from the backlog](https://github.com/vllm-project/vllm/pull/68614)
- **作者**: njhill  **时间**: 2026-11-19 15:39 CST
- **标签**: scheduler
- **摘要**: ## Overview  Two scheduling changes for pipeline parallelism with async scheduling, both active only with PP > 1 (admission also requires DP == 1). The original optimization and first implementation are by @gau-nernst (the first two commits). The later commits rework them onto the existing wait-for-…

### #68613 — [[Kernel][MoE] Run Kimi-K3.1 SiTU routed experts in SM90 DeepGEMM Mega MoE](https://github.com/vllm-project/vllm/pull/68613)
- **作者**: new-TonyWang  **时间**: 2026-11-19 15:36 CST
- **标签**: kimi
- **摘要**: ## Overview  `MEGA_MOE_MXFP4_SM90` advertised only SwiGLU, so `--moe-backend deep_gemm_mega_moe` on Hopper rejected Kimi K3.1's SiTU activation while constructing the experts:  ``` NotImplementedError: DeepGEMM MegaMoE mxfp4 experts support ['swiglu'], not 'situ'. ```  SM100's `MEGA_MOE_MXFP4` alrea…

### #68612 — [[Rust Frontend] Serve /generative_scoring](https://github.com/vllm-project/vllm/pull/68612)
- **作者**: BugenZhao  **时间**: 2026-11-19 15:31 CST
- **标签**: ci/build, rust
- **摘要**: ## Overview  The Rust frontend answers `POST /generative_scoring` with a 501. This PR serves it with the Python route's semantics: each item becomes one `max_tokens=1` request that reads the label tokens' logprobs. Part of #44280.  Supersedes #46427 by @jesco-absolut, credited as co-author: same rou…

### #68611 — [[MRV2][Spec Decode] Rejected-token feature extraction](https://github.com/vllm-project/vllm/pull/68611)
- **作者**: seongyun1104  **时间**: 2026-11-19 15:22 CST
- **标签**: speculative-decoding, mrv2
- **摘要**: ## Overview  MRv2: add `gather_rejected_tokens()`, which returns the rejected draft suffix of each request in a verify step: its raw target aux hidden states, original target positions, and request state index. This is the extraction step that rejected-token-memory drafters such as DNeo need. Normal…

### #68609 — [[Model] Enable NIXL P/D disaggregation for HY4-Flash](https://github.com/vllm-project/vllm/pull/68609)
- **作者**: grYe99  **时间**: 2026-11-19 15:20 CST
- **标签**: documentation, kv-connector, nixl
- **摘要**: ## Overview  Enable NIXL pull-mode P/D disaggregation for HY4-Flash, following #68296.  ## Claims  - Allow the built-in NIXL pull connector (`NixlConnector` / `NixlPullConnector`) in HY4-Flash. - Reuse existing hybrid KV cache and NIXL infrastructure without changes to shared connector or scheduler …

### #68608 — [[Refactor][Weight Transfer] Add backend extension points for packed IPC](https://github.com/vllm-project/vllm/pull/68608)
- **作者**: Liccol  **时间**: 2026-11-19 15:19 CST
- **标签**: documentation, needs-rebase
- **摘要**: ## Overview  Packed IPC binds buffer allocation, handle lookup, and mapping rebuild to CUDA. Expose keyword-only extension points so platform plugins can reuse the packed transfer and importer lifecycle with their own device and IPC handles.  ## Claims  - Trainers can set the packed buffer device ex…

### #68607 — [[Bugfix][Quantization] Ignore AutoRound q/k/v max/min stats when loading checkpoints](https://github.com/vllm-project/vllm/pull/68607)
- **作者**: baodii  **时间**: 2026-11-19 15:10 CST
- **标签**: bug, quantization
- **摘要**: ## Overview  AutoRound FP8-attention checkpoints (e.g. `INCModel2/Qwen3-30B-A3B-Instruct-2507-MXFP8-FP8ATTN-CT-AutoRound`, `INCModel2/Qwen3-32B-MXFP8-FP8ATTN-CT-AutoRound`) fail to load with `ValueError: There is no module or parameter named 'layers.0.self_attn.q_max'`. This PR adds the AutoRound q/…

### #68606 — [[Feature][EPLB] Add an opt-in persistent policy subprocess](https://github.com/vllm-project/vllm/pull/68606)
- **作者**: freyfwt  **时间**: 2026-11-19 15:02 CST
- **标签**: documentation, performance
- **摘要**: ## Overview  Add an experimental, opt-in persistent CPU subprocess for async EPLB policy computation, so the planner can run outside the inference worker's Python GIL. Enable it with `--eplb-config '{"use_async":true,"use_policy_process":true}'`; the default remains `false`.  This is a feasibility d…

### #68605 — [[Bugfix] Pad prompt logprobs of completions aborted mid-prefill](https://github.com/vllm-project/vllm/pull/68605)
- **作者**: ankitgala007  **时间**: 2026-11-19 14:44 CST
- **标签**: bug, frontend
- **摘要**: ## Overview  Fixes #68601. A `/v1/completions` request with `echo` and `logprobs` that is aborted before its prompt is fully processed returned HTTP 500 (`IndexError: list index out of range`). This pads the missing prompt positions with `None` so the request returns normally with `finish_reason="ab…
