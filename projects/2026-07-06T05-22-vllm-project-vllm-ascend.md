# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-06 13:22 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📌 Issue 动态
* **#11468 [Bug] Qwen3.6-27B 多模态大请求卡死问题**
  在 910B3 双机 PD 分离部署环境下，使用 Qwen3.6-27B 模型时，当多模态请求的数据量过大，会导致服务进程卡死（Hang）。此问题目前已被记录，并可能有相关 PR 正在尝试修复。

### 🔧 PR 动态（按类型归纳）

**1. 重要特性与性能优化**
* **#11461 [MRV2] 投机解码图捕获合并**：针对 Eagle 投机解码机制，将原先分散的 N-1 个 draft 步骤合并为**单次 ACL 图捕获**，大幅减少图捕获开销并解决相关问题。
* **#11469 [DSv4][A3] KV Cache 内存分层扩展**：为 A3 上的 DSv4 引入静态 Host-backed `C128` KV 分层路径，将冷数据移出 HBM，有效扩充逻辑 KV 地址空间，缓解显存压力。
* **#11467 [Feature] 新增 Triton SwigluStep 内核**：将 `silu+clamp+mul` 操作融合为单一的 1D-grid row-loop Triton kernel，最大限度降低 Host launch 开销，提升激活函数计算效率。

**2. Bug 修复**
* **#11470 [WIP] Qwen3.x 模型修复**：修复 Qwen3.x 模型部分层存在多个 `layer_indices` 导致的异常（此修复预计直接解决上述 Issue #11468 的卡死问题）。
* **#11466 [BugFix] 修复 Block Table 溢出**：解决在启用投机解码（MTP/EAGLE）并搭配前缀缓存时，Mamba KV cache 组可能发生的 block table 溢出崩溃问题。

**3. CI 优化与文档维护**
* **#11465 / #11464 / #11460 / #11462 (CI 资源与流程优化)**：
  - 因公共 Runner 资源紧张，将 CI 迁移至**自托管 Runner** (#11465)；
  - 增大 Qwen3-30B-QuaRot 夜间测试的 `HCCL_BUFFSIZE` 至 768 以保障稳定 (#11464)；
  - 新增失败分析 Job 及交叉引用功能 (#11460)；
  - 精简 `test_uva.py` 冗余模型测试，加速 E2E 流程 (#11462)。
* **#11463 [Doc] 规范化 MiniMax-M2 文档**：统一 MiniMax-M2 模型文档中的 Docker 安装章节格式，与最新模板对齐。

### 🚀 Release 动态
* **近期无新版本发布记录**。

---

## 🐛 Issues

### #11468 — [[Bug]: Qwen3.6-27B多模态场景，多模态请求过大时会卡死](https://github.com/vllm-project/vllm-ascend/issues/11468)
- **作者**: s-jiayang  **时间**: 2026-07-06 11:57 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Your output of above commands here ```  </details>   ### 🐛 Describe the bug  环境信息：910B3，双机PD分离 模型：Qwen3.6-27B 启动参数： #!/bin/bash export VLLM_LOGGING_LEVEL=DEBUG  export LD_LIBRARY_PATH=/usr/local…

## 🔀 Pull Requests

### #11470 — [[WIP] bugfix for qwen3.x model, because there are multiple layers in some l…](https://github.com/vllm-project/vllm-ascend/pull/11470)
- **作者**: leijie-ww  **时间**: 2026-07-06 12:15 CST
- **摘要**: …ayer_indices  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389

### #11469 — [[DSv4][A3] Static host-backed C128 KV tiering](https://github.com/vllm-project/vllm-ascend/pull/11469)
- **作者**: foraxe  **时间**: 2026-07-06 12:09 CST
- **标签**: module:core
- **摘要**: # [DSv4][A3] Static host-backed C128 KV tiering  ## Summary - add a static host-backed `C128` KV tiering path for DSv4 on A3 - grow logical KV address space by moving the cold `C128` family off HBM into host-backed memory - harden the feature by using spec-based routing instead of layer-id parity an…

### #11467 — [[Feature] Add triton swiglustep kernel for SwigluStep activation](https://github.com/vllm-project/vllm-ascend/pull/11467)
- **作者**: boes129  **时间**: 2026-07-06 11:51 CST
- **标签**: module:ops
- **摘要**: Fuses silu+clamp+mul into a single 1D-grid row-loop triton kernel, following the same launch pattern as swiglu_quant.py / rope.py (grid=(num_vectorcore,), per-row loop) to minimize host launch overhead. Numerically equivalent to SwigluStepAndMul.forward_native: silu-then-clamp on the gate half, +/-l…

### #11466 — [[BugFix] Resolve block table overflow](https://github.com/vllm-project/vllm-ascend/pull/11466)
- **作者**: yuxingcyx  **时间**: 2026-07-06 11:41 CST
- **标签**: ready
- **摘要**: ## PR Description  ```markdown ### What this PR does / why we need it?  This PR fixes a block table overflow issue for Mamba KV cache groups when speculative decoding (MTP/EAGLE) is enabled and prefix caching is disabled.  When initializing the block table in `may_reinitialize_input_batch()`, `max_n…

### #11465 — [[CI] Resources for ubuntu-latest are tight and need to be replaced with self-hosted runners](https://github.com/vllm-project/vllm-ascend/pull/11465)
- **作者**: czc-unac  **时间**: 2026-07-06 11:36 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it?  Resources for ubuntu-latest are tight and need to be replaced with self-hosted runners ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da…

### #11464 — [[main][CI] update nightly Qwen3-30B-QuaRot buffersize](https://github.com/vllm-project/vllm-ascend/pull/11464)
- **作者**: ppppeng  **时间**: 2026-07-06 11:34 CST
- **标签**: module:tests, ready
- **摘要**: <!--  Thanks for sending a pull request!  BEFORE SUBMITTING, PLEASE READ https://docs.vllm.ai/en/latest/contributing/overview.html  --> ### What this PR does / why we need it? Increased the HCCL_BUFFSIZE environment variable from 512 to 768 for the Qwen3-30B-QuaRot nightly test configuration. <!-- -…

### #11463 — [[Doc][Misc] Standardize MiniMax-M2 Docker installation section](https://github.com/vllm-project/vllm-ascend/pull/11463)
- **作者**: xuchi-0808  **时间**: 2026-07-06 10:47 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Standardize the Docker Image Installation section (4.1) in the MiniMax-M2 model doc to align with the latest template (Qwen3-30B-A3B):  - Use `tab-set` for A3/A2 hardware selection instead of flat code blocks - Separate Docker Pull and Docker Run in each tab …

### #11462 — [[CI] optimize e2e test of test_uva.py](https://github.com/vllm-project/vllm-ascend/pull/11462)
- **作者**: Ronald1995  **时间**: 2026-07-06 10:27 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This pull request optimizes the CI pipeline by streamlining the end-to-end testing process in test_uva.py. By pruning unnecessary model tests and removing legacy test cases, the overall runtime for the test suite is reduced, leading to faster feedback loops i…

### #11461 — [[Feature][MRV2] Collapse Eagle decode draft into a single ACL graph capture](https://github.com/vllm-project/vllm-ascend/pull/11461)
- **作者**: wxsIcey  **时间**: 2026-07-06 10:22 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it? Previously, each speculative decode step was captured as a separate graph. This change captures all N-1 draft steps in one unified graph. This pr also solve the acceptance of FULL Graph.  ### Does this PR introduce _any_ user-facing change? N/A  ### How was th…

### #11460 — [[CI] Add analyze-failure-report job (Job 3) with hitest cross-referen…](https://github.com/vllm-project/vllm-ascend/pull/11460)
- **作者**: Xuyzhen  **时间**: 2026-07-06 10:18 CST
- **标签**: ci/build
- **摘要**: …ce and fix missing regex dependency  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389
