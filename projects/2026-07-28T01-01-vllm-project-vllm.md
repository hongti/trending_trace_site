# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-28 09:01 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库最近的动态摘要：

### 🚨 Issue 动态
1. **LoRA 量化模型输出异常** (#50059)：在 compressed-tensors int4 (W4A16) 模型上使用 LoRA 时，rank-32 全层适配器生成的输出效果极差且不可复现，而 rank-8 部分层适配器则表现正常（影响 v0.17.1–0.24.0 版本，RTX 5090）。
2. **Kimi-K3 FP8 KV 缓存报错** (#50056)：使用 `--kvcache-dtype-fp8` 运行 Kimi-K3 时触发 RuntimeError，提示需启用 fp8 prefill query 量化配置。

### 🛠️ Pull Request 动态
**核心修复：**
- **修复 MLA 注意力机制 Padding 问题** (#50060)：仅当 `value` 比 query/key 头部更窄时才进行填充，修复了扩展 MLA 及 packed grouped/multi-query 注意力的兼容问题。
- **修复 ZMQ 缓冲区并发复用问题** (#50053)：避免在 ZMQ 尚未发送完数据时，引擎核心复用 msgpack payload 缓冲区导致的数据错乱。
- **修复索引器 Block 大小不匹配** (#50050)：修正了 `expanded_block_table_buffer` 计算时未与 `BlockTableGroup` 保持一致的 128 对齐圆整逻辑。

**新特性与性能优化：**
- **Kimi-K3 DCP 支持** (#50055)：为 Kimi-K3 引入 DCP 核心，包含直接对称内存 DCP A2A、融合 MLA 层的 DCP 支持及 NVLS 多播 q-gating。
- **ROCm 性能优化** (#50054)：针对 Kimi-K3 KDA 解码门控进行了算子融合。
- **Helion 内核优化** (#50061)：重新生成 H100 Helion 内核，保留特化并将形状值作为 `tl.constexpr` 传递给 Triton，提升调优产物的通用性。

**前端与文档：**
- **Rust 前端更新**：支持 `truncate_prompt_tokens` 参数 (#50052)；修复了随机基准测试中稀疏词表（如 `o200k_base`）生成无效 token ID 的问题 (#50058)。
- **文档完善**：移除了 EP（专家并行）的实验性警告 (#50057)；补充了关于后置生成检查不适合作为 logits processors 使用的说明 (#50051)。

### 🎉 Release 动态
**v0.26.0**
- **版本规模**：包含 411 次提交，来自 212 位贡献者（其中 61 位为新贡献者）。
- **核心亮点**：正式引入了全新的 **Inkling 模型家族**支持（*注：原文 Release 摘要在此截断，该版本应包含更多重要特性更新*）。

---

## 🐛 Issues

### #50059 — [LoRA on a compressed-tensors int4 (W4A16) model: rank-32 all-layer adapters produce weak, non-reproducible outputs; rank-8 partial-coverage adapters work fine (0.17.1–0.24.0, RTX 5090)](https://github.com/vllm-project/vllm/issues/50059)
- **作者**: Paul-Kyle  **时间**: 2026-07-28 08:24 CST
- **标签**: quantization
- **摘要**: ### Summary  Serving LoRA adapters over a compressed-tensors int4 (W4A16, group size 32) checkpoint: rank-32 adapters targeting all layers produce outputs that are much weaker than the same adapter's reference behavior, and **different across identical runs** (same seed, same prompt, temperature 0 —…

### #50056 — [[Bug]: kimi-k3 --kvcache-dtype-fp8 error](https://github.com/vllm-project/vllm/issues/50056)
- **作者**: yiminghub2024  **时间**: 2026-07-28 07:45 CST
- **标签**: bug, kimi, k3
- **摘要**: ### Your current environment  untimeError: Worker failed with error 'Kimi-K3 fp8 KV cache requires an fp8 prefill query; enable --attention-config '{"use_prefill_query_quantization": true}'.', please check the stack trace above for the root cause   i have add   --attention-config '{"use_prefill_quer…

## 🔀 Pull Requests

### #50061 — [[DO NOT REVIEW][Kernel][Helion] Preserve specializations in generated kernels](https://github.com/vllm-project/vllm/pull/50061)
- **作者**: yushangdi  **时间**: 2026-07-28 08:54 CST
- **标签**: quantization
- **摘要**: ## Summary  - regenerate the checked-in H100 Helion kernels with `preserve_specializations=True` - pass specialized shape values to Triton as `tl.constexpr` launch arguments so one tuned artifact can serve compatible runtime shapes - select the nearest tuned static configuration while retaining toke…

### #50060 — [[Bugfix] Only pad transformers backend `value` when it is narrower](https://github.com/vllm-project/vllm/pull/50060)
- **作者**: njhill  **时间**: 2026-07-28 08:34 CST
- **标签**: bug, ready
- **摘要**: #49982 added padding of `value` up to the query/key head size in `vllm_attention_forward` for expanded MLA, guarded on `head_dim_v != head_dim_qk`. #49987 then added support for packed grouped/multi-query QKV projections, whose attention modules keep `key`/`value` packed rather than split per head, …

### #50058 — [[Rust Frontend] Prevent invalid token IDs in random benchmarks](https://github.com/vllm-project/vllm/pull/50058)
- **作者**: reidliu41  **时间**: 2026-07-28 08:01 CST
- **标签**: rust
- **摘要**: ## Purpose    Random benchmark generation treated every ID in `0..vocab_size` as valid for   built-in tiktoken encodings. That assumption is incorrect for sparse   vocabularies:    - `o200k_base` contains 275 unassigned IDs in the configured range.   - `cl100k_base` contains 16 unassigned IDs in the…

### #50057 — [[Docs] Remove experimental warning for EP](https://github.com/vllm-project/vllm/pull/50057)
- **作者**: WoosukKwon  **时间**: 2026-07-28 07:52 CST
- **标签**: documentation

### #50055 — [[Kimi-K3] DCP support](https://github.com/vllm-project/vllm/pull/50055)
- **作者**: GirasoleY  **时间**: 2026-07-28 07:39 CST
- **标签**: ci/build, v1, nvidia, kimi, k3
- **摘要**: ## Summary  DCP core for Kimi-K3, three commits: * adding the direct symmetric-memory DCP A2A (with in-kernel empty-KV-shard masking) * DCP support for the fused MLA layer * NVLS-multicast direct q-gather + multimem chunked-context KV gather  ## Accuracy   .buildkite/kimi-k3/rack5/gsm8k.yaml:  | Arm…

### #50054 — [perf(rocm): fuse Kimi-K3 KDA decode gate](https://github.com/vllm-project/vllm/pull/50054)
- **作者**: JohnQinAMD  **时间**: 2026-07-28 07:27 CST
- **标签**: rocm, kimi, k3
- **摘要**: # perf(rocm): fuse Kimi-K3 KDA decode gate  ## Stack and dependency  This is a focused ROCm performance PR stacked on https://github.com/vllm-project/vllm/pull/50000. Its base branch should be `vllm-project/vllm:kimi-k3` until PR #50000 merges.  The combined result also depends on the companion ROCm…

### #50053 — [[Bugfix] Don't reuse engine core payload buffer while zmq is sending it](https://github.com/vllm-project/vllm/pull/50053)
- **作者**: njhill  **时间**: 2026-07-28 07:17 CST
- **标签**: bug, ready, v1
- **摘要**: `EngineCoreProc.process_output_sockets` recycles the msgpack payload bytearray across messages via `MsgpackEncoder.encode_into`, gated on the tracker returned by `send_multipart(copy=False, track=True)`.  That gate never engages. `Socket.send_multipart()` returns a tracker for the *last* frame only,…

### #50052 — [[Frontend][Rust] Support truncate_prompt_tokens](https://github.com/vllm-project/vllm/pull/50052)
- **作者**: almogtavor  **时间**: 2026-07-28 07:04 CST
- **标签**: rust
- **摘要**: ## Purpose  Roadmap item from #44280: `truncate_prompt_tokens`.  The Rust frontend rejected it with `truncate_prompt_tokens is not supported` on `/v1/completions` and `/v1/chat/completions`, and with `InvalidArgument` over gRPC. This accepts it.  ## Implementation  Truncation is applied once, in `Te…

### #50051 — [docs: clarify post-generation checks vs logits processors](https://github.com/vllm-project/vllm/pull/50051)
- **作者**: kantik001  **时间**: 2026-07-28 07:01 CST
- **标签**: documentation
- **摘要**: ## Summary Adds a short guidance section to `docs/features/custom_logitsprocs.md`:  - Decoded-text checks (numeric grounding, PII on the answer string, external safety classifiers) are a poor fit for logits processors. - Calling gRPC/HTTP from `apply()` / `update_state()` every engine step is discou…

### #50050 — [[Bugfix] Fix indexer expanded_block_table_buffer size mismatch](https://github.com/vllm-project/vllm/pull/50050)
- **作者**: elvircrn  **时间**: 2026-07-28 06:37 CST
- **标签**: bug, v1, deepseek
- **摘要**: The indexer's `expanded_block_table_buffer` was sized with an independent `cdiv(max_model_len, block_size * shard_count)` that didn't apply the 128-alignment rounding that `BlockTableGroup` applies to the actual block table. When those differ (e.g. 3895 vs 3896 columns), `_prepare_decode_tensors` cr…

## 🚀 Releases

### [v0.26.0](https://github.com/vllm-project/vllm/releases/tag/v0.26.0)
- **作者**: khluu  **时间**: 2026-07-27 09:06 CST
- **摘要**: # vLLM v0.26.0 Release Notes  ## Highlights  This release features 411 commits from 212 contributors (61 new)!  * **New Inkling model family** with a full support stack: base modeling (#48799), piecewise CUDA graph support (#48822), Hopper FA4 relative attention (#48858), MTP=1 speculative decoding …
