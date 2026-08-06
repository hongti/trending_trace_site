# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-06 11:53 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库最近动态的中文摘要：

### 📌 Issue 动态
- **架构提案 [RFC]**：提议为 Ascend 设备引入**面向能力的硬件抽象层**。旨在解决当前代码中充斥大量设备特定条件分支的问题，统一平台初始化、Worker选择、模型执行、编译、量化和 MoE 等模块的硬件适配逻辑（#13671）。
- **推理异常 Bug**：在 0.25.0rc1 版本中，使用 DeepSeek-V4-Flash-0731-w8a8 模型进行 PD（Prefill/Decode）分离部署时，中途会出现推理异常（#13674）。
- **并发采样 Bug**：高并发场景下，随机采样路径可能存在**正确性问题**。原因是在 `global_stream()` 上创建指数分布随机张量后，在当前流上消费时存在并发同步隐患（#13673）。

### 🛠️ Pull Request 动态
**1. 核心重构与架构优化**
- **硬件抽象重构**：落实上述 RFC 提案的第一步，集中管理 `AscendDeviceType` 和 SoC 映射，将其提取至轻量级共享模块（#13672）。
- **多连接器优化**：改进 `MultiConnector`，在 `update_state_after_alloc` 阶段为每个子连接器提供请求的真实 blocks，简化内部追踪逻辑（#13668）。

**2. Bug 修复**
- **修复 Mega MoE W8A8**：解决了 Mega MoE 模型在 W8A8 量化下的 Bug（#13675）。
- **修复 SFA OTP**：修复了 SFA（可能是 Flash Attention 相关）的 OTP 问题（#13666）。
- **MooncakeConnector 增强**：为 `MooncakeConnector` 新增 `do_virtual` 参数支持，以优化 Decode 端 KV 接收请求的处理（#13669）。

**3. 文档与 CI/测试**
- **模型支持文档**：更新支持矩阵，新增 `Qwen3.5-397B-A17B` 在 `Ascend 950DT` 上的特性支持说明（包含 BF16、W8A8、Chunked Prefill 等）（#13667）。
- **新增算子文档与测试**：补充算子相关文档及测试用例（#13676）。
- **CI 流水线适配**：适配 A5 场景的 PR 流水线（#13663）；新增 weekly 测试用例（#13662）；修改 235B A5 yaml 配置文件并新增性能文件（#13670）。

### 🚀 Release 动态
- 近期**无新版本发布**。

---
**💡 总结亮点**：近期仓库的核心焦点在于**底层硬件抽象重构**（旨在提升代码可维护性与多设备扩展性）以及**PD分离部署/高并发场景下的Bug修复**，同时持续扩大对业界最新大模型（DeepSeek-V4-Flash, Qwen3.5）及特定硬件（950DT, A5）的适配与文档支持。

---

## 🐛 Issues

### #13674 — [[Bug]: 0.25.0rc1版本镜像+DeepSeek-V4-Flash-0731-w8a8， PD分离部署，中途出现推理异常](https://github.com/vllm-project/vllm-ascend/issues/13674)
- **作者**: 15817778954  **时间**: 2026-08-06 11:09 CST
- **标签**: bug, llm-model, deepseek
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text P实例报错： (Worker_DP0_TP2_EP2 pid=36191) ERROR 08-05 05:55:40 [multiproc_executor.py:945] Error getting async model runner output^M (Worker_DP0_TP2_EP2 pid=36191) ERROR 08-05 05:55:40 [multiproc_ex…

### #13673 — [[Bug]: 随机采样在高并发下可能存在正确性问题](https://github.com/vllm-project/vllm-ascend/issues/13673)
- **作者**: mayunaise  **时间**: 2026-08-06 10:40 CST
- **标签**: bug
- **摘要**: ### Your current environment  ```text CANN 9.0.0 torch 2.9.0 torch_npu 2.9.0.post4 vllm 0.18.0 vllm-ascend 0.18.0 ```  ### 🐛 问题描述  当前普通随机采样路径在 `global_stream()` 上创建并填充指数分布随机张量 `q`，随后在当前流上消费该张量：  ```python with npu_stream_switch(global_stream()):     q = torch.empty_like(probs)     q.exponential_()  …

### #13671 — [[RFC]: Introduce a Capability-Oriented Hardware Abstraction Layer for Ascend Devices](https://github.com/vllm-project/vllm-ascend/issues/13671)
- **作者**: Tflowers-0129  **时间**: 2026-08-06 10:30 CST
- **标签**: RFC
- **摘要**: ### Motivation.  vLLM Ascend currently contains a growing number of device-specific conditionals across platform initialization, worker selection, model execution, compilation, quantization, and MoE-related code paths. Common examples include: ``` if is_310p():     ...  if is_950():     ...  if soc_…

## 🔀 Pull Requests

### #13676 — [add ops md and tests](https://github.com/vllm-project/vllm-ascend/pull/13676)
- **作者**: shiyuan680  **时间**: 2026-08-06 11:23 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/0351e9aa1fdf1a51329d1906881528dfe61fc88e

### #13675 — [[Bugfix] fix bug for mega moe w8a8](https://github.com/vllm-project/vllm-ascend/pull/13675)
- **作者**: nanshen3000  **时间**: 2026-08-06 11:10 CST
- **标签**: module:ops, module:quantization
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/0351e9aa1fdf1a51329d1906881528dfe61fc88e

### #13672 — [[Refactor][device] Refactor Ascend hardware configuration and discovery](https://github.com/vllm-project/vllm-ascend/pull/13672)
- **作者**: Tflowers-0129  **时间**: 2026-08-06 10:33 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  This PR introduces the first step of the Ascend hardware abstraction refactoring:  - Centralizes `AscendDeviceType` and SoC mappings in a lightweight module shared by runtime and build-time code. - Adds a lazily initialized `DeviceConfig` as the unified devic…

### #13670 — [修改 235B A5yaml文件，新增性能文件,在nightly和weekly中增加用例信息](https://github.com/vllm-project/vllm-ascend/pull/13670)
- **作者**: luxixiang2026  **时间**: 2026-08-06 10:27 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/0351e9aa1fdf1a51329d1906881528dfe61fc88e

### #13669 — [[BugFix] Add do_virtual parameter to MooncakeConnector](https://github.com/vllm-project/vllm-ascend/pull/13669)
- **作者**: xyy59516  **时间**: 2026-08-06 10:22 CST
- **摘要**: ### What this PR does / why we need it?  This PR adds support for the optional `do_virtual` flag in `MooncakeConnector`.  When `do_virtual` is enabled for a decode-side KV receive request, this change:  - propagates `do_virtual` from `kv_transfer_params` to `ReqMeta`; - marks the request as complete…

### #13668 — [[Feature] MultiConnector: give every sub-connector the request's real blocks in update_state_after_alloc](https://github.com/vllm-project/vllm-ascend/pull/13668)
- **作者**: HF-001  **时间**: 2026-08-06 10:17 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? refer to: https://github.com/vllm-project/vllm/pull/46865 , MultiConnector does not need to know whether each connector needs to perform store tracking, but rather ensures that: -Blocks are always real; -Whether to load is determined by the num_external_tokens…

### #13667 — [[Doc][Misc] Update Qwen3.5-397B-A17B support features in Ascend 950DT\n](https://github.com/vllm-project/vllm-ascend/pull/13667)
- **作者**: Karryking3  **时间**: 2026-08-06 10:10 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? This PR updates the support matrix for the `Qwen3.5-397B-A17B` model on `Ascend 950DT` hardware, detailing the supported features such as BF16, W8A8, Chunked Prefill, Automatic Prefix Cache, Speculative Decoding, Async Scheduling, Tensor Parallel, Expert Paral…

### #13666 — [[BugFix] Fix sfa otp](https://github.com/vllm-project/vllm-ascend/pull/13666)
- **作者**: lijiahang226  **时间**: 2026-08-06 10:04 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/0351e9aa1fdf1a51329d1906881528dfe61fc88e

### #13663 — [[CI]Adapt vLLM-Ascend PR pipeline for A5 scenarios (copy for pypi-cache verification)](https://github.com/vllm-project/vllm-ascend/pull/13663)
- **作者**: KadenZhang3321  **时间**: 2026-08-06 09:51 CST
- **标签**: ci/build
- **摘要**: - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/0351e9aa1fdf1a51329d1906881528dfe61fc88e

### #13662 — [[CI] add weekly test example](https://github.com/vllm-project/vllm-ascend/pull/13662)
- **作者**: xqchen7  **时间**: 2026-08-06 09:42 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/0351e9aa1fdf1a51329d1906881528dfe61fc88e
