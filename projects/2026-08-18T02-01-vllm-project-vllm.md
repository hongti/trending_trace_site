# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-18 10:01 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
1. **推理模型死循环问题** (#52673)：请求增加新特性，检测并打断推理模型（如 Qwen3.8-27B 结合推测解码时）在推理阶段陷入的**精确重复 token 循环**（ThinkingLoopBreaker）。
2. **PaliGemma 缩放 Bug** (#52667)：由于此前 PR 将 Gemma 的 `sqrt(hidden_size)` 归一化移至文本嵌入层，导致 PaliGemma 的**图像嵌入仍保持逆向缩放**，引发异常。

### 🔀 PR 动态
**新特性与功能增强：**
- **可插拔 KV Cache 数据类型** (#52670)：引入新特性，支持灵活指定 KV Cache 的数据类型，提升量化灵活性。
- **认证方式扩展** (#52666)：API 认证中间件增加 `x-api-key` 请求头作为 `Authorization: Bearer` 的降级/替代方案，兼容更多客户端。

**性能优化 (主要针对 ROCm)：**
- **MiniMax/HunYuan 路由器优化** (#52668)：为 fp32 门控权重添加了 bf16x3 router GEMM，优化 MiniMax-M3/M2 及 HunYuan-V3 在 ROCm 上的表现。
- **MiniMax-M3 稀疏注意力优化** (#52664)：将 aiter indexer scoring 和 top-k 内核集成到 MiniMax-M3 的稀疏注意力路径中。

**Bug 修复：**
- **Dynamic NTK 缩放逻辑修复** (#52675)：当服务长度低于模型训练长度时，跳过 Dynamic NTK 缩放公式，避免不当的上下文扩展。
- **KV Offload 修复** (#52674, #52669)：针对开启 KV 卸载和推测解码时非滑动窗口组的情况，扩展了 eagle 查询范围，修复内部一致性问题。
- **lm-format-enforcer 兼容性修复** (#52661)：恢复被误删的 `vllm.transformers_utils.tokenizer` 桥接代码，修复与 `lm-format-enforcer==0.11.3` 的兼容性问题。

**基础设施与前端：**
- **Rust 前端** (#52671)：多引擎工具调用由“遇错即返”改为“等待所有引擎执行完毕后再返回错误”。
- **XPU CI** (#52672)：升级 pytest 版本以修复 Intel CI 构建失败问题。

### 🚀 Release 动态
- 本期监控时间范围内**无新版本发布**。

---

## 🐛 Issues

### #52673 — [[Feature]: Detect and break exact repeating loops inside reasoning sections (spec-decode-aware ThinkingLoopBreaker)](https://github.com/vllm-project/vllm/issues/52673)
- **作者**: amittell  **时间**: 2026-08-18 09:54 CST
- **摘要**: ### 🚀 The feature, motivation and pitch  Reasoning models (observed in production with Qwen3.8-27B at high reasoning effort, MTP spec-3) occasionally fall into an **exact repeating token cycle inside the `<think>` section** and burn tokens until `max_tokens`. `thinking_token_budget` (merged in #3466…

### #52667 — [[Bug][PaliGemma] Image embeddings remain inverse-scaled after Gemma input scaling change](https://github.com/vllm-project/vllm/issues/52667)
- **作者**: wagnerpatriota  **时间**: 2026-08-18 08:04 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (x86_64) GCC version                  : (Ubuntu 11.…

## 🔀 Pull Requests

### #52675 — [[Bugfix] Skip Dynamic NTK scaling when served length is below trained length](https://github.com/vllm-project/vllm/pull/52675)
- **作者**: AnkitNakhawa  **时间**: 2026-08-18 09:55 CST
- **标签**: bug
- **摘要**: ## Purpose  Fixes #52665.  Dynamic NTK scaling in `dynamic_ntk_scaling_rope.py` unconditionally applies its context-extension formula even when the served `max_position_embeddings` is *below* the checkpoint's `max_trained_positions`. Reducing the served length below the trained length is ordinary tr…

### #52674 — [[Bugfix][KV Offload] Widen the eagle lookup query for non-sliding-window groups](https://github.com/vllm-project/vllm/pull/52674)
- **作者**: yifjiang  **时间**: 2026-08-18 09:55 CST
- **标签**: bug, kv-connector
- **摘要**: ## ⚠️ Draft: the measurement is solid, the mechanism is not fully pinned. Filing for maintainer input.  With `--kv-offloading-size` and `--speculative-config` both enabled, the host tier **writes and never reads back**: `GPU_to_CPU` climbs, `CPU_to_GPU` stays at exactly zero. We hit this in producti…

### #52672 — [[XPU] upgrade pytest](https://github.com/vllm-project/vllm/pull/52672)
- **作者**: mayuyuace  **时间**: 2026-08-18 09:52 CST
- **标签**: intel-gpu, ci/build
- **摘要**: **BUG**:  https://buildkite.com/vllm/intel-ci/builds/9136/canvas?sid=01a01157-7bc4-4dd2-9cbd-d6f16677b983&tab=output tests/engine/test_arg_utils.py::test_prefix_cache_retention_interval_from_deprecated_env FAILED on XPU CI, PASSED on CUDA CI.  **Root cause**: The pytest version specified in `require…

### #52671 — [[Rust Frontend] Wait for all utility calls to finish](https://github.com/vllm-project/vllm/pull/52671)
- **作者**: connorcarpenter15  **时间**: 2026-08-18 09:16 CST
- **标签**: rust
- **摘要**: ## Purpose  Make multi-engine utility fanout wait for every engine outcome before returning an error.  - Replace fail-fast send and response joins with collection that waits for all engines while preserving result order. - Let callers compensate partially applied mutations only after every engine ca…

### #52670 — [[Quantizaiton][Feature]Pluggable KV Cache Data Types](https://github.com/vllm-project/vllm/pull/52670)
- **作者**: lcfenglinwan  **时间**: 2026-08-18 09:06 CST
- **标签**: quantization
- **摘要**: ## Purpose This PR is the implementation of https://github.com/vllm-project/vllm/issues/51751  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this…

### #52669 — [[Bugfix][KV Offload] Widen eagle lookup query for non-sliding-window groups](https://github.com/vllm-project/vllm/pull/52669)
- **作者**: yifjiang  **时间**: 2026-08-18 09:00 CST
- **标签**: bug, kv-connector
- **摘要**: > **Draft — scope corrected after further investigation.** This is offered as an *internal > consistency* fix, not as the cause of any particular symptom. An earlier revision of this > description overstated the production evidence; corrections are at the bottom.  ## The asymmetry  In `OffloadingCon…

### #52668 — [[ROCm][Perf] Add a bf16x3 router GEMM for fp32 gate weights](https://github.com/vllm-project/vllm/pull/52668)
- **作者**: akii96  **时间**: 2026-08-18 08:49 CST
- **标签**: rocm
- **摘要**: ## Motivation  Models like MiniMax-M3, MiniMax-M2 and HunYuan-V3 store their router gate weights in fp32. On ROCm, none of the specialized GateLinear tiers can handle an fp32 weight, so the gate falls back to `F.linear`. That fallback casts the bf16 activations to fp32 in a separate kernel, then run…

### #52666 — [Accept x-api-key header as auth fallback](https://github.com/vllm-project/vllm/pull/52666)
- **作者**: perfectmakuwerere  **时间**: 2026-08-18 08:00 CST
- **标签**: frontend
- **摘要**: ## Purpose  This PR extends `AuthenticationMiddleware.verify_token` to accept an `x-api-key` header as an alternative to the `Authorization: Bearer <token>` scheme. Some clients and tooling (e.g. certain SDKs and API gateways) send credentials via `x-api-key` rather than a Bearer token, and previous…

### #52664 — [[Performance][ROCm]  Integrate aiter indexer scoring and top-k kernels into MiniMax-M3 sparse attention path](https://github.com/vllm-project/vllm/pull/52664)
- **作者**: ykamiset  **时间**: 2026-08-18 07:34 CST
- **标签**: rocm
- **摘要**: Integrate aiter indexer scoring and top-k kernels (AITER PR: https://github.com/ROCm/aiter/pull/4787) into MiniMax-M3 sparse attention path. WIP.

### #52661 — [[Bugfix] Restore vllm.transformers_utils.tokenizer shim for lm-format-enforcer](https://github.com/vllm-project/vllm/pull/52661)
- **作者**: anguszzzz  **时间**: 2026-08-18 06:14 CST
- **标签**: bug
- **摘要**: ## Purpose  Fix #52614.  PR #35024 removed `vllm/transformers_utils/tokenizer.py` after lm-eval dropped its usage. However, `lm-format-enforcer==0.11.3` — pinned in `requirements/common.txt` — still does `from vllm.transformers_utils.tokenizer import MistralTokenizer` at the top of `lmformatenforcer…
