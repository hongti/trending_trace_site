# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-25 13:08 CST

## AI 总结

# vLLM 仓库近期动态摘要（2026-10-25）

## 📋 Issue
本次数据中未包含 Issue 活动。

## 🔀 Pull Request

### Bug 修复
- **#63546**（V1 执行器）：修复 worker 在 `sleep()` 中释放内存后异常时执行器未记录睡眠状态的 bug，避免后续 `wake_up()` 不恢复内存、引擎误调度的问题。
- **#63545**（Qwen2.5/3 Thinker）：修复分块预填充（Chunked Prefill）场景下音频/视频 token 交错的边界问题，避免视频与音频错位。
- **#63543**（前端 SystemOne）：在含图像的 `/v1/systemone` 混合模型请求中，修复共享前缀预热丢失多模态元数据的问题。
- **#63542**（ROCm AITER）：修正 `split_decodes_prefills_and_extends` 假定 extend 先于 new prompt（MRV1 行为）的 bug，适配 MRV2 仅保证 decode 在前的语义。
- **#63538**（Gemma 系列）：在 `EmbeddingGemma2`、`DiffusionGemma` 中补回 Gemma4 的 `_get_max_soft_tokens` 查找（跟随 #60687 重构）。
- **#63537**（CI）：修复 Qwen2 embed 测试的 SentenceTransformer 内存泄漏，以及 `HfRunner` no-cache 模式下 logprobs 的处理。

### 性能优化
- **#63544**（GLM-5.3-Flash）：将 KDA（Kimi Delta Attention）层的单 token 解码与多 token 草稿-验证解码迁移到 FlashInfer kernel（`fused_kda_decode` 等）。
- **#63541**（MoE / Kimi K3）：在 `deep_gemm_mega_moe` 后端 + 专家并行 + TP>1 + PP=1 时，将 MegaMoM 对称 buffer 大小调整为单 sequence-parallel shard，降低显存占用。
- **#63540**（DSv4）：`fused_inv_rope_fp8_quant` 由「每 (token, head) 一个单 warp program」改为「每 program 处理 4 个 head」，突破 1.5 TB/s 的瓶颈。
- **#63539**（MRV2）：对 `_apply_write_kernel` 启动网格设上限，避免长短混合批次中由单个长 staged write 触发 `num_writes × max_chunks` 个近乎空转的 program（#62698 跟进）。

## 🚀 Release
本次数据中未包含 Release 活动。

---

**总体观察**：本轮 PR 集中在 **V1/MRV2 引擎路径的稳定性修复** 与 **特定模型（GLM-5.3-Flash、Kimi K3、DSv4、Qwen2.5/3 Omni）的性能/正确性优化**，体现了 vLLM 在多模态对齐、MoE 显存控制与 kernel 融合方面持续推进。

---

## 🔀 Pull Requests

### #63546 — [[Bugfix][V1] Keep the executor wakeable after a failed sleep](https://github.com/vllm-project/vllm/pull/63546)
- **作者**: bluayer  **时间**: 2026-10-25 12:41 CST
- **标签**: bug
- **摘要**: ## Overview  If a worker releases its memory and then raises during `sleep()`, the executor never records the sleep. A later `wake_up()` then returns without restoring worker memory, and the engine resumes scheduling on released memory. This PR records the sleep tags even when the sleep RPC fails. S…

### #63545 — [[Bugfix / Model] Fix Chunked Prefill Audio/Video Misalignment for Qwen 2.5 & 3 Thinker](https://github.com/vllm-project/vllm/pull/63545)
- **作者**: alex-jw-brooks  **时间**: 2026-10-25 12:41 CST
- **标签**: bug, multi-modality, qwen
- **摘要**: ## Overview There's an edge case in Qwen2.5/Qwen3Omni's thinker where the `use_audio_in_video` check for interleaved looks for alternating A/V tokens, which is normally fine, but can cause video and audio embeddings to map out incorrectly with chunked prefills.  ## Claims  The current code checks fo…

### #63544 — [[Perf][GLM-5.3-Flash] KDA decode and prefill through FlashInfer](https://github.com/vllm-project/vllm/pull/63544)
- **作者**: suiyoubi  **时间**: 2026-10-25 12:28 CST
- **标签**: nvidia, glm
- **摘要**: ## Overview  This PR moves GLM-5.3-Flash's KDA (Kimi Delta Attention) layers onto FlashInfer kernels: - the fused single-token decode (`fused_kda_decode`); - the packed multi-token decode for draft-verify rows (`packed_fused_kda_decode`); - optionally, the CuTe DSL chunked prefill (`recurrent_kda(ba…

### #63543 — [[Bugfix][Frontend] Preserve multimodal inputs in SystemOne prefix warmup](https://github.com/vllm-project/vllm/pull/63543)
- **作者**: waizuichougou  **时间**: 2026-10-25 11:58 CST
- **标签**: bug, frontend
- **摘要**: ## Overview  Fix shared-prefix warmup across questions in an image-bearing `/v1/systemone` request on hybrid models. The token-only warmup from #61192 loses the multimodal metadata required for prefix reuse, including on the image path added in #62181.  ## Claims  - Avoid redundant prefill by preser…

### #63542 — [[ROCm][Bugfix] Fix ROCM_AITER_FA extend/prefill split bug](https://github.com/vllm-project/vllm/pull/63542)
- **作者**: qli88  **时间**: 2026-10-25 11:53 CST
- **标签**: bug, rocm
- **摘要**: ## Overview  split_decodes_prefills_and_extends assumed extends precede new prompts (MRV1's behavior), which MRV2 does not guarantee (it only puts decodes first). An extend that followed a new prompt was classified as a prefill and attended without its KV cache context. This will silently downgrade …

### #63541 — [[Perf][MoE] Size the DeepGEMM MegaMoE symmetric buffer for one sequence-parallel shard](https://github.com/vllm-project/vllm/pull/63541)
- **作者**: mispa-ms  **时间**: 2026-10-25 11:51 CST
- **标签**: kimi, k3
- **摘要**: ## Overview  With `--moe-backend deep_gemm_mega_moe`, `--enable-expert-parallel`, TP > 1 and PP = 1, Kimi K3 runs its MoE layers sequence parallel: each rank feeds MegaMoE at most `cdiv(max_num_batched_tokens, tp_size)` tokens. The MegaMoE symmetric buffer is still sized for `max_num_batched_tokens`…

### #63540 — [[Perf][DSv4] Process 4 heads per program in fused_inv_rope_fp8_quant](https://github.com/vllm-project/vllm/pull/63540)
- **作者**: guanzheh  **时间**: 2026-10-25 11:42 CST
- **标签**: deepseek, quantization, DSv4
- **摘要**: ## Overview  `fused_inv_rope_fp8_quant` applies inverse RoPE to the attention output and quantizes it to block FP8 for `wo_a`. It launched one single-warp program per (token, head), which ran at 1.5 TB/s on B200 and took 8.6% of a 16k-token DeepSeek-V4-Flash prefill step (TP2).  This PR makes each p…

### #63539 — [[Perf][MRV2] Bound staged-write grid size for mixed-length batches](https://github.com/vllm-project/vllm/pull/63539)
- **作者**: njhill  **时间**: 2026-10-25 11:38 CST
- **标签**: mrv2
- **摘要**: ## Overview  Follow-up to #62698. Bound the `_apply_write_kernel` launch grid so a single long staged write next to many short ones no longer launches `num_writes x max_chunks` mostly-idle programs, and have the kernel stride over chunks so any grid size is correct.  ## Claims  - Mixed-length staged…

### #63538 — [[Bugfix][Model] Borrow Gemma4 _get_max_soft_tokens in EmbeddingGemma2 and DiffusionGemma](https://github.com/vllm-project/vllm/pull/63538)
- **作者**: AndreasKaratzas  **时间**: 2026-10-25 11:33 CST
- **标签**: bug
- **摘要**: - `embedding_gemma2.py`, `diffusion_gemma.py`: copy Gemma4's `_get_max_soft_tokens` alongside the `_process_image_input` they already borrow  #60687 moved the `max_soft_tokens` lookup into a new `Gemma4ForConditionalGeneration._get_max_soft_tokens` helper that `_process_image_input` now calls. `Embe…

### #63537 — [[CI] Fix the Qwen2 embed memory leak and HfRunner no-cache logprobs](https://github.com/vllm-project/vllm/pull/63537)
- **作者**: AndreasKaratzas  **时间**: 2026-10-25 11:33 CST
- **标签**: qwen, pooling
- **摘要**: - `test_embedding.py`: edit the Qwen2 embed SentenceTransformer through `hf_model.model` so `HfRunner` can free it before vLLM starts - `conftest.py`: `HfRunner` keeps only the last position of every generation step's logprobs, not just the first step's  These are two separate failures in the Extend…
