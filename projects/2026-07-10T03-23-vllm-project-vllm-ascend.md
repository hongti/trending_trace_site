# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-10 11:23 CST

## AI 总结

# vllm-ascend 仓库近期动态摘要

---

## 🐛 Issue

| 编号 | 要点 |
|------|------|
| **#11788** | **GLM5.1 w4a8 + MTP 概率性 Bug**：调用模型 API 时，`tool_calls` 返回数组中 `id` 缺失、`function.arguments` 为空字符串，`finish_reason` 为 `tool_calls`。环境：910C 560T、vllm==0.20.2、vllm-ascend==0.20.2rc1 |
| **#11784** | **main2main 自动化流程中断**：CI 自动同步未完成所有计划步骤，需要人工审核 |
| **#11782** | **check-symbolic-meta 预提交钩子报错**：`npu_kv_quant_sparse_flash_attention_meta` 中存在 14 处违规，符号形状检查未通过 |

---

## 🔀 Pull Request

### 🚀 新特性 / 模型支持
- **#11791** — 新增 **Gemma4 E2B 与 E4B** 模型支持
- **#11785** — 引入 **SFA packed KV cache** 功能（源自 PR #11443）
- **#11786** — 新增 **A3 多机 PD 分离**测试用例

### 🔧 Bug 修复
- **#11790** — 修复 **fused_infer_attention 算子 contiguous 错误**
- **#11781 / #11780** — 两份 PR 共同修复 **npu_kv_quant_sparse_flash_attention_meta 符号形状问题**：将 `.size(i)` 改为 `.sym_size(i)`，使用 `c10::SymDimVector` 和 `at::empty_symint` 保留符号维度，解决 #11782 所报告的预提交钩子违规

### 📝 文档 / 测试 / 杂项
- **#11787** — 修正 `dflash` 的文档描述
- **#11789** — 调整 Qwen3.5-397B-A17B-w4a8-mtp-A2 性能基线
- **#11783 / #11779** — **两次 vLLM upstream 适配同步**：#11779 完成 6/6 步骤（cc1d020d）；#11783 进行 24/30 步骤（a5d19cbb），涉及上游删除 `deepseekv4_tool_parser` 等变更，需人工介入完成剩余步骤

---

## 📦 Release

> **本期无新版本发布记录。**

---

**整体趋势**：本期重点围绕 **符号形状（symbolic shapes）修复**（多个 PR 联动解决同一 Issue）、**新模型接入**（Gemma4）以及 **vLLM 上游持续同步适配**。GLM5.1 w4a8 的工具调用 Bug 是当前待解决的用户侧痛点。

---

## 🐛 Issues

### #11788 — [[Bug]: GLM5.1 w4a8，MTP，概率性出现，调用模型API接口时出现返回的tool_calls 数组中,id不存在、function.arguments 为空字符串；finish_reason为tool_calls](https://github.com/vllm-project/vllm-ascend/issues/11788)
- **作者**: lml006  **时间**: 2026-07-10 10:40 CST
- **标签**: bug, glm5, advanced-features, mtp/speculative-decode, llm-model
- **摘要**: ### Your current environment  模型GLM5.1 w4a8 机器910C 560T  vllm==0.20.2、vllm-ascend==0.20.2rc1   ### 🐛 Describe the bug  部署参数如下 p： vllm serve \   --model /mnt/paas/kubernetes/kubelet/GLM-5.1-w4a8 \   --trust-remote-code \   --host 10.37.138.16 \   --port 1025 \   --data-parallel-size 1 \   --data-para…

### #11784 — [main2main manual review required (a5d19cbb)](https://github.com/vllm-project/vllm-ascend/issues/11784)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-10 09:59 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/11783 - Commit range: `1f486d96a17303ce8db8e02be39545b2be338446`...`a5d19cbb95872c4b426c06735733568542fa33db` - Status: `failed`  ## Final Summary  …

### #11782 — [[Bug]: check-symbolic-meta hook fails on npu_kv_quant_sparse_flash_attention_meta](https://github.com/vllm-project/vllm-ascend/issues/11782)
- **作者**: gygdh-001  **时间**: 2026-07-10 09:54 CST
- **标签**: bug
- **摘要**: ### Your current environment  ## Description  The `check-symbolic-meta` pre-commit hook (`tools/check_symbolic_meta.py`) reports 14 violations in `csrc/torch_binding_meta.cpp` in the `npu_kv_quant_sparse_flash_attention_meta` function.  ## Steps to Reproduce  ``` python tools/check_symbolic_meta.py …

## 🔀 Pull Requests

### #11791 — [Gemma4  E2B and E4B](https://github.com/vllm-project/vllm-ascend/pull/11791)
- **作者**: 0moyi0-2024  **时间**: 2026-07-10 10:58 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #11790 — [[BugFix]Fix fused_infer_attention ops contiguous err](https://github.com/vllm-project/vllm-ascend/pull/11790)
- **作者**: hust17yixuan  **时间**: 2026-07-10 10:48 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change? No  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #11789 — [[Test] Modify perf baseline of Qwen3.5-397B-A17B-w4a8-mtp-A2.yaml](https://github.com/vllm-project/vllm-ascend/pull/11789)
- **作者**: Wangbei25  **时间**: 2026-07-10 10:44 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Modify perf baseline of Qwen3.5-397B-A17B-w4a8-mtp-A2.yaml ### Does this PR introduce _any_ user-facing change? None ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545…

### #11787 — [[Doc] Correct the interpretation of `dflash`](https://github.com/vllm-project/vllm-ascend/pull/11787)
- **作者**: drslark  **时间**: 2026-07-10 10:37 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? Current interpretation of `dflash` is inaccurate. This pr corrects it.  ### Does this PR introduce _any_ user-facing change?  N/A  ### How was this patch tested?  N/A  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17…

### #11786 — [[TEST]Add A3 multi-machine PD separation test case](https://github.com/vllm-project/vllm-ascend/pull/11786)
- **作者**: taozi01319  **时间**: 2026-07-10 10:25 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Add A3 multi-machine PD separation test case ### Does this PR introduce _any_ user-facing change? no ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #11785 — [Resolve/pr 11443 sfa packed kv cache](https://github.com/vllm-project/vllm-ascend/pull/11785)
- **作者**: ZYang6263  **时间**: 2026-07-10 10:12 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #11783 — [[Misc]feat: adapt to vLLM main (a5d19cbb)](https://github.com/vllm-project/vllm-ascend/pull/11783)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-10 09:58 CST
- **摘要**: ### What this PR does / why we need it?  vllm upstream `1f486d96...a5d19cbb` (24/30 steps).  #### vllm_ascend/patch/platform/__init__.py  - Cause: Upstream deleted vllm/tool_parsers/deepseekv4_tool_parser.py (DeepSeekV4ToolParser), replacing it with new ParserEngine-based parsers (DeepSeekV4EngineTo…

### #11781 — [[BugFix] Fix symbolic shapes in npu_kv_quant_sparse_flash_attention_meta](https://github.com/vllm-project/vllm-ascend/pull/11781)
- **作者**: gygdh-001  **时间**: 2026-07-10 09:50 CST
- **标签**: merge-conflicts
- **摘要**: ## Description  Fixes the `check-symbolic-meta` pre-commit hook violations in `npu_kv_quant_sparse_flash_attention_meta`.  Fixes #11782  ## Changes  - `.size(i)` → `.sym_size(i)` for tensor-derived output dimensions - `at::SmallVector<int64_t, N>` → `c10::SymDimVector` for shape vectors - `at::empty…

### #11780 — [[Bugfix] Preserve symbolic shapes in kv quant sparse FA meta](https://github.com/vllm-project/vllm-ascend/pull/11780)
- **作者**: anning-2026  **时间**: 2026-07-10 08:55 CST
- **标签**: merge-conflicts
- **摘要**: ## Summary - preserve symbolic tensor dimensions in npu_kv_quant_sparse_flash_attention_meta - use c10::SymDimVector and at::empty_symint for meta outputs - avoid concrete shape handling flagged by check-symbolic-meta  ## Validation - python tools/check_symbolic_meta.py - git diff --check - vLLM ver…

### #11779 — [[Misc]feat: adapt to vLLM main (cc1d020d)](https://github.com/vllm-project/vllm-ascend/pull/11779)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-10 03:08 CST
- **摘要**: ### What this PR does / why we need it?  vllm upstream `1f486d96...cc1d020d` (6/6 steps).  #### vllm_ascend/patch/platform/__init__.py  - Cause: Upstream deleted vllm.tool_parsers.deepseekv4_tool_parser.DeepSeekV4ToolParser (replaced by engine-based DeepSeekV4EngineToolParser); the Ascend patch impo…
