# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-17 10:07 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期的动态摘要：

### 📌 Issue (问题反馈)
近期问题主要集中在**模型兼容性报错**及**高级特性下的生成/精度异常**：
1. **模型启动/适配失败**：多位用户反馈 Qwen3.5-122B-A10B-W4A8、Z-Image-Turbo 及 all-MiniLM-L6-v2 模型在 Ascend 环境下无法正常启动或报错。
2. **生成内容异常（幻视）**：DeepSeek-V4-Flash-0731 (W8A8 + DSpark 推测解码) 在 910B2 上流式输出长文本时，间歇性出现随机或幻觉 Token。
3. **精度波动**：GLM-5.2-W4A8C8 在开启 PD 分离与稀疏 Flash Attention C8 时出现精度不稳定现象。

---

### 🔧 PR (代码合并)
本期 PR 包含**关键 Bug 修复、性能优化、架构重构与新特性支持**：
1. **关键修复**：修复 SFA PD（Prefill-Decode 分离）拉取模式的严重上下文泄漏 Bug——原逻辑导致当前请求会解码出**上一个请求**的上下文内容，现通过将 PD 通知排序在 KV scatter 之后解决（#14380）。
2. **性能优化**：使用 Ascend C 实现自定义算子，大幅加速 SFA DCP 稀疏索引重映射，同时保留 CPU 参考路径（#14373）。
3. **新特性**：引入对 Omni-EPLB 的支持（#14374）。
4. **架构重构**：模块化重构 DeepSeek V4 (dsa_v1) 的 indexer 与 compressor 逻辑（#14375）。
5. **上游适配**：尝试适配 vLLM main 分支（对应 v0.27.1），但当前状态为失败（#14376）。
6. **文档与 CI**：同步 v0.23.0 发布文档及 23 个中文翻译文件；为 Qwen3.8 临时更新 Docker 镜像名；CI 自动更新测试预估时间。

---

### 🚀 Release (版本发布)
**v0.23.0 (发布日期: 2026.07.31)**
- **版本亮点**：vLLM-Ascend v0.23.0 官方正式发布，该版本完全**对齐上游 vLLM v0.23.0**。整合了前期的各项功能迭代、Bug 修复及文档更新，标志着当前阶段的稳定版本成型。

---

## 🐛 Issues

### #14386 — [[Bug]: Qwen3.5-122B-A10B-W4A8模型无法启动](https://github.com/vllm-project/vllm-ascend/issues/14386)
- **作者**: sj6969-dear  **时间**: 2026-08-17 10:00 CST
- **标签**: bug, qwen-3.5, multimodal-understanding
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Your output of above commands here ```  </details>   ### 🐛 Describe the bug  命令如下： export ASCEND_RT_VISIBLE_DEVICES="0,1,2,3,4,5,6,7" export PYTORCH_NPU_ALLOC_CONF="expandable_segments:True" exp…

### #14385 — [[Bug]: Z-Image-Turbo模型适配报错](https://github.com/vllm-project/vllm-ascend/issues/14385)
- **作者**: sj6969-dear  **时间**: 2026-08-17 09:57 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Your output of above commands here ```  </details>   ### 🐛 Describe the bug  启动命令如下： #!/bin/bash export ASCEND_VISIBLE_DEVICES=6 export PYTHONUNBUFFERED=1 export HCCL_LOG_LEVEL=ERROR  vllm serve…

### #14384 — [[Bug]: all-MiniLM-L6-v2模型适配报错](https://github.com/vllm-project/vllm-ascend/issues/14384)
- **作者**: sj6969-dear  **时间**: 2026-08-17 09:50 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary> 启动命令如下： #!/bin/bash export ASCEND_VISIBLE_DEVICES=6 export PYTHONUNBUFFERED=1 export HCCL_LOG_LEVEL=ERROR  vllm serve /models/all-MiniLM-L6-v2 \     --trust-remote-code \     --port 8001 \     --host 0.0…

### #14381 — [[Bug]: Random/hallucinated tokens intermittently corrupt long streaming outputs when serving DeepSeek-V4-Flash-0731 (W8A8 + DSpark spec decoding) on Ascend 910B2](https://github.com/vllm-project/vllm-ascend/issues/14381)
- **作者**: wangyiyong  **时间**: 2026-08-17 09:19 CST
- **标签**: bug, llm-model, deepseek
- **摘要**: ### Your current environment  <details> <summary>Environment summary (<code>collect_env.py</code> output not collected yet)</summary>  ```text Image:           quay.io/ascend/vllm-ascend:DeepSeekV4-flash-0731 vllm-ascend:     0.19.1rc2.dev1265+g50e0ce608 vLLM:            0.25.1+empty torch/torch_npu…

### #14378 — [[Bug][0.23.0]: Accuracy Fluctuations with GLM-5.2-W4A8C8 on Ascend](https://github.com/vllm-project/vllm-ascend/issues/14378)
- **作者**: underfituu  **时间**: 2026-08-17 05:38 CST
- **标签**: bug
- **摘要**: ### Description    We observed accuracy fluctuations when serving `GLM-5.2-W4A8C8` with Prefill-Decode (PD) disaggregation and enabling Sparse Flash Attention C8:    ```bash   --additional-config '{"enable_sparse_sfa_c8": true}'   ```    When GPQA is evaluated repeatedly with the same model, dataset…

## 🔀 Pull Requests

### #14387 — [[Doc][Misc] Update navigation titles](https://github.com/vllm-project/vllm-ascend/pull/14387)
- **作者**: herizhen  **时间**: 2026-08-17 10:04 CST
- **摘要**: <!--  Thanks for sending a pull request!  BEFORE SUBMITTING, PLEASE READ https://docs.vllm.ai/en/latest/contributing/overview.html  --> ### What this PR does / why we need it? This PR updates the navigation titles and page headers to more accurately reflect the supported versions .  ### Does this PR…

### #14383 — [[Doc][Misc] Temporarily use temporary image names for Qwen3.8](https://github.com/vllm-project/vllm-ascend/pull/14383)
- **作者**: AJF-cmd  **时间**: 2026-08-17 09:34 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  This PR temporarily updates the Docker image tags in the Qwen3.8 documentation (`Qwen3.8-2.4T-A95B.md` and `Qwen3.8-27B.md`) to use the temporary image names `qwen3.8-a3` and `qwen3.8-a5` instead of the version placeholders. This is needed because the officia…

### #14382 — [[v0.23.0][Doc] Translated Doc files 2026-08-17](https://github.com/vllm-project/vllm-ascend/pull/14382)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-17 09:27 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **23** file(s):  - <code>docs/source/locale/zh_CN/LC_MESSAGES/community/contributors.po</code> - <code>docs/source/locale/zh_CN/LC_MESSAGES/community/versioning_policy.po</code> - <code>docs/source/locale/zh_CN/LC_MESSAGES/faqs.po</code> - <code>docs/source/lo…

### #14380 — [[BugFix][PD] Order SFA PD notification after the KV scatter](https://github.com/vllm-project/vllm-ascend/pull/14380)
- **作者**: chowhsu  **时间**: 2026-08-17 08:21 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? On the SFA pull-mode PD path, every request after the first decoded the *previous* request's context: its answer reproduced the previous prompt's needle, and often its text verbatim.  P notified D that a layer's blocks were ready before that layer's KV scatter…

### #14379 — [[v0.23.0][Doc][Misc] Sync v0.23.0 release docs to main](https://github.com/vllm-project/vllm-ascend/pull/14379)
- **作者**: yiz-liu  **时间**: 2026-08-17 06:34 CST
- **标签**: documentation, ci/build
- **摘要**: ### What this PR does / why we need it?  Synchronizes the official v0.23.0 release documentation from the release branch to `main` after the final release was published.  - adds the cumulative v0.23.0 release notes already reviewed and merged in #12356; - updates the English and Chinese README files…

### #14376 — [[Misc]feat: adapt to vLLM main (3ac95255)](https://github.com/vllm-project/vllm-ascend/pull/14376)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-17 00:17 CST
- **标签**: module:tests, module:ops
- **摘要**: main2main adaptation failed — no steps completed.  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14375 — [[Refactor][DSA][4/N] modularize the indexer/compressor of dsa_v1](https://github.com/vllm-project/vllm-ascend/pull/14375)
- **作者**: weiguihua2  **时间**: 2026-08-17 00:05 CST
- **标签**: module:tests, module:ops, ready
- **摘要**: ### What this PR does / why we need it? The main focus is on layering the indexer and compressor logic in dsa_v1. 1. Create a new directory named "deepseek_v4" inside the "models" folder, and place all the related files for deepseek_v4 in this directory. The directory structure is as follows: vllm_a…

### #14374 — [Support Omni-EPLB](https://github.com/vllm-project/vllm-ascend/pull/14374)
- **作者**: CConory  **时间**: 2026-08-16 21:26 CST
- **标签**: module:core
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #14373 — [perf: accelerate SFA sparse-index remap with Ascend C](https://github.com/vllm-project/vllm-ascend/pull/14373)
- **作者**: pisceskkk  **时间**: 2026-08-16 18:36 CST
- **标签**: module:tests
- **摘要**: ## What changed  - add an Ascend C implementation of SFA DCP sparse-index remapping - route the SFA CP backend through the custom operator on NPU while retaining the CPU reference path - use a vectorized power-of-two DCP fast path with a scalar fallback for non-power-of-two or dynamic TopK cases - a…

### #14372 — [[CI] Auto-update estimated test times in test_config.yaml](https://github.com/vllm-project/vllm-ascend/pull/14372)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-16 17:54 CST
- **标签**: merge-conflicts
- **摘要**: ## Summary  This PR was auto-generated by the **Update estimated test times** [workflow](https://github.com/vllm-project/vllm-ascend/actions/runs/31936173076).  It updates the `estimated_times` values in `.github/workflows/scripts/test_config.yaml` based on actual elapsed times collected from CI wor…

## 🚀 Releases

### [v0.23.0](https://github.com/vllm-project/vllm-ascend/releases/tag/v0.23.0)
- **作者**: yiz-liu  **时间**: 2026-08-17 06:18 CST
- **摘要**: ## v0.23.0 - 2026.07.31  We're excited to announce the official vLLM Ascend v0.23.0 release, aligned with upstream vLLM v0.23.0. This note summarizes the cumulative user-facing changes since the previous official release, v0.18.0, including the v0.19.1rc1, v0.20.2rc1, v0.21.0rc1, v0.22.1rc1, and v0.…
