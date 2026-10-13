# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-13 15:01 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要：

### Issue
本次提供的动态中暂无相关 Issue 记录。

### Pull Request (PR)
近期 PR 主要围绕**新特性支持**、**性能优化**及**平台与测试修复**展开：

1. **新特性与模型支持**
   - **模型扩展**：通过 Transformers 后端新增了 `DistilBERT` 序列分类模型支持 (#61349)。
   - **推测解码**：在流水线并行（PP）场景下，支持针对结构化输出请求的逐请求草稿保留，避免微批次覆盖问题 (#61348)。
   - **前端 API**：OpenAI completions/chat 接口现支持在请求中设置 `return_weight_versions: true`，以按选择返回 `weight_versions` 信息 (#61347)。
   - **CPU 支持**：为不具备原生 AMX 内核的 CPU 添加了 MXFP4 MoE 的 BF16 后备计算支持 (#61346)。

2. **性能优化**
   - **内核融合**：将温度缩放融合到拒绝采样内核中，优化推测解码的低延迟推理 (#61344)。
   - **CUDA 算子加速**：优化 `per_token_group_quant_8bit` CUDA 内核，通过计算每组的倒数并相乘来代替逐元素除法，从而提升性能 (#61343)。

3. **Bug 修复与测试**
   - **ROCm 修复**：修复了 MI355 平台上 AITER AR+RMSNorm 后备测试失败的问题 (#61350)；同时调整 DeepSeek-V4 共享专家融合（FP8 重算为 MXFP4），现要求通过在线量化配置显式开启 (#61345)。
   - **Mamba 测试修复**：修复了因未等待接受计数导致的 `test_mamba_prefix_cache_mrv1` 测试失败问题 (#61342)。
   - **测试基建**：引入了测试拆分预览 (#61351)。

### Release
本次提供的动态中暂无新版本发布信息。

---

## 🔀 Pull Requests

### #61351 — [Review/umbp test split preview](https://github.com/vllm-project/vllm/pull/61351)
- **作者**: charxwu  **时间**: 2026-10-13 14:43 CST
- **标签**: documentation, ci/build, kv-connector
- **摘要**: ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this PR.  - [ ] **Design Fit:** Minimizes…

### #61350 — [[ROCm][Bugfix] Run the AITER AR+RMSNorm fallback test on the platform…](https://github.com/vllm-project/vllm/pull/61350)
- **作者**: stefankoncarevic  **时间**: 2026-10-13 14:38 CST
- **标签**: bug, rocm, ready
- **摘要**: `tests/distributed/test_aiter_rms_fallback.py::test_rocm_aiter_gemma_rmsnorm_falls_back_after_allreduce` fails in `:amd: (MI355) Distributed Features`. The test was added yesterday by #58547, together with the fallback it covers.  [The failing job.](https://buildkite.com/vllm/ci/builds/94315/list?ji…

### #61349 — [feat(models): add DistilBERT sequence classification via Transformers…](https://github.com/vllm-project/vllm/pull/61349)
- **作者**: Sanjeev900  **时间**: 2026-10-13 14:29 CST
- **标签**: pooling
- **摘要**: ## Overview  Adds `DistilBertForSequenceClassification` (e.g. `distilbert/distilbert-base-uncased-finetuned-sst-2-english`) to the pooling runner through the Transformers backend.  ## Claims  - `vllm serve distilbert/distilbert-base-uncased-finetuned-sst-2-english --runner pooling` now works and mat…

### #61348 — [[Feature][Spec Decode] Per-request draft retention for structured output under PP](https://github.com/vllm-project/vllm/pull/61348)
- **作者**: michael-gpu  **时间**: 2026-10-13 14:17 CST
- **标签**: speculative-decoding, mrv2
- **摘要**: ## Overview  Under PP, a structured-output request is rescheduled `pp_size` steps after it was sampled and other microbatches overwrite the single latest-batch slot in `DraftTokensHandler` in between. The scheduler's deferred grammar fetch then returns the wrong batch's drafts; the request's own sta…

### #61347 — [[RL][Frontend] Return weight-version spans on OpenAI completions and chat](https://github.com/vllm-project/vllm/pull/61347)
- **作者**: ShuoleiWang  **时间**: 2026-10-13 14:09 CST
- **标签**: documentation, frontend
- **摘要**: ## Overview  Follow-up to #59582 and #61070 (RFC #54443): `/v1/completions` and `/v1/chat/completions` now return per-choice `weight_versions` when the request sets `return_weight_versions: true`, as `/inference/v1/generate` already does.  ## Claims  - Opt-in `return_weight_versions` on both routes.…

### #61346 — [[CPU] Add BF16 fallback for MXFP4 MoE without AMX](https://github.com/vllm-project/vllm/pull/61346)
- **作者**: galleonli  **时间**: 2026-10-13 14:07 CST
- **标签**: documentation, cpu, nvidia, quantization
- **摘要**: ## Overview  Add a BF16 fallback for MXFP4 MoE checkpoints on CPUs without a native AMX kernel. Addresses #61027 while retaining the AMX capability check and preferring the existing native backend when available.  ## Claims  - Automatic selection and `--moe-backend cpu` can select portable CPU exper…

### #61345 — [[ROCm][DSv4] Require explicit MXFP4 opt-in for MegaMoE shared-expert fusion](https://github.com/vllm-project/vllm/pull/61345)
- **作者**: AMD-yanfeiwang  **时间**: 2026-10-13 14:03 CST
- **标签**: rocm, deepseek, DSv4
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Follow-up to #61004. With this PR, fusing DeepSeek-V4's shared expert into AITER MegaMoE (which requantizes it from FP8 to MXFP4) requires an explicit opt-in through the online quantization API instead of a dedicated env var.  ## Claims  - Removes `VLLM_RO…

### #61344 — [[Kernel][Spec Decode] Fuse temperature scaling into the rejection-sampling kernels](https://github.com/vllm-project/vllm/pull/61344)
- **作者**: AlejandroParedesLT  **时间**: 2026-10-13 13:48 CST
- **标签**: speculative-decoding, mrv2
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Targets #61024 (Q4 2026 roadmap, SIG Spec Decode: "optimize fusion and kernels for low-latency inference, including rejection sampling"). Folds temperature scaling into the rejection-sampling kernels so `target_logits` can be passed raw instead of pre-divi…

### #61343 — [[Perf] per_token_group_quant_8bit: multiply by reciprocal instead of dividing per element](https://github.com/vllm-project/vllm/pull/61343)
- **作者**: gau-nernst  **时间**: 2026-10-13 13:41 CST
- **标签**: quantization
- **摘要**: <!-- markdownlint-disable -->  ## Overview  Speed up the `per_token_group_quant_8bit` CUDA kernel (both the smem and register-resident schedules) by computing the per-group scale reciprocal once and multiplying, instead of dividing per element. ~1.2-1.3x kernel speedup on GB300 across M=1..32768; ~1…

### #61342 — [[Tests][Mamba] Wait for accepted counts in fake MTP proposer](https://github.com/vllm-project/vllm/pull/61342)
- **作者**: ZJY0516  **时间**: 2026-10-13 13:39 CST
- **摘要**: ## Overview  Fix `test_mamba_prefix_cache_mrv1` / `_mrv1_async`, failing on `main` since #61134 on H200 MIG and MI250 (e.g. [main build #94362](https://buildkite.com/vllm/ci/builds/94362#01a1333b-d247-49be-b767-84df48ec72c7)): `ValueError: Mamba state is not equal for key: 576 at index 0`. Test-only…
