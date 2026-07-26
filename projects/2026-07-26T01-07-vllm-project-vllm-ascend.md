# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-26 09:07 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期的动态摘要：

### 📌 Issue 动态
* **CI 自动化流程中断需人工审核** (#12880)：`main2main` 自动同步流程在执行过程中未能完成所有计划步骤，目前已暂停并需要人工介入审核（关联 Draft PR #12879）。

---

### 🔧 PR 动态

**1. 核心适配与重构**
* **适配 vLLM 主分支** (#12879)：将 vllm-ascend 适配至 vLLM 主分支截至7月21日的最新提交。
* **同步权重同步生命周期重构** (#12876)：将 vLLM 上游的权重同步生命周期重构（#44353）移植到 Ascend HCCL 及 NPU IPC 后端，保持架构对齐。
* **移除 mix_placement 功能** (#12871)：清理并移除了 `mix_placement` 特性路径及其相关的“共享专家作为路由专家”的处理逻辑及配置解析。

**2. 新特性与性能优化**
* **CANN 9.1.0 支持 mla_prolog 算子** (#12878)：在 CANN 9.1.0 版本中引入对 `mla_prolog` 算子的支持，以提升性能。
* **新增 NPU 融合 GDN decode 算子** (#12870)：为 Qwen3.5 / Qwen3-Next 的 Gated Delta Net 解码核心新增了 AscendC 融合 GDN decode 算子，直接处理 `mixed_qkv` 数据，提升解码效率。

**3. BugFix 修复**
* **修复 A5 endpoint 配置的设备 ID 映射问题** (#12873)：修正了此前 PR #9690 引入的缺陷，改用物理设备 ID（而非 `local_rank`）来选择每设备的 endpoint 文件。
* **增加 sq/cq 绑定失败的警告日志** (#12877)：在 sq/cq 绑定失败时增加警告日志输出，便于问题排查。

**4. 文档与 CI 更新**
* **新增 MiniCPM-o 4.5 验证教程** (#12874)：增加了 `MiniCPM-o-4_5` 模型在 Ascend NPU 上的验证教程及支持矩阵条目。
* **新增 Ascend 950DT 文档** (#12875)：补充了 Ascend 950DT DSV4 Flash & Pro 的相关文档。
* **自动更新 CI 测试预估时间** (#12872)：由 CI Workflow 自动触发，更新了 `test_config.yaml` 中的测试预估耗时。

---

### 🚀 Release 动态
* **近期无新版本发布**。

---

## 🐛 Issues

### #12880 — [[main2main] main2main manual review required (97a98006)](https://github.com/vllm-project/vllm-ascend/issues/12880)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-26 03:10 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/12879 - Commit range: `fae543015cd49dfb4db3afb3a2e96d2eccb3e1db`...`97a98006b0895bc3d488ec19e898f56c777150b6` - Status: `failed`  ## Final Summary  …

## 🔀 Pull Requests

### #12879 — [[Misc]feat: adapt to vLLM main (97a98006)](https://github.com/vllm-project/vllm-ascend/pull/12879)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-26 03:09 CST
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to July 21.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | `vllm_ascend/patch/worker/patch_process_weights_after_loading.py`<br>`vllm…

### #12878 — [[Performance] 9.1.0 CANN supports the mla_prolog operator](https://github.com/vllm-project/vllm-ascend/pull/12878)
- **作者**: chenxi-hh  **时间**: 2026-07-26 01:05 CST
- **摘要**: <!--  Thanks for sending a pull request!  BEFORE SUBMITTING, PLEASE READ https://docs.vllm.ai/en/latest/contributing/overview.html  --> ### What this PR does / why we need it? 9.1.0 CANN supports the mla_prolog operator  ### Does this PR introduce _any_ user-facing change? <!-- Note that it means *a…

### #12877 — [[BugFix] add warning log while sq/cq fail to bind](https://github.com/vllm-project/vllm-ascend/pull/12877)
- **作者**: zouyida2052  **时间**: 2026-07-25 22:46 CST
- **标签**: module:core
- **摘要**: ### What this PR does / why we need it? [BugFix] add warning log while sq/cq fail to bind  ### Does this PR introduce _any_ user-facing change? no  ### How was this patch tested?  - vLLM version: v0.25.1 - vLLM main: https://github.com/vllm-project/vllm/commit/fe784ff22e630a31fd798f392b01e0a75c18f04…

### #12876 — [[Refactor] Align weight transfer lifecycle with vLLM](https://github.com/vllm-project/vllm-ascend/pull/12876)
- **作者**: Ulrica111  **时间**: 2026-07-25 21:12 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Ports the vLLM weight sync lifecycle refactor from [vllm-project/vllm#44353](https://github.com/vllm-project/vllm/pull/44353) to the Ascend HCCL and NPU IPC backends.  The upstream refactor moves weight-loading decisions out of the worker and into each weight…

### #12875 — [[Doc][Ascend950] Add 950DT  DSV4 Flash& Pro documentation](https://github.com/vllm-project/vllm-ascend/pull/12875)
- **作者**: linfeng-yuan  **时间**: 2026-07-25 20:52 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389

### #12874 — [[Doc][Feature] Add verification tutorial and support matrix entry for MiniCPM-o 4.5](https://github.com/vllm-project/vllm-ascend/pull/12874)
- **作者**: XCgaohz  **时间**: 2026-07-25 20:13 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? This pull request adds a new tutorial document (`docs/source/tutorials/models/MiniCPM-o-4_5.md`) for verifying the `OpenBMB/MiniCPM-o-4_5` model on Ascend NPUs. It covers environment preparation, installation, online service deployment, known issues (such as t…

### #12873 — [[BugFix][Worker] Use physical device ID for A5 endpoint config](https://github.com/vllm-project/vllm-ascend/pull/12873)
- **作者**: Yuli-yx  **时间**: 2026-07-25 20:05 CST
- **标签**: module:core
- **摘要**: ### What this PR does / why we need it?  Fixes the device ID mapping issue introduced by #9690.  `setup_ascend_local_comm_res` used `local_rank` to select a per-device endpoint file. When the vLLM logical device ID differs from the physical NPU ID, this can load the wrong file, for example `ub_endpo…

### #12872 — [[CI] Auto-update estimated test times in test_config.yaml](https://github.com/vllm-project/vllm-ascend/pull/12872)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-25 19:59 CST
- **摘要**: ## Summary  This PR was auto-generated by the **Update estimated test times** [workflow](https://github.com/vllm-project/vllm-ascend/actions/runs/30153141813).  It updates the `estimated_times` values in `.github/workflows/scripts/test_config.yaml` based on actual elapsed times collected from CI wor…

### #12871 — [[Refactor][Misc] Remove mix placement support](https://github.com/vllm-project/vllm-ascend/pull/12871)
- **作者**: ningjingbengxiaohai  **时间**: 2026-07-25 19:43 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization, ready
- **摘要**: ### What this PR does / why we need it?  Remove the `mix_placement` feature path and its related shared-expert-as-routed-expert handling:  - Remove `AscendConfig.mix_placement` parsing and validation. - Remove shared-expert mixed placement logic from EPLB placement generation. - Remove router top-k …

### #12870 — [[Attention][Feature] Add NPU fused GDN decode operator](https://github.com/vllm-project/vllm-ascend/pull/12870)
- **作者**: happyyzy  **时间**: 2026-07-25 19:19 CST
- **摘要**: ### What this PR does  Add an AscendC fused GDN decode operator for the Qwen3.5 / Qwen3-Next Gated Delta Net decode core.  The new operator directly consumes:  ```text mixed_qkv         [B, 2 * H * K + HV * V] a / b             [B, HV] A_log / dt_bias   [HV] state             [num_slots, HV, V, K] s…
