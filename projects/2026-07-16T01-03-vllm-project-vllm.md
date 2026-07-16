# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-16 09:03 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的简洁摘要：

### 📌 Issue 动态
* **Qwen3.5 FP8 KV Cache 性能退化 (#48786)**：在 H100 (SM90 架构) 上运行 Qwen3.5-35B-A3B-FP8 模型时，发现使用 `--kv-cache-dtype fp8` 的推理速度反而比 BF16 更慢。性能下降主要归因于 FlashAttention 3 (FA3) 的相关机制。

---

### 🚀 Pull Request 动态

**1. 核心模型修复（Qwen3.5 MTP 专项）**
* **修复多模态权重加载错误 (#48794)**：修复了 `Qwen3_5MTP.load_weights()` 误将多模态专属的 `embed_tokens_extend` 权重当作共享 token embedding 加载的 Bug。
* **修复多层 MTP Dispatch 失效 (#48793)**：修复了通用投机解码提议器未向 `Qwen3_5MTP.forward` 传递 `spec_step_idx`，导致多层 MTP (多Token预测) 调度逻辑失效的问题。

**2. 新特性与性能优化**
* **投机解码 KV Cache 独立控制 (#48787)**：为 `speculative_config` 新增 `kv_cache_dtype` 参数，允许投机模型的 KV Cache 数据类型与目标模型不同，解决了此前同时开启 FP8 KV Cache 与投机解码时的崩溃问题。
* **Gated DeltaNet 解码加速 (#48792)**：引入 ReplaySSM 缓存机制，通过缓存 SSM 输入显著加速 Gated DeltaNet 的标准解码过程（依赖前置 PR #48018）。
* **ModelRunner V2 序列池化 (#48791)**：为嵌入和分类模型启用了序列池化功能，修复了相关 Issue 并为后续 PR 解除阻塞。
* **ROCm 稀疏解码性能提升 (#48788)**：优化了 AMD gfx950 上 split-K 稀疏解码降维的占用率，解决了此前每个工作组处理 16 个头部时 FP32 累加器限制并行度的问题。

**3. 其他修复、工具与文档**
* **修复激活量化 Dispatch (#48785)**：修复了 WNA4Int/WNA8Int 量化路径中，子类调用 `from_config` 时错误回退到 wNa16 的问题。
* **新增 Triton Proton Profiler (#48789)**：引入 Triton Proton 作为继 PyTorch 和 CUDA 之后第三个可选的 Worker 性能分析后端。
* **新增 Pixeltable 集成文档 (#48790)**：在官方文档的推理与服务集成章节中，新增了 Pixeltable 集成指南（与 LangChain、LlamaIndex 并列）。
* **ROCm CI 精度修复 (#48784)**：为 `test_bert_for_masked_lm` 测试中的参考 runner 设置了“最高”矩阵乘法精度，解决了 ROCm 环境下因细微精度差异导致的测试失败。

---

### 📦 Release 动态
* 本次提供的动态数据中**无新版本发布**信息。

---

## 🐛 Issues

### #48786 — [[Bug]: Qwen3.5-35B FP8 kv cache is slower than BF16 kv cache on SM90 (Hopper)](https://github.com/vllm-project/vllm/issues/48786)
- **作者**: amukkara  **时间**: 2026-07-16 05:18 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Your output of `python collect_env.py` here ```  Collecting environment information... ==============================         System Info ============================== OS            …

## 🔀 Pull Requests

### #48794 — [[Bugfix][Model] Fix Qwen3.5 MTP multimodal weight loading](https://github.com/vllm-project/vllm/pull/48794)
- **作者**: liuwei-git  **时间**: 2026-07-16 08:40 CST
- **标签**: bug, qwen
- **摘要**: ## Purpose  `Qwen3_5MTP.load_weights()` treats any weight name containing `embed_tokens` as the shared token embedding. Multimodal checkpoints may also contain `embed_tokens_extend` weights, which belong to multimodal components and must not be loaded into the MTP drafter's embedding table.  This ch…

### #48793 — [[Bugfix][Spec Decode] Fix multi-layer Qwen3.5 MTP dispatch](https://github.com/vllm-project/vllm/pull/48793)
- **作者**: liuwei-git  **时间**: 2026-07-16 08:19 CST
- **标签**: bug, speculative-decoding, v1, qwen
- **摘要**: ## Purpose  Qwen3.5's MTP predictor already selects a trained layer using `spec_step_idx % num_mtp_layers`. However, the generic speculative proposer did not pass `spec_step_idx`, and `Qwen3_5MTP.forward()` did not forward it to the inner predictor. Every draft pass therefore used the default index …

### #48792 — [[Kernel] ReplaySSM: cache SSM inputs for faster Gated DeltaNet standard decode](https://github.com/vllm-project/vllm/pull/48792)
- **作者**: Johnny-Liou  **时间**: 2026-07-16 06:59 CST
- **标签**: performance, v1, qwen
- **摘要**: > [!IMPORTANT] > This PR is stacked on #48018 (ReplaySSM Mamba2 standard decode) and should be merged only after #48018 lands. Until then, the diff here also includes #48018's commits; for the GDN-only changes, see the stacked branch view in [Johnny-Liou/vllm#1](https://github.com/Johnny-Liou/vllm/p…

### #48791 — [[ModelRunner V2] Enable sequence pooling for embedding and classification models](https://github.com/vllm-project/vllm/pull/48791)
- **作者**: taneem-ibrahim  **时间**: 2026-07-16 06:14 CST
- **标签**: v1
- **摘要**: ## Purpose  Fixes part of https://github.com/vllm-project/vllm/issues/41286 and unblocks the sequence-level portion of https://github.com/vllm-project/vllm/pull/48290 (failing build [77582]( https://buildkite.com/vllm/ci/builds/77582)).   Model Runner V2 previously hardcoded last-token gathering and…

### #48790 — [[Doc]: Add Pixeltable integration to inference & serving/integrations docs](https://github.com/vllm-project/vllm/pull/48790)
- **作者**: apreshill  **时间**: 2026-07-16 05:31 CST
- **标签**: documentation
- **摘要**: ## Summary  Add a Pixeltable integration page under **Inference and Serving → Integrations**, alongside LangChain and LlamaIndex.  The page: - Briefly explains Pixeltable (https://github.com/pixeltable/pixeltable) - Shows how to run Qwen2.5-VL on an image with `pxtf.vllm.chat_completions` - Links to…

### #48789 — [[Profiler] Add Triton Proton profiling backend](https://github.com/vllm-project/vllm/pull/48789)
- **作者**: Luosuu  **时间**: 2026-07-16 05:26 CST
- **标签**: documentation, performance, v1
- **摘要**: ## Purpose  Add Triton Proton as an optional third worker profiling backend alongside PyTorch and CUDA profiling.  - Expose typed Proton context, data, backend, mode, and Triton hook configuration. - Keep Proton lazily imported so existing profiler users do not need a Proton-capable Triton installat…

### #48788 — [[ROCm][Perf][DSV4] Improve sparse decode reduction occupancy on gfx950](https://github.com/vllm-project/vllm/pull/48788)
- **作者**: Fangzhou-Ai  **时间**: 2026-07-16 05:20 CST
- **标签**: rocm, ready, v1
- **摘要**: ## Summary  The split-K sparse decode reducer currently processes 16 heads per workgroup, which keeps a `[16, COMB_DIM]` FP32 accumulator in each workgroup and limits workgroup-level parallelism.  This change processes one head per reducer workgroup. It lowers per-workgroup accumulator/register pres…

### #48787 — [[Spec Decode] Add kv_cache_dtype to speculative_config to control separately from target](https://github.com/vllm-project/vllm/pull/48787)
- **作者**: mgoin  **时间**: 2026-07-16 05:19 CST
- **标签**: speculative-decoding, ready, v1
- **摘要**: ## Purpose  Currently running `vllm serve RedHatAI/GLM-5.2-NVFP4-FP8 --tensor-parallel-size 4 --spec-model GLM-5.2-speculator.dspark --spec-method dspark --spec-tokens 7 --kv-cache-dtype fp8` will fail on Blackwell since the global `--kv-cache-dtype fp8` will apply to both the GLM 5.2 target and the…

### #48785 — [[Bugfix] Fix activation quantization dispatch for WNA4Int/WNA8Int](https://github.com/vllm-project/vllm/pull/48785)
- **作者**: HDCharles  **时间**: 2026-07-16 04:09 CST
- **标签**: bug
- **摘要**: ## Summary  CT humming path was pretending to do wNaM but was falling back to wNa16   if you call X.from_config from a subclass it won't enter into the   ```         if cls is BaseInputSchema:             quant_method = config["quant_method"]             ... ```  [branch](https://github.com/inclusio…

### #48784 — [[ROCm][CI] Set "highest" matmul precision for reference hf_runner in `test_bert_for_masked_lm`](https://github.com/vllm-project/vllm/pull/48784)
- **作者**: micah-wil  **时间**: 2026-07-16 04:05 CST
- **标签**: rocm
- **摘要**: https://github.com/vllm-project/vllm/pull/48463 added `test_bert_for_masked_lm` which fails on ROCm due to subtle accuracy differences:  ``` >           torch.testing.assert_close(hf_output, vllm_output, atol=3.2e-2, rtol=1e-3) E           AssertionError: Tensor-likes are not close! E            E  …
