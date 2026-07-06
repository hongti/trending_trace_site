# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-06 01:34 UTC

## AI 总结

# vllm-ascend 仓库近期动态摘要

---

## 🐛 Issue

| # | 标题 | 要点 |
|---|------|------|
| #11456 | Qwen 3.6-27b-w8a8 使用 nightly-main-310p 镜像无法启动 | 在 310P 环境下部署 W8A8 量化模型启动失败，涉及 PyTorch 2.10.0 + CPU 版本环境 |
| #11454 | AscendStoreConnector Store Put 失败 (HcclBatchPut hccl_result=4) | 在 910B 上即使最小 1K prompt、禁用 SSD offload，Mooncake Store Put 仍无法完成，Prefill worker 报 HCCL 批量写入错误 |
| #11450 | 300 I Duo 部署 PaddleOCR-VL-1.6 报错 | 在 300 I Duo 硬件 + 驱动 25.2.0 + v0.21.0rc 容器环境下部署 PaddleOCR-VL 模型出错 |

> **共性趋势**：近期 Issue 主要集中在**特定硬件型号兼容性**（310P / 300 I Duo）和**分布式存储/通信链路**（HCCL BatchPut）两大方向。

---

## 🔀 Pull Request

| # | 类型 | 标题 | 要点 |
|---|------|------|------|
| #11455 | **BugFix** | 优化 sampler batch sampling + 增加 MTP acceptance 监控 | 修复采样器批量采样逻辑，新增 Multi-Token Prediction 接受率监控（v0.20.2rc 分支） |
| #11449 | **Performance** | 复用 DP metadata 同步缓冲区 | 避免每次 `_sync_metadata_across_dp` 重新分配 CPU tensor，改为 per-runner 可复用缓冲区，减少内存开销和 GC 压力 |
| #11451 | **Feature / Test** | 为 A5 启用 `c8_enable_reshape_optim` + 补充 UT | 开启 reshape 优化路径并覆盖 `slot_mapping_cpu` / `store_kv_block_pre` 相关单测 |
| #11452 | **Compat** | A2G3 适配海外版 (CH) HDK | 使 vllm-ascend 可在海外版 HDK A2G3 硬件上正常安装与运行 |
| #11453 | **Refactor** | 移除 GLM parser 旧版 monkey patch | 删除已被新 parser-engine 路径取代的 `patch_glm47_tool_call_parser` 和 `patch_glm_tool_call_streaming`，同步清理相关 UT 与 patch 注册表 |
| #11457 | **Test** | A2 测试 | 基于 vLLM v0.23.0 (commit ee0da84) 的 A2 硬件测试流程 |

> **重点变更**：
> - **#11455（BugFix）** 直接修复采样逻辑并引入 MTP 监控，影响推理精度与可观测性
> - **#11449（Performance）** 是一项来自 vllm-ascend-hust 的上游优化，对 Data Parallel 场景下的元数据同步有显著性能改善
> - **#11452（Compat）** 扩展了海外版 HDK 的硬件支持范围

---

## 📦 Release

本次观察周期内**无新版本发布**。

---

**总结**：近期活动重心在**硬件兼容性扩展**（310P / 300 I Duo / A2G3 海外版 / A5 优化）、**推理核心模块修复与优化**（sampler / DP sync buffer），以及**代码瘦身**（移除过时 GLM patch）。Issue 侧暴露的 HCCL 通信和量化模型启动问题值得关注后续修复进展。

---

## 🐛 Issues

### #11456 — [[Bug]: 使用 nightly-main-310p 镜像部署 Qwen 3.6-27b-w8a8 无法启动](https://github.com/vllm-project/vllm-ascend/issues/11456)
- **作者**: etnAtker  **时间**: 2026-07-05 13:30 UTC
- **标签**: bug, 310p
- **摘要**: ### Your current environment  ```log Collecting environment information... PyTorch version: 2.10.0+cpu Is debug build: False  OS: Ubuntu 22.04.5 LTS (x86_64) GCC version: (Ubuntu 11.4.0-1ubuntu1~22.04.3) 11.4.0 Clang version: Could not collect CMake version: version 4.3.4 Libc version: glibc-2.35  P…

### #11454 — [AscendStoreConnector 1K no-SSD Store Put fails with HcclBatchPut hccl_result=4](https://github.com/vllm-project/vllm-ascend/issues/11454)
- **作者**: yanceng305-collab  **时间**: 2026-07-05 09:09 UTC
- **摘要**: ### Summary  `AscendStoreConnector` Store Put fails on Ascend 910B even with a minimal 1K prompt and embedded SSD offload disabled. The HTTP request completes successfully, but Mooncake Store never reaches `PutEnd`; the Prefill worker logs `Failed to invoke HcclBatchPut, hccl_result = 4` followed by…

### #11450 — [[Misc]: 300 I Duo部署PaddleOCR-VL-1.6模型报错](https://github.com/vllm-project/vllm-ascend/issues/11450)
- **作者**: robin-programmer  **时间**: 2026-07-05 05:24 UTC
- **标签**: paddle
- **摘要**: ### Anything you want to discuss about vllm on ascend.  驱动版本：25.2.0 容器：v0.21.0rc 容器命令： ``` docker run \   --name ocr \   --privileged \   --net=host \   --device /dev/davinci0 \   --device /dev/davinci1 \   --device /dev/davinci2 \   --device /dev/davinci3 \   --device /dev/davinci4 \   --device /de…

## 🔀 Pull Requests

### #11457 — [A2 测试](https://github.com/vllm-project/vllm-ascend/pull/11457)
- **作者**: czydyy  **时间**: 2026-07-06 00:58 UTC
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389

### #11455 — [[BugFix] [v0.20.2rc] Optimize sampler batch sampling and add MTP acceptan…](https://github.com/vllm-project/vllm-ascend/pull/11455)
- **作者**: zhaochuang001  **时间**: 2026-07-05 09:49 UTC
- **摘要**: …ce monitoring  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version:  - vLLM main: https://github.com/vllm-project/vllm/commit/

### #11453 — [[Refactor][Patch] Remove GLM parser backports](https://github.com/vllm-project/vllm-ascend/pull/11453)
- **作者**: QwertyJack  **时间**: 2026-07-05 08:00 UTC
- **标签**: module:tests
- **摘要**: ## What this PR does / why we need it?  Removes obsolete GLM parser monkey patches covered by the post #10935 vLLM parser-engine path:  - `patch_glm47_tool_call_parser.py` - `patch_glm_tool_call_streaming.py`  Also removes their patch-specific UTs and updates the patch registry notes for remaining p…

### #11452 — [A2G3 adapted for the CH version of HDK](https://github.com/vllm-project/vllm-ascend/pull/11452)
- **作者**: ZT-AIA  **时间**: 2026-07-05 07:58 UTC
- **摘要**: ### What this PR does / why we need it? VLLM ASCEND is adapted for the overseas version of HDK A2G3. To ensure that VLLM ASCEND can be properly installed and used in scenarios where the overseas version of HDK is being used.  ### Does this PR introduce _any_ user-facing change? No  ### How was this …

### #11451 — [[Feature][Test] Enable "c8_enable_reshape_optim" for A5 and add ut coverage](https://github.com/vllm-project/vllm-ascend/pull/11451)
- **作者**: lijiahang226  **时间**: 2026-07-05 07:36 UTC
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? This PR addresses two items:  1. **Add unit test for the `c8_enable_reshape_optim = True` code path**     When `c8_enable_reshape_optim` is `True`, `AscendSFAMetadataBuilder._build()` reads `slot_mapping_cpu`, invokes `torch.ops._C_ascend.store_kv_block_pre`, …

### #11449 — [[Performance] Reuse DP metadata sync buffers](https://github.com/vllm-project/vllm-ascend/pull/11449)
- **作者**: ShuhaoZhangTony  **时间**: 2026-07-05 01:57 UTC
- **标签**: module:tests
- **摘要**: This extracts the DP metadata sync buffer reuse optimization from vllm-ascend-hust for upstream review.  The current DP metadata sync path allocates fresh CPU tensors on every `_sync_metadata_across_dp` call. This PR keeps reusable per-runner buffers for:  - the packed DP metadata tensor - the `num_…
