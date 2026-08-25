# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-25 10:06 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue (问题反馈)
本期主要报告了几个核心功能的崩溃和兼容性 Bug，以及 CI 流程中断问题：
1. **投机解码与调度器冲突崩溃**：在 PD 分离部署中，Decode 节点开启 `recompute_scheduler_enable=true` 并结合投机解码（`deepseek_mtp`）时，V1/V2 运行器均会因 `Request` 缺少 `async_tokens_to_discard` 属性而崩溃 (#14871)。
2. **W4A8 量化通信 Bug**：W4A8 FUSED_MC2 CombineV2 混淆了 packed-row 和 token-row 的容量单位，导致计算错误 (#14869)。
3. **上游接口不兼容**：Anthropic 接口测试中，`AsyncMessages.create()` 出现了意外的 `temperature` 参数报错 (#14870)。
4. **CI 自动化中断**：`main2main` 自动同步流程两次中断，需要人工审查介入 (#14868, #14866)。

---

### 🔀 Pull Request (代码合并)
本期 PR 聚焦于关键 Bug 修复、性能优化以及对 vLLM 上游的紧密跟进：
1. **关键 Bug 修复**：
   - **MC2 通信逻辑修复**：修正了 `dispatch_v2` 的 tokens 容量计算，使其遵循标准 MC2 分支逻辑 (#14864)。
   - **310P 硬件兼容修复**：针对 310P 设备，为 MoE aclgraph 引入了 router 权重预转换机制，避免了运行时不支持的 Cast 操作报错 (#14863)。
2. **性能优化**：
   - **MRoPE 缓存优化**：针对 Qwen2.5-Omni 等不在 NPU 白名单内的 MRoPE 模型，实现了按步缓存 cos/sin 值，提升了推理性能 (#14858)。
3. **上游适配与补丁**：
   - **紧密跟进上游**：连续两次提交 PR 适配 vLLM 主分支最新代码（截至8月22日及24日），并更新了 CI 基线锚点 (#14867, #14865, #14872)。
   - **Triton 补丁**：添加了 Triton ops 补丁，防止上游代码变更导致运行失败 (#14862)。
   - **Graph 重构**：进行了 Graph 重构相关工作 (#14874)。
4. **CI 与文档**：
   - **CI 门禁升级**：引入分层治理模型，增加了 Nightly 核心测试套件门禁及 L1 熔断机制 (#14861)。
   - **文档翻译**：自动翻译了 63 个文档文件至中文 (#14873)。

---

### 🚀 Release (版本发布)
本期暂无新版本发布。

---

## 🐛 Issues

### #14871 — [[Bug]: recompute_scheduler + speculative decoding crashes with AttributeError: 'Request' object has no attribute 'async_tokens_to_discard' (both V1 and V2 runners)](https://github.com/vllm-project/vllm-ascend/issues/14871)
- **作者**: xuchi-0808  **时间**: 2026-08-25 09:55 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text PyTorch version: 2.10.0+cpu OS: Ubuntu 24.04.4 LTS (aarch64) Python version: 3.12.13 CPU: Kunpeng 920 7280Z (aarch64, 640 CPUs) [pip3] torch_npu==2.10.0.post4 vLLM Version: 0.27.1 vLLM Ascend Ve…

### #14870 — [[Bug][Upstream]: TypeError: AsyncMessages.create() got an unexpected keyword argument 'temperature'](https://github.com/vllm-project/vllm-ascend/issues/14870)
- **作者**: jiangyunfan1  **时间**: 2026-08-25 09:50 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Your output of above commands here ```  </details>   ### 🐛 Describe the bug  tests/entrypoints/anthropic/test_messages.py https://github.com/vllm-project/vllm-ascend/actions/runs/32628148725/job…

### #14869 — [[Bug]: W4A8 FUSED_MC2 CombineV2 mixes packed-row and token-row capacity units](https://github.com/vllm-project/vllm-ascend/issues/14869)
- **作者**: QwertyJack  **时间**: 2026-08-25 08:36 CST
- **标签**: bug, custom-op
- **摘要**: ### Environment  ```text Hardware: 2 Ascend A3 hosts, 16 visible NPU devices per host CANN: 9.0.1, innerversion V100R001C10SPC002B220 torch: 2.10.0+cpu torch-npu: 2.10.0.post2 vLLM: 0.23.0 vLLM-Ascend: 0.23.0rc1  Reproduction topology: DP2 x TP8 x EP16, eager prefill Quantization: W4A8_DYNAMIC VLLM_…

### #14868 — [[main2main] main2main manual review required (8bdc70ec)](https://github.com/vllm-project/vllm-ascend/issues/14868)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-25 06:36 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/14867 - Commit range: `ba07e4a48fc951300d97eb506217dd530583dea3`...`8bdc70ec7b379279ec0152343239c2d50aced687` - Status: `failed`  ## Final Summary  …

### #14866 — [[main2main] main2main manual review required (702e1d71)](https://github.com/vllm-project/vllm-ascend/issues/14866)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-25 02:08 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/14865 - Commit range: `ba07e4a48fc951300d97eb506217dd530583dea3`...`702e1d718646b5290f17533c04932d58bf03dad6` - Status: `failed`  ## Final Summary  …

## 🔀 Pull Requests

### #14874 — [Graph recon](https://github.com/vllm-project/vllm-ascend/pull/14874)
- **作者**: zhiyu-wa  **时间**: 2026-08-25 10:03 CST
- **标签**: module:core, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/ba07e4a48fc951300d97eb506217dd530583dea3

### #14873 — [[Doc] Translated Doc files 2026-08-25](https://github.com/vllm-project/vllm-ascend/pull/14873)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-25 09:58 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **63** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/contributors.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/versioning_policy.po</co…

### #14872 — [[CI] main2main vllm 0828](https://github.com/vllm-project/vllm-ascend/pull/14872)
- **作者**: LQDLove  **时间**: 2026-08-25 09:58 CST
- **摘要**: ### What this PR does / why we need it?  #### Upgrade baseline  - Update the verified vLLM main anchor from [`ba07e4a48fc951300d97eb506217dd530583dea3`](https://github.com/vllm-project/vllm/commit/ba07e4a48fc951300d97eb506217dd530583dea3) to [`7b7e5cbfcd99bc286ba2ae13e1f967b66a1a5c00`](https://githu…

### #14867 — [[Misc]feat: adapt to vLLM main (8bdc70ec)](https://github.com/vllm-project/vllm-ascend/pull/14867)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-25 06:36 CST
- **标签**: ready-all
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to August 22.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | — | [27ec8ac6](https://github.com/vllm-project/vllm/commit/27ec8ac626345…

### #14865 — [[Misc]feat: adapt to vLLM main (702e1d71)](https://github.com/vllm-project/vllm-ascend/pull/14865)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-25 02:08 CST
- **标签**: ready-all
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to August 24.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | — | [27ec8ac6](https://github.com/vllm-project/vllm/commit/27ec8ac626345…

### #14864 — [[v0.26.0rc][BugFix] Fix dispatch_v2 tokens capacity to follow normal MC2 branch logic](https://github.com/vllm-project/vllm-ascend/pull/14864)
- **作者**: kunpengW-code  **时间**: 2026-08-24 23:01 CST
- **标签**: module:core, ready
- **摘要**: ### What this PR does / why we need it? Follow-up modifications for PR #14747 This PR fixes the calculation of `_dispatch_v2_tokens_capacity` (normal MC2 tokens capacity) to follow the normal MC2 branch logic. Previously, when `enable_prefill_mc2` or `use_mega_moe` was true, `max_num_tokens` was bas…

### #14863 — [[BugFix][310P] Fix Moe aclgraph aclop error on 310p](https://github.com/vllm-project/vllm-ascend/pull/14863)
- **作者**: YangShuai52  **时间**: 2026-08-24 21:13 CST
- **摘要**: ### What this PR does / why we need it? ACLGraph Compatibility: Implemented a pre-casting mechanism for router weights to prevent unsupported forward-time Cast operations on 310P hardware. ### Does this PR introduce _any_ user-facing change? No ### How was this patch tested?  - vLLM version: v0.27.1…

### #14862 — [[MRV2][PATCH][Triton] Add a triton ops patch to avoid failure from upstream](https://github.com/vllm-project/vllm-ascend/pull/14862)
- **作者**: AuroraEmiya  **时间**: 2026-08-24 20:48 CST
- **标签**: module:ops
- **摘要**: …stream  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/ba07e4a48fc951300d97eb506217dd530583dea3

### #14861 — [[CI] Add nightly core suite gate with commit status propagation](https://github.com/vllm-project/vllm-ascend/pull/14861)
- **作者**: zhangxinyuehfad  **时间**: 2026-08-24 20:46 CST
- **标签**: ci/build
- **摘要**: <!--  Thanks for sending a pull request!  BEFORE SUBMITTING, PLEASE READ https://docs.vllm.ai/en/latest/contributing/overview.html  --> ### What this PR does / why we need it?  [[RFC]: PR Merge Tiered Governance (L1 Circuit Breaker + Exemption) ](https://github.com/vllm-project/vllm-ascend/issues/14…

### #14858 — [[Perf] Cache MRoPE cos/sin per step for sections outside the npu_mrop…](https://github.com/vllm-project/vllm-ascend/pull/14858)
- **作者**: l-wave  **时间**: 2026-08-24 20:14 CST
- **标签**: module:ops
- **摘要**: ## 🚀 Motivation  Models using MRoPE sections outside the `npu_mrope` allowlist (e.g. **Qwen2.5-Omni thinker's `[16, 16, 0]`**, rejected by `aclnnRopeWithSinCosCacheV2` with *"mropeSection must be in the supported list"*, verified on CANN 9.1.0) fall back to the stock eager native path in `AscendMRot…
