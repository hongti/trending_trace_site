# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-07 14:21 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issues (问题反馈)
本期报告了多个与流式处理、调度和配置相关的 Bug：
* **流式与工具解析问题**：开启 `tool_choice="required"` 时，若单个增量完成多个调用，流式输出会丢失 tool calls（#60351）；InternLM2 流式工具解析器会丢失函数参数中的字符（#60348）。
* **内存与调度死锁**：FP8 KV cache 启动时会 OOM，原因是 CUDA graph 显存未计入缓存预算（#60350）；`max_num_scheduled_tokens=0` 会导致 `LLM.generate()` 永久挂起无响应（#60349）。
* **配置失效**：`TensorizerConfig` 指定 `lora_dir` 时未能正确推导 `tensorizer_dir`，导致序列化往返失败（#60347）。

### 🚀 Pull Requests (合并请求)
PR 活动涵盖了新特性、性能优化及大量 Bug 修复：
* **新特性**：
  * 前端音频 API 新增 `srt` (SubRip) 和 `vtt` (WebVTT) 字幕响应格式（#60336）。
  * 引入基于 GPU 的后缀解码投机器 (`suffix_gpu`)，支持 Model Runner V2（#60339）。
* **性能优化**：
  * **MLA 解码加速**：TokenSpeed MLA 解码内核支持配置最小 KV 分割数，特定场景下性能最高提升 4 倍（#60345）。
  * **Blackwell 优化**：在 TP2（张量并行度为2）时启用融合 all-reduce + mHC 边界优化，适配新的单节点默认配方（#60355）。
* **Bug 修复**：
  * 修复上述 `tool_choice="required"` 流式丢失 tool calls 的问题（#60354）。
  * 修复 Olmo3 模型流式推理时缓冲区在流结束时未刷新的问题（#60353）。
  * 修复 Voxtral 多模态模型在 bf16 下因强制转换导致 log-mel 精度丢失的问题，现保持原始音频为 fp32（#60352）。
  * 量化方面，为 Humming A16 兼容性检查接受折叠的 NVFP4 scales（#60337）。
* **基建与测试**：
  * XPU Docker 镜像将 nixl 版本对齐升级至 1.5.0，UCX 同步升至 v1.23.x（#60356）。
  * 新增 NVIDIA 机密计算（Confidential Computing）下 CPU 与 GPU 间数据拷贝的微基准测试（#60338）。

### 📦 Release (版本发布)
本期提供的动态中未包含版本发布信息。

---

## 🐛 Issues

### #60351 — [[Bug]: tool_choice="required"` streaming drops tool calls when one delta completes more than one call](https://github.com/vllm-project/vllm/issues/60351)
- **作者**: the-Shallow  **时间**: 2026-10-07 13:06 CST
- **标签**: bug, tool-calling
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text vLLM 0.30.0 (the latest release) and 0.28.0, CPU, Python 3.12, Ubuntu 24.04, x86_64 ```  </details>   ### 🐛 Describe the bug   With `tool_choice="required"` the model writes a JSON ar…

### #60350 — [[Bug]: FP8 KV cache startup can OOM because CUDA graph memory is omitted from the cache budget](https://github.com/vllm-project/vllm/issues/60350)
- **作者**: the-Shallow  **时间**: 2026-10-07 12:52 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text vLLM 0.28.0 on Linux with an NVIDIA GPU ```  </details>   ### 🐛 Describe the bug   On a 24 GiB GPU, `LLM(..., kv_cache_dtype="fp8")` is accepted but fails during startup with an out-o…

### #60349 — [[Bug]: `LLM.generate()` never returns when `max_num_scheduled_tokens=0`](https://github.com/vllm-project/vllm/issues/60349)
- **作者**: the-Shallow  **时间**: 2026-10-07 12:51 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text vLLM 0.28.0 on Linux with an NVIDIA GPU. ```  </details>   ### 🐛 Describe the bug   `SchedulerConfig.max_num_scheduled_tokens` uses a lower bound of `ge=0`, so vLLM accepts zero as a …

### #60348 — [[Bug]: InternLM2 streaming tool parser drops characters from function arguments](https://github.com/vllm-project/vllm/issues/60348)
- **作者**: the-Shallow  **时间**: 2026-10-07 12:50 CST
- **标签**: bug, tool-calling
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text vLLM 0.30.0 and 0.28.0, CPU, Python 3.12, Ubuntu 24.04, x86_64 ```  </details>   ### 🐛 Describe the bug   `Internlm2ToolParser.extract_tool_calls_streaming` can emit argument deltas w…

### #60347 — [[Bug]: `TensorizerConfig(lora_dir=...)` leaves `tensorizer_dir=None` and breaks serialization round trips](https://github.com/vllm-project/vllm/issues/60347)
- **作者**: the-Shallow  **时间**: 2026-10-07 12:49 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text vLLM 0.30.0 and 0.28.0, CPU, Python 3.12, Ubuntu 24.04, x86_64 ```  </details>   ### 🐛 Describe the bug   `TensorizerConfig.__post_init__` derives `tensorizer_dir` only when `tensoriz…

## 🔀 Pull Requests

### #60356 — [[XPU][Docker] Bump nixl to 1.5.0](https://github.com/vllm-project/vllm/pull/60356)
- **作者**: sandeep-maddipatla  **时间**: 2026-10-07 14:15 CST
- **标签**: intel-gpu, ci/build, kv-connector
- **摘要**: Bring the XPU image in line with the nixl == 1.5.0 pin that requirements/kv_connectors.txt already carries (#56907).  UCX moves from v1.21.x to v1.23.x along with it, following the rule in the Dockerfile: UCX_VERSION tracks the UCX_REF that nixl's own contrib/Dockerfile.manylinux builds against for …

### #60355 — [[Perf][DSV4.1] Enable the fused all-reduce + mHC boundary at TP2](https://github.com/vllm-project/vllm/pull/60355)
- **作者**: andakai  **时间**: 2026-10-07 14:07 CST
- **标签**: deepseek, DSv4.1
- **摘要**: ## Motivation  The fused CuTe DSL Lamport all-reduce + mHC post/collapse/RMSNorm (#57643, #58586) only runs at TP4. The Blackwell single-node recipes now default to TP2 (vllm-project/recipes#1002), so at TP2 every small-batch sublayer boundary still runs a separate all-reduce plus unfused mHC. The k…

### #60354 — [[Bugfix][ToolParser] Stream every tool call when one delta completes several](https://github.com/vllm-project/vllm/pull/60354)
- **作者**: Kylinny  **时间**: 2026-10-07 13:56 CST
- **标签**: bug, tool-calling, kimi, k3
- **摘要**: ## Overview  Fixes https://github.com/vllm-project/vllm/issues/60351: with `tool_choice="required"`, streaming silently dropped tool calls whenever a single delta completed more than one call.  ## Claims  - Every tool call in the streamed JSON array is now emitted with its name and arguments, regard…

### #60353 — [[Bugfix][Reasoning] Flush the Olmo3 streaming hold-back at end of stream](https://github.com/vllm-project/vllm/pull/60353)
- **作者**: he-yufeng  **时间**: 2026-10-07 13:38 CST
- **标签**: bug, tool-calling
- **摘要**: ## Overview  Fixes #60340. In streaming mode `Olmo3ReasoningBuffer.add_text` decides whether to hold a delta back using `string_overlap(delta_text, marker)`, and that helper reports *containment* as an overlap: any delta that merely occurs inside `<think>` / `</think>` (`think`, `in`, `k`, `>`, `/`,…

### #60352 — [[Multimodal][Voxtral] Keep raw audio waveforms in fp32](https://github.com/vllm-project/vllm/pull/60352)
- **作者**: jimburtoft  **时间**: 2026-10-07 13:23 CST
- **标签**: multi-modality, mistral
- **摘要**: ## Overview  Voxtral's processor returns raw waveforms (`audio_arrays`), and `call_hf_processor` casts every float output to the model dtype. With `--dtype bfloat16` the log-mel is computed from bf16 audio. This lets a processor keep named fields out of that cast; Voxtral opts in for `audio_arrays`.…

### #60345 — [[Perf] Support minimum KV splits configuration for TokenSpeed MLA decode ](https://github.com/vllm-project/vllm/pull/60345)
- **作者**: wzhao18  **时间**: 2026-10-07 12:47 CST
- **摘要**: ## Overview  The TokenSpeed MLA decode kernel supports passing an optional flag `min_split_kv` to force a minimum number of KV splits, which can accelerate performance by up to 4x speedup under certain workloads, see https://github.com/lightseekorg/tokenspeed/pull/1693.  This PR supports passing thi…

### #60339 — [[Spec Decode][MRV2] GPU suffix decoding speculator (suffix_gpu)](https://github.com/vllm-project/vllm/pull/60339)
- **作者**: nishantrevur  **时间**: 2026-10-07 11:50 CST
- **标签**: documentation, speculative-decoding, mrv2
- **摘要**: ## Purpose  Suffix decoding on Model Runner V2 (the "suffix" item in #47172). Today `method="suffix"` needs Arctic Inference, runs on the CPU, forces Model Runner V1 and disables async scheduling. This PR adds `method="suffix_gpu"`, built on the `ngram_gpu` speculator from #40704 and borrowing ideas…

### #60338 — [[Benchmark] NVIDIA Confidential Computing CPU<->GPU bridge microbench](https://github.com/vllm-project/vllm/pull/60338)
- **作者**: Alex-Carter01  **时间**: 2026-10-07 11:50 CST
- **标签**: performance, nvidia
- **摘要**: ## Purpose  Adds a standalone microbenchmark (`benchmarks/confidential_compute/`) that measures how host<->device copies behave under NVIDIA GPU Confidential Computing (CC) and what that does to a vLLM-style decode loop. It does not import vLLM and changes no runtime code.  Tests: `semantics` (does …

### #60337 — [[Bugfix][Quantization] Accept folded NVFP4 scales for Humming A16](https://github.com/vllm-project/vllm/pull/60337)
- **作者**: jinzhen-lin  **时间**: 2026-10-07 10:37 CST
- **标签**: bug, quantization
- **摘要**: ## Overview  Accept folded NVFP4 FP16/BF16 scales in Humming's A16 compatibility check, without changing backend priority. Alternative to #60315.  ## Claims  - Handles unequal gate/up scales, including dummy-weight initialization. - Preserves actual weight metadata, other compatibility checks, and s…

### #60336 — [[Frontend] Add `srt` and `vtt` response formats for audio transcription/translation](https://github.com/vllm-project/vllm/pull/60336)
- **作者**: G4tsby  **时间**: 2026-10-07 10:36 CST
- **标签**: documentation, frontend
- **摘要**: ## Overview  Add the `srt` (SubRip) and `vtt` (WebVTT) `response_format` options to `/v1/audio/transcriptions` and `/v1/audio/translations`, as in the [OpenAI Audio API](https://platform.openai.com/docs/api-reference/audio/createTranscription). Part of #25750 (claimed in [this comment](https://githu…
