# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-19 12:55 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要：

### 🐛 Issue 动态
近期主要报告了 2 个问题，均已有对应的修复 PR：
1. **CI 失败问题 (#57653)**：在 MI250 多模态处理器上运行 DeepSeek-V4.1-Flash 的多模态处理正确性测试时失败。
2. **资源释放缺陷 (#57650)**：在 v0.26.0 版本中，若调度器初始化失败，Worker 进程不会主动退出或释放资源，导致资源泄漏。

---

### 🔧 PR 动态
近期共有 10 个 PR，涵盖核心缺陷修复、性能优化及模型适配，主要亮点如下：

**1. 核心缺陷修复**
*   **引擎与调度器修复 (#57657)**：修复了 EngineCore 初始化失败时执行器 Worker 进程未关闭的问题（对应 Issue #57650），防止资源挂起。
*   **KV 缓存修复 (#57656)**：修复了启动时 `--kv-cache-memory` 建议值未考虑其他进程启动后内存分配导致的问题。
*   **投机解码修复 (#57658)**：修复了调度器选择动态投机 Token 数量（K）时，Hybrid Mamba KV-cache 准入机制的准确性问题。

**2. 模型与多模态支持**
*   **DeepSeek V4.1 导入修复 (#57654)**：解决了在无 Triton 环境下，因 `tl.constexpr` 回退为 None 导致 DeepSeek V4.1 无法导入的问题（对应 Issue #57653）。
*   **多模态音频丢失修复 (#57655)**：修复了 Dots3 NOTE 模型在多模态处理器缓存未命中时，错误丢弃视频音轨的问题。

**3. 性能优化与分布式改进**
*   **DeepSeek-V4.1 性能优化 (#57659)**：在压缩注意力层中跨消费者层复用解码 top-k 全局索引，避免重复计算，提升性能。
*   **Engram 内存共享 (#57651)**：Engram 默认在共置的 DP 副本间共享宿主机表，节省 CPU 内存并消除每步的 DP 集体通信。
*   **KV 卸载与 TP 优化 (#57652)**：扩展了 `replicated_layout` 检测，支持多组 MLA 模型的 KV 卸载，减少张量并行时的冗余卸载。
*   **DP 依赖解耦 (#57648)**：移除了 `add_dp_placement_groups` 对 `ray[default]` 开发者 API 的依赖。

**4. CI 修复**
*   **测试阈值修正 (#57647)**：修正了 Laguna DFlash CI 测试中不合理的 `expected_acceptance_len` 参考值，修复误报。

---

### 🚀 Release 动态
*   本期输入数据中未包含 Release 相关信息。

---

## 🐛 Issues

### #57653 — [[CI Failure]: MI250 Multimodal Processor - test_common.py::test_processing_correctness[DeepSeek-V4.1-Flash]](https://github.com/vllm-project/vllm/issues/57653)
- **作者**: JiangLLM  **时间**: 2026-09-19 11:27 CST
- **标签**: rocm, multi-modality, ci-failure, deepseek, DSv4.1
- **摘要**: ### Name of failing test  MI250 Multimodal Processor shards 3 and 5:  ```text tests/models/multimodal/processing/test_common.py::test_processing_correctness[1.0-32-0.3-deepseek-ai/DeepSeek-V4.1-Flash] tests/models/multimodal/processing/test_common.py::test_processing_correctness[1.0-32-0.5-deepseek-…

### #57650 — [[Bug]: In version 0.26.0, when scheduler initialization fails, the worker process will not proactively exit or release resources.](https://github.com/vllm-project/vllm/issues/57650)
- **作者**: UESTC-AHao  **时间**: 2026-09-19 10:25 CST
- **标签**: bug, scheduler
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Your output of `python collect_env.py` here ``` [W919 02:16:36.852657847 FunctionLoader.cpp:48] Warning: LD_PRELOAD detected, FunctionLoader prefers RTLD_DEFAULT for symbol resolution…

## 🔀 Pull Requests

### #57659 — [[Perf][DSv4.1] Reuse decode topk global indices across consumer layers](https://github.com/vllm-project/vllm/pull/57659)
- **作者**: Juntian777  **时间**: 2026-09-19 12:53 CST
- **标签**: deepseek, nvidia, DSv4.1
- **摘要**: ## Purpose  On DeepSeek-V4.1, every compressed attention layer maps its index source's local top-k indices to global KV-cache slots on its own (`compute_global_topk_indices_and_lens`), although the indices and the block table are shared by every layer below the same index source. DSv4.1-Flash has 38…

### #57658 — [[Spec Decode] Fix Hybrid Mamba KV-cache admission for dynamic K](https://github.com/vllm-project/vllm/pull/57658)
- **作者**: zlj20020601  **时间**: 2026-09-19 12:08 CST
- **标签**: scheduler, kv-cache-manager
- **摘要**: ## Summary  Fix a correctness issue in Hybrid Mamba KV-cache admission when the existing scheduler selects a dynamic speculative-token count (`K`) based on the current batch size.  The scheduler already supports dynamic `K`, but Mamba state admission was still accounted using the maximum `K`. This o…

### #57657 — [[Bugfix][Core] Shut down executor workers when EngineCore init fails](https://github.com/vllm-project/vllm/pull/57657)
- **作者**: ItsRoy69  **时间**: 2026-09-19 11:59 CST
- **标签**: bug
- **摘要**: ## Purpose  Fixes #57650.   EngineCore.__init__ spawns the executor's worker processes before building the scheduler, so if scheduler construction (or any later step) raises, the `engine_core = EngineCoreProc(...)` assignment in `run_engine_core` never completes and its `finally` cannot call `shutdo…

### #57656 — [[Bugfix] Clamp the kv-cache-memory advisory by memory allocated since startup](https://github.com/vllm-project/vllm/pull/57656)
- **作者**: ssubbotin  **时间**: 2026-09-19 11:45 CST
- **标签**: bug
- **摘要**: ## Purpose  The boot-time suggestion `--kv-cache-memory=N` "to fully utilize gpu memory" is computed from `init_snapshot.free_memory`, taken before the model loads. Anything another process allocates on the device after that snapshot is invisible to the formula: a co-tenant, or a helper the engine i…

### #57655 — [[Bugfix][Multimodal] Preserve Dots3 NOTE video audio on MM processor cache misses](https://github.com/vllm-project/vllm/pull/57655)
- **作者**: waizuichougou  **时间**: 2026-09-19 11:35 CST
- **标签**: bug, multi-modality
- **摘要**: ## Purpose  Fix Dots3 NOTE dropping video audio with the default MM processor cache. The first miss stores a video-only result, so later hits also lose the audio track.  `_get_cache_missing_items()` uses `__getitem__()`, which unwraps the `MediaWithBytes` containing the source video required for aud…

### #57654 — [[Bugfix][CPU] Fix DeepSeek V4.1 import without Triton](https://github.com/vllm-project/vllm/pull/57654)
- **作者**: JiangLLM  **时间**: 2026-09-19 11:27 CST
- **标签**: bug, ready, deepseek, mrv2, DSv4.1
- **摘要**: ## Purpose  Fixes #57653.  DeepSeek V4.1 fails to import when Triton is unavailable. `sparse_mqa_logits.py` calls `tl.constexpr` at import time, but vLLM's fallback sets it to `None`. This blocks model inspection and causes the MI250 processor tests to fail on main.  Make the placeholder's `constexp…

### #57652 — [[KV-Offloading][TP] : Expand replicated_layout detection to multi-group MLA ](https://github.com/vllm-project/vllm/pull/57652)
- **作者**: varun-sundar-rabindranath  **时间**: 2026-09-19 11:25 CST
- **标签**: kv-connector
- **摘要**: ## Purpose KV offloading for MLA models + TP need only offload KV values from one TP rank.  This is controlled by replicated_layout determination in `vllm/distributed/kv_transfer/kv_connector/v1/offloading/config.py`  Building on top of @almogtavor 's work https://github.com/vllm-project/vllm/pull/5…

### #57651 — [[Model][Engram] Share host tables across co-located DP replicas by default](https://github.com/vllm-project/vllm/pull/57651)
- **作者**: Juntian777  **时间**: 2026-09-19 10:56 CST
- **标签**: deepseek, DSv4.1
- **摘要**: ## Purpose  `EngramConfig.dp_shared_memory` (#56512) keeps one copy of the CPU-offloaded Engram tables per node instead of one per DP replica, with no per-step Engram DP collectives, but it had to be requested by hand and failed startup whenever its requirements were not met. This PR turns it on by …

### #57648 — [[Bugfix][DP] add_dp_placement_groups does not require ray[default]](https://github.com/vllm-project/vllm/pull/57648)
- **作者**: beenpow  **时间**: 2026-09-19 09:35 CST
- **标签**: bug, ray
- **摘要**: ## Purpose  Follow-up to #23822.  That PR removed the `ray.util.state.list_nodes()` call from `CoreEngineActorManager.create_dp_placement_groups()` because `list_nodes()` is a developer API backed by the Ray dashboard HTTP server (`127.0.0.1:8265/api/v0/nodes`), which only exists when Ray is install…

### #57647 — [[CI][Bugfix] Correct the Laguna DFlash acceptance-length reference](https://github.com/vllm-project/vllm/pull/57647)
- **作者**: okorzh-amd  **时间**: 2026-09-19 08:05 CST
- **标签**: bug, speculative-decoding, ready, dflash
- **摘要**: `test_dflash_correctness[laguna-nvfp4-mrv2]` enforces `expected_acceptance_len=3.55 * 0.9` = 3.195, which nothing reaches:      AssertionError: acceptance_len 3.051 is below 3.195; accuracy=0.895  The reference is wrong rather than the platform slow. Measured with the same test, 200 GSM8K questions,…
