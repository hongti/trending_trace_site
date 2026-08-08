# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-08 10:31 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
*   **变长 DP/EP 请求挂起 Bug (#13826)**：在 Ascend A2 环境下运行 DeepSeek-V4-Flash W8A8 + DSpark 投机解码时，当 MC2 跳过 ACLGraph 模式同步，会导致变长 DP/EP 请求挂起。
*   **main2main 自动化流程中断 (#13824)**：主分支同步自动化流程未完成所有计划步骤，需人工审核介入。

### 🔀 Pull Request 动态
**1. 核心修复**
*   **修复 MC2 ACLGraph 模式同步问题 (#13828, #13827)**：针对上述 Issue #13826，修复了当 MC2 支持不均匀 token 数时，`_sync_metadata_across_dp()` 提前返回导致 ACLGraph 模式同步被意外跳过的问题。#13827 为该修复向 `v0.25.1rc` 分支的回移。
*   **修复 Mooncake 负载归属问题 (#13825)**：修复了 `AscendMultiConnector` 中使用 FirstWin 且 AscendStore 赢得 Decode 侧负载时，特定状态更新函数被跳过的问题。

**2. 重要新特性**
*   **统一动态投机解码逻辑 (#13819)**：统一了 DFlash 和 DSpark 的动态投机解码逻辑，使两者现在使用相同的动态选择机制。
*   **为 MLA 和 GQA 保留带步幅的 PA 缓存 (#13821)**：将 FIA 迁移至 vllm-ascend 自定义 op 路径，并为 MLA 和 GQA 保留了 first-axis-strided PA KV view 描述符。
*   **新增 SFA PD RD2H 连接器 (#13818)**：向 `v0.26.0rc` 分支回移，支持 KV cache 卸载。
*   **支持 MTP 和 sparse C8 (#13817)**：向 `v0.26.0rc` 分支回移，在逐层 prefill 卸载中支持多 token 预测（MTP）和 sparse C8。

**3. 工程与文档**
*   **适配 vLLM 主分支 (#13823, #13822)**：将 vllm-ascend 适配至 8 月 7 日的 vLLM 上游最新提交。
*   **废弃环境变量 (#13820)**：在全仓库范围（文档、教程、测试配置等）废弃并移除 `ASCEND_BUFFER_POOL` 环境变量。

### 🚀 Release 动态
*   近期无正式版发布。但从 PR 活动可见，目前正积极向 **v0.25.1rc** 回移关键 BugFix（如 ACLGraph 同步修复），并向 **v0.26.0rc** 回移重要的 KV Transfer/Pool 新特性（如 SFA PD RD2H connector 和 MTP 支持）。

---

## 🐛 Issues

### #13826 — [[Bug]: Variable-length DP/EP requests can hang when MC2 skips ACLGraph mode sync](https://github.com/vllm-project/vllm-ascend/issues/13826)
- **作者**: QwertyJack  **时间**: 2026-08-08 09:48 CST
- **标签**: core-features, aclgraph
- **摘要**: ### Environment  - Hardware: Ascend A2, 8 nodes (4 prefill + 4 decode) - Model: DeepSeek-V4-Flash-0731 W8A8 with DSpark speculative decoding - Decode topology: DP32 / EP32 / TP1, PD KV consumer - vLLM: `v0.25.1` (`752a3a504485790a2e8491cacbb35c137339ad34`) - vLLM Ascend: `releases/v0.25.1rc` (`50e0c…

### #13824 — [[main2main] main2main manual review required (45273b8d)](https://github.com/vllm-project/vllm-ascend/issues/13824)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-08 03:12 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/13823 - Commit range: `448344c0e29383adfe606a5c7ede72dd74705321`...`45273b8dcbfb2d2c300c2f4a55c4dc283adca06a` - Status: `failed`  ## Final Summary  …

## 🔀 Pull Requests

### #13828 — [[BugFix][Worker] Synchronize ACLGraph mode across MC2 DP ranks](https://github.com/vllm-project/vllm-ascend/pull/13828)
- **作者**: QwertyJack  **时间**: 2026-08-08 10:07 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  When MC2 supports uneven token counts, `_sync_metadata_across_dp()` skips the token-count all-reduce. The same early return also skipped ACLGraph mode synchronization, allowing active and idle DP ranks to submit graph and eager work into the same EP collectiv…

### #13827 — [[v0.25.1rc][BugFix][Worker] Synchronize ACLGraph mode across MC2 DP ranks](https://github.com/vllm-project/vllm-ascend/pull/13827)
- **作者**: QwertyJack  **时间**: 2026-08-08 10:02 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  When MC2 supports uneven token counts, `_sync_metadata_across_dp()` skips the token-count all-reduce. The same early return also skipped ACLGraph mode synchronization, allowing active and idle DP ranks to submit graph and eager work into the same EP collectiv…

### #13825 — [[BugFix] Fix Mooncake load ownership in AscendMultiConnector](https://github.com/vllm-project/vllm-ascend/pull/13825)
- **作者**: Futuresxy  **时间**: 2026-08-08 07:18 CST
- **标签**: module:tests
- **摘要**: This PR aims to fix #13474.  When `MultiConnector[AscendStoreConnector, MooncakeLayerwiseConnector]` uses FirstWin and AscendStore wins a Decode-side load, the Ascend-specific `update_state_after_alloc` override currently gives the losing Mooncake connector both the real blocks and the winner's `num…

### #13823 — [[Misc]feat: adapt to vLLM main (45273b8d)](https://github.com/vllm-project/vllm-ascend/pull/13823)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-08 03:11 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to August 07.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | `tests/e2e/conftest.py` | [3ee2bd13](https://github.com/vllm-project/vll…

### #13822 — [[Misc]feat: adapt to vLLM main (448344c0)](https://github.com/vllm-project/vllm-ascend/pull/13822)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-08 01:28 CST
- **标签**: module:tests, module:ops, module:core, ready
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to August 07.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | — | [21ea5b4f](https://github.com/vllm-project/vllm/commit/21ea5b4fa1062…

### #13821 — [[Feature][Ops][FIA] Preserve strided PA caches for MLA and GQA](https://github.com/vllm-project/vllm-ascend/pull/13821)
- **作者**: maoxx241  **时间**: 2026-08-07 22:20 CST
- **标签**: module:tests
- **摘要**: ## Summary  - Migrate FIA from ops-transformer mainline into the vllm-ascend custom-op path, including the requested 8739/9608/9631 change set. - Preserve first-axis-strided PA KV view descriptors through vllm-ascend's ACLNN adapter and host tiling into the arch22 GQA/MLA device kernels. - Add singl…

### #13820 — [[Doc] Sunset ASCEND_BUFFER_POOL environment variable](https://github.com/vllm-project/vllm-ascend/pull/13820)
- **作者**: lizy124  **时间**: 2026-08-07 21:21 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Sunsets the `ASCEND_BUFFER_POOL` environment variable across the entire repository (docs, tutorials, locale translations, and test configurations).  ### Does this PR introduce _any_ user-facing change?  Yes. `ASCEND_BUFFER_POOL` is removed and replaced by `AS…

### #13819 — [[Feature] Support dynamic speculative decoding with DFlash and Unified dynamic selection logic](https://github.com/vllm-project/vllm-ascend/pull/13819)
- **作者**: StanislavII  **时间**: 2026-08-07 20:34 CST
- **标签**: module:core
- **摘要**: ## Summary Follow up from https://github.com/vllm-project/vllm-ascend/pull/13216 This PR unifies the dynamic speculative decoding logic for **DFlash** and **DSpark**.  Both methods now use the same dynamic verification pipeline:  ```text token confidence     -> token acceptance probabilities     -> …

### #13818 — [[Feature][KV Transfer][v0.26.0rc] Add SFA PD RD2H connector for KV cache offload](https://github.com/vllm-project/vllm-ascend/pull/13818)
- **作者**: ader47  **时间**: 2026-08-07 19:30 CST
- **标签**: documentation, module:tests, module:core, module:tools
- **摘要**: ### What this PR does / why we need it?  > [!NOTE] > This is the `releases/v0.26.0rc` backport of #12831 and the third PR in a stacked series. > It depends on #13711, #13816, and #13817. Until those PRs are merged, the GitHub diff will also contain their prerequisite changes.  Building on decode-sid…

### #13817 — [[Feature][KV Pool][v0.26.0rc] Support MTP and sparse C8 in layerwise prefill offload](https://github.com/vllm-project/vllm-ascend/pull/13817)
- **作者**: ader47  **时间**: 2026-08-07 19:30 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  > [!NOTE] > This is the `releases/v0.26.0rc` backport of #12853 and the second PR in a stacked series. > It depends on #13816. Until that PR is merged, the GitHub diff will also contain its layerwise buffer reuse changes.  This PR extends the layerwise KV Poo…
