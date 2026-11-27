# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-27 16:58 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 📦 Release
*近期无新版本发布动态。*

### 🐛 Issue
近期问题主要集中在混合模型（如 Mamba/GDN 架构）的适配与稳定性上：
1. **特性请求**：针对多轮混合模型，建议 AscendStore 限制冗余的边界状态写入，以优化性能 (#23546)。
2. **前缀缓存 Bug**：`KimiLinearForCausalLM` 在共享前缀的不同 prompt 间错误复用了 mamba state，导致返回结果错误 (#23545)。
3. **引擎崩溃 Bug**：`KimiLinearForCausalLM` (GDN) 在处理 prefill 与 decode 混合批次时，前向传播触发 `NotImplementedError` 导致引擎崩溃 (#23544)。

### 🔧 Pull Request
PR 活动涵盖了新特性引入、关键 Bug 修复以及 CI/社区维护：

**1. 重要新特性与性能优化**
* **AscendStore 优化**：新增可选的边界状态卸载步长，直接修复了 Issue #23546 中提到的冗余写入问题 (#23547)。
* **HiSparse 性能提升**：引入按需 TopK 行收集和异构 prefill 流水线，优化 Prefill 阶段的数据处理路径 (#23539)。
* **算子适配**：将 `attn_res_fwd_fused` 算子适配至 Ascend960DT 硬件 (#23542)。

**2. 关键 Bug 修复**
* **DCP/Mooncake 路由对齐**：修复 DCP KV gathers 和 Mooncake V2 路由排序，使其与 vLLM 主干保持一致 (#23541)。
* **DSpark KV 写入修复**：修复了在 chunked prefill 且 draft count 为 0 时，DSpark 仍错误更新 context KV 的问题 (#23540)。

**3. CI 与社区维护**
* **测试覆盖**：新增 GLM-5.3 A3 的周度长序列负载测试覆盖 (#23543)，并更新端到端算子 CI 测试 (#23535)。
* **流程优化**：允许带有 `[Revert]` 标题的紧急回退 PR 使用 `skip_ci` 标签跳过测试，加速紧急修复合入 (#23537)。
* **代码审查**：为 `HiSparse` 包指定了专属 CODEOWNERS，确保相关代码变更能被正确路由审查 (#23536)。

---

## 🐛 Issues

### #23546 — [[Feature] AscendStore: throttle redundant boundary-state writes for multi-turn hybrid models](https://github.com/vllm-project/vllm-ascend/issues/23546)
- **作者**: haoyuzhang123  **时间**: 2026-11-27 16:55 CST
- **摘要**: # [Feature] AscendStore: throttle redundant boundary-state writes for multi-turn hybrid models  ## Summary  AscendStore already supports hybrid models (Mamba/GDN linear-attention groups) and stores Core boundary-state checkpoints through `_handle_boundary_state_offloads` and `register_finished_parti…

### #23545 — [[Bug] Prefix caching returns wrong results for KimiLinearForCausalLM: mamba state reused across different prompts sharing a prefix](https://github.com/vllm-project/vllm-ascend/issues/23545)
- **作者**: dahai80  **时间**: 2026-11-27 16:46 CST
- **摘要**: # [Bug] Prefix caching returns wrong results for KimiLinearForCausalLM: mamba state ('align' mode) reused across different prompts sharing a prefix  ## Environment  - vllm-ascend: 0.32.0rc2.dev48 (vllm 0.32.0), Ascend 950DT, TP=2, eager mode - Model: Kimi-Linear-48B-A3B-Instruct (KimiLinearForCausal…

### #23544 — [[Bug] KimiLinearForCausalLM (GDN) engine dies on mixed prefill+decode batches: NotImplementedError raised inside forward pass](https://github.com/vllm-project/vllm-ascend/issues/23544)
- **作者**: dahai80  **时间**: 2026-11-27 16:46 CST
- **摘要**: # [Bug] KimiLinearForCausalLM (GDN) engine dies on mixed prefill+decode batches: NotImplementedError raised inside forward pass  ## Environment  - vllm-ascend: 0.32.0rc2.dev48 (vllm 0.32.0), Ascend 950DT, TP=2, eager mode - Model: Kimi-Linear-48B-A3B-Instruct (KimiLinearForCausalLM, hybrid GDN + ful…

## 🔀 Pull Requests

### #23547 — [[Feature] AscendStore: add opt-in boundary-state offload stride](https://github.com/vllm-project/vllm-ascend/pull/23547)
- **作者**: haoyuzhang123  **时间**: 2026-11-27 16:56 CST
- **标签**: documentation, module:tests
- **摘要**: Fixes #23546  # [Feature] AscendStore: add opt-in boundary-state offload stride  ## What this PR does / why we need it  AscendStore supports hybrid Mamba/GDN models and receives Core boundary-state checkpoints through `_handle_boundary_state_offloads` and `register_finished_partial_tail`. Today ever…

### #23543 — [[CI][Model] Add GLM-5.3 A3 weekly workload coverage](https://github.com/vllm-project/vllm-ascend/pull/23543)
- **作者**: Wyz-134  **时间**: 2026-11-27 16:45 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  The full GLM-5.3 A3 nightly currently measures only 16k prefill, 16k PD and one 16k decode concurrency point. Add three weekly entries to cover longer inputs, shared-prefix PD and higher decode concurrency as GLM-5.1 coverage is retired.  All entries reuse th…

### #23542 — [feat(attn_res_fwd): adapt attn_res_fwd_fused operator to Ascend960DT](https://github.com/vllm-project/vllm-ascend/pull/23542)
- **作者**: Night-lbk  **时间**: 2026-11-27 16:44 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…

### #23541 — [[BugFix]: align DCP KV gathers and Mooncake V2 routing with vLLM](https://github.com/vllm-project/vllm-ascend/pull/23541)
- **作者**: u626cyale  **时间**: 2026-11-27 16:43 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? This PR depends on #22176. Both vLLM v0.34.0 and the updated main pin order each DCP group by ascending global rank, matching the order of HCCL AllGather output. MLA and SFA can therefore use the DCP rank directly to locate gathered KV blocks. This removes red…

### #23540 — [[BugFix] Finalize DSpark layerwise KV writes when drafts are skipped](https://github.com/vllm-project/vllm-ascend/pull/23540)
- **作者**: tyy0829  **时间**: 2026-11-27 16:41 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  During chunked prefill, DSpark still updates context KV when the scheduler sets the draft count to zero. Its query attention does not run, so attention-triggered layerwise saves stop at the target model and never submit the final draft-layer task carrying `wr…

### #23539 — [[Feature][Perf][HiSparse] On-demand TopK row gather and heterogeneous prefill pipeline](https://github.com/vllm-project/vllm-ascend/pull/23539)
- **作者**: ZD-euler  **时间**: 2026-11-27 16:37 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  In `HiSparseConnector`, Decode already performs sparse incremental H2D row swaps into a fixed per-request `Hot` window (`4,096` rows), whereas **Prefill** bypasses `Hot` and allocates a transient per-layer `staged` buffer (`torch.empty`) on the main compute s…

### #23538 — [[Doc] doc quality test](https://github.com/vllm-project/vllm-ascend/pull/23538)
- **作者**: herizhen  **时间**: 2026-11-27 16:37 CST
- **标签**: documentation, ai-review
- **摘要**: ### What this PR does / why we need it? 仅测试  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero e…

### #23537 — [[CI] Allow skip_ci for revert PRs](https://github.com/vllm-project/vllm-ascend/pull/23537)
- **作者**: czydyy  **时间**: 2026-11-27 16:28 CST
- **标签**: documentation, ci/build
- **摘要**: ### What this PR does / why we need it?  Urgent reverts can wait on CPU unit tests and NPU end-to-end tests before maintainers can merge them. A PR with `[Revert]` in its title and the `skip_ci` label can now skip the repository workflow's static checks, CPU UT, NPU E2E, and CODEOWNERS approval job.…

### #23536 — [[Misc][Community] Add CODEOWNERS for HiSparse](https://github.com/vllm-project/vllm-ascend/pull/23536)
- **作者**: Esir98  **时间**: 2026-11-27 16:28 CST
- **摘要**: ### What this PR does / why we need it?  Assign Code Owners for the HiSparse package so changes under `vllm_ascend/hisparse/` are routed to its maintainers.  ```text /vllm_ascend/hisparse/ @zhenwenqi2024 @weiguihua2 @Esir98 @LifeOfCoffee ```  ### Does this PR introduce _any_ user-facing change?  No.…

### #23535 — [[CI]Pr e2e ops 20261127](https://github.com/vllm-project/vllm-ascend/pull/23535)
- **作者**: shiqiangA  **时间**: 2026-11-27 16:28 CST
- **标签**: ci/build, module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…
