# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-31 09:06 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 📌 Issue 动态
近期 Issue 主要集中在核心 Bug 报告与社区流程优化讨论：
*   **误分类 Bug**：ACL Graph replay 阶段出现的 NPU 流同步超时（错误码 ACL 107020/107027）被错误地归类为 OOM（内存溢出）错误 (#13218)。
*   **逻辑 Bug**：在解耦预填充示例中，`_release_load` 逻辑未做下限限制，导致 decoder 的 `active_tokens` 可能出现负数 (#13214)。
*   **模型兼容性 Bug**：在 Atlas 800I A2 硬件上运行 GLM-5.2 时，MLA target 搭配 GQA 滑动窗口 draft 模型导致 KV cache 页面统一失败 (#13213)。
*   **社区流程优化讨论**：针对近期 RC 版本质量压力上升的问题（如 0.23.0.rc1 收到 67 个 bug），社区提议优化 RC 发布流程，引入测试与资料 Maintainer，并建立发布质量评审机制 (#13217)。

---

### 🚀 PR 动态
近期合并/提交的 PR 涵盖重要 Bug 修复、推测解码性能优化及 CI 建设等：

**🛠️ 重要修复**
*   **修复同步超时误报**：为 ACL Graph replay 中的 `synchronize()` 添加异常处理，修正 107020/107027 超时被误分为 OOM 的问题 (#13219，对应 Issue #13218)。
*   **修复 active_tokens 负数问题**：在 `_release_load` 中为 `active_tokens` 增加 `max(0)` 的钳位限制，防止数值减为负数 (#13215，对应 Issue #13214)。
*   **修复 RL 训练权重同步**：修复 RL disaggregated 训练模式下，`NPUWorker` 仅同步 target 模型权重而遗漏 MTP draft 模型权重的问题 (#13209)。
*   **解耦 MiniMax M3 缓存处理**：从 `model_runner_v1.py` 中移除 MiniMax M3 模型的特定依赖，优化代码结构（针对 v0.24.0rc 分支）(#13208)。

**✨ 新特性与性能优化**
*   **DSpark 推测解码增强**：为验证流水线引入置信度头和动态长度调度，利用置信度估算 token 来动态调整验证长度 (#13216)。
*   **减少冗余计算**：[Draft] 为 DSA 上下文并行引入 OPT3 Compressor 序列并行实现，减少预填充阶段的冗余计算 (#13212)。

**🤖 CI 与工程化**
*   **上游同步**：适配 vLLM main 分支最新代码（截至7月25日）(#13207)。
*   **测试用例**：新增 Qwen3.6-35B-A3B 模型的每周性能与精度测试 (#13211)；降低 Qwen3.5 MTP nightly 精度阈值以减少因用例波动导致的 CI 误报 (#13210)；修复提取 PR 修改测试用例信息的脚本 Bug (#13206)。

---

### 📦 Release 动态
*   近期**无新版本发布**。（注：PR #13208 提及了 `releases/v0.24.0rc` 分支的修复，表明 v0.24.0 正在 RC 准备阶段）。!

---

## 🐛 Issues

### #13218 — [[Bug]: NPU stream synchronize timeout (ACL 107020/107027) on aclgraph replay is misclassified as OutOfMemoryError](https://github.com/vllm-project/vllm-ascend/issues/13218)
- **作者**: sicnuyudidi  **时间**: 2026-07-31 07:35 CST
- **标签**: core-features, aclgraph
- **摘要**: ## Summary  During acl graph **replay**, `ACLGraphWrapper.__call__` calls `torch.npu.current_stream().synchronize()` with no exception handling. When the synchronize call fails with ACL `107020` (`AclrtSynchronizeStreamWithTimeout`) or `107027` (synchronize on a captured stream), the `RuntimeError` …

### #13217 — [[Performance]: [讨论] 优化 RC 发布流程：引入测试与资料 Maintainer，建立发布质量评审机制](https://github.com/vllm-project/vllm-ascend/issues/13217)
- **作者**: hl843901190  **时间**: 2026-07-31 04:17 CST
- **标签**: performance
- **摘要**: ### Proposal to improve performance  ### 1、背景与问题 vLLM-Ascend 社区正处于快速成长阶段。自 0.18.0 release 版本发布以来，社区已迭代 5 个 RC 版本，伴随使用量增长，质量压力也在同步上升——仅 0.23.0.rc1 粗略统计即收到 67 个 bug 类 issue。  这种高速增长的社区规模，意味着每一次 RC 发布都直接影响数万开发者的使用体验。然而，当前 RC 发布流程中缺少一个系统性的质量检查环节：Release Manager 承担了代码审查、业务特性开发、版本发布等复合职责，很难同时逐一追踪社区 Issue …

### #13214 — [[Bug]: decoder active_tokens can go negative after release](https://github.com/vllm-project/vllm-ascend/issues/13214)
- **作者**: hongweiyi2026  **时间**: 2026-07-30 23:16 CST
- **标签**: bug
- **摘要**: ### Your current environment  ##   <details> <summary>The output of <code>python collect_env.py</code></summary>   ```text - OS: Ubuntu 24.04 (WSL2) - Python: 3.12.3 - vLLM: 0.25.1 - Hardware: NVIDIA RTX 5070 Ti 16GB - Note: the reported defect is in pure-Python scheduling logic (heapq + httpx),   h…

### #13213 — [[Bug][GLM-5.2][DSpark][A2] MLA target + GQA sliding-window draft fails KV cache page unification](https://github.com/vllm-project/vllm-ascend/issues/13213)
- **作者**: yanceng305-collab  **时间**: 2026-07-30 23:12 CST
- **摘要**: ### Your current environment  - Hardware: 4 × Atlas 800I A2 nodes, each with 8 × Ascend 910B1 - Container: `quay.io/ascend/vllm-ascend:nightly-main` - Target model: `Eco-Tech/GLM-5.2-w8a8`-compatible W8A8 checkpoint - Draft model: `RedHatAI/GLM-5.2-speculator.dspark` - Parallelism: `DP4 / local DP1 …

## 🔀 Pull Requests

### #13219 — [[Bugfix] Classify ACL stream synchronize timeout (107020/107027) during aclgraph replay](https://github.com/vllm-project/vllm-ascend/pull/13219)
- **作者**: sicnuyudidi  **时间**: 2026-07-31 07:35 CST
- **标签**: documentation, ci/build, module:tests, module:ops, module:core, module:quantization, merge-conflicts
- **摘要**: ## Summary  During acl graph **replay**, `ACLGraphWrapper.__call__` calls `torch.npu.current_stream().synchronize()` with no exception handling. When the synchronize call fails with ACL `107020` (`AclrtSynchronizeStreamWithTimeout`) or `107027` (synchronize on a captured stream), the `RuntimeError` …

### #13216 — [Add Confidence head and Dynamic Lengths to Verification Pipeline](https://github.com/vllm-project/vllm-ascend/pull/13216)
- **作者**: StanislavII  **时间**: 2026-07-31 01:51 CST
- **标签**: module:core
- **摘要**: ## Summary  This push adds confidence-based dynamic verification length scheduling for the DSpark speculative decoding path on Ascend.  The implementation uses the DSpark confidence head to estimate token survival probabilities and dynamically distribute a verification-token budget across active req…

### #13215 — [[BugFix] Clamp decoder active_tokens on release to prevent negative values](https://github.com/vllm-project/vllm-ascend/pull/13215)
- **作者**: hongweiyi2026  **时间**: 2026-07-30 23:23 CST
- **摘要**: ### What this PR does / why we need it?  In `_release_load`, the `active_tokens` branch subtracts `load` **without a `max(0)` clamp**, while the adjacent `kv_cache` branch is clamped:  ```python if active_tokens:     entry.active_tokens -= load                                   # can go negative if …

### #13212 — [[Draft][Performance] Reduce redundant DSA Compressor computation with sequence parallelism](https://github.com/vllm-project/vllm-ascend/pull/13212)
- **作者**: chuanbowang2026  **时间**: 2026-07-30 22:55 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it  This PR adds the current OPT3 Compressor sequence-parallel implementation for DSA context parallelism.  The goal is to reduce redundant DSA Compressor work in prefill. Baseline computes `Compressor(full_hidden)` on every rank even though each rank only consume…

### #13211 — [[CI] Add weely performance and accuracy test cases for Qwen3.6-35B-A3B](https://github.com/vllm-project/vllm-ascend/pull/13211)
- **作者**: weixinAc  **时间**: 2026-07-30 21:39 CST
- **标签**: module:tests, model-dataset-download
- **摘要**: ### What this PR does / why we need it?   Add weely performance and accuracy test cases for Qwen3.6-35B-A3B  ### Does this PR introduce _any_ user-facing change? NO  ### How was this patch tested? by the running the test  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/comm…

### #13210 — [[CI]modify qwen3.5 mtp nightly threshold](https://github.com/vllm-project/vllm-ascend/pull/13210)
- **作者**: weiguihua2  **时间**: 2026-07-30 21:03 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? If the use case precision threshold is set too high and the use cases fluctuate, it may cause the use cases to fail. Lower the threshold to maintain the same settings as in the 0.23.0 branch. In later versions, the PCP function will be temporarily taken offlin…

### #13209 — [[BugFix] Fix MTP draft model weight sync in RL training disaggregated mode](https://github.com/vllm-project/vllm-ascend/pull/13209)
- **作者**: CalvinXKY  **时间**: 2026-07-30 20:30 CST
- **摘要**: ### What this PR does / why we need it? In RL disaggregated training, `NPUWorker` only synced target-model weights and skipped the MTP draft model (`model_runner.drafter`). After the first weight update, draft stayed at HF init while target moved, so MTP acceptance dropped to 0% and rollout became g…

### #13208 — [[v0.24.0rc][BugFix][ModelRunner] Decouple MiniMax M3 runner cache handling](https://github.com/vllm-project/vllm-ascend/pull/13208)
- **作者**: muziyuhui666  **时间**: 2026-07-30 20:15 CST
- **摘要**: ## What this PR does / why we need it?  This PR is for the `releases/v0.24.0rc` branch.  It removes MiniMax M3 model-specific dependencies from `vllm_ascend/worker/model_runner_v1.py`.  Previously, `model_runner_v1.py` directly imported MiniMax M3 modeling classes to handle sparse attention and inde…

### #13207 — [[Misc]feat: adapt to vLLM main (0b0bd2b5)](https://github.com/vllm-project/vllm-ascend/pull/13207)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-30 19:55 CST
- **标签**: ready
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to July 25.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | `vllm_ascend/attention/utils.py`<br>`vllm_ascend/worker/npu_input_batch.py…

### #13206 — [[CI]Fix the bug in extracting modified test case information from the PR. ](https://github.com/vllm-project/vllm-ascend/pull/13206)
- **作者**: shenhui-cli  **时间**: 2026-07-30 19:34 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/d02df748bf9efd99022f1a062597dc3cb3808485
