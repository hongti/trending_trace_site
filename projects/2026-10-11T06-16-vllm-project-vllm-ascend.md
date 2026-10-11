# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-11 14:16 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的简洁摘要：

### 📝 Pull Requests (PR)

近期 PR 主要集中在性能优化、Bug 修复以及针对 `v0.30.0` 版本分支的维护与配置更新：

*   **性能优化**
    *   **#18277 (KVPool 性能)**: 优化了非分层 AscendStore 的查找和保存逻辑，避免在历史数据块中进行全前缀扫描，显著降低 CPU 时间消耗。
*   **Bug 修复**
    *   **#18270 (MoE 内存探测修复)**: 修复了 MRV2 在初始内存探测时绕过 PCP 分区的问题。限制了在 EP 和 fused MC2 场景下 `DispatchFFNCombine` 接收的全局 token 预算，防止错误发送。
    *   **#18272 (oproject 容量超限提示)**: 优化了当 `oproj` 容量超限时的 `ValueError` 报错信息，直接引导用户使用相应的 recipe 进行修复（已 Cherry-pick 至 `v0.30.0`）。
    *   **#18269 (P/D 投机解码修复)**: 提供了一个示例补丁，修复了 P/D 分离部署下，D 节点在 EAGLE 投机解码 + v2 `FULL_DECODE_ONLY` 图模式下，新请求首个 verify 步骤静默退化为 eager 执行并被 DP 级联放大到所有 rank 的问题。
*   **文档与配置清理**
    *   **#18276 / #18275**: 刷新了 GLM-5.3-Flash A3 PD 部署指南，将配置更新为 DP2/TP8 Prefill 和 DP8/TP2 Decode（同步至主分支与 `v0.30.0` 分支）。
    *   **#18271**: 清理了 sparse kv offload 相关的冗余代码和配置，移除了未使用的 "none" 模式及 `fused_op_type` 参数。
*   **CI 与测试**
    *   **#18278**: 移除了 GLM-5 和 GLM-4.7 的计划测试用例及 YAML 文件（Cherry-pick 至 `v0.30.0`）。
    *   **#18273 / #18274**: 更新了 Qwen3.6 310P 的测试配置，移除了 27B DUO 和 35B MTP 案例中未使用的采样参数（已同步至 `v0.30.0`）。

### 🐛 Issues
*   本次提供的动态中未包含具体的 Issue 记录。

### 🚀 Releases
*   本次提供的动态中未包含正式的 Release 发布信息。
*   **注**：尽管无新版本发布，但大量 PR（#18278, #18275, #18274, #18272 等）正在向 `releases/v0.30.0` 分支进行 Cherry-pick，表明该版本正在密集进行测试收敛、配置更新与稳定性修复。

---

## 🔀 Pull Requests

### #18278 — [[Cherry-pick][releases/v0.30.0][CI] Remove GLM5 and GLM4.7 scheduled cases and YAML files (from #18013)](https://github.com/vllm-project/vllm-ascend/pull/18278)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-11 14:15 CST
- **摘要**: Cherry-pick of PR #18013 onto `releases/v0.30.0`.  Original PR: #18013 Original author: @Wyz-134  --- ### What this PR does / why we need it?  Remove the remaining GLM-5 (original version) and GLM-4.7 nightly/weekly cases, their YAML files, and obsolete scheduled-test references, following the clean…

### #18277 — [[Performance][KVPool] Avoid full-prefix scans during lookup and save](https://github.com/vllm-project/vllm-ascend/pull/18277)
- **作者**: bowgneo  **时间**: 2026-10-11 14:07 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Non-layerwise AscendStore lookup and save spend CPU time walking historical chunks that cannot be queried or addressed. Lookup also expands the locally computed HBM prefix into per-group hit sets, repeating hash conversion and set construction across groups. …

### #18276 — [[Doc][Misc] Refresh GLM-5.3-Flash A3 PD configuration](https://github.com/vllm-project/vllm-ascend/pull/18276)
- **作者**: YanpengDing  **时间**: 2026-10-11 13:41 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Refresh the GLM-5.3-Flash A3 PD deployment guide to use DP2/TP8 Prefill and DP8/TP2 Decode instead of DP16/TP1 Decode. Align the role scripts and descriptions with the supplied A3 configuration, including HCCS communication, graph modes, MTP settings, int8 KV…

### #18275 — [[releases/v0.30.0][Doc][Misc] Refresh GLM-5.3-Flash A3 PD configuration](https://github.com/vllm-project/vllm-ascend/pull/18275)
- **作者**: YanpengDing  **时间**: 2026-10-11 13:31 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Refresh the GLM-5.3-Flash A3 PD deployment guide to use DP2/TP8 Prefill and DP8/TP2 Decode instead of DP16/TP1 Decode. Align the role scripts and descriptions with the supplied A3 configuration, including HCCS communication, graph modes, MTP settings, int8 KV…

### #18274 — [[Cherry-pick][releases/v0.30.0][CI][Test] Update Qwen3.6 310P test configurations (from #18273)](https://github.com/vllm-project/vllm-ascend/pull/18274)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-11 13:21 CST
- **标签**: module:tests
- **摘要**: Cherry-pick of PR #18273 onto `releases/v0.30.0`.  Original PR: #18273 Original author: @czydyy  --- ### What this PR does / why we need it? Backports the Qwen3.6 310P test configuration changes from #18000 to main. It removes unused sampling parameters from the 27B DUO and 35B MTP cases and updates…

### #18273 — [[CI][Test] Update Qwen3.6 310P test configurations](https://github.com/vllm-project/vllm-ascend/pull/18273)
- **作者**: czydyy  **时间**: 2026-10-11 13:16 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it? Backports the Qwen3.6 310P test configuration changes from #18000 to main. It removes unused sampling parameters from the 27B DUO and 35B MTP cases and updates the 27B and 35B DUO performance baselines. GLM changes from #18000 are excluded.  ### Does this PR i…

### #18272 — [[BugFix][Model] Point users at the recipe fix when oproj capacity is exceeded](https://github.com/vllm-project/vllm-ascend/pull/18272)
- **作者**: xuchi-0808  **时间**: 2026-10-11 13:11 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  Companion to #18126 (against `main`), cherry-picked to the `releases/v0.30.0` branch where users actually run this code.  Improves the `ValueError` raised in `dsa_v1.py::_forward_o_proj` when the fine-grained oproj TP static exchange capacity cannot cover the…

### #18271 — [[Doc][Misc][v0.30.0] Remove redundant codes and configs for sparse kv offload](https://github.com/vllm-project/vllm-ascend/pull/18271)
- **作者**: Angazenn  **时间**: 2026-10-11 13:00 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it? This PR updates config for sparse kv offload from the following aspects: 1. Drop unused "none" mode for `fused_op_type` along with `fused_op_type` argument. Now fused_copy_sfa is the default option. We still maintain the `use_fused_overlap` for legacy, and can…

### #18270 — [[Bugfix] Bound PCP MoE inputs during MRV2 memory profiling](https://github.com/vllm-project/vllm-ascend/pull/18270)
- **作者**: recky-c  **时间**: 2026-10-11 12:45 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  MRV2's initial memory-profile dummy batch bypasses PCP partitioning. With EP and fused MC2, it sends the global token budget to `DispatchFFNCombine` on every PCP rank. For example, a PCP4 run with `max_num_batched_tokens=16384` profiles M=16384 per rank, unne…

### #18269 — [[Bugfix][P/D] Reclassify first spec-verify step on PD decode nodes to keep graph replay (example patch)](https://github.com/vllm-project/vllm-ascend/pull/18269)
- **作者**: xuchi-0808  **时间**: 2026-10-11 12:13 CST
- **摘要**: ## 这个 PR 的定位（请先读这段）  这是一个**示例代码 PR（不要求按原样合入）**，目的：向 P/D 分离方向的维护者完整展示一个已在生产环境实测定位的问题——**PD 部署的 D 节点在 EAGLE 投机解码 + v2 `FULL_DECODE_ONLY` 图模式下，新请求的首个 verify 步会静默退化为 eager，再被 DP 的 cg_mode min 级联放大到全部 rank**。生产实测 decode 迭代从 131ms 劣化到 471ms（3.6×），而启动日志显示 49 档图全部捕获、dispatch 覆盖索引完好——问题不在"桶缺了"，而在 dispatch 的输…
