# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-28 09:02 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
*   **Atlas 300I Duo 环境导入报错** (#12954)：用户反馈在 Atlas 300I Duo 上运行时出现 `triton.language` 导入错误。官方明确指出该硬件不支持 triton/triton-ascend，而当前 Dockerfile 中的依赖导致了此冲突。
*   **Gemma4 31B EAGLE3 投机解码方案** (#12947)：探讨了如何在 Ascend 上运行 Gemma4 31B 结合 EAGLE3 的投机解码，并记录了目标模型与草稿模型的配置流程。

### 🔀 PR 动态
**🚀 新特性与模型支持**
*   **Kimi K3 模型全面支持** (#12950, #12951)：同时向主分支和 v0.23.0 分支添加了 Kimi K3 模型支持，实现了包括 KDA 注意力机制、量化 MoE 执行、多模态处理、工具调用解析及 KV-cache 集成等核心功能。
*   **Gemma4 投机解码支持图像 Token** (#12948)：为 Gemma4 相关模型添加了多模态图像 Token 映射，修复了投机解码在处理图像时的兼容性问题。

**🛠️ 核心修复**
*   **修复 MRV2 ACL Graph 挂起问题** (#12944)：优化了 ACL graph replay 时的同步机制，避免了全局同步，从而修复了异步调度下参数更新流导致的挂起问题。

**📝 文档更新**
*   **新增 Kimi-K3 部署指南** (#12952)：补充了 v0.23.0 版本下 Kimi-K3 的部署教程，包含 ModelScope W4A8 权重引用及四节点 DP4/TP16 配置说明。
*   **修正 Kimi-K3 镜像标签** (#12953)：作为上述文档的补充，将 openEuler 镜像标签更正为 Atlas 800 A3 专用变体。
*   **文档模板规范化** (#12949)：按最新模板要求更新了相关文档材料。

**🤖 CI 与测试改进**
*   **修复覆盖率计算报错** (#12943, #12946)：补充了覆盖率数据处理时缺失的 `regex` 依赖安装，修复了 CI 流程中的上传报错。
*   **优化 CPU 单元测试日志** (#12945)：改进了 CPU 单元测试失败时的日志汇总逻辑，保留了下载的 artifact 目录以便独立排查，并将失败项归纳为单一结果。

### 🚀 Release 动态
*   **本期无新版本发布**。

---

## 🐛 Issues

### #12954 — [[Bug]: triton.language import error about Atlas 300I Duo](https://github.com/vllm-project/vllm-ascend/issues/12954)
- **作者**: Yikun  **时间**: 2026-07-28 08:48 CST
- **标签**: bug, 310p
- **摘要**: ### Your current environment  From: https://mp.weixin.qq.com/s/D_Md1zY8MWuXuFCFLThClQ ``` 坑三：vLLM Ascend 找不到 triton.language 属性 vLLM Ascend 官方明确说 Atlas 300I Duo 不支持 triton 或 triton-ascend，所以官方 Dockerfile 里直接卸载了 triton。  但这就导致一个问题：torch 和 vllm 的某些模块启动时会尝试 import triton.language，找不到就报错——即使你不需要 triton …

### #12947 — [Gemma4 31B EAGLE3 speculative decoding on Ascend](https://github.com/vllm-project/vllm-ascend/issues/12947)
- **作者**: Alex-stack-hub  **时间**: 2026-07-27 20:27 CST
- **摘要**: # Gemma4 31B EAGLE3 speculative decoding on Ascend  This issue documents how to run Gemma4 31B with EAGLE3 speculative decoding on vLLM Ascend.  ## Models  - Target model: Gemma4 31B Instruct - Draft model: [RedHatAI/gemma-4-31B-it-speculator.eagle3](https://huggingface.co/RedHatAI/gemma-4-31B-it-sp…

## 🔀 Pull Requests

### #12953 — [[Doc][Model] Fix Kimi-K3 openEuler image tag](https://github.com/vllm-project/vllm-ascend/pull/12953)
- **作者**: q664171689  **时间**: 2026-07-27 23:38 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Follow-up to #12952.  Updates the Kimi-K3 openEuler image reference from `quay.io/ascend/vllm-ascend:kimi-k3-openeuler` to the Atlas 800 A3 variant `quay.io/ascend/vllm-ascend:kimi-k3-a3-openeuler` in both the image table and Docker startup example.  ### Does…

### #12952 — [[Doc][Model] Add Kimi-K3 deployment guide](https://github.com/vllm-project/vllm-ascend/pull/12952)
- **作者**: q664171689  **时间**: 2026-07-27 23:19 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Adds a Kimi-K3 deployment tutorial for vLLM Ascend v0.23.0, including:  - ModelScope W4A8 weight and validated Atlas 800 A3 image references. - Four-node DP4/TP16/EP64 mixed deployment. - Sixteen-node prefill-decode separation deployment. - Functional verific…

### #12951 — [[Feature][v0.23.0] Support Kimi K3](https://github.com/vllm-project/vllm-ascend/pull/12951)
- **作者**: maoxx241  **时间**: 2026-07-27 23:17 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization
- **摘要**: ## Overview  Add Kimi K3 model support to vllm-ascend, including the model architecture, KDA attention, quantized MoE execution, multimodal processing, tool-call parsing, KV-cache integration, and Ascend custom operators.  ## Key Components  1. Model Architecture (`vllm_ascend/models/`)    - `kimi_k…

### #12950 — [[Feature] Support Kimi K3](https://github.com/vllm-project/vllm-ascend/pull/12950)
- **作者**: maoxx241  **时间**: 2026-07-27 23:17 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization, ready
- **摘要**: ## Overview  Add Kimi K3 model support to vllm-ascend, including the model architecture, KDA attention, quantized MoE execution, multimodal processing, tool-call parsing, KV-cache integration, and Ascend custom operators.  ## Key Components  1. Model Architecture (`vllm_ascend/models/`)    - `kimi_k…

### #12949 — [[v0.23.0][Doc][Misc]Update the materials using the latest template as required](https://github.com/vllm-project/vllm-ascend/pull/12949)
- **作者**: aisong1988  **时间**: 2026-07-27 21:19 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? Update the materials using the latest template as required  ### Does this PR introduce _any_ user-facing change? No, this is a documentation-only update. ### How was this patch tested? documentation changes only.  - vLLM version: v0.23.0 - vLLM main: https://g…

### #12948 — [[BugFix][SpecDecode] Support Gemma4 image token mapping](https://github.com/vllm-project/vllm-ascend/pull/12948)
- **作者**: Alex-stack-hub  **时间**: 2026-07-27 20:45 CST
- **摘要**: ### What this PR does / why we need it?  Adds `Gemma4ForConditionalGeneration` and `Gemma4UnifiedForConditionalGeneration` to the multimodal image token mapping list used by `AscendSpecDecodeBaseProposer.load_model`.  Gemma4 VL configs use `image_token_id`, so Gemma4 EAGLE3 draft loading should foll…

### #12946 — [[CI]Coverage rate upload lacks regex installation and repair.](https://github.com/vllm-project/vllm-ascend/pull/12946)
- **作者**: shiqiangA  **时间**: 2026-07-27 19:52 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it?  When converting coverage data into a map, regex is needed, but the installation is missing.Add regex installation here.  ### Does this PR introduce _any_ user-facing change? no  ### How was this patch tested? After modification, it can correctly generate the …

### #12945 — [[CI][Fix]: summarize CPU unit test failures](https://github.com/vllm-project/vllm-ascend/pull/12945)
- **作者**: Xuyzhen  **时间**: 2026-07-27 19:49 CST
- **标签**: ci/build
- **摘要**: Preserve downloaded artifact directories so CPU logs can be identified independently. Collapse CPU pytest failures into a single cpu-ut result while retaining existing matching for other devices.  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How …

### #12944 — [[BugFix][MRV2] Avoid global synchronization for ACL graph replay](https://github.com/vllm-project/vllm-ascend/pull/12944)
- **作者**: yiz-liu  **时间**: 2026-07-27 19:45 CST
- **标签**: ready
- **摘要**: ### What this PR does / why we need it?  This PR refines the synchronization introduced for the async-scheduling ACL graph hang described in #4233.  The hang occurs when the parameter-update stream records the current iteration event before the previous graph replay has completed. A device-wide sync…

### #12943 — [[CI]Coverage regex fix](https://github.com/vllm-project/vllm-ascend/pull/12943)
- **作者**: shiqiangA  **时间**: 2026-07-27 19:43 CST
- **标签**: ci/build, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.25.1 - vLLM main: https://github.com/vllm-project/vllm/commit/fe784ff22e630a31fd798f392b01e0a75c18f047
