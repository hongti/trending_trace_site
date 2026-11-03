# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-03 15:09 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🐛 Issue (问题)
*   **结构化输出并发异常 (#20748)**：在结构化输出场景的并发测试中，当输出整型字段时会出现异常，导致模型陷入无限数字流，直到达到 `max_tokens` 才停止。
*   **MooncakeHybridConnector 角色配置失效 (#20743)**：当配置 `kv_role="kv_both"` 时，worker 仅启动了接收线程，未创建发送线程，导致引擎功能不完整。

### 🔧 Pull Request (拉取请求)
**核心特性与性能优化：**
*   **支持 Mooncake `kv_both` 角色 (#20746)**：修复上述 Issue #20743，使 `MooncakeHybridConnector` 在 `kv_both` 模式下同时启动发送和接收线程，与上游 vLLM 对齐。
*   **新增 DeepGEMM-Ascend MegaMoE 后端 (#20742)**：为 Ascend950 引入实验性的 routed FP8 x FP4 MegaMoE 后端，专门支持 DeepSeek V4.1 模型。
*   **DeepSeek V4.1 性能融合优化 (#20744)**：在 DeepSeek-V4.1 的 shared-expert 路径上，将 shared/routed 输出加法与 FFN HcPost 算子融合，避免写入中间 BF16 数据，提升执行效率。

**Bug 修复：**
*   **WeightTransfer 修复 (#20740)**：优化 NPU IPC 传输逻辑，每个 packed NPU IPC export 仅导入一次，避免重复重建和释放生产者的暂存缓冲区。
*   **EPLB 单测修复 (#20738)**：修复 EPLB 工具单元测试中期望张量在 CPU 上而实际在 NPU 上导致的设备不一致报错。
*   **CI 修复 (#20747, #20749)**：修复 nightly profile benchmark 参数解析回归问题；修复在创建 App token 前 recipe preview dispatch 失败的问题。

**文档与测试：**
*   **文档更新 (#20745, #20741)**：自动翻译 v0.30.0 相关文档；为 950DT 的 Qwen3.8 教程新增 `/lib64/libm.so.6` 链接器错误排查指南。
*   **新增测试 (#20739)**：添加 GLM-5.2-layerwise 相关测试。

### 🚀 Release (发布)
*   近期无版本发布信息。

---

## 🐛 Issues

### #20748 — [[Bug]: 结构化输出场景并发测试输出异常,content输出在整型字段时会进入无限数字流直到max_tokens才停止](https://github.com/vllm-project/vllm-ascend/issues/20748)
- **作者**: mengshuaixiao  **时间**: 2026-11-03 15:00 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details>GLM5.3-Flash模型在结构化输出场景并发测试输出异常,content输出在整型字段时会进入无限数字流直到max_tokens才停止 <summary> 正常请求响应:data: {"id":"b6eebd0f-2059-43af-bffd-b0602c6ad7cd","object":"chat.completion.chunk","created":1793688231,"model":"auto","choices":[{"index":0,"delta":{"content":"","reasoning…

### #20743 — [[Bug]: MooncakeHybridConnector starts only the receive thread with kv_role="kv_both"](https://github.com/vllm-project/vllm-ascend/issues/20743)
- **作者**: ZRJ026  **时间**: 2026-11-03 14:45 CST
- **摘要**: ### Describe the bug  `MooncakeHybridConnector` accepts `kv_role="kv_both"` but the worker starts only the receive thread. The engine starts successfully; the sender is never created, so the engine can receive KV cache but can never serve it to a peer.  This matters for deployments where the same in…

## 🔀 Pull Requests

### #20749 — [[CI] Fix recipe preview dispatch and publish links immediately after labeling](https://github.com/vllm-project/vllm-ascend/pull/20749)
- **作者**: SOMEONEUNSEEN  **时间**: 2026-11-03 15:05 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it?  Recipe preview dispatch fails on the proxied self-hosted runner before an App token is created, as reproduced by [the smoke-test PR](https://github.com/vllm-project/vllm-ascend/pull/20728). The initial preview comment can also wait for a separately queued wor…

### #20747 — [[BugFix][CI] Fix nightly profile benchmark argument parsing](https://github.com/vllm-project/vllm-ascend/pull/20747)
- **作者**: MrZ20  **时间**: 2026-11-03 14:58 CST
- **摘要**: ### What this PR does / why we need it?  Fix a regression introduced by #17634. Its new `--vllm-ref` branch separated `--profile-benchmark-names` from its `PENDING_OPTION` reset, causing valid commands to fail with `requires a value`.  Restore the reset immediately after consuming the benchmark name…

### #20746 — [[Feature][KV Transfer] Support kv_both in MooncakeHybridConnector](https://github.com/vllm-project/vllm-ascend/pull/20746)
- **作者**: ZRJ026  **时间**: 2026-11-03 14:52 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  `kv_role="kv_both"` is a valid `KVRole`, and upstream vLLM's Mooncake connector supports it by starting both the sender and the receiver.  `MooncakeHybridConnector` was written for the one-engine-one-role model:  ```python if self.kv_role == "kv_producer":   …

### #20745 — [[v0.30.0][Doc] Translated Doc files 2026-11-03](https://github.com/vllm-project/vllm-ascend/pull/20745)
- **作者**: vllm-ascend-ci  **时间**: 2026-11-03 14:47 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **4** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/getting_started/installation/install_vllm_ascend.inc.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/getti…

### #20744 — [[Performance][Model][v0.30.0] Fuse DeepSeek V4.1 shared/routed Add with FFN HcPost](https://github.com/vllm-project/vllm-ascend/pull/20744)
- **作者**: Abstrey  **时间**: 2026-11-03 14:46 CST
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  On the DeepSeek-V4.1 shared-expert path, shared/routed output addition writes a BF16 intermediate that FFN HcPost immediately reads. This PR adds an opt-in Triton kernel that consumes both finalized expert outputs and performs Add + HcPost in one launch.  - P…

### #20742 — [[Ops][Feature] Add DeepGEMM-Ascend routed FP8 x FP4 MegaMoE backend](https://github.com/vllm-project/vllm-ascend/pull/20742)
- **作者**: lengrongfu  **时间**: 2026-11-03 14:36 CST
- **标签**: module:tests, module:ops, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it? This pull request introduces an experimental DeepGEMM-Ascend routed FP8 x FP4 MegaMoE backend for Ascend950, specifically supporting DeepSeek V4.1. It adds configuration validation, weight preparation, and integration with the fused MoE communication implement…

### #20741 — [[Doc][Qwen3.8] Add libm.so.6 linker troubleshooting for 950DT](https://github.com/vllm-project/vllm-ascend/pull/20741)
- **作者**: kc2608  **时间**: 2026-11-03 14:30 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Add a `5.2 Troubleshooting` subsection to the Qwen3.8-27B tutorial covering the `/lib64/libm.so.6` linker error reported in #20245.  The 950DT Qwen3.8 image ships the math libraries under `/usr/lib64/` but the linker defaults to `/lib64/`. Document the symlin…

### #20740 — [[BugFix][WeightTransfer] Import each packed NPU IPC export once](https://github.com/vllm-project/vllm-ascend/pull/20740)
- **作者**: yeyunu  **时间**: 2026-11-03 14:29 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Packed NPU IPC transfers currently rebuild and then release the producer's shared mapping once per chunk. The producer, however, exports its reusable staging buffer only once and sends the same rebuild arguments with every chunk. Since torch pairs one IPC exp…

### #20739 — [[Test] GLM-5.2-layerwise](https://github.com/vllm-project/vllm-ascend/pull/20739)
- **作者**: chen-commits  **时间**: 2026-11-03 14:26 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…

### #20738 — [[BugFix][EPLB][Test] Align expected tensors to NPU device in eplb_utils unit tests](https://github.com/vllm-project/vllm-ascend/pull/20738)
- **作者**: kc2608  **时间**: 2026-11-03 14:03 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  The EPLB unit tests build expected tensors on CPU while the code under test produces tensors on NPU, so `torch.equal` raises `"Expected all tensors to be on the same device"` and these tests fail on NPU hosts:  - `TestAscendConfig::test_generate_log2phy_map_r…
