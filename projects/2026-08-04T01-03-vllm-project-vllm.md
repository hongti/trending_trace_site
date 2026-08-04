# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-04 09:03 CST

## AI 总结

# vllm-project/vllm 近期动态摘要

---

## 🐛 Issue

| # | 标题 | 要点 |
|---|------|------|
| **#50938** | Gemma4 解析器 + 结构化输出 Bug | 推理起始标签前的文本逃逸了 Gemma4 解析器和结构化输出校验，导致 `json_schema` 响应被破坏 |
| **#50935** | Qwen3.5 批次不变性支持 Bug | Qwen3.5 系列在批量推理时 EngineCore 崩溃（Traceback） |
| **#50934** | GB10 CUDA 地址对齐崩溃 | Nemotron NVFP4 + Marlin MoE + MTP 投机解码组合下，稳定运行约 10 天后出现 `CUDA error: misaligned address` |

> **趋势**：新模型（Gemma-4、Qwen3.5、Nemotron）的推理/量化/解码路径仍存在稳定性问题，涉及解析器、批量处理和长时间运行场景。

---

## 🔀 Pull Request

### 🐛 缺陷修复

| # | 要点 |
|---|------|
| **#50939** | **Model Runner V2**：拒绝采样器中修复 `-1` 占位 draft token id 未被过滤的问题，避免其被当作有效 token |
| **#50937** | 权重加载重构后，跳过不支持的空 bias 权重，修复 `RoutedExperts` 加载报错 |
| **#50933** | 修复 Pydantic v2 生成的工具 schema 中 `$ref`/`$defs` 嵌套类型丢失问题，使结构化输出正确解析嵌套对象 |
| **#50932** | 修复 FlashInfer MNNVL allreduce buffer 大小计算不足，调整 `max_num_tokens` 的门控逻辑 |
| **#50928** | **Kimi K3**：按有效 MoE TP size 分片 MoE intermediate，修复因 padding 导致的分片错误 |
| **#50926** | CI：修复 `test_store_orders_after_compute_write` 因无屏障控制阶段残留内核竞争导致的 flaky 测试 |

### ⚡ 性能优化

| # | 要点 |
|---|------|
| **#50936** | 流式输入循环中跳过重复 `sampling_params` 的校验，减少不必要的 `_validate_params()` 开销 |

### ✨ 新特性 / 功能增强

| # | 要点 |
|---|------|
| **#50931** | **Model Runner V2**：启用 decoder token-wise pooling（支持 chunked accumulation），补全 `token_embed`/`token_classify` 能力 |
| **#50929** | **Kimi-K2.5/K2.6**：支持 ViT 全 CUDA Graph，提升多模态推理性能 |
| **#50930** | ROCm AITER MLA 自定义算子注册、fake-tensor 支持及环境门控测试 |

---

## 🚀 Release

> 本期无新版本发布。

---

### 📌 总结

- **Bug 修复**是本期主旋律，涵盖解析器逃逸、权重加载、工具 schema 嵌套、MoE 分片、buffer 计算等多个方向。
- **Model Runner V2** 持续迭代，修复了拒绝采样器和 token-wise pooling 的问题。
- **Kimi 系列模型**（K2.5/K2.6/K3）获得密集支持（ViT CUDA Graph、MoE 分片修复）。
- **性能优化**方面，流式采样参数校验跳过是一个低开销高收益的改进。
- 长时间运行稳定性（10 天 CUDA 崩溃）和结构化输出可靠性仍是社区关注重点。

---

## 🐛 Issues

### #50938 — [[Bug]: Text emitted before the reasoning start tag escapes both the gemma4 parser and structured output, breaking json_schema responses](https://github.com/vllm-project/vllm/issues/50938)
- **作者**: yasutoshi-lab  **时间**: 2026-08-04 08:32 CST
- **摘要**: ### Your current environment  <details> <summary>The output of <code>vllm collect-env</code> (trimmed: CPU flags, vulnerability list, GPU topology legend and the remaining nvidia-* wheels removed)</summary>  ```text ==============================         System Info ============================== OS…

### #50935 — [[Bug]: Qwen3.5 Series Batch Invariance Support](https://github.com/vllm-project/vllm/issues/50935)
- **作者**: bigben-dev777  **时间**: 2026-08-04 08:09 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Your output of `python collect_env.py` here ``` Collecting environment information... ==============================         System Info ============================== OS             …

### #50934 — [[Bug]: CUDA misaligned address crash on GB10 (sm_121) after ~10 days uptime — Nemotron NVFP4 + Marlin MoE + MTP speculative decoding](https://github.com/vllm-project/vllm/issues/50934)
- **作者**: hckhead  **时间**: 2026-08-04 08:07 CST
- **标签**: bug, quantization
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information... ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (aarch64) GCC…

## 🔀 Pull Requests

### #50939 — [[Model Runner V2] Fix -1 placeholder draft token ids in rejection sam…](https://github.com/vllm-project/vllm/pull/50939)
- **作者**: TheEpicDolphin  **时间**: 2026-08-04 08:49 CST
- **标签**: ready, mrv2
- **摘要**: # Summary The block verification path does not currently guard against -1 placeholder draft token ids, which can be used as padding. This PR simply ensures that these draft tokens are rejected, and that they don't cause OOB for block verification. This was already handled for standard/greedy rejecti…

### #50937 — [[Bugfix] skip empty bias weights when not supported](https://github.com/vllm-project/vllm/pull/50937)
- **作者**: walterbm  **时间**: 2026-08-04 08:29 CST
- **标签**: bug
- **摘要**: ## Purpose  After the weight loading refactor in https://github.com/vllm-project/vllm/pull/47058 models with unused bias tensors started to fail on load with `AttributeError: 'RoutedExperts' object has no attribute 'w2_bias'`  ``` (EngineCore pid=1916176) ERROR 08-03 22:26:34 [core.py:1349]   File "…

### #50936 — [perf: skip re-validating reused sampling params in streaming input loop](https://github.com/vllm-project/vllm/pull/50936)
- **作者**: ji24077  **时间**: 2026-08-04 08:28 CST
- **标签**: needs-rebase
- **摘要**: ## Summary  In `generate_from_stream()`, each input chunk calls `process_inputs()`, which unconditionally runs `_validate_params()` → `params.verify(...)`. When the chunk carries no per-chunk `sampling_params`, it reuses the same object that was already fully validated for the `final_req` at stream …

### #50933 — [[Bugfix] Resolve $ref/$defs in tool schemas and preserve definitions](https://github.com/vllm-project/vllm/pull/50933)
- **作者**: ben7am1n  **时间**: 2026-08-04 08:01 CST
- **标签**: bug, tool-calling
- **摘要**: ## Purpose  Fixes #46924. Supersedes #46925 (incomplete fix) and #46170 (`pop`→`get` only).  Tool schemas generated by Pydantic v2 use `$defs`/`$ref` to define nested types. Two bugs caused nested object arguments (e.g. `{"kind": "week"}`) to be double-encoded as JSON strings:  1. **No `$ref` resolu…

### #50932 — [buffer size insuffient Dspark sd for FlashInfer MNNVL allreduce](https://github.com/vllm-project/vllm/pull/50932)
- **作者**: khushali9  **时间**: 2026-08-04 07:46 CST
- **标签**: nvidia
- **摘要**: fixes [#50877](https://github.com/vllm-project/vllm/issues/50877)  ## Purpose FlashInferAllReduce.should_use_fi_ar gates on:      self.max_num_tokens = max_workspace_size // (hidden_dim * element_size)  max_workspace_size is the size of the whole MNNVL allocation (2 MB for TP8). But the MNNVL backen…

### #50931 — [[ModelRunner v2] Enable decoder token-wise pooling](https://github.com/vllm-project/vllm/pull/50931)
- **作者**: taneem-ibrahim  **时间**: 2026-08-04 07:39 CST
- **标签**: mrv2
- **摘要**: ## Purpose  Advances #41286 by enabling decoder token-wise pooling on Model Runner V2. The pooling already supports chunked accumulation, but MRV2 filtered `token_embed` and `token_classify` for non-encoder models. This removes that flag.  This unblocks text decoder embedding models, reward/process-…

### #50930 — [[Test] Add ROCm AITER MLA op registration and env gating tests](https://github.com/vllm-project/vllm/pull/50930)
- **作者**: aarushjain29  **时间**: 2026-08-04 07:31 CST
- **标签**: rocm
- **摘要**: ## [Test] ROCm AITER MLA op registration, fake-tensor, and env gating tests  Adds kernel-level tests verifying that `rocm_aiter_mla_decode_fwd` custom op is correctly registered, supports fake tensors for `torch.compile` tracing, and respects `VLLM_ROCM_USE_AITER` / `VLLM_ROCM_USE_AITER_MLA` environ…

### #50929 — [[MM][CG] Support ViT full CUDA graph for Kimi-K2.5](https://github.com/vllm-project/vllm/pull/50929)
- **作者**: lk-chen  **时间**: 2026-08-04 07:01 CST
- **标签**: nvidia, kimi
- **摘要**: ## Purpose  Add ViT CG support (#38175 )  for Kimi K2.5/K2.6  ## Test Plan  ```bash   VLLM_USE_V2_MODEL_RUNNER=0 FLASHINFER_DISABLE_VERSION_CHECK=1 \     vllm serve moonshotai/Kimi-K2.6 --trust-remote-code \       --tensor-parallel-size 8 --enable-expert-parallel \       --max-model-len 4096 --max-n…

### #50928 — [[Bugfix][Model] Kimi K3: shard MoE intermediate by the effective MoE TP size](https://github.com/vllm-project/vllm/pull/50928)
- **作者**: hongyeon-yu  **时间**: 2026-08-04 06:36 CST
- **标签**: bug, kimi, k3
- **摘要**: ## Purpose  `KimiMoE` pads `moe_intermediate_size` up to `min_moe_intermediate_per_partition * tp_size` so every MoE shard is at least 256 columns wide, and records `intermediate_size_per_partition_unpadded = moe_intermediate_size // tp_size` (`vllm/models/kimi_k3/nvidia/model.py:496` and `:652`). B…

### #50926 — [[CI][Bugfix] Fix flaky `test_store_orders_after_compute_write`](https://github.com/vllm-project/vllm/pull/50926)
- **作者**: njhill  **时间**: 2026-08-04 06:29 CST
- **标签**: bug, ready
- **摘要**: The no-barrier control phase leaves a ~800ms backlog of sleep+fill kernels in flight on its compute stream (the host loop only waits on the store-copy events). Those leftover fills race the barrier phase's fill->copy window on the shared gpu tensor and flakily corrupt one iteration, tripping the 'st…
