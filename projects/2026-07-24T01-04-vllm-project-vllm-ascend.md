# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-24 09:04 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🔄 Issue 动态
- **#12755 main2main 自动化流程异常**：main2main 同步自动化流程在完成所有计划步骤前中断，目前需人工介入审查（关联 Draft PR #12754 及特定提交范围）。

### 🛠 PR 动态
**1. 核心架构与上游适配**
- **#12754 / #12753 / #12752 [Misc] 适配 vLLM 主分支**：连续3个 PR 将 vllm-ascend 代码与 vLLM 上游主分支最新提交（截至7月12日-13日）进行同步适配，保持项目与上游演进一致。

**2. 性能优化与 Bug 修复**
- **#12749 [Performance][KV Pool] 远距多缓冲 API 升级**：更新 AscendStore 的远距后端，采用同步多缓冲 APIs（`mget_h2d_from_multi_b`）并支持可配置超时，提升 KV Pool 性能。
- **#12748 [BugFix] 修复 SP/DP 精度问题**：修复了在序列并行和数据并行场景下出现的精度异常问题。

**3. 新特性与功能支持**
- **#12747 / #12746 [Ops][Feature] 支持 Motor 同构 P/D 架构**：适配 hixlep endpoint 选择功能，为 Motor colocated P/D（Prefill/Decode 同构）架构提供支持。

**4. CI、安全与文档维护**
- **#12751 [CI] 提升 CI 可复现性**：将 GitHub Action 依赖 `dorny/paths-filter` 从浮动的 v4 标签锁定至特定版本 v4.0.2，避免 CI 因上游更新而意外中断。
- **#12750 新增安全漏洞报告政策**：为仓库新增 `SECURITY.md` 文件，基于上游 vllm 模板建立规范的安全漏洞披露与报告渠道。
- **#12745 [Cherry-pick] 统一 Qwen3-30B-A3B 命名**：将 Qwen3-30B-A3B 模型在 Atlas 310P3 上的文档指引及命名规范 Cherry-pick 合入 `releases/v0.24.0rc` 分支。

### 🚀 Release 动态
- **本期暂无新版本发布信息**。

---

## 🐛 Issues

### #12755 — [[main2main] main2main manual review required (9427c453)](https://github.com/vllm-project/vllm-ascend/issues/12755)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-24 05:19 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/12754 - Commit range: `27c3e579f0e5f345a86e512e26e1231d3689931f`...`9427c453863f3ab9e720748f04b9d6dd404ef602` - Status: `failed`  ## Final Summary  …

## 🔀 Pull Requests

### #12754 — [[Misc]feat: adapt to vLLM main (9427c453)](https://github.com/vllm-project/vllm-ascend/pull/12754)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-24 05:19 CST
- **标签**: ready
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to July 13.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | `vllm_ascend/core/single_type_kv_cache_manager.py`<br>`vllm_ascend/distrib…

### #12753 — [[Misc]feat: adapt to vLLM main (27c3e579)](https://github.com/vllm-project/vllm-ascend/pull/12753)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-24 02:13 CST
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to July 12.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------|  - vLLM version: v0.25.1 - vLLM main: https://github.com/vllm-project/vllm/c…

### #12752 — [[Misc]feat: adapt to vLLM main (27c3e579)](https://github.com/vllm-project/vllm-ascend/pull/12752)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-24 01:19 CST
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to July 12.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | `vllm_ascend/core/single_type_kv_cache_manager.py`<br>`vllm_ascend/distrib…

### #12751 — [[CI] Pin dorny/paths-filter to v4.0.2 for reproducibility](https://github.com/vllm-project/vllm-ascend/pull/12751)
- **作者**: arijitroy003  **时间**: 2026-07-23 23:08 CST
- **标签**: ci/build
- **摘要**: ## Summary  Pins dorny/paths-filter from floating @v4 tag to specific @v4.0.2 release for CI reproducibility.  ## Changes  - Updated dorny/paths-filter from @v4 to @v4.0.2 in .github/workflows/pr_test.yaml (2 instances) - Updated dorny/paths-filter from @v4 to @v4.0.2 in .github/workflows/_selected_…

### #12750 — [Add SECURITY.md with vulnerability reporting policy](https://github.com/vllm-project/vllm-ascend/pull/12750)
- **作者**: arijitroy003  **时间**: 2026-07-23 23:08 CST
- **标签**: documentation
- **摘要**: ## Summary  Adds SECURITY.md for responsible vulnerability disclosure. 2.5k-star repo currently has no security reporting path.  ## Changes  - Add SECURITY.md based on upstream vllm-project/vllm template - Adapted references to vllm-ascend repository and maintainers - Includes severity classificatio…

### #12749 — [[Performance][KV Pool] Use Yuanrong multi-buffer APIs with configurable timeouts](https://github.com/vllm-project/vllm-ascend/pull/12749)
- **作者**: yangsonglin13  **时间**: 2026-07-23 22:05 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  This PR updates AscendStore's Yuanrong backend to use the synchronous multi-buffer APIs introduced by openyuanrong-datasystem PR #1412:  - `mget_h2d_from_multi_buffers` - `mset_d2h_from_multi_buffers` - `batch_is_exist`  The connector already owns per-key dev…

### #12748 — [[BugFix] accuracy issue under SP and DP](https://github.com/vllm-project/vllm-ascend/pull/12748)
- **作者**: jiaqi-lee  **时间**: 2026-07-23 21:58 CST
- **标签**: module:ops
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.25.1 - vLLM main: https://github.com/vllm-project/vllm/commit/54503ecec0f3ac31e5ecfc5f28652e4cc42307b5

### #12747 — [[Ops][Feature] adapt hixlep endpoint selection for Motor colocated P/D](https://github.com/vllm-project/vllm-ascend/pull/12747)
- **作者**: 1478931959  **时间**: 2026-07-23 21:44 CST
- **标签**: documentation, module:core
- **摘要**: Suggested PR Title:  ```markdown [Ops][Feature] adapt hixlep endpoint selection for Motor colocated P/D ```  Suggested PR Summary:  ```markdown ### What this PR does / why we need it?  This is an adaptation for the MindIE Motor / K8s scenario where Prefill and Decode are colocated on one node but ea…

### #12746 — [[Ops][Feature] adapt hixlep endpoint selection for Motor colocated P/D](https://github.com/vllm-project/vllm-ascend/pull/12746)
- **作者**: 1478931959  **时间**: 2026-07-23 21:44 CST
- **标签**: documentation, module:core
- **摘要**: Suggested PR Title:  ```markdown [Ops][Feature] adapt hixlep endpoint selection for Motor colocated P/D ```  Suggested PR Summary:  ```markdown ### What this PR does / why we need it?  This is an adaptation for the MindIE Motor / K8s scenario where Prefill and Decode are colocated on one node but ea…

### #12745 — [[Cherry-pick][releases/v0.24.0rc][Doc][Misc] Unify the naming of Qwen3-30B-A3B for Atlas 310P (from #12322)](https://github.com/vllm-project/vllm-ascend/pull/12745)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-23 21:28 CST
- **标签**: documentation
- **摘要**: Cherry-pick of PR #12322 onto `releases/v0.24.0rc`.  Original PR: #12322 Original author: @liuhanhui  --- ### What this PR does / why we need it? add 310P3 guidance of  Qwen3-30B-A3B model, refresh  Qwen3-30B-A3B.md in the docs/source/tutorials/ ### Does this PR introduce _any_ user-facing change?  …
