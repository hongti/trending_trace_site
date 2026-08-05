# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-05 09:01 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🚀 Release（版本发布）
本次输入数据中**未包含**新的 Release 版本发布信息。
*(注：PR #51090 提到了 Deepseek v4 v0.26.0，属于针对特定模型的支持更新，已归入下方 PR 新特性中)*

### 🐛 Issue（问题）
本次输入数据中**未包含** Issue 动态。

### 🔧 Pull Request（代码合并请求）
近期 PR 主要集中在**新模型/特性支持、核心 Bug 修复、性能分析器优化及 CI 构建改进**四个方面：

**1. 新特性与模型支持**
*   **支持 Deepseek v4**：引入了对 Deepseek v4 v0.26.0 的支持 (#51090)。
*   **HTTP 请求优先级解析**：新增从 HTTP Header 解析请求优先级的功能，优化调度控制 (#51089)。
*   **ROCm MLA 解码优化**：为 Kimi-K3 TP8 新增了小头（small-head <16）PS ASM 解码路由，替代原有的 Gluon 路径以提升性能 (#51088)。

**2. 核心 Bug 修复**
*   **量化权重维度丢失**：修复了 ModelOpt FP8 在转置序列化权重时，未保留原始权重维度信息的问题 (#51093)。
*   **推测解码崩溃**：修复了 EAGLE3 DeepSeek 在非 YaRN RoPE 配置下运行草稿模型时崩溃的问题 (#51092)。
*   **KV Cache 数据损坏**：修复了在 KV-sharing 模型（如 Gemma-4, Gemma-3n）使用 `int8_per_token_head` 等 KV 缓存时，引擎启动后前几个请求生成数据损坏的问题 (#51091)。
*   **ROCm MLA 潜在缺陷**：修复了现有 head-padding 辅助函数中的潜在 Bug (#51088)。

**3. Profiler（性能分析器）优化**
*   **分布式分析事务化**：使分布式性能分析过程具备事务性，确保数据一致性 (#51085)。
*   **CUDA Graph 归因**：为 Proton 分析器添加了 CUDA Graph 归因功能 (#51084)。
*(注：以上两项均依赖于前置 PR #48789)*

**4. CI 与构建改进**
*   **新增 CI 触发指令**：为受信任用户和 PR 作者添加了未公开的评论指令 `/ci run all` 和 `/ci run nightly` (#51087)。
*   **依赖版本锁定**：将 AITER 依赖锁定至已知良好 commit，防止代码漂移破坏构建 (#51086)。

---

## 🔀 Pull Requests

### #51093 — [[Bugfix][Quantization] Preserve ModelOpt FP8 weight dimensions](https://github.com/vllm-project/vllm/pull/51093)
- **作者**: netanel-haber  **时间**: 2026-08-05 09:00 CST
- **摘要**: ### Purpose  ModelOpt FP8 transposes serialized weights from `[N, K]` to `[K, N]` before dispatching to the selected linear kernel, but the replacement `Parameter` does not preserve the weight's dimension metadata. Layout-aware backends then fall back to interpreting the tensor as `[N, K]`.  Record …

### #51092 — [[Bugfix][Spec Decode] Fix EAGLE3 DeepSeek draft crash on non-YaRN rope configs](https://github.com/vllm-project/vllm/pull/51092)
- **作者**: zixi-qi  **时间**: 2026-08-05 08:54 CST
- **标签**: bug, speculative-decoding, deepseek
- **摘要**: ## Purpose  `DeepseekV2Eagle3DecoderLayer.__init__` rebuilds `config.rope_parameters` from `config.rope_scaling` before handing the config to `DeepseekV2MLAAttention`:  ```python rope_scaling = getattr(config, "rope_scaling", None) ... config = copy.copy(config) if rope_scaling:     rope_params = ro…

### #51091 — [[Bugfix] Initialize per-token-head KV scale regions once per storage](https://github.com/vllm-project/vllm/pull/51091)
- **作者**: haregali  **时间**: 2026-08-05 08:42 CST
- **标签**: bug
- **摘要**: ## Purpose  With `int8_per_token_head` (or fp8/int4 per-token-head) KV cache on KV-sharing models (Gemma-4, Gemma-3n), the first `max_num_seqs` requests after engine start generate corrupted leading tokens. Likely the root cause of #50702 and #50749. Supersedes #51061, closed while we finished verif…

### #51090 — [Deepseek v4 v0.26.0](https://github.com/vllm-project/vllm/pull/51090)
- **作者**: Baekpica  **时间**: 2026-08-05 08:18 CST
- **标签**: structured-output, needs-rebase, ci/build, deepseek, nvidia, quantization
- **摘要**: ## Purpose  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such as providing test command. - [ ] The …

### #51089 — [[Feature] Parse request priority from HTTP header](https://github.com/vllm-project/vllm/pull/51089)
- **作者**: chaunceyjiang  **时间**: 2026-08-05 08:12 CST
- **标签**: frontend
- **摘要**: FIX https://github.com/vllm-project/vllm/issues/51023  ## Purpose  Parse request priority from HTTP header ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing…

### #51088 — [[ROCm][MLA] Add small-head PS ASM decode route (Kimi-K3 TP8)](https://github.com/vllm-project/vllm/pull/51088)
- **作者**: rebklee  **时间**: 2026-08-05 07:09 CST
- **标签**: rocm, kimi, k3
- **摘要**: ## Summary Adds an opt-in path that routes small-head (<16) AITER MLA decode through the persistent-scheduling (PS) ASM kernel instead of Gluon, and fixes a latent bug in the existing head-padding helpers so non-divisor head counts (e.g. 12 heads/rank) pad correctly.  The path is switched on using t…

### #51087 — [[CI] Add run-all comment commands](https://github.com/vllm-project/vllm/pull/51087)
- **作者**: khluu  **时间**: 2026-08-05 06:36 CST
- **标签**: ci/build
- **摘要**: ## Purpose  Add two exact, intentionally undocumented CI comment variants for trusted users and authorized PR authors:  - `/ci run all` triggers a Buildkite build with `RUN_ALL=1`. - `/ci run nightly` triggers a Buildkite build with `RUN_ALL=1` and `NIGHTLY=1`.  Both variants use the existing author…

### #51086 — [Pin AITER for 1250 build](https://github.com/vllm-project/vllm/pull/51086)
- **作者**: jpvillam-amd  **时间**: 2026-08-05 06:28 CST
- **标签**: rocm, ci/build
- **摘要**: Pinning AITER to known good commit to preserve functionality   ## Purpose Pin aiter on build to prevent code drift ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link …

### #51085 — [[Profiler] Make distributed profiling transactional](https://github.com/vllm-project/vllm/pull/51085)
- **作者**: Luosuu  **时间**: 2026-08-05 06:09 CST
- **标签**: documentation, performance, frontend
- **摘要**: ## Purpose  **Depends on #48789. Please review and merge #48789 first.**  This PR currently shows the eager Proton backend from #48789 as well because the parent has not merged yet. Once #48789 merges, I will rebase this branch onto `main` and force-push; the diff will reduce to the single reliabili…

### #51084 — [[Profiler] Add Proton CUDA graph attribution](https://github.com/vllm-project/vllm/pull/51084)
- **作者**: Luosuu  **时间**: 2026-08-05 06:09 CST
- **标签**: documentation, performance, nvidia
- **摘要**: ## Purpose  **Depends on #48789. Please review and merge #48789 first.**  This PR currently shows the eager Proton backend from #48789 as well because the parent has not merged yet. Once #48789 merges, I will rebase this branch onto `main` and force-push; the diff will reduce to the single CUDA grap…
