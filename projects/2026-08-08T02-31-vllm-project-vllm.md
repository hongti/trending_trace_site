# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-08 10:31 CST

## AI 总结

以下是 **vllm-project/vllm** 最近的动态摘要：

### 📌 Issue 动态

1. **[RFC] 支持原始多模态输入的 `/generate` 端点 (#51472)**
   - 提出在 `/generate` 端点支持原始多模态输入，以满足强化学习（RL）框架（如 prime-rl）在 token 级别推理时的需求。
2. **[Bug] DeepSeek-V4-Flash 结构化输出导致引擎崩溃 (#51467)**
   - 使用 `response_format`（如 JSON schema）时，`apply_grammar_bitmask` 出现张量大小不匹配（4040 vs 4041），导致 EngineCore 崩溃。
3. **[Bug] Kimi K3 prompt_tokens 计数多算 3 个 token (#51465)**
   - 在带有强制生成/解码前缀的模型（如 Kimi K3）中，OpenAI API 返回的 `usage.prompt_tokens` 错误地将尾部的 channel-open stub 计入了总数。

---

### 📌 PR 动态

**核心与引擎修复**
- **修复 EngineCore 握手端口竞态条件 (#51469)**：在广播端口之前先绑定 ZMQ ROUTER，防止并发连接误连。
- **修复 V2 模型运行器权重卸载器未安装的问题 (#51462)**：解决 V2 runner 因未安装 offloader 导致权重无法正常卸载的 Bug。
- **保留外部 Mamba 状态下的分歧 FA 命中 (#51468)**：修复外部 Mamba 状态下 `truncate_computed_blocks()` 的断言错误。

**模型与硬件支持**
- **[CPU][MLA] 修复 MLA 预填充后端选择 (#51471)**：使 DeepSeek 风格的 MLA 模型能在 CPU 上端到端运行，修复了之前进入预填充阶段崩溃的问题。
- **[ROCm] 更新基础 Docker 中的 Triton (#51464)**：修复 gluon MLA 内核编译时 DistributedLinearLayout 被拒的问题。
- **[MM][CG] 修复 Ernie-4.5-VL 编码器 CUDA Graph 后处理 (#51461)**：修复多路径输出时的崩溃问题，避免回退代码。

**投机解码**
- **修复 DSD K-lookup 计数错误 (#51466)**：仅计算纯采样请求，防止批次边界处的 K 颠簸。
- **多层 MTP 优化 (#51460)**：在多层 MTP 期间仅嵌入多模态（MM）输入，提升效率。

**前端与 API**
- **修复工具解析器构建逻辑 (#51470)**：从渲染到提示词中的工具构建解析器。
- **`/derender` 端点 `model` 字段改为可选 (#51463)**：在所有 4 个 `/derender` 请求类中将 `model` 参数设为可选。

---

### 🚀 Release 动态

近期无新的版本发布记录。

---

## 🐛 Issues

### #51472 — [[RFC] Raw multimodal input for /generate endpoint (RL workloads)](https://github.com/vllm-project/vllm/issues/51472)
- **作者**: aoshen02  **时间**: 2026-08-08 10:27 CST
- **标签**: RFC
- **摘要**: ## Motivation  RL frameworks (e.g. prime-rl) need token-level inference with multimodal inputs. Their typical call pattern is:  ``` rollout produces token_ids + raw media references   → call /generate for next-step inference ```  Today there are three paths, none ideal:  | Path | Problem | |---|---|…

### #51467 — [[Bug]: DeepSeek-V4-Flash-0731 `response_format` (structured output) crashes the vLLM EngineCore — `apply_grammar_bitmask` tensor size mismatch (4040 vs 4041)](https://github.com/vllm-project/vllm/issues/51467)
- **作者**: PaimonLumine  **时间**: 2026-08-08 09:27 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (x86_64) GCC version                  : (Ubuntu 11.…

### #51465 — [[Bug]: Kimi K3 usage.prompt_tokens over-counts trailing channel-open stub (+3)](https://github.com/vllm-project/vllm/issues/51465)
- **作者**: smurthy024  **时间**: 2026-08-08 08:12 CST
- **标签**: bug, kimi, k3
- **摘要**: ### Your current environment  ## Summary For Kimi K3 (and potentially other models that append a forced generation / decode prefix), OpenAI `usage.prompt_tokens` reports the **full engine** `prompt_token_ids` length. Vendor-style / billing groundtruth counts only user-provided prompt tokens: ```text…

## 🔀 Pull Requests

### #51471 — [[CPU][MLA] Fix prefill backend selection so MLA runs end-to-end on CPU](https://github.com/vllm-project/vllm/pull/51471)
- **作者**: maobaolong  **时间**: 2026-08-08 10:24 CST
- **标签**: cpu
- **摘要**: ## Purpose  Make DeepSeek-style MLA models run **end-to-end** on CPU. This builds on #49453, which added the CPU MLA backend but left the model crashing the moment it entered the prefill phase.  ## Relation to #49453 (why it could not run on CPU, and what this PR adds)  #49453 ("[CPU] Add MLA backen…

### #51470 — [[Bugfix][Frontend] Build parsers from the tools rendered into the prompt](https://github.com/vllm-project/vllm/pull/51470)
- **作者**: Vegetog  **时间**: 2026-08-08 10:00 CST
- **标签**: bug, frontend, tool-calling
- **摘要**: > **Depends on #49876** and is branched on top of it, so the commit list here > includes that PR's three commits. Only the two commits below belong to this > one: > > - `fix: construct parsers from the tools rendered into the prompt` > - `fix: give each batched conversation its own parser` > > Happy…

### #51469 — [[BugFix] Bind EngineCore handshake port before advertising it](https://github.com/vllm-project/vllm/pull/51469)
- **作者**: aoshen02  **时间**: 2026-08-08 10:00 CST
- **标签**: bug, ready
- **摘要**: ## Purpose  The EngineCore startup handshake selected an automatic TCP port with `get_open_port()`, released the probe socket, constructed the endpoint, and only later bound the ZMQ ROUTER. A concurrent process could claim the port in that interval and abort startup with `zmq.error.ZMQError: Address…

### #51468 — [[BugFix] Preserve divergent FA hits with external Mamba state](https://github.com/vllm-project/vllm/pull/51468)
- **作者**: majunze2001  **时间**: 2026-08-08 09:28 CST
- **标签**: bug
- **摘要**: The original fix is by @ywang96, with a follow-up refinement by @ivanium  ## Summary  `truncate_computed_blocks()` asserts that every KV cache group holds at least `num_computed_tokens // block_size` blocks. For a hybrid (full-attention + Mamba) model served with a KV connector, a Mamba group can le…

### #51466 — [[Bugfix][Spec Decode] Fix DSD K-lookup to count sampling-only requests, preventing K thrashing at batch boundaries](https://github.com/vllm-project/vllm/pull/51466)
- **作者**: Suppressor72  **时间**: 2026-08-08 08:14 CST
- **标签**: bug, speculative-decoding
- **摘要**: ## Description  The dynamic speculative decoding (DSD) K-lookup in `scheduler.py` uses `len(num_scheduled_tokens)` to index into `dynamic_sd_lookup`. This counts **all** scheduled requests, including mid-prefill/chunked-prefill requests. When prefills are mixed with decodes, they temporarily inflate…

### #51464 — [[ROCm] update triton in base docker for gluon compatibility](https://github.com/vllm-project/vllm/pull/51464)
- **作者**: hongxiayang  **时间**: 2026-08-08 08:07 CST
- **标签**: rocm, ci/build
- **摘要**: ## Purpose  To pick up https://github.com/ROCm/triton/pull/960 for fixing the gluon mla kernel compilation problem related to DistributedLinearLayout was rejected in the previous triton commit. With this, the gluon kernel can compile and serve K3 with accuracy.  Context: Initially vllm/vllm-openai-r…

### #51463 — [[Frontend] Make `model` optional on all `/derender` request classes](https://github.com/vllm-project/vllm/pull/51463)
- **作者**: vrdn-23  **时间**: 2026-08-08 08:07 CST
- **标签**: frontend
- **摘要**: ## Summary  Makes `model` optional on all four `/derender` request classes — `DerenderChatRequest`, `DerenderCompletionRequest`, `DerenderChatStreamRequest`, `DerenderCompletionStreamRequest` — and resolves the served model name server-side via `request.model or self.models.model_name()` when the fi…

### #51462 — [[Bugfix] Install the weight offloader in the V2 model runner](https://github.com/vllm-project/vllm/pull/51462)
- **作者**: guptaishaan  **时间**: 2026-08-08 07:47 CST
- **标签**: bug, needs-rebase, mrv2
- **摘要**: Fixes #50672  `set_offloader(create_offloader(...))` is only called from the V1 GPU model runner. The V2 runner never installs an offloader, so `make_layers()` gets the process-global default `NoopOffloader` and `--cpu-offload-gb` is silently a no-op. `KimiK3ForConditionalGeneration` is in `DEFAULT_…

### #51461 — [[MM][CG][BugFix] Fix Ernie-4.5-VL encoder CG postprocess for multi-path outputs](https://github.com/vllm-project/vllm/pull/51461)
- **作者**: qyYue1389  **时间**: 2026-08-08 07:23 CST
- **标签**: bug
- **摘要**: ## Purpose  Fix-forward for the crash that prompted #51263 (revert of #45254), instead of reverting.  `SupportsEncoderCudaGraph.postprocess_encoder_output` now receives `outputs: dict[str, torch.Tensor]` keyed by encoder path (multi-path graph support), but `Ernie4_5_VLMoeForConditionalGeneration`'s…

### #51460 — [[Model Runner V2][Spec Decode] Only embed MM inputs during multi-layer MTP](https://github.com/vllm-project/vllm/pull/51460)
- **作者**: TheEpicDolphin  **时间**: 2026-08-08 07:17 CST
- **标签**: mrv2
