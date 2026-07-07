# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-07 09:12 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要：

### 📋 Issue (问题与讨论)

1. **架构重构提议 (RFC #47790)**：提议将 Flashinfer GQA 拆分为 TRTLLM 和 flashinfer 两个独立后端。原因是目前 `flashinfer.py` 文件过于臃肿，支持了过多特性各异的内核，维护难度大。
2. **多模态服务严重 Bug (#47788)**：Qwen3-Omni 在在线服务中使用 `--mm-processor-kwargs` 配置 `use_audio_in_video` 时崩溃；存在音频数据静默丢失、交错占位符及 mRoPE 报错等问题（环境：vLLM 0.20.2 + 2×H100）。
3. **MLA 解码输出乱码 Bug (#47783)**：DeepSeek-V4 (DSV4) 使用 sparse MLA decode 后端时，因受 PR #44577 引入的 packed KV layout 影响，输出完全乱码（gsm8k 准确率从 ~95% 暴跌至 ~0%）。

---

### 🔧 Pull Request (代码变更与修复)

**🚀 重要重构与新特性**
- **Flashinfer 架构重构 (#47786)**：作为上述 RFC (#47790) 的第一步，重构了 Flashinfer 的 `BatchPrefill*` / `BatchDecode*` 及 `trtllm_batch*` wrappers 的 forward 路径。
- **Rust 前端延迟统计修正 (#47787)**：将 `arrival_time` 的打点时机从渲染和分词之后提前到前端入口处，使 Rust 前端的 TTFT 和 e2e 延迟指标与 Python 前端对齐，修复了之前统计偏短的问题。
- **MoE Expert 路由对齐 (#47785)**：在 align sum kernel 中增加了对 `topk_ids` padding 的处理（支持 `-1` 值），以兼容 DeepEP 的弹性调度逻辑（表示 token 路由到非本地专家）。
- **Marconi 缓存优化 (#47782)**：改进了 Marconi（Mamba 状态缓存）机制，在选择性的混合缓存保留策略下，避免了因 KV cache 逐出而意外丢失共享前缀的 Mamba 状态。
- **SM120 MLA DCP 支持 (#47779)**：为 SM120 架构的 FlashInfer sparse MLA decode 后端启用了 DCP（Decode Context Parallelism，解码上下文并行）。

**🐛 关键 Bug 修复**
- **5D KV Cache 索引修复 (#47791)**：修复了在接收 KV cache 进行 HND 到 blocks-first layout 转换时，5D KV cache 触发 `IndexError` 的问题。
- **DSV2 量化加载修复 (#47780)**：修复了 Compressed Tensors (CT) 量化下 DeepSeek-V2 模型的加载问题，减少了过去针对 fp8 和特定量化配置的大量硬编码。
- **源码运行环境报错修复 (#47789)**：修复了在未安装为包的源码树下运行 vLLM 时，因找不到 `vllm` metadata 导致 Triton 报错 `PackageNotFoundError` 的问题。
- **The Rock GPU 路径修复 (#47781)**：修复了 MI250 等 GPU 名称包含特殊字符（如 `/`）时，导致文件路径错误进而引发 kernel 测试失败的问题。

**📝 其他**
- **测试规范更新 (#47784)**：在 AGENTS MD 文档中增加了关于如何为新 PR 编写和纳入测试的建议。

---

### 🚀 Release (版本发布)

*近期无新增版本发布记录。*

---

## 🐛 Issues

### #47790 — [[RFC]: Separate Flashinfer GQA into TRTLLM and flashinfer backends.](https://github.com/vllm-project/vllm/issues/47790)
- **作者**: pavanimajety  **时间**: 2026-07-07 08:58 CST
- **标签**: RFC
- **摘要**: ### Motivation.  vllm/v1/attention/backends/flashinfer.py has grown and supports too many kernels at this point. Each has diverse set of features, [standard attention support matrix](https://github.com/vllm-project/vllm/blob/main/docs/design/attention_backends.md#standard-attention-mha-mqa-gqa-backe…

### #47788 — [[Bug]: Qwen3-Omni use_audio_in_video via --mm-processor-kwargs is broken in online serving: engine crash at startup, audio silently never fed, interleave placeholder/mRoPE errors](https://github.com/vllm-project/vllm/issues/47788)
- **作者**: LukeLIN-web  **时间**: 2026-07-07 08:52 CST
- **摘要**: ### Your current environment  - vLLM 0.20.2 (pip), verified the relevant code is unchanged on current `main` - transformers 5.13.0, torch 2.11.0+cu130 - 2× H100 NVL, TP=2 (also reproduces on a single GPU) - Model: `Qwen/Qwen3-Omni-30B-A3B-Instruct`  ### 🐛 Describe the bug  Enabling audio-in-video fo…

### #47783 — [[Bug]: DSV4 sparse MLA emits garbage (gsm8k ~0%) with the packed KV layout from #44577](https://github.com/vllm-project/vllm/issues/47783)
- **作者**: majunze2001  **时间**: 2026-07-07 06:29 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>   - vLLM `0.23.1rc1.dev713+gd63c8e944` (`main` @ `d63c8e944`); regression bisected to `01192139b` (#44577), good parent `b9a7cd464` - PyTorch `2.11.0+cu130`, FlashInfer `0.6.13`, CUDA 13 (nvcc…

## 🔀 Pull Requests

### #47791 — [[Bugfix] Fix handling 5D KV cache in kv_postprocess_layout_on_receive](https://github.com/vllm-project/vllm/pull/47791)
- **作者**: dsocek  **时间**: 2026-07-07 09:00 CST
- **标签**: bug, kv-connector
- **摘要**: ## PR Purpose  Fixes `IndexError` in `kv_postprocess_layout_on_receive()` for 5D blocks-first KV caches.  `kv_postprocess_layout_on_receive()` (added in #30275) permutes a received KV cache from HND to NHD layout. It assumes the cache is 4D. But since #42095 is recently merged, non-MLA backends allo…

### #47789 — [fix: tolerate source checkout without vllm metadata](https://github.com/vllm-project/vllm/pull/47789)
- **作者**: ShuhaoZhangTony  **时间**: 2026-07-07 08:54 CST
- **摘要**: ### Summary  Handle source-tree execution where `importlib.metadata.version("vllm")` raises `PackageNotFoundError` because vLLM has not been installed as a package yet.  Without this guard, Triton probing can fail before runtime initialization reaches the backend checks. In a source checkout, the ab…

### #47787 — [[Rust Frontend] Stamp `arrival_time` at the frontend entry](https://github.com/vllm-project/vllm/pull/47787)
- **作者**: tahsintunan  **时间**: 2026-07-07 08:48 CST
- **标签**: rust
- **摘要**: ### Summary  Rust stamps `arrival_time` after render and tokenize. Python stamps it at the renderer entry, before both. So Rust's TTFT and e2e histograms run short. This stamps it at the frontend entry to match. No engine or protocol change.  _Refs: roadmap #44280. Part of RFC #44757._

### #47786 — [[Core] Refactor Flashinfer BatchPrefill* / BatchDecode* and trtllm_batch* wrappers' forward paths. ](https://github.com/vllm-project/vllm/pull/47786)
- **作者**: pavanimajety  **时间**: 2026-07-07 08:43 CST
- **标签**: v1, nvidia
- **摘要**: ## Purpose Step 1 of RFC - https://github.com/vllm-project/vllm/issues/47790   ## Test Plan    ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resol…

### #47785 — [handle topk_ids padding in align sum kernel](https://github.com/vllm-project/vllm/pull/47785)
- **作者**: gnovack  **时间**: 2026-07-07 07:20 CST
- **摘要**: ## Purpose Some all-to-all backends use `-1` in the `topk_ids` to indicate that a token is routed to a non-local expert (e.g. see [DeepEP elastic dispatch logic](https://github.com/deepseek-ai/DeepEP/blob/main/deep_ep/include/deep_ep/impls/dispatch_copy_epilogue.cuh#L106-L109)), but currently the `m…

### #47784 — [AGENTS MD: Add suggestion on how to incorporate tests](https://github.com/vllm-project/vllm/pull/47784)
- **作者**: simon-mo  **时间**: 2026-07-07 06:56 CST
- **标签**: documentation
- **摘要**: I have seeing more PRs with newly created test files by agents.

### #47782 — [[Core] Preserve Marconi caching with selective hybrid cache retention](https://github.com/vllm-project/vllm/pull/47782)
- **作者**: njhill  **时间**: 2026-07-07 06:25 CST
- **标签**: v1
- **摘要**: The [Marconi](https://arxiv.org/abs/2411.19379) paper was implemented in https://github.com/vllm-project/vllm/pull/37898, which enables caching of mamba state for shared prefixes in "align" mode.  This PR extends the shared prefix boundary caching to the retention-interval sparse caching added in ht…

### #47781 — [[CI/Build][BugFix][The Rock] Fix get_ssm_device_name to return sanitized, usable filename](https://github.com/vllm-project/vllm/pull/47781)
- **作者**: rasmith  **时间**: 2026-07-07 06:21 CST
- **标签**: bug
- **摘要**: ## Purpose On The Rock, the GPU name for MI250 will be `AMD Instinct MI250X / MI250`.  The old output was: `AMD_Instinct_MI250X_/_MI250` which causes pathing errors during  the run of: `pytest -sv  kernels/mamba/test_mamba_ssm_configs.py` The new output is: `AMD_Instinct_MI250X_MI250` which does not…

### #47780 — [[Bugfix] [Quantization] Fix loading for CT DSV2](https://github.com/vllm-project/vllm/pull/47780)
- **作者**: kylesayrs  **时间**: 2026-07-07 06:17 CST
- **标签**: bug, ready, deepseek
- **摘要**: ## Purpose ## * Fix loading for DSV2 model + compressed tensors   * The deepseek architectures have always included a lot of hard-coding around fp8 and particular quantization config layouts. This PR does not do things like (support non-fp8 quantization of attention layers, support wkw quantization,…

### #47779 — [[Bugfix][SM120][MLA] Enable DCP for FlashInfer sparse MLA decode](https://github.com/vllm-project/vllm/pull/47779)
- **作者**: sebastiaanvduijn  **时间**: 2026-07-07 06:04 CST
- **标签**: bug, v1, nvidia
- **摘要**: ## Purpose  Enable decode context parallelism for the SM120 FlashInfer sparse MLA decode backend (`FLASHINFER_MLA_SPARSE_SM120`).  The generic FlashInfer sparse MLA backend already participates in DCP by filtering sparse top-k indices to the local DCP rank, passing valid per-row counts as `seq_lens`…
