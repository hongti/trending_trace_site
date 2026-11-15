# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-15 15:01 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue (问题反馈)
*   **Profiler 截断问题 (#21904)**：`torch_npu profiler` 在处理长上下文或多请求捕获时，会静默生成截断的跟踪数据，且 `_stop` 阶段缺乏输出完整性检查。
*   **310P SplitFuse 异常 (#21902)**：在 Atlas 300I DUO (310P3) 设备上，当上下文长度为 2049 时，压缩 SplitFuse 存在错误读取未来 V 值的 Bug。

### 🛠️ Pull Request (代码提交)
**性能优化**
*   **MoE 通信优化 (#21907)**：消除了 `All2AllV` dispatch split 中的两次主机同步阻塞，提升 MoE 模型调度效率。
*   **Qwen4Exp 模型提速 (#21901)**：针对 A3 芯片新增了大批次行块 Triton kernel，优化了 Qwen4Exp HyperConnection 的 `gate_mix` 性能。
*   **DeepSeek V4.1 解码优化 (#21896)**：在非打包缓存执行时，使用“首消费者 SWA 优先级”策略捕获原生解码元数据，移除了对 A5 打包缓存后端的硬性限制。

**重构与修复**
*   **思考预算修复 (#21906)**：完善了 Ascend Model Runner V2 上 thinking budget 的缺失修复与验收测试覆盖率。
*   **KV 缓存分配重构 (#21903)**：使 MRV2 Kimi K3 复用此前为 DeepSeek V4/V4.1 引入的统一描述符驱动 KV 缓存分配器。
*   **KVTransfer 健壮性 (#21895)**：当客户端在 KV push 期间中止导致请求未被跟踪时，Mooncake `KVCacheTaskTracker` 会发出警告，避免静默丢弃条目。

**文档与 CI**
*   **CI 修复 (#21905)**：跳过容器化 `pre-commit` 作业中引发故障的主机 problem-matcher 注册。
*   **文档清理 (#21900, #21898)**：修复了 14 个失效的页内锚点；从部署指南中移除了 vLLM Ascend 实际上从不读取的无效环境变量。
*   **杂项 (#21899)**：修正了不符合命名规范的 GDN checkpoint release notes 片段文件名。

### 🚀 Release (版本发布)
*   本次监控周期内无新的 Release 发布。

---

## 🐛 Issues

### #21904 — [[Bug]: torch_npu profiler silently produces truncated traces for long-context / multi-request captures (no output integrity check in _stop)](https://github.com/vllm-project/vllm-ascend/issues/21904)
- **作者**: wuming2027  **时间**: 2026-11-15 11:53 CST
- **摘要**: ### Your current environment  `collect_env.py` output is not available from the benchmark host at the moment (will be attached in a follow-up comment). Environment details from our deployment record:  - Hardware: Atlas A3 single node, 16 dies; tested topologies `DP4 x TP4 x EP16` and `DP2 x TP8 x EP…

### #21902 — [[Bug]: 310P compressed SplitFuse reads future V at context length 2049](https://github.com/vllm-project/vllm-ascend/issues/21902)
- **作者**: ychy7001  **时间**: 2026-11-15 09:58 CST
- **标签**: 310p
- **摘要**: # [Bug]: 310P compressed SplitFuse reads future V at context length 2049  ### Your current environment  Atlas 300I DUO (Ascend 310P3), reproduced independently on both chips. CANN 9.2.1, PyTorch 2.10.0+cpu, torch-npu 2.10.0.post7. The image has vllm-ascend revision `76ea9d24f155afe032ad9ac6c98717f1f…

## 🔀 Pull Requests

### #21907 — [[Performance][MoE] Eliminate All2AllV dispatch split host sync](https://github.com/vllm-project/vllm-ascend/pull/21907)
- **作者**: diaochengxi  **时间**: 2026-11-15 14:26 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  `TokenDispatcherWithAll2AllV._preprocess` currently blocks the host twice per MoE dispatch:  - `torch.repeat_interleave(..., num_global_tokens_per_local_expert.ravel())`   infers its output length from device values, causing an implicit D2H copy and   stream …

### #21906 — [[BugFix] Complete V2 thinking budget validation and acceptance coverage](https://github.com/vllm-project/vllm-ascend/pull/21906)
- **作者**: jryxxx  **时间**: 2026-11-15 13:38 CST
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Complete the missing fixes and acceptance coverage for thinking budgets on Ascend Model Runner V2, while retaining upstream `ThinkingBudgetState` and sampling behavior.  This PR:  - Fixes signed-int32 overflow in token-history addressing by widening request-s…

### #21905 — [[CI] Avoid host problem-matcher registration in containerized pre-commit](https://github.com/vllm-project/vllm-ascend/pull/21905)
- **作者**: ychy7001  **时间**: 2026-11-15 12:01 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it?  Skip optional custom problem-matcher registration in the containerized `pre-commit` job. In PR #21864, the Kubernetes runner tried to resolve the container's `$RUNNER_TEMP/mypy.json` on the host and could not find it. GitHub marked the step failed before lint…

### #21903 — [[Refactor][MRV2] Reuse unified KV cache allocation for Kimi K3](https://github.com/vllm-project/vllm-ascend/pull/21903)
- **作者**: yxk-fdu  **时间**: 2026-11-15 11:05 CST
- **标签**: documentation, module:tests, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  Route MRV2 Kimi K3 through the descriptor-driven allocator introduced for DeepSeek V4/V4.1. The worker uses the existing cache placements to allocate backing storage. KDA state views come from `MambaSpec`. MLA initialization builds the kernel-block views requ…

### #21901 — [[Performance][Model] Tile Qwen4Exp HC gate mix on A3](https://github.com/vllm-project/vllm-ascend/pull/21901)
- **作者**: diaochengxi  **时间**: 2026-11-15 09:56 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  This PR adds a large-batch row-block Triton kernel for Qwen4Exp HyperConnection `gate_mix`.  - The existing one-row kernel remains the default for small batches. - When `rows >= 1024` and the hardware profile supports `QWEN4EXP_HC_GATE_MIX_ROW_BLOCK`, the new…

### #21900 — [[Doc] Fix broken in-page anchors in docs](https://github.com/vllm-project/vllm-ascend/pull/21900)
- **作者**: shuxiachai  **时间**: 2026-11-15 08:32 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Fixes 14 in-page links whose anchors do not exist, so clicking them just lands at the top of the page.  The docs site is built with MkDocs, whose default slugify (Python-Markdown `toc`) collapses runs of spaces and hyphens into a single hyphen. Several links …

### #21899 — [[Misc] Rename GDN checkpoint release note fragment to its PR number](https://github.com/vllm-project/vllm-ascend/pull/21899)
- **作者**: shuxiachai  **时间**: 2026-11-15 08:26 CST
- **摘要**: ### What this PR does / why we need it?  `release_notes/fragments/pending-gdn-internal-checkpoints.yaml` (added in #21040, `pr: 21040`) does not follow the fragment naming rule in `release_notes/README.md`: `<PR number>.yaml`, or `<PR number>-<slug>.yaml` for a second file.  `collect_release_notes.p…

### #21898 — [[Doc] Remove env vars that vLLM Ascend never reads from deployment guides](https://github.com/vllm-project/vllm-ascend/pull/21898)
- **作者**: shuxiachai  **时间**: 2026-11-15 08:05 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Two deployment guides export environment variables that no vLLM Ascend code on `main` reads, so setting them has no effect:  | File | Removed line | Why it has no effect | |---|---|---| | `docs/source/tutorials/models/GLM5.2.md` (3 launch scripts: 5.1.3, 5.1.…

### #21896 — [[Performance][DSV4.1] Capture native decode metadata with first-consumer SWA priority](https://github.com/vllm-project/vllm-ascend/pull/21896)
- **作者**: KeyanHu-git  **时间**: 2026-11-15 03:32 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  On non-packed-cache (native) DeepSeek-V4.1-Flash `FULL_DECODE_ONLY` execution, `main` currently restricts `TargetDeviceMetadata` to the A5 packed-cache backend, leaving native slot coordinate conversion, C2 compressed lengths, and SMLA / QLI metadata preparat…

### #21895 — [[Misc][KVTransfer] Warn on untracked delayed-free requests in Mooncake KVCacheTaskTracker](https://github.com/vllm-project/vllm-ascend/pull/21895)
- **作者**: lcz479  **时间**: 2026-11-15 02:39 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  `KVCacheTaskTracker.add_delayed_request` drops the entry when the request is not tracked in `reqs_to_process` (e.g. the client aborted between the KV push and the metadata handling). The drop itself must stay: recording such an entry could publish finished_se…
