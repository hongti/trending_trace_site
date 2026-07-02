# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-02 14:09 UTC

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文简洁摘要：

### 📌 Issue
- **SM120 Blackwell GPU 上 FP8 加载崩溃 (#47436)**：在 RTX PRO 6000 Blackwell（计算能力 12.0）上使用 v0.24.0 加载 Block-scaled FP8（compressed-tensors W8A8）模型时崩溃，触发 DeepGEMM 的 `"Unknown SF transformation"` 断言错误。

---

### 🔧 Pull Request

**🚀 新特性与支持**
- **支持 AutoRound Block-Wise FP8 (#47434)**：新增对 AutoRound 格式的 Block-Wise FP8 量化模型的支持。

**⚡ 性能与架构优化**
- **HPC Attention 动态 Split-K 选择 (#47443)**：解码路径不再硬编码 `splitk=True`，而是根据批次大小和序列长度动态选择。经验证，Split-K 仅在小批次/长序列下占优，中等批次/短序列下非 Split-K 路径更快。（注：同类 PR #47441 已被废弃）
- **Rust 前端性能与对齐优化 (#47444, #47435)**：预缓存 metrics handles 以减少调度器和请求级别的 `get_or_create` 重复查找开销；补充了与 Python 前端对齐的统计日志（如延迟请求、抢占、外部前缀缓存命中率及投机解码指标等）。
- **CI/构建修复 (#47442)**：升级 `nvidia-cutlass-dsl` 至 4.6.0，移除因子 wheel 文件系统路径冲突导致 `uv pip install` 出现竞态条件的旧版补丁。

**🐛 Bug 修复**
- **结构化输出与工具调用冲突 (#47439)**：修复了当 `tool_choice: auto` 和 `response_format` 并存时，格式约束将解码限制为纯 JSON，导致无法生成 `tool_calls` 的问题。
- **Nemotron MTP 加载修复 (#47440)**：修复了量化情况下 Nemotron 模型 MTP lm head 被重复加载的问题。
- **Transformers 5 兼容性 (#47438)**：修复了 OpenCUA 模型在新版 transformers 5 上的处理错误。
- **Whisper 滑动窗口计算修复 (#47437)**：修复了 Pooled Whisper encoder 滑动窗口 kernel 大小因池化单位与未池化单位混淆导致的计算错误。

---

### 🎉 Release
- 本次提供的数据中**无新增版本发布 (Release)** 信息。

---

## 🐛 Issues

### #47436 — [[Bug]: Block-scaled FP8 (compressed-tensors W8A8) crashes on load on SM120 Blackwell (RTX PRO 6000), v0.24.0 — DeepGEMM "Unknown SF transformation" assertion](https://github.com/vllm-project/vllm/issues/47436)
- **作者**: Odrec  **时间**: 2026-07-02 12:51 UTC
- **摘要**: ### Your current environment  <details> <summary>Environment</summary>  - **vLLM**: v0.24.0 (official `vllm/vllm-openai:v0.24.0` Docker image) - **GPU**: NVIDIA RTX PRO 6000 Blackwell Server Edition — **compute capability 12.0 (SM120)** - **Driver**: 580.126.20 - **CUDA (image)**: 13.0, **torch** 2.…

## 🔀 Pull Requests

### #47444 — [[Rust Frontend] Cache metric handles for scheduler & request stats](https://github.com/vllm-project/vllm/pull/47444)
- **作者**: BugenZhao  **时间**: 2026-07-02 13:49 UTC
- **标签**: rust
- **摘要**: ## Purpose  This PR reduces repeated `get_or_create` lookups on scheduler & request-level metrics paths. Instead of calling `get_or_create` every time when we want to access the global metrics handle for a request, we pre-resolve per-engine metrics handles once `ClientInner` and `GenerateOutputStrea…

### #47443 — [[Attention Backend] Select split-K per shape in HPC Attention Decode Backend, add integration tests](https://github.com/vllm-project/vllm/pull/47443)
- **作者**: Religious-J  **时间**: 2026-07-02 13:47 UTC
- **标签**: performance, v1
- **摘要**: ## Purpose  `HpcAttentionImpl.forward()` hard-coded `splitk=True` on the decode path, but empirical measurements show split-K only wins for **small batches or long sequences** — for mid-sized batches with short contexts (e.g. `num_decode_tokens=8, max_seq_len=1k`) the non-split-K path is faster. Thi…

### #47442 — [[CI/Build][Docker] Bump nvidia-cutlass-dsl to 4.6.0 and drop packaging workarounds](https://github.com/vllm-project/vllm/pull/47442)
- **作者**: arpera  **时间**: 2026-07-02 13:33 UTC
- **标签**: ci/build, nvidia
- **摘要**: ## Purpose  In `nvidia-cutlass-dsl` wheel package there was a bug in using the same paths in filesystem by two different sub-wheels that resulted in race conditions during `uv pip install` process in vLLM. Original bug reports https://github.com/NVIDIA/cutlass/issues/3259, https://github.com/NVIDIA/…

### #47441 — [[Abandoned][Attention Backend] Select split-K per shape in HPC Attention Decode Backend BF16, add integration tests](https://github.com/vllm-project/vllm/pull/47441)
- **作者**: Religious-J  **时间**: 2026-07-02 13:25 UTC
- **标签**: performance, v1
- **摘要**: ## Purpose  `HpcAttentionImpl.forward()` hard-coded `splitk=True` on the decode path, but empirical measurements show split-K only wins for **small batches or long sequences** — for mid-sized batches with short contexts (e.g. `num_decode_tokens=8, max_seq_len=1k`) the non-split-K path is faster. Thi…

### #47440 — [fix: ensure no double load of lm head in nemotron mtp](https://github.com/vllm-project/vllm/pull/47440)
- **作者**: shaunkotek  **时间**: 2026-07-02 13:24 UTC
- **摘要**: ## Purpose fix nemotron mtp head loading for quantized heads ## Test Plan load nemotron 3.5 nano, v3 super and v3 ultra and see that it works ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [X] The purpose of the PR, such as "Fix some…

### #47439 — [[Bugfix] Clear response format constraints when tool_choice is auto](https://github.com/vllm-project/vllm/pull/47439)
- **作者**: pablopupo  **时间**: 2026-07-02 13:16 UTC
- **标签**: bug, ci/build, tool-calling, rust
- **摘要**: FIX #39929  ## Purpose  When a chat request has tools with `tool_choice: "auto"` (or unset/null, which default to auto) plus a `response_format`, the format becomes a structured output constraint that boxes decoding into plain JSON, so tool-call tokens can never be emitted and `tool_calls` comes bac…

### #47438 — [[BUGFIX] Fix opencua processor on transformers 5](https://github.com/vllm-project/vllm/pull/47438)
- **作者**: microslaw  **时间**: 2026-07-02 13:07 UTC
- **标签**: bug
- **摘要**: ## Purpose Fix opencua models with new transformers version ## Test Plan vllm serve xlangai/OpenCUA-7B --dtype bfloat16 --tensor-parallel-size 1 --max-model-len 8192 --gpu-memory-utilization 0.85 -cc '{"inductor_compile_config":{"benchmark_combo_kernel":false}}' --port 8000 --trust-remote-code --lim…

### #47437 — [[Bugfix] Fix pooled Whisper encoder sliding-window kernel size](https://github.com/vllm-project/vllm/pull/47437)
- **作者**: njhill  **时间**: 2026-07-02 12:57 UTC
- **标签**: bug, ready, multi-modality, mistral
- **摘要**: The causal Whisper encoder pools `block_pool_size` encoder tokens per KV block, so its `SlidingWindowSpec.sliding_window` is expressed in pooled units for the KV cache manager. The attention kernel, however, reads its window from the same spec field but runs on the `block_pool_size`x-expanded (unpoo…

### #47435 — [[Rust Frontend] Improve scheduler stats logging parity](https://github.com/vllm-project/vllm/pull/47435)
- **作者**: BugenZhao  **时间**: 2026-07-02 12:14 UTC
- **标签**: ready, rust
- **摘要**: Signed-off-by: Bugen Zhao <i@bugenzhao.com><!-- markdownlint-disable -->   ## Purpose  This PR fills several straightforward Rust frontend periodic log-stats parity gaps with Python:  - deferred requests, preemptions, and external prefix cache hit rate - spec decoding: accepted/draft throughput, acc…

### #47434 — [[AutoRound] Support AutoRound Format Block-Wise FP8 in vLLM](https://github.com/vllm-project/vllm/pull/47434)
- **作者**: Zhenzhong1  **时间**: 2026-07-02 12:10 UTC
- **摘要**: Test:  ```bash CUDA_VISIBLE_DEVICES=1 python examples/basic/offline_inference/generate.py --model ../tmp_autoround/Llama-3.1-8B-fp-w8g128x128 --enforce-eager --gpu_memory_utilization 0.8  -------------------------------------------------- Prompt: 'Hello, my name is' Generated text: ' Adelina. I am a…
