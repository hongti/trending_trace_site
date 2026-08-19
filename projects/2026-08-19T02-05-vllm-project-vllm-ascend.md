# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-19 10:05 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 最近动态的简洁摘要：

### 🐛 Issue 动态
*   **#14518 main2main 自动化流程中断**：CI 机器人在执行 main2main 同步时未能完成所有计划步骤，需人工审查 Draft PR #14517。

### 🔀 PR 动态

**🚀 新特性与适配**
*   **#14524 DFlash2 Spec Decode 架构**：为 DFlash drafter 引入独立架构（DFlash2DraftModel），新增 local convolution 与 candidate selector 支持。
*   **#14517 适配 vLLM Main 分支**：将 vllm-ascend 适配至 8 月 18 日的 vLLM 上游最新提交。
*   **#14514 DeepSeek V4 默认思考设置**：为 `dsv4` 模型的 `ChatCompletionRequest` 增加默认 thinking 行为及预验证补丁。

**🛠️ Bug 修复与性能优化**
*   **#14521 修复 DeepSeek V4 工具调用流式输出**：解决长字符串 tool argument 在流式输出时被缓冲的问题，优化解析器避免不必要的转换。
*   **#14519 修复 MegaMoe Prefill 缓冲区大小计算**：适配 CANN 9.1，使 v0.25 fused-MC2 路径选用 CANN MegaMoe 替代旧版 `dispatch_ffn_combine` 实现。
*   **#14516 修复 A5 构建链接问题**：移除旧版自定义 ACLNN 链接配置中对单体 `libopapi.so` 的直接依赖，避免 A5 上的链接错误。
*   **#14515 性能优化：避免有效 token 计数拷贝的 dtype 转换**：作为 #10205 的后续，避免 MTP 路径中 `valid_sampled_token_count_cpu` 的隐式数据类型转换，提升性能。

**📄 文档与 CI**
*   **#14522 补充 DeepSeek V4 CI 数据**：为 `test_deepseek_v4.py` 补充缺失的 MTP acceptance rate golden 基准数据。
*   **#14523 文档翻译**：自动翻译 16 个文档文件至中文社区目录。
*   **#14520 更新 Qwen3.8-27B Cookbook**：新增 950PR 配置指南及 v0.27.1 版本的基准测试结果。

### 🚀 Release 动态
*   近期暂无新版本发布。

---

## 🐛 Issues

### #14518 — [[main2main] main2main manual review required (d29dc3ab)](https://github.com/vllm-project/vllm-ascend/issues/14518)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-19 03:59 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/14517 - Commit range: `1d2d83a07fd3180068917a031c28dcc83141d0be`...`d29dc3ab87840aef42129b30825176295ea73b07` - Status: `failed`  ## Final Summary  …

## 🔀 Pull Requests

### #14524 — [[wip][Feature]DFlash2 Spec Decode: local convolution + candidate selector](https://github.com/vllm-project/vllm-ascend/pull/14524)
- **作者**: HF-001  **时间**: 2026-08-19 10:01 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Two additions to the DFlash drafter, carried by a separate architecture: a checkpoint declaring DFlash2DraftModel gets them, and every existing DFlashDraftModel checkpoint resolves to the class it resolves to today, untouched by this PR.  Grouped dynamic depth…

### #14523 — [[Doc] Translated Doc files 2026-08-19](https://github.com/vllm-project/vllm-ascend/pull/14523)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-19 09:57 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **16** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/contributors.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/governance.po</code> - <…

### #14522 — [[CI] Add DeepSeek V4 MTP acceptance rate golden](https://github.com/vllm-project/vllm-ascend/pull/14522)
- **作者**: jiajinzhu2  **时间**: 2026-08-19 09:46 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? ` test_deepseek_v4.py` missed acceptance rate golden.  ### Does this PR introduce _any_ user-facing change? N/A ### How was this patch tested? Existing CI tests all pass. - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e…

### #14521 — [[BugFix][Frontend] Stream DeepSeek V4 tool string arguments](https://github.com/vllm-project/vllm-ascend/pull/14521)
- **作者**: QwertyJack  **时间**: 2026-08-19 09:34 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  DeepSeek V4 currently buffers the body of a long string tool argument until `</parameter>` arrives. The parser's structural-character optimization skips conversion for ordinary parameter-body deltas, so clients receive one large arguments fragment instead of …

### #14520 — [[Doc] Update Qwen3.8-27B cookbook with 950PR](https://github.com/vllm-project/vllm-ascend/pull/14520)
- **作者**: yiminghub2024  **时间**: 2026-08-19 09:32 CST
- **标签**: documentation
- **摘要**: Update Qwen3.8-27B cookbook with 950PR configguide and benchmark results it tested on 950PR  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14519 — [[v0.25.1rc][BugFix][MoE] Fix MegaMoe prefill buffer sizing](https://github.com/vllm-project/vllm-ascend/pull/14519)
- **作者**: QwertyJack  **时间**: 2026-08-19 07:13 CST
- **标签**: module:tests, module:ops, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  CANN 9.1 makes `cann_ops_transformer` available, so the v0.25 fused-MC2 path can select CANN MegaMoe instead of the legacy `dispatch_ffn_combine` implementation. When `enable_prefill_mc2` is disabled, MegaMoe was still selected for eager prefill, but its symm…

### #14517 — [[Misc]feat: adapt to vLLM main (d29dc3ab)](https://github.com/vllm-project/vllm-ascend/pull/14517)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-19 03:58 CST
- **标签**: module:tests, module:ops, ready
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to August 18.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | — | [3ac95255](https://github.com/vllm-project/vllm/commit/3ac95255) | —…

### #14516 — [fix(csrc): avoid monolithic opapi link on A5](https://github.com/vllm-project/vllm-ascend/pull/14516)
- **作者**: maoxx241  **时间**: 2026-08-19 02:14 CST
- **摘要**: ### What this PR does / why we need it?  This PR updates the legacy custom ACLNN link configuration on `releases/v0.18.0` for A5 builds:  - removes the direct dependency on the monolithic `libopapi.so` - adds `-Wl,-Bsymbolic` so references inside `libcust_opapi.so` bind to the custom implementation …

### #14515 — [[BugFix][Performance] Avoid dtype-converting valid token count copies](https://github.com/vllm-project/vllm-ascend/pull/14515)
- **作者**: Republix-ch  **时间**: 2026-08-18 22:20 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This is a follow-up to #10205.  #10205 changed `valid_sampled_token_count_cpu` to `torch.int64` so the MTP path would no longer perform an implicit dtype conversion during the asynchronous D2H `copy_`. However, the shared copy helper also receives `torch.int3…

### #14514 — [feat(api): default dsv4 thinking settings](https://github.com/vllm-project/vllm-ascend/pull/14514)
- **作者**: wjc129  **时间**: 2026-08-18 22:08 CST
- **标签**: documentation, module:tests
- **摘要**: ## What this PR does / why we need it?  This PR defines the default thinking behavior for requests using the served model name `"dsv4"`.  A pre-validation patch is added to `ChatCompletionRequest`. When a request uses `"model": "dsv4"` and omits the DeepSeek thinking controls, the following defaults…
