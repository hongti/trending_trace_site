# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-17 10:06 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文简洁摘要：

### 🐛 Issue 动态
- **Qwen3.6-27B 模型导致 V1 引擎死锁 (#52551)**
  在 vLLM 0.23.0 版本中，使用 Qwen3.6-27B（全注意力+Gated-DeltaNet）单 GPU 推理时，遇到长多轮文本或大图输入，V1 引擎会发生永久性卡死，原因疑似与特定注意力配置相关。

---

### 🔀 PR 动态

**🚀 性能优化与新特性**
- **投机解码自适应 K 值 (#52559)**：为 DFlash 引入图感知自适应 K 值，解决大批次下固定 draft 长度导致吞吐量下降的问题。
- **MoE 负载均衡向量化 (#52556)**：新增可选的 `batched` EPLB 策略，在 CPU 端跨独立 MoE 层向量化平衡打包，提升性能。
- **同节点 TP=2 自定义 All-Reduce 上限 (#52555)**：允许用户为同节点 TP=2 场景自定义 all-reduce 最大尺寸，进一步压榨性能。
- **解码词重复停止机制 (#52554)**：针对语音生成模型，新增基于解码词的重复检测停止机制，解决分词/标点差异掩盖下的短语死循环重复问题。
- **自回归 Draft 支持动态 K (#52548)**：在自回归投机解码中支持并遵守正动态 K 值的传递。

**🛠️ 关键 Bug 修复**
- **修复 SM120 NVFP4 服务崩溃 (#52553)**：根本原因在于 GEMM workspace 在 CUDA graph 捕获后设备地址发生改变，现已确保其地址稳定。
- **修复 LoRA 专家参数映射顺序错误 (#52552)**：修复先前重构导致的 `lora_base_layer` 与 `routed_experts` 参数映射顺序错乱问题。

**⚙️ 调试、配置与清理**
- **V2 调试张量导出 (#52558)**：为 V2 Model Runner 新增可选的 decoder-layer 张量导出路径（`--debug-tensor-dump-output-folder`），便于 rollout 正确性比对。
- **标记无效参数 (#52557)**：弃用并警告 `use_prefill_decode_attention` 参数，该参数目前已被接受但无实际效果。
- **统一 Indexer 缓存数据类型配置 (#52550)**：将原本分散的两个无关配置项统一收敛至 `attention_config.indexer_kv_dtype`，简化稀疏注意力 K-cache 的 dtype 设置。

---

### 📦 Release 动态
- **近期无新版本发布**。

---

## 🐛 Issues

### #52551 — [[Bug]: Qwen3.6-27B (dense Gated-DeltaNet) permanently hard-wedges the V1 engine — two reproducible modes (2 large images; long multi-turn text), possibly related](https://github.com/vllm-project/vllm/issues/52551)
- **作者**: weathon  **时间**: 2026-08-17 05:28 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text OS: Ubuntu 22.04.5 LTS (x86_64), glibc 2.35 Python: 3.12.13 | platform Linux-6.17.0-1026-nvidia-x86_64 PyTorch: 2.11.0+cu130 (CUDA build 13.0) vLLM Version: 0.23.0 CUDA available: Tru…

## 🔀 Pull Requests

### #52559 — [[Spec Decode] Add graph-aware adaptive K for DFlash](https://github.com/vllm-project/vllm/pull/52559)
- **作者**: mo-ke-ke  **时间**: 2026-08-17 09:11 CST
- **标签**: documentation, speculative-decoding, qwen, nvidia, mrv2
- **摘要**: ## Purpose  Addresses the throughput degradation reported in #49730 when a fixed DFlash draft length remains enabled after target verification becomes more expensive than ordinary decoding at larger batches.  This change extends `enable_adaptive_verification` to DFlash and selects a batch-level K fr…

### #52558 — [[Debug][V2] Add opt-in rollout layer tensor dumps](https://github.com/vllm-project/vllm/pull/52558)
- **作者**: aoshen02  **时间**: 2026-08-17 08:32 CST
- **标签**: ready, mrv2
- **摘要**: ## Purpose  Add an opt-in, eager-only decoder-layer tensor dump path for rollout correctness comparisons in Model Runner V2.  - Adds `--debug-tensor-dump-output-folder` and optional `--debug-tensor-dump-layers`. - Dumps all decoder layers when no layer list is provided. - Writes one `PassN.pt` per f…

### #52557 — [[Deprecation] Warn that use_prefill_decode_attention has no effect](https://github.com/vllm-project/vllm/pull/52557)
- **作者**: brianosaurus  **时间**: 2026-08-17 08:15 CST
- **摘要**: ## Purpose  `--attention-config.use_prefill_decode_attention` is accepted, documented, and has no effect.  It used to select `ROCM_ATTN` on ROCm. #36702 made `ROCM_ATTN` the unconditional first entry in the ROCm priority list and removed the branch that read the flag, so the field has had no reader …

### #52556 — [[Performance] Vectorize EPLB packing across MoE layers](https://github.com/vllm-project/vllm/pull/52556)
- **作者**: BochuanBob  **时间**: 2026-08-17 06:31 CST
- **标签**: performance
- **摘要**: ## Purpose  Add an opt-in `batched` EPLB policy that vectorizes balanced packing across independent MoE layers on CPU.  The existing `default` policy and implementation remain unchanged. The new policy subclasses `DefaultEplbPolicy` and overrides only `balanced_packing`. It preserves the default pol…

### #52555 — [[Perf] Opt-in custom all-reduce max size for same-node TP=2](https://github.com/vllm-project/vllm/pull/52555)
- **作者**: Suppressor72  **时间**: 2026-08-17 06:29 CST
- **摘要**: ## Purpose  Give operators an **opt-in** way to **set** the custom all-reduce size ceiling on **same-node TP=2**. This is a **performance** change, not a correctness bugfix.  The constructor default remains **8 MiB**. Prefill chunks at `max_num_batched_tokens=8192` with hidden 5120 and bf16 activati…

### #52554 — [[Frontend] Add decoded-word repetition stopping](https://github.com/vllm-project/vllm/pull/52554)
- **作者**: DongjiGao  **时间**: 2026-08-17 06:05 CST
- **摘要**: ## Purpose  Some speech-generation models repeat the same decoded phrase after intervening text even when tokenizer segmentation, case, or punctuation differs. The existing repetition detector matches only adjacent token-ID N-grams at the sequence tail, so it cannot express this stopping policy.  Th…

### #52553 — [[Bugfix] Keep GEMM workspace addresses stable once CUDA graphs may have captured them](https://github.com/vllm-project/vllm/pull/52553)
- **作者**: yavarb  **时间**: 2026-08-17 05:59 CST
- **标签**: bug, nvidia
- **摘要**: Related: #52540, #34948  ## Purpose  Root-cause fix-out for the SM120 NVFP4 serving crashes in #52540 (and at least some reports in #34948): a **shared GEMM workspace that changes device address after CUDA graphs have captured launches against it** turns every subsequent replay into a use-after-free…

### #52552 — [[BugFix] lora_base_layer / routed_experts order in expert param mapping](https://github.com/vllm-project/vllm/pull/52552)
- **作者**: HollowMan6  **时间**: 2026-08-17 05:55 CST
- **标签**: bug, ready
- **摘要**: ## Purpose  Follow up of https://github.com/vllm-project/vllm/pull/31104 as broken by refactor at https://github.com/vllm-project/vllm/pull/41184  In RoutedExperts.build_expert_params_mapping the LoRA base-layer prefix was concatenated AFTER routed_experts_prefix, producing ``experts.routed_experts.…

### #52550 — [[Config] Unify indexer cache dtype under attention_config.indexer_kv_dtype](https://github.com/vllm-project/vllm/pull/52550)
- **作者**: zyongye  **时间**: 2026-08-17 05:06 CST
- **标签**: ready
- **摘要**: ## Purpose  The sparse-attention indexer picked its K-cache dtype through two unrelated knobs:  | flag | type | read by | | --- | --- | --- | | `attention_config.use_fp4_indexer_cache` | `bool` | DeepSeek V3.2/V4 only | | `attention_config.indexer_kv_dtype` | enum | MiniMax M3 only |  Because each m…

### #52548 — [[Spec Decode] Honor positive dynamic K in autoregressive drafting](https://github.com/vllm-project/vllm/pull/52548)
- **作者**: jpezzulli  **时间**: 2026-08-17 04:17 CST
- **标签**: speculative-decoding, nvidia, mrv2
- **摘要**: # STACKED / DEPENDENT PR  Depends on:  - #49652 - fixed-query-length autoregressive draft CUDA graph capture - #51575 - runtime K propagation, K0 behavior, and active-width publication  This branch currently contains those prerequisite commits solely so the positive-K follow-on can be built and vali…
