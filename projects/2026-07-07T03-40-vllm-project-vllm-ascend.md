# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-07 11:40 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库最近的动态摘要：

### 🚨 Issue 动态
* **算子兼容性报错 (#11519)**：在 `ascend910_93` 环境下运行 vLLM-Ascend 0.18.0 时，出现 `aclnnAddRmsNormBias` 调用失败的 RuntimeError。错误提示该 SoC 版本的配置文件不支持 `AddRmsNormBias` 算子类型，影响了特定硬件上的正常运行。

### 🛠 Pull Request 动态
**1. 重要新特性与适配**
* **批量感知调度器 (#11526)**：引入专为**离线批量推理**场景设计的 Batch-Job-Aware Scheduler，重点提升该场景下的吞吐量与硬件利用率。
* **MiniCPM 模型适配 (#11523)**：完成 **MiniCPM-2B-dpo-bf16** 模型在 Ascend NPU 上的适配，添加了端到端测试配置及模型使用教程。
* **Hunyuan3 (Hy3) 0-day 适配 (#11527)**：修复 vllm-ascend 0.22.1rc 上的两个 Hy3 兼容性问题，特别是补充了缺失的 MTP 模型类型转换（`hy_v3` → `hy_v3_m`）。

**2. Bug 修复**
* **PD Disaggregation 修复 (#11517)**：修复了启用 PD 分离部署（Prefill/Decode Disaggregation）时产生的相关 Bug。

**3. 文档与易用性改进**
* **DeepSeek-V4 文档优化 (#11525, #11520)**：对 DeepSeek-V4-Flash 和 Pro 教程进行一致性修复，补充了缺失的 Docker 卷挂载（`-v /root/.cache:/root/.c`）等关键信息，提升实操体验。
* **Issue 反馈按钮 (#11524)**：在文档页面新增浮动的“报告问题”按钮，可根据当前语言路径（如 `/zh-cn/`）动态构建并跳转 GitHub Issue 提交页。

**4. CI 与测试维护**
* **更新周级别 CI 用例 (#11518)**：新增 Deepseekv3.2-w8a8-A3、GLM5.1-W8A8-A3、Qwen3.5-397B-A17B-w8a8-mtp-A3 等大模型量化/评估的周测配置。
* **修复 CI 阻塞 (#11522)**：通过跳过 MRV2 DFlash 端到端测试，解决当前 CI 流水线失败问题。
* **重构二分定位脚本路径 (#11521)**：将 bisect 模块路径从 `tests.e2e.nightly.bisect` 更新至 `tools.bisect`，优化单节点与多节点 AOP 脚本的调用入口。

### 📦 Release 动态
* 本次提供的时间范围内**无新的 Release 版本发布**。

---

## 🐛 Issues

### #11519 — [[Bug]: RuntimeError: call aclnnAddRmsNormBias failed, detail:[PID: 364676] 2026-07-07-09:36:12.120.702 AclNN_Parameter_Error(EZ1001): Get regInfo failed, The binary_info_config.json of socVersion [ascend910_93] does not support opType [AddRmsNormBias].](https://github.com/vllm-project/vllm-ascend/issues/11519)
- **作者**: mushan09  **时间**: 2026-07-07 09:41 CST
- **标签**: bug, triaged
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Your output of above commands here ```  </details>   ### 🐛 Describe the bug  Step3.5  VLLM 0.18.0 VLLM-Ascend 0.18.0

## 🔀 Pull Requests

### #11527 — [[0-day] Hy3 0-day adaptation](https://github.com/vllm-project/vllm-ascend/pull/11527)
- **作者**: rjg-lyh  **时间**: 2026-07-07 11:36 CST
- **标签**: module:quantization
- **摘要**: ### What this PR does / why we need it?   Fix two Hunyuan3 (Hy3) compatibility issues on vllm-ascend 0.22.1rc:      1. **MTP model type conversion**: `patch_speculative_config` was missing `hy_v3` → `hy_v3_mtp` conversion, causing   `Unsupported speculative method: 'mtp'` when MTP is enabled.   2. *…

### #11526 — [[Feature] Support batch job aware scheduler](https://github.com/vllm-project/vllm-ascend/pull/11526)
- **作者**: wanqi01  **时间**: 2026-07-07 10:46 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it? This PR introduces a **Batch-Job-Aware Scheduler** designed for **offline batch inference** scenarios where throughput and hardware utilisation are the primary goals. It is particularly effective when processing multiple batch jobs concurrently, each with a di…

### #11525 — [[Doc] The usability of the DeepSeek-V4 documentation is improved](https://github.com/vllm-project/vllm-ascend/pull/11525)
- **作者**: GDzhu01  **时间**: 2026-07-07 10:45 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Three consistency fixes applied to both the DeepSeek-V4-Flash and DeepSeek-V4-Pro tutorials:    1. Added missing Docker volume mount   Added -v /root/.cache:/root/.cache to 4 docker run commands across the two docs. This exposes the host's   ~/.cache (Hugging…

### #11524 — [[Doc] add issue feedback button](https://github.com/vllm-project/vllm-ascend/pull/11524)
- **作者**: wangxiyuan  **时间**: 2026-07-07 10:45 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? This PR adds a floating "report issue" button to the documentation pages. It dynamically constructs the issue URL using the current language path (`/zh-cn/` or `/en/`) and the Read the Docs version. The necessary environment variables (`READTHEDOCS_VERSION` an…

### #11523 — [[Doc][Feature] Adapt MiniCPM-2B-dpo-bf16 to Ascend NPU](https://github.com/vllm-project/vllm-ascend/pull/11523)
- **作者**: zhangkx-777  **时间**: 2026-07-07 10:43 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it? This PR adapts the MiniCPM-2B-dpo-bf16 model to run on Ascend NPU via `vllm-ascend`. It adds the end-to-end test configuration, model adaptation tutorial, and a skill record documenting lessons learned and troubleshooting tips. It also registers the model in t…

### #11522 — [[Misc] Fix CI by skip MRV2 DFlash e2e test](https://github.com/vllm-project/vllm-ascend/pull/11522)
- **作者**: wxsIcey  **时间**: 2026-07-07 10:25 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it? [Misc] Fix CI by skip MRV2 DFlash e2e test  ### Does this PR introduce _any_ user-facing change? N/A  ### How was this patch tested? CI passed with new added/existing test.  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486…

### #11521 — [[Test] feat: update bisect module path from tests to tools](https://github.com/vllm-project/vllm-ascend/pull/11521)
- **作者**: czydyy  **时间**: 2026-07-07 10:05 CST
- **标签**: module:tests
- **摘要**: Update `python -m tests.e2e.nightly.bisect.auto_bisect` to `python -m tools.bisect.auto_bisect` in AOP scripts.  - aop_process.sh: single-node AOP bisect entry - run.sh: multi-node worker bisect and aop_pipeline bisect entry  ### What this PR does / why we need it?  ### Does this PR introduce _any_ …

### #11520 — [[Doc] The usability of the DeepSeek-V4 documentation is improved.](https://github.com/vllm-project/vllm-ascend/pull/11520)
- **作者**: GDzhu01  **时间**: 2026-07-07 09:45 CST
- **标签**: documentation
- **摘要**: <!--  Thanks for sending a pull request!  BEFORE SUBMITTING, PLEASE READ https://docs.vllm.ai/en/latest/contributing/overview.html  --> ### What this PR does / why we need it?  Three consistency fixes applied to both the DeepSeek-V4-Flash and DeepSeek-V4-Pro tutorials:    1. Added missing Docker vol…

### #11518 — [[CI] update weekly cases: Deepseekv3.2-w8a8-A3.yaml GLM5.1-W8A8-a3-weekly.yaml Qwen3.5-397B-A17B-w8a8-mtp-A3-weekly.yaml](https://github.com/vllm-project/vllm-ascend/pull/11518)
- **作者**: guxin108  **时间**: 2026-07-07 09:09 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  we update weekly cases: Deepseekv3.2-w8a8-A3.yaml GLM5.1-W8A8-a3-weekly.yaml Qwen3.5-397B-A17B-w8a8-mtp-A3-weekly.yaml  ### Does this PR introduce _any_ user-facing change? no  ### How was this patch tested? run the cases weekly  - vLLM version: v0.23.0 - vLL…

### #11517 — [fix p bugs while enabling PD Disaggregation](https://github.com/vllm-project/vllm-ascend/pull/11517)
- **作者**: chenhy97  **时间**: 2026-07-07 09:00 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.22.1 - vLLM main: https://github.com/vllm-project/vllm/commit/967c5c3bc38891f4465d3f4e99917ed837bb3833
