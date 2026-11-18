# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-18 15:53 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 最近动态的中文摘要：

### 🚀 Pull Requests (PR)
本次共收录 10 个 PR，主要涵盖新特性、Bug 修复、代码重构与社区治理：

**1. 重要新特性与功能增强**
* **原生 INT8 C8 支持**：为 Qwen3.8 Flash Next 模型添加了原生 INT8 C8 量化支持（#22278）。
* **昇腾容错与恢复机制**：引入了昇腾 FT 重试、缩容以及 HBM UCE（不可纠正错误）恢复功能，支持跨节点隔离受损的 KV-cache 块，提升集群容错能力（#22274）。
* **Gemma4 MTP 验证优化**：为 Gemma4 的 512 维 MTP 验证步骤添加了 B1 paged verify 支持（#22275）。

**2. Bug 修复**
* **MRV2 残差读取修复**：修复了 `releases/v0.32.0rc` 版本中，Model Runner V2 在使用 block verification 且带有 draft logits 时，residual-mass launch grid 未初始化读取的 Bug（#22273）。
* **Mooncake 滑动窗口修复**：修复了 Mooncake PD 架构在 Prefill 阶段处理整个 prompt 时，错误退役 Decode 阶段仍需使用的 page 的问题（#22270）。

**3. 重构与 CI 维护**
* **算子合并**：将 DeepSeek V4/V4.1 模型路径与 vLLM 自定义算子注册中两份字节相同的 Triton scale-and-add 算子代码进行了合并去重（#22276）。
* **清理 CI 测试**：移除了不再需要持续覆盖的 GLM-5.1 W4A4 A5 nightly 和 weekly 测试（#22277）。
* **Nightly 验证**：为 `releases/v0.30.0` 分支自动创建了夜间验证 PR（#22271）。

**4. 社区动态**
* 提名 `winson-00178005`（#22272）与 `lcfenglinwan`（#22269，量化及 DeepSeek 稀疏注意力/MoE领域）为新的 Committer/Maintainer。

### 🐛 Issues
* 本周期内未提供相关 Issue 动态。

### 📦 Release
* 本周期内无新的版本发布。

---

## 🔀 Pull Requests

### #22278 — [feat: add native INT8 C8 support for Qwen3.8 Flash Next](https://github.com/vllm-project/vllm-ascend/pull/22278)
- **作者**: ffggs  **时间**: 2026-11-18 15:50 CST
- **摘要**: <!--  Thanks for sending a pull request!  BEFORE SUBMITTING, PLEASE READ https://docs.vllm.ai/en/latest/contributing/overview.html  --> ### What this PR does / why we need it? <!-- - Please clarify what changes you are proposing. The purpose of this section is to outline the changes and how this PR …

### #22277 — [[CI] Remove GLM-5.1 A5 nightly and weekly tests](https://github.com/vllm-project/vllm-ascend/pull/22277)
- **作者**: 31-ryougishiki  **时间**: 2026-11-18 15:43 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  GLM-5.1 no longer needs ongoing CI coverage. Remove its W4A4 nightly and weekly tests on A5:  - Delete `GLM5_1_W4A4_A5.yaml` and `GLM5_1_W4A4_A5_weekly.yaml` under `tests/SIT/cases/models/configs/GLM/`. - Remove the corresponding `GLM5_1_W4A4_A5` entries from…

### #22276 — [[Refactor][Ops] Consolidate the Triton scale-and-add operator](https://github.com/vllm-project/vllm-ascend/pull/22276)
- **作者**: AuroraEmiya  **时间**: 2026-11-18 15:28 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  The DeepSeek V4/V4.1 model paths and the vLLM custom-op registration use two byte-identical copies of the same Triton kernel. Consolidate the implementation in `vllm_ascend/ops/triton/scale_and_add.py`, which computes `x * scale + y`.  - Update the DeepSeek c…

### #22275 — [[Feature][Attention] Add B1 paged verify for Gemma4 512-dim MTP verify](https://github.com/vllm-project/vllm-ascend/pull/22275)
- **作者**: JinGuoju001  **时间**: 2026-11-18 15:28 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it? This is a rebased and adapted continuation of #20125 on current main, where Model Runner V1 has been removed.  The Gemma4 target runs its MTP verify step with k+1 queries per request on head_dim=512 global-attention layers. On A2/A3, FIA TND rejects head_dim=5…

### #22274 — [[Feature] Add Ascend FT retry, scale-down and HBM UCE recovery](https://github.com/vllm-project/vllm-ascend/pull/22274)
- **作者**: WZN-JJB  **时间**: 2026-11-18 15:14 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  Add Ascend HBM UCE recovery using the existing fault-tolerance retry flow. A recoverable KV-cache UCE is classified across participating workers, damaged KV blocks are cleared, and old ACL graphs are discarded and recaptured before serving resumes. Non-KV or …

### #22273 — [[BugFix][v0.32.0rc] Fix MRV2 block verification residual-mass launch grid](https://github.com/vllm-project/vllm-ascend/pull/22273)
- **作者**: SOMEONEUNSEEN  **时间**: 2026-11-18 15:01 CST
- **摘要**: ### What this PR does / why we need it?  Fix an uninitialized residual-mass read when Model Runner V2 uses block verification with draft logits on `releases/v0.32.0rc`.  In vLLM v0.32.0, `_compute_local_residual_mass_kernel` skips the final bonus row with `logit_idx + 1 >= tl.num_programs(0)`. Ascen…

### #22272 — [[Community] nominate winson-00178005 as committer](https://github.com/vllm-project/vllm-ascend/pull/22272)
- **作者**: wangxiyuan  **时间**: 2026-11-18 15:01 CST
- **标签**: documentation, ci/build
- **摘要**: - ✅Review Quality: 31 PR reviewed and 48 PR commented in the main repository:[ https://github.com/vllm-project/vllm-ascend/pulls?q=is%3Apr+reviewed-by%3Awinson-00178005](https://github.com/vllm-project/vllm-ascend/pulls?q=is%3Apr+reviewed-by%3Awinson-00178005) Three of them are worth calling out. Ea…

### #22271 — [[Verify] Nightly test on releases/v0.30.0](https://github.com/vllm-project/vllm-ascend/pull/22271)
- **作者**: LQDLove  **时间**: 2026-11-18 14:53 CST
- **标签**: documentation
- **摘要**: Auto-created empty PR for nightly verification. - vLLM main: https://github.com/vllm-project/vllm/commit/ced6857afa0ea7b2e3f0846a62e1394e90f15607

### #22270 — [[BugFix] Fix Mooncake SWA prefill handoff at N-1](https://github.com/vllm-project/vllm-ascend/pull/22270)
- **作者**: Mouse27-bit  **时间**: 2026-11-18 14:46 CST
- **标签**: module:tests, ci-run
- **摘要**: ### What this PR does / why we need it?  Mooncake PD replays the final prompt token on Decode. For sliding-window caches, computing the entire prompt on Prefill can retire a page that Decode still needs for that replay. With prompt length 1023, window 512 and block size 128, the page containing posi…

### #22269 — [[Community] nominate lcfenglinwan as committer](https://github.com/vllm-project/vllm-ascend/pull/22269)
- **作者**: wangxiyuan  **时间**: 2026-11-18 14:45 CST
- **标签**: documentation, ci/build
- **摘要**: I would like to nominate @lcfenglinwan as a vLLM Ascend maintainer for the quantization and DeepSeek sparse attention / MoE domains. He has been a sustained, high-signal contributor since early 2026 and has become one of the busiest reviewers of the quantization, DSA/SFA attention and MoE dispatch c…
