# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-21 15:37 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 最近动态的中文摘要：

### 🐛 Issue (问题反馈与讨论)
*   **#22725 [Bug] 静态内核编译缓存失效与目录膨胀：** 开启 `enable_static_kernel` 后，因未向 `torch_npu` 正确传入 `use_cache_compile` 缓存键，导致每次重启均触发全量重编，编译产物无限累积（基于 Atlas 800I A3 环境）。
*   **#22724 [Bug] DeepSeek V4.1-Flash 推理轨迹偏移：** 在应用 PCP local row compression 相关优化后，长 search-RAG prompt 在贪婪解码下会确定性地进入输出空内容的推理死循环。
*   **#22719 [RFC] 为 KDA RecoverSSM 定义无 epsilon 的 L2 归一化：** 提议在 A3 和 A5 硬件上统一 RecoverSSM 的 Q/K 归一化规则：非零向量执行标准 L2 归一化，零向量返回零。

### 🔧 Pull Request (代码合并请求)
**新特性与性能优化：**
*   **#22722 [Profiler] 新增算子级调用栈支持：** 允许 Ascend 用户在使用 `torch_npu.profiler` 时可选开启算子级调用栈分析功能。
*   **#22726 [Doc] 扩展 Kimi-K2.7-Code 适配：** 为 `Kimi-K2.7-Code` 新增 Atlas 800I A2 (910B) 硬件下 W4A8 检查点的 PD（Prefill/Decode 分离）部署支持文档。
*   **#22720 [Performance] 跨层复用 A5 C8 Indexer 元数据：** 改用 CANN V2 接口，使得兼容层的 C8 indexing 批次元数据只需准备一次，提升执行效率。
*   **#22723 [Performance] 恢复并稳定 QSA 准备融合：** 在主干分支恢复 QSA preparation 优化，并修复了影响旧版 rotary callers 的 JIT 特化输入问题。

**关键 Bug 修复：**
*   **#22721 [BugFix] 修复 RecoverSSM 归一化：** 落实 #22719 的方案，在 A3/A5 的 RecoverSSM 验证及提交逻辑中应用无 epsilon 的 L2 归一化。
*   **#22718 [BugFix] 修复调度器游标错位：** 修复在 `PRIORITY` 优先级抢占发生时，KV 块分配失败导致的调度游标不正确的问题。
*   **#22716 [BugFix] 修复 HiSparse 路径偏离：** 修复 HMA 非页对齐步长和 PCP padding 场景下，C8 resident prefill 意外偏离 NPU 常驻快速路径的问题。

**测试与 CI 维护：**
*   **#22717** 调整 MiniMax-M3 PP PD nightly 任务端口以修复重试失败。
*   **#22714** 为 DeepSeek V4 Flash A5 nightly 测试配置 `CCU_SCHED` 环境变量。
*   **#22715** 主分支常规 nightly 验证自动 PR。

### 🚀 Release (版本发布)
*   *本期时间段内无新版 Release 发布。*

---

## 🐛 Issues

### #22725 — [[Bug]: 开启 enable_static_kernel 后每次重启全量重编 static kernel（未向 torch_npu 传入 use_cache_compile 缓存键），编译产物目录无限累积](https://github.com/vllm-project/vllm-ascend/issues/22725)
- **作者**: xuyucheng220  **时间**: 2026-11-21 15:04 CST
- **摘要**: ### 环境信息  ```text - vllm-ascend: 0.19.1rc2.dev3585+g568d12495 (nightly main, 2026-11-16 构建) - vllm: 0.33.0+empty（源码位于 /vllm-workspace/vllm） - torch-npu: 2.10.0.post7 - CANN: 9.2.1 - 硬件: Atlas 800I A3 - 模型: GLM-5.3 W8A8C8, decode TP4/DCP4/DP8, MTP=3 - 配置: additional_config.ascend_compilation_config.e…

### #22724 — [[Bug] DeepSeek V4.1-Flash PCP local row compression (#19362 + #20374) shifts greedy decoding trajectories: long search-RAG prompt enters deterministic reasoning loop with empty content](https://github.com/vllm-project/vllm-ascend/issues/22724)
- **作者**: Mrxiashu  **时间**: 2026-11-21 14:52 CST
- **摘要**: ## Summary  After applying #19362 and #20374 (PCP prefill local row compression for DeepSeek V4.1-Flash) on top of v0.32.0rc1, a specific greedy (temperature=0) reasoning request deterministically enters an infinite enumeration loop inside `reasoning_content` and returns **empty content**. The same …

### #22719 — [[RFC] Define epsilon-free L2 normalization for KDA RecoverSSM](https://github.com/vllm-project/vllm-ascend/issues/22719)
- **作者**: drslark  **时间**: 2026-11-21 14:34 CST
- **摘要**: ### Motivation  Define one Q/K normalization contract for Ascend KDA RecoverSSM on A3 and A5. For a nonzero vector `x`, use `x / sqrt(sum(x * x))`. Return zero for a zero vector. Apply the same rule in verify, regular commit, and grouped commit.  #22523 aligned RecoverSSM with the then-current A3 wh…

## 🔀 Pull Requests

### #22726 — [[Doc][Recipe] add Ascend 910B (Atlas 800I A2) W4A8 PD support to Kimi-K2.7-Code overlay](https://github.com/vllm-project/vllm-ascend/pull/22726)
- **作者**: troubadourHUI  **时间**: 2026-11-21 15:09 CST
- **标签**: recipe
- **摘要**: ### What this PR does / why we need it?  Adds a `moonshotai/Kimi-K2.7-Code` overlay with validated **Ascend 910B** (Atlas 800I A2 / 910B4) support for the W4A8 checkpoint in prefill/decode disaggregation, expressed with the generic PD pools introduced in #1226.  Why: Kimi-K2.7-Code is a 1T-parameter…

### #22723 — [[Performance][Qwen4Exp] Restore QSA preparation fusion and stabilize legacy JIT parameters](https://github.com/vllm-project/vllm-ascend/pull/22723)
- **作者**: Zoezxb  **时间**: 2026-11-21 14:50 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Restore the QSA preparation optimizations from #22086 on current main, then address an unnecessary JIT specialization input affecting legacy rotary callers. The branch contains two commits for separate review:  1. Restore #22086 after its rollback in #22633, …

### #22722 — [[Feature][Profiler] Add opt-in operator call stacks for the torch_npu profiler](https://github.com/vllm-project/vllm-ascend/pull/22722)
- **作者**: changan593  **时间**: 2026-11-21 14:46 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Allow Ascend users to opt in to operator-level call stacks when profiling with `torch_npu.profiler`. The existing vLLM `torch_profiler_with_stack` setting enables module hierarchy collection on Ascend. The Ascend option controls the additional cost of operato…

### #22721 — [[BugFix] Use epsilon-free L2 normalization for KDA RecoverSSM](https://github.com/vllm-project/vllm-ascend/pull/22721)
- **作者**: drslark  **时间**: 2026-11-21 14:36 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Fixes #22719.  Normalize nonzero Q/K to unit L2 norm in RecoverSSM verify, regular commit, and grouped commit on A3 and A5. Preserve zero vectors with a zero-norm guard. Use the independent PyTorch FP64 recurrence as the KDA arithmetic reference for small-nor…

### #22720 — [[Performance] Reuse A5 C8 indexer metadata across layers](https://github.com/vllm-project/vllm-ascend/pull/22720)
- **作者**: danziheng1024  **时间**: 2026-11-21 14:35 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  A5 C8 indexing currently uses the older QLI interface in each layer. This change uses the CANN V2 interface and prepares its batch metadata once per compatible indexer cache group during metadata build, so all layers in the group consume the same result.  - U…

### #22718 — [[BugFix][Scheduler] Keep the scheduling cursor correct on PRIORITY preemption](https://github.com/vllm-project/vllm-ascend/pull/22718)
- **作者**: alex7092  **时间**: 2026-11-21 14:32 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Fixes #22017.  The Ascend schedulers are copies of upstream vLLM's `Scheduler.schedule()`. When a running request cannot allocate its KV blocks under `SchedulingPolicy.PRIORITY`, the lowest-priority request is preempted and removed from `self.running`.  `self…

### #22717 — [[CI] Change MiniMax-M3 PP PD prefill KV port for nightly retry](https://github.com/vllm-project/vllm-ascend/pull/22717)
- **作者**: yingyingzizi  **时间**: 2026-11-21 14:25 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  The [MiniMax-M3-w8a8-PP-prefix-PD nightly job](https://github.com/vllm-project/vllm-ascend/actions/runs/44783873338/job/136641842926) failed before Prefill became ready: a KV sending worker could not bind port 36001 and raised `zmq.error.ZMQError: Address alr…

### #22716 — [[BugFix][HiSparse] Support TND resident prefill for HMA stride and PCP padding](https://github.com/vllm-project/vllm-ascend/pull/22716)
- **作者**: zyz111222  **时间**: 2026-11-21 14:22 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Fixes `_try_c8_resident_prefill` in `vllm_ascend/attention/sfa_hisparse.py` so C8 resident prefill stays on the NPU resident fast path under HMA non-page-aligned strides and PCP padding instead of falling back 100% to Host-staging.  #### Root Cause 1. **HMA n…

### #22715 — [[Verify] Nightly test on main](https://github.com/vllm-project/vllm-ascend/pull/22715)
- **作者**: LQDLove  **时间**: 2026-11-21 14:13 CST
- **标签**: documentation
- **摘要**: Auto-created empty PR for nightly verification. - vLLM main: https://github.com/vllm-project/vllm/commit/c410543d9ffff03f5f6c4a8c0ae1d3ad082c4a15

### #22714 — [[Test] Use CCU_SCHED for DeepSeek V4 Flash A5 nightly](https://github.com/vllm-project/vllm-ascend/pull/22714)
- **作者**: LXX1122  **时间**: 2026-11-21 14:11 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Set `HCCL_OP_EXPANSION_MODE: "CCU_SCHED"` in the shared environment of `tests/SIT/cases/models/configs/DeepSeek/DeepSeek_V4_Flash_A5.yaml` so the `DeepSeek_V4_Flash_A5` nightly suite runs with CCU scheduling.  The variable was previously unset in this config.…
