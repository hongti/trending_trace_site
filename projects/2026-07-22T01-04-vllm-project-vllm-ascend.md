# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-22 09:04 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
近期 Issue 主要集中在**分布式推理性能异常**和**底层缓存组件 Bug**，同时 CI 自动化流程出现中断：
- **MTP（多Token预测）接收率异常** (#12521, #12516)：在特定镜像版本（B119）及 mooncake 池化（双机1P1D）场景下，MTP 接收率极低（<10%）；在 v0.23.0rc1 的 GLM5.2 1M PD 分离推理测试中，D实例 MTP 接收率甚至降为 0。
- **KV 缓存与乱码 Bug** (#12522)：`AscendStoreConnector`（memcache）在 KV `get` 失败后产生乱码输出，且 `get_num_new_matched_tokens` 缺少对失败加载的追踪。
- **CI 流程中断** (#12524, #12520)：main2main 自动化同步流程未能完成所有计划步骤，触发人工审核要求。

### 🔧 PR 动态
PR 涵盖关键算子 Bug 修复、CI/质量门禁引入及大量文档更新：
- **重要 Bug 修复**：
  - **FusedMoE 权重连续性修复** (#12512, #12518)：修复 Ascend NPU 上因 `transpose` 操作导致权重张量内存不连续，进而引发 `npu_format_cast` 报错的问题。#12518 为该修复向 `v0.24.0rc` 版本的 cherry-pick。
  - **CI eplb 修复** (#12515)：修复 main2main 同步流程中的 eplb 问题。
- **新特性与适配**：
  - **Phase-1 日志质量门禁** (#12517)：引入第一阶段日志质量框架，在 pre-commit 阶段增加增量硬门禁检查工具。
  - **vLLM main 分支适配** (#12519, #12523)：尝试升级适配 vLLM 最新 commit（主要涉及 KV 缓存管理器及 Ascend Store 连接池等），但 #12523 显示适配失败，未完成任何步骤。
- **文档与维护** (#12510, #12511, #12513, #12514)：修复 Qwen3-ASR-1.7B 文档的失效链接；统一 Atlas 310P 下 Qwen3 模型的命名规范；Cherry-pick 更新 `v0.24.0rc` 版本的部署教程模板；按 AIDD 要求修改部分文档。

### 🚀 Release 动态
- 近期**无新版本发布**。但值得注意的是，多个 PR（#12514, #12518）正在向 `releases/v0.24.0rc` 分支进行 cherry-pick 和维护，表明该候选版本正在积极修复文档与核心算子问题，为正式发布做准备。

---

## 🐛 Issues

### #12524 — [[main2main] main2main manual review required (0885b519)](https://github.com/vllm-project/vllm-ascend/issues/12524)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-22 01:43 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/12523 - Commit range: `56a357ed33dec39357344816ccba2732109a96a8`...`0885b51981d09089e7a067276046b04a20e5b59d` - Status: `failed`  ## Final Summary  …

### #12522 — [[Bug]:  AscendStoreConnector (memcache): garbled output after KV `get` failure; `get_num_new_matched_tokens` lacks failed-load tracking](https://github.com/vllm-project/vllm-ascend/issues/12522)
- **作者**: Oven2002  **时间**: 2026-07-21 23:46 CST
- **标签**: bug, triaged
- **摘要**: ### Your current environment  ## Environment  - Image: quay.io/ascend/vllm-ascend:v0.22.1rc1-openeuler (bundles vllm-ascend v0.22.1rc1, vllm v0.22.1, memcache v1.1.2, CANN) - Hardware: NPU A2, HDK 25.5.2 - Model: DeepSeek-V3.2-w8a8-mtp-QuaRot  ### 🐛 Describe the bug  # [Bug] AscendStoreConnector (me…

### #12521 — [[Bug]: mtp acceptance rate < 10% on img version B119](https://github.com/vllm-project/vllm-ascend/issues/12521)
- **作者**: AlanisZomeg  **时间**: 2026-07-21 23:30 CST
- **标签**: bug, advanced-features, mtp/speculative-decode
- **摘要**: <!-- Failed to upload "1-池化采信率.png" --> <!-- Failed to upload "2-去掉池化采信率.png" -->  ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Collecting environment information... PyTorch version: 2.10.0+cpu Is debug build: False  OS: Ubuntu 24.04.4 LTS…

### #12520 — [[main2main] main2main manual review required (1ff94296)](https://github.com/vllm-project/vllm-ascend/issues/12520)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-21 22:15 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/12519 - Commit range: `54503ecec0f3ac31e5ecfc5f28652e4cc42307b5`...`1ff9429655f01c804a7b17412144323e804f1324` - Status: `failed`  ## Final Summary  …

### #12516 — [[Bug]: 0.23.0rc1  GLM5.2 1M场景PD分离 推理测试，D实例mtp接收率为0](https://github.com/vllm-project/vllm-ascend/issues/12516)
- **作者**: Sfeching  **时间**: 2026-07-21 20:21 CST
- **标签**: bug, glm5, triaged, advanced-features, mtp/speculative-decode, llm-model
- **摘要**: ### Your current environment  torch                                    2.10.0+cpu torch_npu                                2.10.0.post2 torchaudio                               2.10.0+cpu torchvision                              0.25.0+cpu transformers                             5.5.4 triton       …

## 🔀 Pull Requests

### #12523 — [[Misc]feat: adapt to vLLM main (0885b519)](https://github.com/vllm-project/vllm-ascend/pull/12523)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-22 01:42 CST
- **摘要**: main2main adaptation failed — no steps completed.  - vLLM version: v0.25.1 - vLLM main: https://github.com/vllm-project/vllm/commit/54503ecec0f3ac31e5ecfc5f28652e4cc42307b5

### #12519 — [[Misc]feat: adapt to vLLM main (1ff94296)](https://github.com/vllm-project/vllm-ascend/pull/12519)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-21 22:14 CST
- **标签**: ready
- **摘要**: ### What this PR does / why we need it?  Upgrade vLLM commit to `1ff94296`  1. Adapt `vllm_ascend/core/single_type_kv_cache_manager.py`, `vllm_ascend/distributed/kv_transfer/kv_pool/ascend_store/pool_scheduler.py`, `vllm_ascend/distributed/kv_transfer/kv_pool/ascend_store/pool_worker.py`, `vllm_asce…

### #12518 — [[v0.24.0rc][BugFix] Ensure contiguous weight tensors after transpose in FusedMoE](https://github.com/vllm-project/vllm-ascend/pull/12518)
- **作者**: zhangxinyuehfad  **时间**: 2026-07-21 21:02 CST
- **标签**: module:tests, module:ops, ready
- **摘要**: ### What this PR does / why we need it?  cherry-pick: https://github.com/vllm-project/vllm-ascend/pull/12494  fix: https://github.com/vllm-project/vllm-ascend/actions/runs/29756582240/job/88436386887  On Ascend NPUs, operators like npu_format_cast require contiguous memory layouts. Non-contiguous te…

### #12517 — [[feature] phase1 log quality gate](https://github.com/vllm-project/vllm-ascend/pull/12517)
- **作者**: xiaoshudian555  **时间**: 2026-07-21 20:46 CST
- **标签**: documentation, module:tests, module:tools
- **摘要**: ### What this PR does / why we need it?  This PR introduces the **Phase-1 log quality framework** for `vllm_ascend/` in two layers:  **1. Incremental hard gate (pre-commit, commit 1)**  - Add `tools/check_log_quality.py`: AST-based checker for `vllm_ascend/**/*.py` logger calls, with **incremental m…

### #12515 — [[Bugfix] Fix main2main eplb](https://github.com/vllm-project/vllm-ascend/pull/12515)
- **作者**: Spicy-Stick  **时间**: 2026-07-21 20:13 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.25.1 - vLLM main: https://github.com/vllm-project/vllm/commit/54503ecec0f3ac31e5ecfc5f28652e4cc42307b5

### #12514 — [[Cherry-pick][releases/v0.24.0rc][Doc][Misc] Update deployment tutorial templates (from #12474)](https://github.com/vllm-project/vllm-ascend/pull/12514)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-21 19:58 CST
- **标签**: documentation
- **摘要**: Cherry-pick of PR #12474 onto `releases/v0.24.0rc`.  Original PR: #12474 Original author: @herizhen  --- ### What this PR does / why we need it? This PR updates the model deployment tutorial templates and various model documentation files to use Jinja-style placeholders (`{{ vllm_ascend_version }}`)…

### #12513 — [[Doc][Misc]Unify the naming of Qwen3-30B-A3B and Qwen3-ASR-310 for Atlas 310P](https://github.com/vllm-project/vllm-ascend/pull/12513)
- **作者**: liuhanhui  **时间**: 2026-07-21 19:49 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  This PR unifies the naming of Qwen3-30B-A3B and Qwen3-ASR-310 for Atlas 310P (Atlas inference products) in the documentation, updating the guides for Qwen3-30B-A3B and Qwen3-ASR-1.7B to include Atlas inference products deployment details, Docker commands, and…

### #12512 — [[BugFix] Ensure contiguous weight tensors after transpose in FusedMoE](https://github.com/vllm-project/vllm-ascend/pull/12512)
- **作者**: zhangxinyuehfad  **时间**: 2026-07-21 19:00 CST
- **标签**: module:tests, module:ops, ready
- **摘要**: ### What this PR does / why we need it? fix: https://github.com/vllm-project/vllm-ascend/actions/runs/29756582240/job/88436386887  On Ascend NPUs, operators like npu_format_cast require contiguous memory layouts. Non-contiguous tensors from .transpose(1,2) caused runtime failures in nightly tests.  …

### #12511 — [[Doc] fix link failure issue in Qwen3-ASR-1.7B.md](https://github.com/vllm-project/vllm-ascend/pull/12511)
- **作者**: MrZ20  **时间**: 2026-07-21 18:19 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? fix:  ``` WARNING -  Doc file 'tutorials/models/Qwen3-ASR-1.7B.md' contains a link '../hardwares/310p.md',  but the target 'tutorials/hardwares/310p.md' is not found among documentation files. ```  ### Does this PR introduce _any_ user-facing change?  ### How …

### #12510 — [[Doc] According to the modifications by AIDD](https://github.com/vllm-project/vllm-ascend/pull/12510)
- **作者**: axx-ty911  **时间**: 2026-07-21 18:01 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? Made some modifications according to AIDD.  ### Does this PR introduce _any_ user-facing change? No.  ### How was this patch tested? Not involved.  - vLLM version: v0.25.1 - vLLM main: https://github.com/vllm-project/vllm/commit/54503ecec0f3ac31e5ecfc5f28652e4…
