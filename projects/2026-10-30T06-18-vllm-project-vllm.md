# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-30 14:18 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
*   **Qwen GDN 模型启动失败 (#64669)**：用户报告在无 UVA 环境下，Qwen GDN 模型开启 CPU 权重卸载 时无法启动。

### 🚀 PR 动态
**新特性与模型支持**
*   **新增 Raon-1-mini 模型支持 (#64672)**：引入了 `Raon1MiniForCausalLM`（总参数 28.8B，激活参数 4.3B）。
*   **新增请求 ID 日志选项 (#64667)**：前端新增 `--enable-log-request-ids-only` 参数，允许仅记录请求 ID 和生命周期事件，不记录任何敏感的请求内容（如提示词、参数等）。
*   **CPU 端 Gloo DP/EP 支持 (#64671)**：为宽专家并行 添加了 Gloo/TCP CPU DP-attention + EP-MoE 的后续支持。

**性能优化**
*   **流式工具调用解析优化 (#64674, #64673)**：针对 Responses API 的自定义工具，将流式输入改为增量解码，避免每次增量都重新解析全部累积参数，降低了开销；同时在遇到重复键已替换值时跳过转换，提升解析器性能。

**核心架构与修复**
*   **修复 Qwen GDN 启动 Bug (#64670)**：修复了上述 Issue #64669，在 Triton warmup 阶段将位于 CPU 的 `A_log` 和 `dt_bias` 移回 GPU。
*   **解耦 Model Runner V2/V1 (#64668)**：移除了 Model Runner V2 DBO 对 V1 切片代码的依赖，停止在测试中使用 V1 作为对照，为未来彻底移除 MRV1 铺路。

**文档更新**
*   **修复失效链接与文档勘误 (#64666, #64665, #64663)**：修复了指向 llm-d、Dynamo、TPU 等外部页面的死链；更新了 Kimi-K3 的 GB300 AgentX 预设文档；修正了 `tool_calling.md` 中关于 `tool_choice="required"` 和命名函数调用默认即可工作的不准确描述（实际需要配置 tool parser）。

### 📦 Release 动态
*   近期无新的正式版本发布。

---

## 🐛 Issues

### #64669 — [[Bug]: Qwen GDN models fail to start with CPU weight offloading without UVA](https://github.com/vllm-project/vllm/issues/64669)
- **作者**: subin9  **时间**: 2026-10-30 14:01 CST
- **标签**: qwen, kimi
- **摘要**: ### Your current environment  vLLM main (0.31.1rc1.dev1609+g335dd77b0), torch 2.14.0+cu130, RTX 4070 Ti SUPER 16 GB under WSL2.  ### 🐛 Describe the bug  With CPU weight offloading and no UVA, Qwen GDN models fail to start in the Triton kernel warmup:  ``` File "vllm/model_executor/warmup/qwen_triton…

## 🔀 Pull Requests

### #64674 — [[Perf][Frontend] Decode streamed custom-tool input incrementally](https://github.com/vllm-project/vllm/pull/64674)
- **作者**: vMaroon  **时间**: 2026-10-30 14:17 CST
- **摘要**: ## Overview  For a Responses `custom` tool, every streamed argument delta re-runs `partial_json_parser` over the whole accumulated arguments, so the API server's cost per delta grows with the tool input. This decodes only the new tail of the input string. Follow-up to #57322.  ## Claims  - Cost per …

### #64673 — [[Perf][Parser] Skip tool-argument conversions after a replaced value](https://github.com/vllm-project/vllm/pull/64673)
- **作者**: vMaroon  **时间**: 2026-10-30 14:17 CST
- **摘要**: ## Overview  When a tool call repeats a key whose value was already streamed, `ParserEngine._compute_arg_delta` fails the prefix check before it installs the argument value extender. Every later delta then re-converts the whole argument text, only to fail the same check, so the API server's cost per…

### #64672 — [[Model] Add Raon-1-mini support](https://github.com/vllm-project/vllm/pull/64672)
- **作者**: ugonfor  **时间**: 2026-10-30 14:13 CST
- **标签**: documentation, new-model, speculative-decoding, mrv2, dflash
- **摘要**: ## Overview  - Adds `Raon1MiniForCausalLM` for [KRAFTON/Raon-1-mini-preview](https://huggingface.co/KRAFTON/Raon-1-mini-preview) (28.8B total / 4.3B active, about 3.1k downloads in the last 30 days). - Includes a DFlash fix that its [DFlash2 draft](https://huggingface.co/KRAFTON/Raon-1-mini-preview-…

### #64671 — [[CPU] Add Gloo DP/EP follow-up for wide expert parallelism](https://github.com/vllm-project/vllm/pull/64671)
- **作者**: DeshErBojhaa  **时间**: 2026-10-30 14:05 CST
- **标签**: documentation, torch.compile, ci/build, cpu
- **摘要**: ## Overview  Part of #57837. This is a dependent Gloo/TCP CPU DP-attention + EP-MoE follow-up to #47778, with the expert-map fix from #61599.  **Draft: testing on two independent Linux hosts is NOT complete.** Native Linux x86-64 default-runner/model acceptance is also pending; the successful macOS …

### #64670 — [[Bugfix] Move offloaded A_log and dt_bias to the GPU in the Qwen GDN warmup](https://github.com/vllm-project/vllm/pull/64670)
- **作者**: subin9  **时间**: 2026-10-30 14:01 CST
- **标签**: bug, qwen
- **摘要**: ## Overview  Fixes #64669. With CPU weight offloading and no UVA, Qwen GDN models fail to start because the Triton warmup passes the CPU-resident `A_log` and `dt_bias` to the kernels. This moves the two tensors to the cache's device when the warmup config is built.  ## Claims  - `--cpu-offload-gb` w…

### #64668 — [[Core] Remove the dependency of model runner V2 DBO from model runner V1](https://github.com/vllm-project/vllm/pull/64668)
- **作者**: algebra-MCX  **时间**: 2026-10-30 13:59 CST
- **标签**: mrv2
- **摘要**: ## Overview  Model Runner V2 DBO no longer calls Model Runner V1 slicing code, and its tests no longer use V1 as an oracle or import the V1 runner. Once #63373 removes MRV1, the V1-only slicing helpers can be deleted without changing MRV2.  ## Claims  - `create_ubatch_slices` (MRV2) computes its pad…

### #64667 — [[Frontend] Add --enable-log-request-ids-only](https://github.com/vllm-project/vllm/pull/64667)
- **作者**: Mramaran  **时间**: 2026-10-30 13:59 CST
- **标签**: frontend
- **摘要**: ## Overview  Adds `--enable-log-request-ids-only`, which logs request IDs and lifecycle events without any request content (prompts, parameters, LoRA requests, JSON bodies). Fixes #64654.  ## Claims  - `--enable-log-request-ids-only` turns on request logging by itself. The frontend logs `Received re…

### #64666 — [[Docs] Fix dead links to moved llm-d, Dynamo, TPU, Ray and Tensorizer pages](https://github.com/vllm-project/vllm/pull/64666)
- **作者**: sarankumar1325  **时间**: 2026-10-30 13:58 CST
- **标签**: documentation, ray
- **摘要**: ## Overview  Fix nine dead external links in the docs. Each one now points at the page's current location.  ## Claims  - llm-d integration page: the `kv-management`, `workload-autoscaling` and `flow-control` pages moved to `/docs/architecture/kv-management`, `/docs/operations/autoscaling` and `/docs…

### #64665 — [[Doc] Kimi-K3: move the GB300 AgentX presets to the Oct 26 points with KV-balanced decode](https://github.com/vllm-project/vllm/pull/64665)
- **作者**: Ankur-singh  **时间**: 2026-10-30 13:54 CST
- **标签**: ready, kimi, k3, recipe
- **摘要**: ## Overview  Update the two GB300 Kimi-K3 AgentX disaggregated presets in `recipes/models/moonshotai/Kimi-K3.yaml` to the lanes qualified in [SemiAnalysisAI/InferenceX#4638](https://github.com/SemiAnalysisAI/InferenceX/pull/4638) (head `bc4d49f5`, sweep run [40830592414](https://github.com/SemiAnaly…

### #64663 — [[Doc] Named and required tool_choice need a tool parser](https://github.com/vllm-project/vllm/pull/64663)
- **作者**: KeitaW  **时间**: 2026-10-30 13:34 CST
- **标签**: documentation, tool-calling
- **摘要**: ## Overview  `docs/features/tool_calling.md` says named function calling works "by default", and that `tool_choice="required"` "is enabled by default and will work with any supported model". Since #30613 (first released in v0.14.0), both return HTTP 400 unless the server has loaded a tool parser, an…
