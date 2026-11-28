# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-28 16:20 CST

## AI 总结

## vllm-ascend 仓库近期动态摘要

本次更新仅涉及 PR 活动，未包含 Issue 与 Release。

---

### Issue 动态
本期暂无相关 Issue。

---

### Release 动态
本期无新版本发布。

---

### PR 动态

#### 🔧 KV Cache / MLA 重构
- **#23665**（recky-c）：重构混合模型 cache planners，使其自包含且纯净，为后续从 `get_kv_cache_spec` 移除跨层对齐做铺垫（KV Cache upstream-alignment 系列 PR-B）。
- **#23658**（recky-c）：将 HiSparse MLA source spec 的定制逻辑从 `attn_utils.py` 下沉到 `AscendMLAAttention` 及对应 binding，提升模块归属清晰度。

#### 🧩 MoE 相关修复与重构
- **#23664**（lenghuixing0330）：修复 DSpark 场景下 MegaMoE EP 验证问题，适配 draft attention query view 为 PCP=1 的重构逻辑。
- **#23661**（Liamup777）：改用上游 `restore_routed_experts_rows` hook 捕获 routed experts，替代此前已关闭的 #20765，与 vllm 主仓对齐。

#### ⚡ 性能优化
- **#23659**（Dwight2458）：针对 Mamba，当 fine-grained prefix match unit 命中同一物理块时，避免不必要的两步 prefill split，降低开销。
- **#23657**（johnnysluckydays）：优化 MRV2 DSpark 概率采样路径，针对 reduced vocabulary 不再每个 Markov 步都展开到目标词表，显著降低开销。

#### 🐛 BugFix
- **#23656**（djh95）：修复 `logits_finite` 在缺少 `logits_indices` 时错误地用 `req_ids[row]` 推断请求的缺陷，避免 spec / 多 token 输出等场景误报。
- **#23660**（tomleeai）：为 Qwen3.8-Flash-Next 在非对称 PD MTP（如 P=1, D=3）下保留 recurrent-state 布局一致性，修复卷积状态行数不匹配问题。

#### 🔄 CI / 测试
- **#23663**（tangdafu）：将 A2/A3/A5 的 Ubuntu 与 openEuler 镜像中 `flash-attn-npu` 升级至 0.6.0。
- **#23662**（1u363）：为 A5 主配置上的 Kimi K3.1 Pro（W4A8, E256）注册 nightly GSM8K 精度测试用例。

---

**总体趋势**：本批 PR 集中在 vLLM Ascend 后端与主仓特性对齐，重点在 KV Cache/MLA 重构、MoE 路由 hook 上游化、Mamba 与 DSpark 性能优化，并补齐了 Kimi K3.1 Pro 的 nightly 精度回归测试。

---

## 🔀 Pull Requests

### #23665 — [[Refactor][KV Cache] Make hybrid model cache planners self-sufficient and pure](https://github.com/vllm-project/vllm-ascend/pull/23665)
- **作者**: recky-c  **时间**: 2026-11-28 16:19 CST
- **摘要**: ### What this PR does / why we need it? Part of the KV Cache Spec upstream-alignment series (**PR-B**; prepares model planners before removing cross-layer alignment from `get_kv_cache_spec` in **PR-D**).  Makes the model-specific KV cache planners self-sufficient and pure (idempotent and non-mutatin…

### #23664 — [[BugFix] Fix MegaMoE EP validation for DSpark](https://github.com/vllm-project/vllm-ascend/pull/23664)
- **作者**: lenghuixing0330  **时间**: 2026-11-28 16:12 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  DSpark reconstructs the draft attention query view as PCP=1 while retaining the target model's physical EP group for routed experts. MegaMoE validation currently derives EP from the draft parallel config, which can incorrectly compute EP=1 and disable MegaMoE…

### #23663 — [[CI] Bump flash-attn-npu to 0.6.0](https://github.com/vllm-project/vllm-ascend/pull/23663)
- **作者**: tangdafu  **时间**: 2026-11-28 16:12 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Bump the flash-attn-npu pin to 0.6.0 in the A2, A3, and A5 Ubuntu and openEuler images.  ### Does this PR introduce _any_ user-facing change?  Yes. The affected images will ship with flash-attn-npu 0.6.0.  ### How was this patch tested?  Verified all six pins…

### #23662 — [[CI] Register A5 Kimi K3.1 Pro 1P1D GSM8K nightly case](https://github.com/vllm-project/vllm-ascend/pull/23662)
- **作者**: 1u363  **时间**: 2026-11-28 16:11 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Add nightly GSM8K accuracy coverage for Kimi K3.1 Pro W4A8 E256 on the A5 main configuration.  - **SIT case** — `tests/SIT/cases/models/configs/Kimi/Kimi-K3.1-Pro-W4A8-E256-A5-1P1D-main.yaml` assigns node 0 to prefill and node 1 to decode. Each uses eight NPU…

### #23661 — [[Refactor][BugFix] Use upstream hook for routed-experts capture](https://github.com/vllm-project/vllm-ascend/pull/23661)
- **作者**: Liamup777  **时间**: 2026-11-28 16:10 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Supersedes closed #20765 now that vllm-project/vllm#64689 has merged. `AscendMoERunner` provides the upstream `restore_routed_experts_rows(topk_ids, ctx)` hook. Upstream continues to own `capture()`, buffer writes, microbatch offsets, snapshots and layer boun…

### #23660 — [[Feature][Qwen4Exp] Preserve recurrent-state layout for asymmetric PD MTP](https://github.com/vllm-project/vllm-ascend/pull/23660)
- **作者**: tomleeai  **时间**: 2026-11-28 16:10 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Qwen3.8-Flash-Next uses the local MTP depth in the GDN and PLE convolution-state shapes. With P=1 and D=3, the convolution state on prefill has two fewer rows than decode expects. The PLE runtime also selects a trailing state view, so the view must use the ph…

### #23659 — [[Performance][Mamba] Avoid unnecessary two-step prefill split on same-block fine-PMU hits](https://github.com/vllm-project/vllm-ascend/pull/23659)
- **作者**: Dwight2458  **时间**: 2026-11-28 16:01 CST
- **标签**: module:tests
- **摘要**: ## What this PR does  Eliminates unnecessary two-step `tail_boundary` prefill splits when fine-grained prefix match unit (`--prefix-match-unit`, e.g. `128`) is smaller than the physical Mamba block size (`--block-size`, e.g. `2048`) with `gdn_internal_prefill_checkpoints=true`.  ### Root Cause 1. In…

### #23658 — [[Refactor][KV Cache] Move HiSparse MLA source spec customization to owner](https://github.com/vllm-project/vllm-ascend/pull/23658)
- **作者**: recky-c  **时间**: 2026-11-28 15:55 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? Move HiSparse MLA source spec customization out of `worker/v2/attn_utils.py::get_kv_cache_spec` into `AscendMLAAttention` and `vllm_ascend.hisparse.binding`, preserving byte-for-byte identical physical KV cache geometry and `block_stride_alignment`.  - **Owner…

### #23657 — [[Performance] Reduce MRV2 DSpark probabilistic sampling overhead for reduced vocabularies](https://github.com/vllm-project/vllm-ascend/pull/23657)
- **作者**: johnnysluckydays  **时间**: 2026-11-28 15:49 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  MRV2 already supports probabilistic DSpark drafting, but its reduced-vocabulary path still expands logits to the target vocabulary before **each** Markov step. The Ascend categorical prepare kernel then scans and caches those mostly masked columns. This remai…

### #23656 — [[BugFix][Observability] logits_finite: refuse row→req guess without mapping](https://github.com/vllm-project/vllm-ascend/pull/23656)
- **作者**: djh95  **时间**: 2026-11-28 15:44 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? `logits_finite` attributed non-finite logits by indexing `req_ids[row]` whenever `logits_indices` was missing — including when `nrows != num_reqs`.  Spec / multi-token layouts such as three logits rows belonging to `[A, A, B]` then mis-fired: a NaN on the seco…
