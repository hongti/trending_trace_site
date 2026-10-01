# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-01 14:25 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 📌 Issue 动态
1. **模型输出异常问题 (#59551)**：Qwen3.6-35B-A3B 模型在 Intel B70 显卡上使用 TP=2 和 DP=2 配置时，输出内容为乱码。
2. **推测解码性能波动问题 (#59548)**：在 L4 显卡上，推测解码的吞吐量存在启动依赖导致的波动（CV 高达 13.92%），该问题已在 0.30.0 版本中得到解决。作者借此提出了针对单次启动基准测试的测量建议。

### 🚀 PR 动态
1. **前端与 API 优化**：
   - **[Rust 前端]** 为 OpenAI 兼容端点 (`/v1/chat/completions` 等) 新增了 `logprob_token_ids` 支持 (#59549)。
   - **[Bug修复]** 修复了在流式输出前，未对复用的提示词 token ids 进行词汇表边界检查的问题，防止越界 (#59555)。

2. **推测解码与水印技术**：
   - **[新特性]** 支持在 DFlash 推测解码中使用 `dual_key_gumbel` 水印技术。通过单次前向传递计算所有草稿 logits 并逐步进行水印采样 (#59554)。

3. **ROCm 性能与修复**：
   - **[性能优化]** 将 MXFP4 激活量化融合到 GDN 门控 RMSNorm 中，减少了 decode 阶段的内核启动次数 (#59553)。
   - **[Bug修复]** 修复了 `ROCM_ATTN` 中滑动窗口边界设置错误的问题（之前存储为 `W-1, 0` 导致底层解码越界） (#59550)。
   - K3 chunk kda aiter 启用 (#59552)。

4. **KV 卸载 内存与并发深度优化**：作者 Etelis 提交了系列 PR，大幅优化了多卡/多 worker 环境下的 CPU 内存使用效率：
   - 将 CPU 卸载区域拆分到不同的共享内存文件中，解决单 `/dev/shm` 文件导致的页缓存插入串行化瓶颈 (#59546)。
   - 在复制布局外为每个 rank 分配独立的 CPU 区域，避免跨 rank 读取 (#59547)。
   - 将复制布局的预分页拆分到不同 worker 中，避免 TP4 时四次遍历同一文件 (#59545)。
   - 仅锁定每个 worker 自身的卸载槽位，避免在 TP4 下对所有页重复进行 `cudaHostRegister` 四次 (#59544)。

### 📦 Release 动态
*近期无新版本发布信息。*

---

## 🐛 Issues

### #59551 — [[Bug]: Qwen3.6-35B-A3B model with TP 2 and DP 2 returns gibberish output on Intel B70 cards](https://github.com/vllm-project/vllm/issues/59551)
- **作者**: nvsreerag  **时间**: 2026-10-01 13:45 CST
- **标签**: bug, intel-gpu, quantization
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Your output of `python collect_env.py` here ```  </details>   ### 🐛 Describe the bug  Qwen3.6-35B-A3B model with TP 2 and DP 2 returns gibberish output on Intel B70 cards  **Steps to …

### #59548 — [[Performance]: spec-decode boot-to-boot throughput dispersion on L4 (CV up to 13.92%), resolved in 0.30.0](https://github.com/vllm-project/vllm/issues/59548)
- **作者**: akshathtiwari  **时间**: 2026-10-01 12:43 CST
- **标签**: performance
- **摘要**: ### Proposal to improve performance  Not a proposal so much as a measurement suggestion. Speculative-decoding throughput on this stack is boot-dependent to a degree a single-boot benchmark cannot see: on 0.29.0 / L4, twelve boots of a byte-identical configuration gave CV 13.92% in one container and …

## 🔀 Pull Requests

### #59555 — [[Bugfix][Frontend] Check reused prompt token ids against the vocab before streaming](https://github.com/vllm-project/vllm/pull/59555)
- **作者**: shijie-lyu  **时间**: 2026-10-01 14:24 CST
- **标签**: bug
- **摘要**: ## Purpose  Follow-up to #55771, from @NickLucche's review: check that the reused `kv_transfer_params["prompt_token_ids"]` are inside the vocabulary when the prompt is rendered.  Today only `InputProcessor._validate_model_input` checks the vocab bound. With `stream=true` it runs after the stream is …

### #59554 — [[Spec Decode][Watermarking] Support DFlash with dual_key_gumbel](https://github.com/vllm-project/vllm/pull/59554)
- **作者**: shernshiou  **时间**: 2026-10-01 14:13 CST
- **标签**: documentation, speculative-decoding, mrv2, dflash
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Enable `dual_key_gumbel` watermarking with DFlash speculative decoding. DFlash computes all N draft logits in one forward pass. This PR watermark-samples them one step at a time, so draft *k*'s context includes drafts 0…k−1, which is what the target verifi…

### #59553 — [[ROCm][Perf] Fuse the MXFP4 activation quant into the GDN gated RMSNorm](https://github.com/vllm-project/vllm/pull/59553)
- **作者**: mjkvaak-amd  **时间**: 2026-10-01 14:13 CST
- **标签**: rocm, torch.compile, quantization
- **摘要**: ## Purpose  With online MXFP4 on the GDN `out_proj` and `VLLM_ROCM_USE_AITER_FP4_ASM_GEMM=1`, the GDN output path at decode runs three launches: `RMSNormGated`, a separate `per_1x32_f4_quant_hip` activation quant, and the GEMM.   This PR adds `AiterRMSNormGatedMxfp4GemmPattern` to `RocmAiterRMSNormQ…

### #59552 — [k3 chunk kda aiter enablment](https://github.com/vllm-project/vllm/pull/59552)
- **作者**: omuhamma  **时间**: 2026-10-01 14:12 CST
- **标签**: kimi, k3
- **摘要**: ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this PR.  - [ ] **Design Fit:** Minimizes…

### #59550 — [[ROCm][Bugfix] Fix ROCM_ATTN sliding-window boundary](https://github.com/vllm-project/vllm/pull/59550)
- **作者**: tangzzycc  **时间**: 2026-10-01 13:42 CST
- **标签**: bug, rocm
- **摘要**: ## Purpose  `ROCM_ATTN` stores the decoder window as `(W - 1, 0)`, where `W - 1` is the left span. The underlying [decode](https://github.com/vllm-project/vllm/blob/4c2d277643e217344056e1d2c42115d5f005912f/vllm/v1/attention/ops/chunked_prefill_paged_decode.py#L234) and [prefix-prefill](https://githu…

### #59549 — [[Rust Frontend] Support logprob_token_ids on OpenAI endpoints](https://github.com/vllm-project/vllm/pull/59549)
- **作者**: zupengwang  **时间**: 2026-10-01 13:14 CST
- **标签**: rust
- **摘要**: ## Overview  Add `logprob_token_ids` support to the Rust `/v1/chat/completions` and `/v1/completions` endpoints. Related to the selected-token item in #44280; [scope claim](https://github.com/vllm-project/vllm/issues/44280#issuecomment-5914657390).  ## Claims  - Forward non-empty token selections th…

### #59547 — [[KV Offload] Give each rank its own CPU region outside the replicated layout](https://github.com/vllm-project/vllm/pull/59547)
- **作者**: Etelis  **时间**: 2026-10-01 12:35 CST
- **摘要**: Outside the replicated layout, ranks never read each other's slots, but they still share one region whose rows interleave all ranks. Every rank pre-faults and pins its strided slots in the same shm file, contending with the others.  Give each rank its own region holding only its slots, in its own sh…

### #59546 — [[KV Offload] Split the CPU offload region across shm files](https://github.com/vllm-project/vllm/pull/59546)
- **作者**: Etelis  **时间**: 2026-10-01 12:35 CST
- **摘要**: Pre-faulting the CPU offload region is capped by its single `/dev/shm` file: page-cache inserts into one shmem file are serialized, so more workers or threads stop helping. On one H200 node, eight processes pre-faulting disjoint parts of a 64 GiB file reach 9.8 GiB/s in total. Given one file each, t…

### #59545 — [[KV Offload] Split replicated-layout pre-faulting across workers](https://github.com/vllm-project/vllm/pull/59545)
- **作者**: Etelis  **时间**: 2026-10-01 12:35 CST
- **摘要**: With the replicated layout, every worker maps the same bytes, yet every worker still pre-faults the whole region. At TP4, that is four passes over one shm file, all contending for the same pages.  Give each worker one contiguous, page-aligned share to pre-fault instead. The pages are shared, so the …

### #59544 — [[KV Offload] Pin only each worker's own offload slots](https://github.com/vllm-project/vllm/pull/59544)
- **作者**: Etelis  **时间**: 2026-10-01 12:35 CST
- **摘要**: Outside the replicated layout, each worker only uses its own slot of every row in the shared CPU region, yet every worker `cudaHostRegister`s the whole region. At TP4, every page gets pinned four times.  Register only the worker's slot of each row, rounded out to pages. The replicated layout keeps w…
