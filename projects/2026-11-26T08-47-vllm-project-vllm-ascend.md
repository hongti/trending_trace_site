# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-26 16:47 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🐛 Issue (问题)
*主要围绕 DeepSeek-V4.1-Flash 模型在 Ascend 硬件上的运行缺陷：*
* **解码端伪存活卡死 / 超时问题 (#23391)**：在 PD 分离式部署 DeepSeek-V4.1-Flash 时，解码侧出现全 DP rank 伪存活卡死现象，导致 `shm_broadcast` 60秒超时触发 3 次，最终引发 Vector Core 超时（`acl_graph synchronize` 失败）。
* **910B 启动失败问题 (#23382)**：DeepSeek-V4.1-Flash 在 8卡 Ascend 910B 环境下因缺少 QSLI metadata symbols 无法正常启动。

### 🔧 Pull Request (拉取请求)
*涵盖新特性、性能优化及重要修复：*
* **新特性**：
  * **SFA 预处理控制**：新增启动选项以支持原生 SFA 预处理（Native SFA preprocessing）及强制覆盖（#23388, #23390），解决在禁用 MLAPO 时自动选择 MLA Prolog V3 导致难以进行融合与原生预处理对比的问题。
  * **GLM-5.3-Flash 支持**：新增对带 DCP 的 GLM-5.3-Flash SFA 模型的支持（#23384）。
* **性能优化**：
  * **DeepSeek V4.1 DSpark 性能提升**：在超连接扩展之前对 embedding 进行切分，避免每个 TP rank 物化不需要的 HC 行，从而减少冗余内存开销（#23389）。
* **Bug 修复**：
  * **PD 分离连接负载不均**：修复了 disaggregated-prefill 代理服务器在新连接突发时集中于少数 worker 的问题，改用独立的 Linux 监听器来平衡 worker 连接（#23386）。
  * **LoRA 适配器与 MLA 兼容性**：修复了 Ascend 的 MLA/SFA 后端因缓存投影引用并吸收 KV-B 权重导致 LoRA 适配器无法正常应用的问题（#23381）；同时修复了 LoRA 投影产生的非连续视图或窄输出导致的原生内核收缩/扩展报错（#23380）。
* **CI 临时验证**：提交了多个临时分支用于 A2+A3 环境下的离线 karc 并行验证测试（#23383, #23385, #23387）。

### 🚀 Release (发布)
*近期无新版本发布信息。*

---

## 🐛 Issues

### #23391 — [[Bug]: Decode-side all-DP-rank pseudo-liveness stall → shm_broadcast 60s ×3 → vector core timeout (acl_graph synchronize) in PD-disaggregated DeepSeek-V4.1-Flash serving](https://github.com/vllm-project/vllm-ascend/issues/23391)
- **作者**: gsycool-haha  **时间**: 2026-11-26 16:42 CST
- **标签**: bug, llm-model, deepseek
- **摘要**: ### Your current environment      Component: vLLM     Value: 0.30.0     ────────────────────────────────────────     Component: vllm-ascend     Value: 0.30.0rc2 (editable install, /vllm-workspace/vllm-ascend)     ────────────────────────────────────────     Component: torch_npu     Value: 2.10.0.pos…

### #23382 — [[Bug][A2] DeepSeek-V4.1-Flash fails to start on 910B due to missing QSLI metadata symbols](https://github.com/vllm-project/vllm-ascend/issues/23382)
- **作者**: Dunnn-Smith  **时间**: 2026-11-26 16:05 CST
- **标签**: bug
- **摘要**: ### Your current environment  ### Environment  - Hardware: 8 × Ascend 910B - Ascend commit: `07fb73958c3f85e246bc825292adf6363b2f5360` - Model: `DeepSeek-V4.1-Flash-1017-w4a8`   ### 🐛 Describe the bug  ### Error  The worker fails during startup in `vllm_ascend/models/deepseek_v41/indexer.py`, line 2…

## 🔀 Pull Requests

### #23390 — [[Feature][Attention] Add startup native SFA preprocessing override](https://github.com/vllm-project/vllm-ascend/pull/23390)
- **作者**: danziheng1024  **时间**: 2026-11-26 16:35 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  SFA can select PROLOG_V3 even when MLAPO is disabled, so existing configuration cannot force native preprocessing for controlled comparisons. This adds a typed `sfa_preprocess_mode` option and selects the native path before fused weight preparation can transp…

### #23389 — [[Performance][DSpark] Shard embeddings before hyper-connection expansion in DeepSeek V4.1](https://github.com/vllm-project/vllm-ascend/pull/23389)
- **作者**: zzy-6666  **时间**: 2026-11-26 16:27 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  DeepSeek V4.1 DSpark currently expands the full embedding `[T,H]` into `[T,M,H]` before sequence-parallel sharding. Each TP rank materializes HC rows that it immediately discards. Shard the 2-D embeddings first, then repeat only the local token rows, matching…

### #23388 — [[Feature][Attention] Add a startup option for native SFA preprocessing](https://github.com/vllm-project/vllm-ascend/pull/23388)
- **作者**: 31-ryougishiki  **时间**: 2026-11-26 16:25 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  On an eligible SFA layer, automatic preprocessing can select MLA Prolog V3 even when MLAPO is disabled. This makes it difficult to compare fused and native preprocessing on the same serving configuration. The new startup option selects the native path before …

### #23387 — [ci(temp): full offline a2+a3 karc validation, parallel r5](https://github.com/vllm-project/vllm-ascend/pull/23387)
- **作者**: Longwt123  **时间**: 2026-11-26 16:23 CST
- **标签**: ci/build
- **摘要**: Temporary CI validation branch, based on vllm-project/main (f35a583dc) with the same changes as #23357.  Purpose: run the offline a2+a3 karc validation **in parallel** with the in-progress run of #23357 (separate refs = separate concurrency groups, no mutual cancellation).  Do not merge. - vLLM main…

### #23386 — [[BugFix][PD] Balance proxy worker connections with independent Linux listeners](https://github.com/vllm-project/vllm-ascend/pull/23386)
- **作者**: nwpu-zxr  **时间**: 2026-11-26 16:22 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  The disaggregated-prefill example proxy can concentrate a burst of new connections on a few workers even when many Uvicorn workers are running. Its existing startup passes one parent-created listening socket to all workers.  On Linux, `--workers > 1` now give…

### #23385 — [ci(temp): full offline a2+a3 karc validation, parallel r4](https://github.com/vllm-project/vllm-ascend/pull/23385)
- **作者**: Longwt123  **时间**: 2026-11-26 16:19 CST
- **标签**: ci/build, merge-conflicts
- **摘要**: Temporary CI validation branch, rebased onto latest main with the same changes as #23357.  Purpose: run the offline a2+a3 karc validation **in parallel** with the in-progress run of #23357 (separate refs = separate concurrency groups, no mutual cancellation).  Do not merge. - vLLM main: https://gith…

### #23384 — [[Feature][Model] Support GLM-5.3-Flash SFA with DCP](https://github.com/vllm-project/vllm-ascend/pull/23384)
- **作者**: Zyang123123  **时间**: 2026-11-26 16:19 CST
- **标签**: module:tests, module:core, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  Continues #20188 from its original commit `869cd6821b56da5e0597e2ffb26ce102efaf9994` by @Zhaocen. This draft retains the exact commit and author attribution.  The change enables GLM-5.3-Flash SFA to use DCP, including PCP+DCP. It shards the main latent KV cac…

### #23383 — [ci(temp): full offline a2+a3 validation on karc direct-sync runners (parallel r3)](https://github.com/vllm-project/vllm-ascend/pull/23383)
- **作者**: Longwt123  **时间**: 2026-11-26 16:12 CST
- **标签**: merge-conflicts
- **摘要**: Temporary CI validation branch. Same content as #23357 (commit b731c91b) plus one empty commit to obtain a unique head SHA.  Purpose: run the offline a2+a3 karc validation **in parallel** with the in-progress run of #23357. The workflow concurrency group is per-ref with `cancel-in-progress: true`, s…

### #23381 — [[BugFix][LoRA] Apply adapters to absorbed MLA projections](https://github.com/vllm-project/vllm-ascend/pull/23381)
- **作者**: Annarine  **时间**: 2026-11-26 16:01 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Ascend's MLA/SFA backends cache projection references and absorb KV-B weights into query/value batch matrix multiplications. Replacing a model projection with a LoRA wrapper does not update a stale backend reference, and the absorbed computations otherwise om…

### #23380 — [[BugFix][LoRA] Preserve fused projection layouts and adapter slots](https://github.com/vllm-project/vllm-ascend/pull/23380)
- **作者**: Annarine  **时间**: 2026-11-26 15:58 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  LoRA projections can return non-contiguous views or TP-local outputs narrower than the adapter rank. Passing those layouts to the native shrink/expand kernels can produce incorrect increments. Variable fused projections also need to retain replicated B shards…
