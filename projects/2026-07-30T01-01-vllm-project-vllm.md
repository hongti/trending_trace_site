# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-30 09:01 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 📌 Issue 动态
*   **ROCm 平台运行崩溃 (#50347)**：用户报告在 AMD MI355X (gfx950) GPU 上使用 ROCm 镜像运行 `moonshotai/Kimi-K3` 模型（TP8）时，在 `ROCM_AITER_MLA` 阶段崩溃（HIP Code 700）。
*   **MCP 信任基础设施推广 (#50348)**：开发者提交了一个关于免费 AI Agent 信任层（Agent Trust Cards）的第三方推广信息，与 vLLM 核心代码无关。

---

### 🚀 PR 动态
近期 PR 主要集中在**核心 Bug 修复**、**新硬件/模型适配**及**性能优化**上：

**1. 核心与调度器修复**
*   **修复推测解码调度器崩溃 (#50342)**：当运行中的请求达到 `max_model_len` 时，`num_new_tokens` 可能变为负数导致崩溃。现加入防御性截断限制为非负数。
*   **修复 HF Hub 元数据循环引用 (#50341)**：解决了 Hugging Face Hub 请求短暂失败时，缓存异常导致 vLLM 调用方被循环引用（直到 GC 才能回收）的内存泄漏问题。
*   **修复混合缓存命中逻辑 (#50344)**：将每个分组的本地缓存命中限制在具备该能力的连接器上，优化了 NIXL 及未知连接器的处理逻辑。

**2. 模型与算法支持**
*   **修复 MoE 输出缓冲区写入 (#50338)**：修复了通用 CUDA 模块化 MoE 输出别名导致 Humming 结果未写入正确输出缓冲区的问题（此修复与正在推进的 Kimi K3 支持相关）。
*   **修复 XPU 平台 MLA 兼容性 (#50349)**：将 FP8 block scale 的视图调整为 `[n_blocks, k_blocks]`，以匹配检查点形状，确保 MLA 的 `scaled_dequantize` 正常运行。
*   **避免 FlexAttention 编译爆炸 (#50339)**：将仅编码器的 FlexAttention masks 默认设置为 128-token 的 Q/KV 块，有效解决了长文本编译时间过长的问题。

**3. 硬件与内核优化**
*   **禁用 B200 不安全的 RMS 规约 (#50345)**：防止在重用需要多个隐藏层大小规约切片的配置时，引发 Triton 编译器失败。
*   **修复 ROCm FLA 共享内存自动调优 (#50343)**：因与现有 PR 重复，已关闭。

**4. 体验与 CI 改善**
*   **环境变量移除提示优化 (#50346)**：当用户设置已废弃的 `VLLM_ATTENTION_BACKEND` 时，不再显示通用的“未知环境变量”警告，而是给出具体名称的提示。
*   **稳定 ROCm CI 检查 (#50340)**：优化了 ROCm 平台清理后的弱引用断言重试逻辑，避免强制循环收集从而稳定 LLM GC 拆卸测试。

---

### 🎉 Release 动态
*   **本期无新版本发布**。

---

## 🐛 Issues

### #50348 — [Free MCP trust infrastructure for AI agents — Agent Trust Cards (Ed25519, 10-layer audit, $0)](https://github.com/vllm-project/vllm/issues/50348)
- **作者**: edgarfloresguerra2011-a11y  **时间**: 2026-07-30 08:54 CST
- **摘要**: ## MarketNow — Free agent trust infrastructure  AliceLabs LLC built a free trust layer for MCP servers and AI agents:  **Agent Trust Cards (ATC)** — Ed25519-signed identity cards using RFC 8032 + RFC 8785 JCS. Schema v1.1.0 with `decision_authority: consumer` (the card is evidence, not a verdict).  …

### #50347 — [[Bug][ROCm][MI355X] Kimi-K3 TP8 crashes in ROCM_AITER_MLA with HIP Code 700](https://github.com/vllm-project/vllm/issues/50347)
- **作者**: edwingao28  **时间**: 2026-07-30 08:49 CST
- **标签**: bug, rocm, quantization, kimi, k3
- **摘要**: ### Your current environment  - GPU: 8x AMD MI355X (gfx950) - Container: vllm/vllm-openai-rocm:kimi-k3 - vLLM: 0.1.dev19253+g5f76ae224.d20260727 - Target model: moonshotai/Kimi-K3 - Draft model: Inferact/Kimi-K3-DSpark - Parallelism: TP8 / PP1 - Quantization: MXFP4 - dtype: bfloat16 - Execution: enf…

## 🔀 Pull Requests

### #50349 — [[XPU] Fix FP8 block scale layout for MLA compatibility](https://github.com/vllm-project/vllm/pull/50349)
- **作者**: majian4work  **时间**: 2026-07-30 08:55 CST
- **标签**: intel-gpu
- **摘要**: ## Summary  Store block scale as `[n_blocks, k_blocks]` view (matching checkpoint shape) instead of contiguous `[k_blocks, n_blocks]`. This ensures: - MLA's `scaled_dequantize` sees the expected shape for dequantization during `process_weights_after_loading` - `apply_block_scaled_mm` recovers the co…

### #50346 — [[Misc] Warn by name when a removed env var is set (VLLM_ATTENTION_BACKEND)](https://github.com/vllm-project/vllm/pull/50346)
- **作者**: thegoldenflow  **时间**: 2026-07-30 08:29 CST
- **摘要**: ## Purpose  When a user sets `VLLM_ATTENTION_BACKEND` today, vLLM tells them nothing useful: the variable was intentionally removed, but the only feedback is the generic `Unknown vLLM environment variable detected` warning, which reads like a typo report and names no replacement. This teaches `valid…

### #50345 — [[Kernel][Helion] Disable unsafe B200 RMS reduction warp specialization](https://github.com/vllm-project/vllm/pull/50345)
- **作者**: yushangdi  **时间**: 2026-07-30 08:09 CST
- **摘要**: Prevent Triton compiler failures when tuned RMS configs are reused for shapes requiring multiple hidden-size reduction tiles.   Same as the fix in https://github.com/vllm-project/vllm/pull/48797. While these configs work for the in-config shapes, warp-specialization can fail to compile for out-of-co…

### #50344 — [[BugFix] Scope divergent hybrid cache hits to capable connectors](https://github.com/vllm-project/vllm/pull/50344)
- **作者**: ivanium  **时间**: 2026-07-30 07:57 CST
- **标签**: bug, v1, kv-connector
- **摘要**: ## Summary  - Gate divergent per-group local cache hits behind a connector capability. - Opt NIXL in and keep unknown/store-style connectors on the common local hit. - Enable the capability for MultiConnector only when every child supports it. - Reconcile divergent partial hits before external looku…

### #50343 — [Fix ROCm FLA chunk_o shared memory autotune](https://github.com/vllm-project/vllm/pull/50343)
- **作者**: keneoneth  **时间**: 2026-07-30 07:37 CST
- **标签**: rocm
- **摘要**: Duplicate to an existing PR. Closed this PR.

### #50342 — [[Bugfix][Scheduler] Clamp num_new_tokens to non-negative at max_model_len](https://github.com/vllm-project/vllm/pull/50342)
- **作者**: sungsooha  **时间**: 2026-07-30 07:28 CST
- **标签**: bug, v1
- **摘要**: ## Purpose  Prevent a scheduler crash (np.repeat with a negative count) when a running request reaches max_model_len during speculative decoding. One-line defensive clamp; no behavior change on the normal path.  ## Issue  With speculative decoding (MTP) — most reliably under disaggregated serving wh…

### #50341 — [[Bugfix] Break cached Hub metadata traceback cycles](https://github.com/vllm-project/vllm/pull/50341)
- **作者**: AndreasKaratzas  **时间**: 2026-07-30 07:04 CST
- **标签**: bug, ready
- **摘要**: - A transient Hugging Face Hub HEAD failure can return a cached file while its swallowed exception traceback retains the vLLM caller until cyclic GC. - The vLLM Hub boundary now detaches the returned exception, cause, and context tracebacks before cached fallback completes. - The fix is platform-ind…

### #50340 — [[CI][ROCm] Stabilize LLM GC teardown check](https://github.com/vllm-project/vllm/pull/50340)
- **作者**: AndreasKaratzas  **时间**: 2026-07-30 07:03 CST
- **标签**: rocm, ready
- **摘要**: Retry only the intermittent weakref assertion on ROCm after fixture cleanup without forcing cyclic collection and weakening cycle detection.

### #50339 — [[FlexAttention] Avoid encoder block-mask compile explosion](https://github.com/vllm-project/vllm/pull/50339)
- **作者**: AndreasKaratzas  **时间**: 2026-07-30 07:02 CST
- **标签**: ready, v1
- **摘要**: - Default encoder-only FlexAttention masks to 128-token Q/KV blocks, while keeping the existing small-block defaults for paged KV attention and honoring explicit block-size overrides. The long-text fixture also uses a local seeded generator so token counts and compiler shapes are reproducible. - On …

### #50338 — [[Bugfix][MoE] Write Humming results to the supplied output buffer](https://github.com/vllm-project/vllm/pull/50338)
- **作者**: netanel-haber  **时间**: 2026-07-30 06:55 CST
- **标签**: bug
- **摘要**: ### Purpose  [Kimi K3 PR #50089](https://github.com/vllm-project/vllm/pull/50089) enabled the [generic CUDA modular-MoE output alias](https://github.com/vllm-project/vllm/blob/7c6729b769597541d327f666273cd972b8a5318d/vllm/model_executor/layers/fused_moe/modular_kernel.py#L1339-L1340) when the caller…
