# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-14 20:28 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期的动态摘要：

### 🛠 Issue 动态
- **310P 部署 Qwen3 报错** (#12015)：用户在 310P 环境下使用 `v0.22.1rc1-310p-openeuler` 镜像部署 `Qwen3-14B-w8a8sc` 时遇到 Bug（ERR00100 PTA call acl api failed 相关报错）。

### 🚀 PR 动态
**1. 版本发布与 CI 升级（核心重点）**
- **推进 v0.23.0 发版**：多个 PR 为 v0.23.0 版本发布做准备，包括添加镜像构建与推送选项 (#12014)、构建 A5 及 nightly 镜像 (#12024)。
- **vLLM 版本升级**：CI 环境弃用 v0.23.0，将 vLLM 底层版本升级至 v0.24.0 (#12019)。
- **适配上游最新代码**：同步适配 vLLM main 分支最新变更，主要涉及 QwenGatedDeltaNetAttention 及 Qwen3NextAttention 的 forward 接口修改 (#12020)。

**2. 新特性与功能支持**
- **MRV2 支持 DSpark FullGraph**：在 MRV2 (Model Runner V2) 中支持 DSpark 的 FullGraph 模式 (#12017)。
- **重新引入 SFA C8 支持**：为 A3 重新应用 SFA sparse C8 支持，并重构 KV cache layout 以共享 packed/merged 表示 (#12023)。

**3. Bug 修复**
- **修复模型加载与推测解码问题**：修复 D2D netloader 加载模型时的 `static_forward_context` 清理失败及 MTP draft 加载问题（影响推测解码场景） (#12022)。
- **修复 GLM5.1 w8a8 兼容性**：解决 GLM5.1 新版本代码与 PTA API 不适配的问题，增加 GLM5.1 库版本检查及警告提示 (#12018)。

**4. 文档更新**
- 新增 `RecomputeCPUOffload` 特性文档 (#12016)。
- 更新文档头部通知栏，提示用户当前正在查看稳定版 (v0.13.0) 文档 (#12021)。

### 🎉 Release 动态
近期暂无正式 Release 发布记录，但通过 CI 和文档相关 PR 可以看出，**仓库正处于 v0.23.0 版本的密集发版筹备阶段**，涵盖镜像构建、文档补充、特性合入及 Bug 修复。同时，开发分支已开始向 v0.24.0 演进。

---

## 🐛 Issues

### #12015 — [[Bug]: 310P上部署Qwen3-14B-w8a8sc报错](https://github.com/vllm-project/vllm-ascend/issues/12015)
- **作者**: zhang822188  **时间**: 2026-07-14 16:54 CST
- **标签**: bug, 310p
- **摘要**: ### Your current environment  https://docs.vllm.ai/projects/ascend/zh-cn/latest/tutorials/hardwares/310p.html#_3  参照这个文档部署的。 用的 quay.io/ascend/vllm-ascend:v0.22.1rc1-310p-openeuler这个镜像  ### 🐛 Describe the bug  root@openEuler workspace]# vllm serve /model/Qwen3-VL-2B-Instruct/ --port 8000 --max-model…

## 🔀 Pull Requests

### #12024 — [[CI] A5 image build Release 23.0.0](https://github.com/vllm-project/vllm-ascend/pull/12024)
- **作者**: xqchen7  **时间**: 2026-07-14 19:50 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? build A5 image and nightly image  ### Does this PR introduce _any_ user-facing change? NA  ### How was this patch tested? whether build A5 image and nightly image successfully. - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/e5…

### #12023 — [[v0.23.0] Re-apply SFA C8 support](https://github.com/vllm-project/vllm-ascend/pull/12023)
- **作者**: ZYang6263  **时间**: 2026-07-14 19:25 CST
- **标签**: module:tests, ready
- **摘要**: Re-apply PR https://github.com/vllm-project/vllm-ascend/pull/11846  This PR adds SFA sparse C8 support for A3 and refactors the KV cache layout to share the same packed/merged representation used by the sparse C8 path. Before this change, SFA KV quantization used a separate switch and could allocate…

### #12022 — [[BugFix] Fix static_forward_context cleanup and MTP draft loading](https://github.com/vllm-project/vllm-ascend/pull/12022)
- **作者**: yilunh998  **时间**: 2026-07-14 19:06 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? This PR fixes two issues in ModelNetLoaderElastic when loading models via D2D netloader, especially in MTP / speculative decoding scenarios. **1. Clear static_forward_context before fallback** When elastic loading fails and the loader falls back to DefaultMode…

### #12021 — [[Doc][Misc] Update documentation header notification bar](https://github.com/vllm-project/vllm-ascend/pull/12021)
- **作者**: herizhen  **时间**: 2026-07-14 18:35 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? This pull request updates the notification bar in the documentation header template to show that the user is viewing the stable release (v0.13.0) documentation and links to the latest developer preview.   ### Does this PR introduce _any_ user-facing change? Ye…

### #12020 — [[Misc]feat: adapt to vLLM main (85c09e98)](https://github.com/vllm-project/vllm-ascend/pull/12020)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-14 18:22 CST
- **标签**: module:ops, ready
- **摘要**: ### What this PR does / why we need it?  vllm upstream `e5588e49...85c09e98` (1/1 steps).  #### vllm_ascend/ops/gdn.py  - Cause: Upstream changed QwenGatedDeltaNetAttention.forward, Qwen3NextAttention.forward, and Qwen3NextDecoderLayer.forward to return tensors instead of writing to output buffers. …

### #12019 — [[CI] vllm drop v0.23.0 and upgrade v0.24.0](https://github.com/vllm-project/vllm-ascend/pull/12019)
- **作者**: MrZ20  **时间**: 2026-07-14 18:16 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization, ready
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/e5588e49bc2642670116664a7fc4096e27adb179

### #12018 — [Solution (#9554): [Bug]: GLM5.1 w8a8新版本代码不适配，报错ERR00100 PTA call acl api failed.](https://github.com/vllm-project/vllm-ascend/pull/12018)
- **作者**: TFGSUMIT  **时间**: 2026-07-14 18:15 CST
- **摘要**: This pull request fixes the compatibility issue between the GLM5.1 library and the PTA API by checking the version of the GLM5.1 library and warning if it is 5.1.0. The NPUBridge object is updated to initialize and call the PTA API successfully.  Changes made:  * Added `check_glm_version` function t…

### #12017 — [[Feature][MRV2] Support FullGraph for DSpark](https://github.com/vllm-project/vllm-ascend/pull/12017)
- **作者**: wxsIcey  **时间**: 2026-07-14 17:29 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Support FullGraph for DSpark in MRV2.  ### Does this PR introduce _any_ user-facing change? N/A  ### How was this patch tested? ``` VLLM_USE_V2_MODEL_RUNNER=1 python examples/test_dspark_acceptance.py  --model-dir ~/.cache/modelscope/hub/models/Qwen/Qwen3-8B  …

### #12016 — [[Doc][v0.23.0] Add RecomputeCPUOffload doc](https://github.com/vllm-project/vllm-ascend/pull/12016)
- **作者**: nwpu-zxr  **时间**: 2026-07-14 17:20 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? Add RecomputeCPUOffload doc.  ### Does this PR introduce _any_ user-facing change? No.  ### How was this patch tested? By CI.  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389

### #12014 — [[CI] add options for releases/v0.23.0](https://github.com/vllm-project/vllm-ascend/pull/12014)
- **作者**: zhangxinyuehfad  **时间**: 2026-07-14 16:51 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? add image build and push options for releases/v0.23.0  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/e5588e49bc2642670116664a7fc4096e27adb1…
