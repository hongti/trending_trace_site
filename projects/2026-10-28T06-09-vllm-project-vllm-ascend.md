# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-28 14:09 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue (问题)
* **性能问题修复提议 (#20078)**：未量化的 Ascend MoE 路径在存在 LoRA context 时会强制走 LoRA 分支，而不是检查批次是否实际选择了 adapter，导致空闲的 adapter 会带来约 40% 的端到端性能损耗。
* **每日夜间测试失败 (#20077)**：2026-10-28 夜间测试报错，失败模型为 DeepSeek-V4-Flash-W8A8-A3，单节点测试 (`test_single_node`) 中出现断言错误。

### 🚀 Pull Request (拉取请求)
* **性能优化 - UVA 脏行追踪 (#20076, #20073)**：优化 `patch_v2/patch_uva.py` 中的主机影子记账逻辑，消除逐行 Python 操作。将连续前缀写入复杂度降至 O(1) 并进行向量化处理，大幅提升性能。
* **新特性 - 跨阶段保留 SP Token 分片 (#20075)**：在 Model Runner V2 流水线边界处保留序列并行 (SP) 主激活，以支持 DeepSeek-V4、Kimi-K3 AttnRes 和 MiniMax-M3。
* **新特性 - Gemma4 512维 MTP 分页验证 (#20070)**：引入 B1 paged verify (FIA-v2 BNSD)，解决 Gemma4 31B W8A8 在 A2/A3 平台上因 head_dim=512 被拒绝而导致无法进行 MTP verify 的问题，使其不再局限于 PIECEWISE 运行。
* **文档更新 - GLM-5.2 部署并行度调整 (#20071, #20069)**：更新 A2 PD 分离式部署配置，将 Prefill 从 PP2/TP16 调整为 PP4/TP8，层分区设为 `20,20,20,18`，Decode 维持 DP16/TP2。
* **文档更新 - MiniMax-M3.1-Flash A2 部署 (#20067)**：为 MiniMax-M3.1-Flash 教程补充 Atlas A2 平台的部署说明与配置。
* **文档与 CI 改进 (#20074, #20068)**：自动翻译 61 个中文文档文件；优化 CI 流水线，将本地 actions 暂存在 runner temp 目录以减少任务启动时间。
* **测试验证 (#20072)**：为 `releases/v0.30.0` 分支创建夜间验证自动 PR。

### 📦 Release (发布)
* 近期无新的 Release 版本发布。

---

## 🐛 Issues

### #20078 — [[Performance]: Unquantized MoE path takes the LoRA branch on context presence instead of has_lora(), so idle adapters cost ~40% e2e](https://github.com/vllm-project/vllm-ascend/issues/20078)
- **作者**: tzhouam  **时间**: 2026-10-28 14:05 CST
- **摘要**: ### Proposal to improve performance  The unquantized Ascend MoE path takes the LoRA branch whenever a `lora_context` is **published**, not when the batch actually selects an adapter. A deployment that enables LoRA but is currently running without an adapter therefore pays the full LoRA path for a gu…

### #20077 — [Nightly Daily test failure on 2026-10-28 12:00 - please check](https://github.com/vllm-project/vllm-ascend/issues/20077)
- **作者**: czydyy  **时间**: 2026-10-28 14:01 CST
- **摘要**: The nightly Daily test for vllm-ascend reported failures at 12:00 on 2026-10-28.  **Failed on:** DeepSeek-V4-Flash-W8A8-A3  **Failed test cases:** - test_single_node — AssertionError: some aisbench cases failed, info were shown above.  **Failed run(s):** https://github.com/vllm-project/vllm-ascend/a…

## 🔀 Pull Requests

### #20076 — [[Performance][Patch] Track UVA dirty rows without per-row Python work](https://github.com/vllm-project/vllm-ascend/pull/20076)
- **作者**: pjh456  **时间**: 2026-10-28 13:51 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Make the emulated-UVA host-shadow bookkeeping in `patch_v2/patch_uva.py` cost O(1) for the contiguous-prefix writes the V2 model runner performs, and vectorised for everything else. The rows to refresh are now tracked with a high-water mark plus a lazily allo…

### #20075 — [[Feature][MRV2] Preserve SP token shards across pipeline stages](https://github.com/vllm-project/vllm-ascend/pull/20075)
- **作者**: xujiaz2000  **时间**: 2026-10-28 13:43 CST
- **标签**: documentation, module:tests, module:tools
- **摘要**: ### What this PR does / why we need it?  Preserve sequence-parallel main activations across Model Runner V2 pipeline boundaries for DeepSeek-V4, Kimi-K3 AttnRes, and MiniMax-M3. Each TP rank sends its token shard to the matching rank in the next stage instead of transmitting replicated full-token ac…

### #20074 — [[Doc] Translated Doc files 2026-10-28](https://github.com/vllm-project/vllm-ascend/pull/20074)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-28 13:32 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **61** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents/context_parallel.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/develop…

### #20073 — [[Performance][Patch] Track UVA dirty rows without per-row Python work](https://github.com/vllm-project/vllm-ascend/pull/20073)
- **作者**: pjh456  **时间**: 2026-10-28 13:23 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Make the emulated-UVA host-shadow bookkeeping in `patch_v2/patch_uva.py` cost O(1) for the contiguous-prefix writes the V2 model runner performs, and vectorised for everything else. The rows to refresh are now tracked with a high-water mark plus a lazily allo…

### #20072 — [[Verify] Nightly test on releases/v0.30.0](https://github.com/vllm-project/vllm-ascend/pull/20072)
- **作者**: LQDLove  **时间**: 2026-10-28 13:04 CST
- **标签**: documentation
- **摘要**: Auto-created empty PR for nightly verification. - vLLM main: https://github.com/vllm-project/vllm/commit/ced6857afa0ea7b2e3f0846a62e1394e90f15607

### #20071 — [[Doc][30.0][Model] Update A2 GLM-5.2 deployment parallelism](https://github.com/vllm-project/vllm-ascend/pull/20071)
- **作者**: panther-zhu  **时间**: 2026-10-28 13:00 CST
- **标签**: documentation
- **摘要**: ### What does this PR do?     - Update the A2 PD disaggregation Prefill configuration from PP2/TP16 to PP4/TP8.   - Set the A2 Prefill layer partition to `20,20,20,18` and keep Decode at DP16/TP2.   - Update the A2 MemCache KV pooling deployment to PP4/TP8.   - Raise the A2 PD context limit to 200k.…

### #20070 — [[Attention][Feature] B1 paged verify for Gemma4 512-dim MTP verify (FIA-v2 BNSD)](https://github.com/vllm-project/vllm-ascend/pull/20070)
- **作者**: swimming2007-doge  **时间**: 2026-10-28 13:00 CST
- **摘要**: ## What this does Gemma4 31B W8A8 with MTP k=3 could only run under PIECEWISE on A2/A3: the target's global-attention layers use head_dim=512, which FIA TND rejects, so the MTP verify step could not be captured in the FULL decode graph. This ports the B1 paged-verify path from the MTP-A2-b1-paged-ve…

### #20069 — [[Doc][Model] Update A2 GLM-5.2 deployment parallelism](https://github.com/vllm-project/vllm-ascend/pull/20069)
- **作者**: panther-zhu  **时间**: 2026-10-28 12:59 CST
- **标签**: documentation
- **摘要**: ### What does this PR do?     - Update the A2 PD disaggregation Prefill configuration from PP2/TP16 to PP4/TP8.   - Set the A2 Prefill layer partition to `20,20,20,18` and keep Decode at DP16/TP2.   - Update the A2 MemCache KV pooling deployment to PP4/TP8.   - Raise the A2 PD context limit to 200k.…

### #20068 — [[CI] Stage local actions in runner temp to reduce job setup time](https://github.com/vllm-project/vllm-ascend/pull/20068)
- **作者**: MrZ20  **时间**: 2026-10-28 12:49 CST
- **标签**: documentation, ci/build, ready-all
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…

### #20067 — [[Doc][Feature] Add A2 platform in MiniMax-M3.1-Flash.md](https://github.com/vllm-project/vllm-ascend/pull/20067)
- **作者**: xszbuaa  **时间**: 2026-10-28 12:20 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? This PR adds deployment instructions and configurations for the Atlas A2 platform in the MiniMax-M3.1-Flash tutorial.  ### Does this PR introduce _any_ user-facing change? No. This is a documentation-only change.  ### How was this patch tested? The A2 commands…
