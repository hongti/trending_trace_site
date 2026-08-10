# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-10 10:43 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📌 Issue 动态
*   **文档反馈**：DeepSeek-V4-Flash 教程文档中提供了参考参数，但缺少对应的参考性能数据，希望补充 (#13885)。
*   **CI 异常**：`main2main` 自动化流程在完成所有计划步骤前中断，需人工介入审查 (#13884)。

---

### 🛠️ PR 动态

**🚀 新特性与模型支持**
*   **支持 DeepSeek-V4 DSpark**：新增 DeepSeek-V4 DSpark 模型及针对 A3 机器的 FloatQuantization 量化方法 (#13892)。
*   **新增 Kimi K2 测试**：为 Kimi K2 thinking 模型添加测试用例 (基于 v0.24.0) (#13889)。

**🔧 修复与 CI**
*   **修复 GLM5.2 DSpark CI**：因 PR #13530 与 #13216 产生冲突导致 `confidence_head` 张量异常，此 PR 修复了该问题 (#13890)。
*   **修复周测配置**：修正了每周测试用例的配置问题 (#13881)。
*   **适配上游 vLLM 主分支**：尝试适配 vLLM main (83ad767e)，但自动化适配失败，未完成任何步骤 (#13883)。

**♻️ 重构与代码优化**
*   **Fused MoE 模块重构**（关联 #13220）：
    *   将 Fused MoE 运行时的参数和契约拆分为独立的 dataclass 模块，使 prepare、dispatch、MLP 和 quant 契约各自独立成文件 (#13886, #13888)。
*   **对齐上游路由重放**：将路由重放捕获与上游 vLLM 对齐，移除了 Ascend 特有的 `model-runner` 覆盖和 `AscendRoutedExperts` 中的手动钩子 (#13882)。
*   **清理无用补丁**：移除了已被上游 vLLM 修复的无用 patch 代码 (#13887)。

**📖 文档**
*   **文档中文化**：自动翻译了 51 个文档文件至中文 (zh_CN) (#13891)。

---

### 🚀 Release 动态
*   近期无新版本发布。

---

## 🐛 Issues

### #13885 — [[Doc]: Feedback for `/zh-cn/latest/tutorials/models/DeepSeek-V4-Flash.html`](https://github.com/vllm-project/vllm-ascend/issues/13885)
- **作者**: zfxSteven  **时间**: 2026-08-10 09:23 CST
- **标签**: documentation, llm-model, deepseek
- **摘要**: ### 📚 The doc issue  https://docs.vllm.ai/projects/ascend/zh-cn/latest/tutorials/models/DeepSeek-V4-Flash.html#91 给的参考参数，有没有对应的参考性能呢？   ### Suggest a potential alternative/fix  _No response_

### #13884 — [[main2main] main2main manual review required ()](https://github.com/vllm-project/vllm-ascend/issues/13884)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-10 01:36 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/13883 - Commit range: `073c510c916f385315a5366173c883781762bb9e`...`` - Status: `failed`  ## Final Summary  main2main adaptation failed — no steps c…

## 🔀 Pull Requests

### #13892 — [feat: add DeepSeek-V4 DSpark model and FloatQuantization method for A3 mechine](https://github.com/vllm-project/vllm-ascend/pull/13892)
- **作者**: QiGS  **时间**: 2026-08-10 10:37 CST
- **标签**: module:quantization
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #13891 — [[Doc] Translated Doc files 2026-08-10](https://github.com/vllm-project/vllm-ascend/pull/13891)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-10 10:37 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **51** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/versioning_policy.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Docume…

### #13890 — [[BugFix][CI] Fix GLM5.2 DSpark CI](https://github.com/vllm-project/vllm-ascend/pull/13890)
- **作者**: wangbj127  **时间**: 2026-08-10 10:37 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it? https://github.com/vllm-project/vllm-ascend/pull/13530 conflicts with https://github.com/vllm-project/vllm-ascend/pull/13216, and causes `confidence_head` tensors missing in `load_weights()`.  ### Does this PR introduce _any_ user-facing change? No.  ### How w…

### #13889 — [[test] test for kimi k2 thinking(v0.24.0)](https://github.com/vllm-project/vllm-ascend/pull/13889)
- **作者**: aipaes  **时间**: 2026-08-10 10:23 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.24.0 - vLLM main: https://github.com/vllm-project/vllm/commit/85c09e9885e346ea1612da30ebff5a75f67d2350

### #13888 — [[Refactor][Ops] Split fused MoE runtime dataclasses](https://github.com/vllm-project/vllm-ascend/pull/13888)
- **作者**: weijinqian0  **时间**: 2026-08-10 10:10 CST
- **标签**: module:tests, module:ops, module:quantization
- **摘要**: ### What this PR does / why we need it? Come from : https://github.com/vllm-project/vllm-ascend/issues/13220 Refactor the Ascend fused MoE runtime argument layer by moving the fused-expert inputs, router inputs, token-dispatch metadata, MLP compute inputs, and prepare/finalize outputs into a dedicat…

### #13887 — [[Misc] Remove useless patch](https://github.com/vllm-project/vllm-ascend/pull/13887)
- **作者**: wangxiyuan  **时间**: 2026-08-10 10:06 CST
- **标签**: module:tests, ready
- **摘要**: Clean up useless patch which has been fixed by vllm already.   - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/0351e9aa1fdf1a51329d1906881528dfe61fc88e

### #13886 — [[Refactor][Misc] Split fused MoE runtime contracts into dataclass modules](https://github.com/vllm-project/vllm-ascend/pull/13886)
- **作者**: weijinqian0  **时间**: 2026-08-10 09:47 CST
- **标签**: module:tests, module:ops, module:core, module:quantization, merge-conflicts
- **摘要**: ## What this PR does / why we need it?  Split the fused MoE runtime payloads into dedicated dataclass modules so the prepare, dispatch, MLP, and quant contracts each live in their owning file instead of a single monolithic runtime-args module.  This keeps the `token_dispatcher` facade backward compa…

### #13883 — [[Misc]feat: adapt to vLLM main (83ad767e)](https://github.com/vllm-project/vllm-ascend/pull/13883)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-10 01:35 CST
- **标签**: module:tests, module:ops
- **摘要**: main2main adaptation failed — no steps completed.  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/0351e9aa1fdf1a51329d1906881528dfe61fc88e

### #13882 — [[Refactor][Ops] Align routing replay capture with upstream](https://github.com/vllm-project/vllm-ascend/pull/13882)
- **作者**: zjchenn  **时间**: 2026-08-09 23:29 CST
- **标签**: module:tests, module:ops, ready
- **摘要**: ### What this PR does / why we need it?  Related to #13220.  Routing replay capture was still wired through an Ascend-specific model-runner override and a manual hook in `AscendRoutedExperts`. The override replaced the upstream binder on the vLLM 0.26 path, while the manual hook ran after Ascend `lo…

### #13881 — [[Test] Fix the weekly test case configuration](https://github.com/vllm-project/vllm-ascend/pull/13881)
- **作者**: chen-commits  **时间**: 2026-08-09 20:52 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389
