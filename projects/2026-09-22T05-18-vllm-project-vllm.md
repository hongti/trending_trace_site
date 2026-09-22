# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-22 13:18 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要：

### 📋 Issue 动态
* **[Feature] 优先级调度可观测性 (#58077)**：
  作者 allenz92 提出功能请求，希望为 `--scheduling-policy priority`（优先级调度策略）增加按优先级分类的可观测性功能。目前优先级调度已支持并持续改进（如抢占排序、KV-cache 驱逐和准入控制等），但缺乏针对不同优先级的监控与指标展示。

### 🔧 PR 动态
近期 PR 主要集中在**模型架构优化、核心 Bug 修复与依赖升级**：

**重要优化与新特性：**
* **DeepSeek V4 内存优化 (#58074)**：将 DeepSeek-V4 C128 压缩器状态改为使用循环缓冲区，避免将其误认为可复用的注意力 KV cache，从而减少不必要的内存块分配。
* **数据并行（DP）调度优化 (#58070)**：在数据并行负载中引入结果指标惩罚和超线性在途请求控制，优化整体调度效率。
* **DeepGEMM 架构检查逻辑优化 (#58073)**：在检查 DeepGEMM 可用性前，优先短路检查硬件架构是否支持，避免在不支持的架构上进行无意义的环境/包检查。

**关键 Bug 修复：**
* **MoE 路由器精度问题 (#58076)**：修复 XPU 下 openai/privacy-filter 中 MoE 路由器的精度不匹配问题（gpt-oss 使用 bf16 计算，而 HF 参考实现强制使用 fp32），防止在 logits 相近时因精度差异导致路由翻转。
* **FA4 内核修复 (#58075)**：修复 sparse MLA 中 `flash_attn_varlen_func` 调用 FA4 时未固定 `num_splits` 的问题，解决 Blackwell (SM100/SM110) 架构下 `head_size == 256` 时的报错。
* **工具解析器修复 (#58072)**：修复 `phi4_mini_json` 解析器在 payload 为空时误报工具调用的问题。
* **注意力内存复用修复 (#58068)**：修复固定宽度稀疏索引器预填充 logits 的问题，使得长预填充场景能够正确复用分配器块。
* **NIXL DCP 校验修复 (#58071)**：在读取 `use_mla` 配置前优先短路检查 DCP size，避免非 MLA 模型触发断言错误。
* **xgrammar 空格逻辑修复 (#58067)**：修复 `disable_any_whitespace` 在 xgrammar 后端非但没禁用空格、反而将空格设为必选的严重逻辑错误。

**依赖升级：**
* **FlashInfer 升级 (#58069)**：将 FlashInfer 依赖版本升级至 0.7.0。

### 🚀 Release 动态
* 本次提供的动态列表中**无新版本发布**信息。

---

## 🐛 Issues

### #58077 — [[Feature]: Per-priority observability for --scheduling-policy priority](https://github.com/vllm-project/vllm/issues/58077)
- **作者**: allenz92  **时间**: 2026-09-22 13:17 CST
- **摘要**: Body:  ### 🚀 Feature request  Priority scheduling (`--scheduling-policy priority`) is supported and actively improved (preemption ordering, KV-cache eviction, admission control, etc.), but there is currently **no way to observe it**: all finished-request metrics (`vllm:e2e_request_latency_seconds`, …

## 🔀 Pull Requests

### #58076 — [[XPU][UT] Fix MoE router precision mismatch in openai/privacy-filter](https://github.com/vllm-project/vllm/pull/58076)
- **作者**: ZhouXiaoya25  **时间**: 2026-09-22 13:12 CST
- **标签**: intel-gpu
- **摘要**: gpt-oss computes router logits in bf16, while the HF reference forces the router to use fp32. When the router at tokens with nearly tied logits, the precision gap between bf16 and fp32 can flip which experts get selected, causing a margin at that token position, while most tokens remain unaffected. …

### #58075 — [[Bugfix] Pin num_splits for the FA4 hd256 kernel in sparse MLA](https://github.com/vllm-project/vllm/pull/58075)
- **作者**: seawolf2357  **时间**: 2026-09-22 13:10 CST
- **标签**: bug
- **摘要**: ## Purpose  `SparseMLAAttention._run_masked_mha` calls `flash_attn_varlen_func` with `fa_version=4` but never pins `num_splits`. On Blackwell (SM100/SM110) with `head_size == head_size_v == 256`, FA4 dispatches its dedicated hd256 kernel, which asserts `not is_split_kv`. Once the batch is large enou…

### #58074 — [[Model] Use circular buffer for DeepSeek V4 C128 state](https://github.com/vllm-project/vllm/pull/58074)
- **作者**: andakai  **时间**: 2026-09-22 12:51 CST
- **标签**: deepseek, DSv4
- **摘要**: ## Summary  DeepSeek-V4 C128 compressor state is request-local bounded history rather than reusable attention KV cache. Using `SlidingWindowMLASpec` therefore allocates unnecessary blocks and applies semantics C128 does not need.  This change moves C128 state to `CircularBufferSpec`, reducing its KV…

### #58073 — [fix: prioritize architecture capability before DeepGEMM availability check](https://github.com/vllm-project/vllm/pull/58073)
- **作者**: hlin99  **时间**: 2026-09-22 12:48 CST
- **摘要**: Check architecture support before the remaining DeepGEMM conditions. Unsupported architectures cannot use DeepGEMM, so short-circuiting avoids unnecessary environment/package checks and prevents has_deep_gemm() from importing unavailable platform-specific modules that may emit irrelevant warnings.  …

### #58072 — [[Bugfix][Tool Parser] phi4_mini_json: report no tool call for an empty payload](https://github.com/vllm-project/vllm/pull/58072)
- **作者**: lvdousha26  **时间**: 2026-09-22 12:46 CST
- **标签**: bug, tool-calling
- **摘要**: ## Purpose  `Phi4MiniJsonToolParser` reports a tool call that does not exist.  `extract_tool_calls` looks for the `functools[...]` marker, parses the payload, and returns `tools_called=True`. Two payloads parse to nothing:  - `functools[]` — the regex captures an empty string, `json.loads("[]")` yie…

### #58071 — [[Bugfix][NIXL] Short-circuit DCP size check before reading use_mla](https://github.com/vllm-project/vllm/pull/58071)
- **作者**: ywang96  **时间**: 2026-09-22 12:06 CST
- **标签**: bug, kv-connector
- **摘要**: ## Purpose  The NIXL-gated DCP validation added in #50611 asserts:  ```python assert self.model_config.use_mla or dcp_size == 1, (     "PD with decode_context_parallel_size > 1 is only "     "supported for MLA models." ) ```  Python evaluates `use_mla` first, so even in the common `dcp_size == 1` ca…

### #58070 — [[Core][DP] Result-metric penalties and superlinear in-flight in DP lo…](https://github.com/vllm-project/vllm/pull/58070)
- **作者**: Jolin1993  **时间**: 2026-09-22 12:04 CST
- **标签**: scheduler
- **摘要**: ## Why this is not duplicating an existing PR  Upstream survey (as of 2026-09):  - PR #47420 (rotating tie-break) and #49204 (stats publishing + in-flight floor) are already merged and are ** prerequisites this patch builds on**; they do not address the linear in-flight overtaking a frozen snapshot …

### #58069 — [[Dependency] Upgrade FlashInfer version to 0.7.0](https://github.com/vllm-project/vllm/pull/58069)
- **作者**: wzhao18  **时间**: 2026-09-22 12:00 CST
- **标签**: ci/build
- **摘要**: ## Purpose Upgrade FlashInfer version to 0.7.0  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such a…

### #58068 — [[Bugfix][Attention] Fixed-width sparse indexer prefill logits so long prefills reuse allocator blocks (#55569)](https://github.com/vllm-project/vllm/pull/58068)
- **作者**: bit-incarnas  **时间**: 2026-09-22 11:56 CST
- **标签**: bug
- **摘要**: ## Purpose  Fixes #55569. Related: #56457 reports the same allocation pattern in the Qwen4Exp QSA indexer, with fixes proposed for that path in #56500 and #57105 (both open); #55572 (closed) lowered the default logits budget on integrated GPUs instead.  The dense sparse-MLA indexer (`sparse_attn_ind…

### #58067 — [[Bugfix] Make disable_any_whitespace actually disable whitespace on xgrammar](https://github.com/vllm-project/vllm/pull/58067)
- **作者**: stu-cao  **时间**: 2026-09-22 11:55 CST
- **标签**: bug, structured-output
- **摘要**: ## Purpose  `disable_any_whitespace` does not disable whitespace on the xgrammar backend — it **makes whitespace mandatory**, and as a side effect masks out the token the model wants at every JSON value position.  xgrammar's `any_whitespace=False` selects a *fixed* JSON format rather than a whitespa…
