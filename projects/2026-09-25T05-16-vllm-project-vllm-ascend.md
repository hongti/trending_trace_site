# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-25 13:16 CST

## AI 总结

# vllm-ascend 仓库动态摘要（截至 2026-09-25）

## Issue 动态（2 项）

1. **#17520** — DFlash 推测解码验证阶段输出错误
   - 在 Ascend 上跑 DFlash speculative decode 时，verify 步骤把多个 query token（如每序列 16 个）折叠进单次 `npu_fused_infer_attention_score`（FIA），不同 query 组误共享 causal KV 长度，导致输出错误。
2. **#17518** — `npu_gather_pa_kv_cache` 挂死（EZ9999 / 507035）
   - Kimi-K3（MLA + MegaMoe，TP16×16 卡）在 chunked prefill 跨边界的第一步稳定挂死；触发条件为聚合上下文跨 ≥12 个 KV block，或 KV 池为非连续交错视图。

## PR 动态（10 项）

### Bugfix / 回滚
- **#17519** — 修复 DFlash FIA 调用中 per-query causal KV 边界丢失问题，针对 #17520 的根因修复。
- **#17522** — 修复 MRV2 `kv_caches_dict.register_kv_caches` 只取首张 tensor 的问题，正确处理 Ascend 的 per-layer K/V 元组与 Conv/SSM 列表。
- **#17525** — 回滚 #17184（glm5.2 dspark d2h free_bubble 修复），将 `seq_lens` 来源选择从 main 分支撤回。

### 新特性
- **#17524** — 支持 FullAttention 在 `DCP>1` 且 `num_kv_heads>1` 时启用 TP 并行；典型受益场景为 Kimi K3 + MHA dspark draft model。
- **#17521** — 量化新增 `W4A8_DYNAMIC` 动态线性层支持；此前 ModelSlim 该格式仅对 fused MoE 层注册，现扩展到独立线性投影。

### CI / 文档 / 测试
- **#17523** — 优化 nightly 镜像构建与发布工作流（原仅支持指定 vLLM + vLLM Ascend commit 组合，现已增强）。
- **#17526** — 自动翻译 10 个文档文件（含 DeepSeek-V4-Flash 等教程）。
- **#17516** — 验证 GLM-5.2 W8A8C8 在 A3 单节点（DP1/TP16, 32K 上下文）的部署配置，替换原超出 KV-cache 容量的 135K 设置。
- **#17515** — CI 测试 PR。
- **#17517** — 与 v0.27.1rc1 相关：补充模型验证与发布结论报告，并从 release notes 中移除全部 Kimi K3 相关条目（模型/性能/精度/配置/可靠性）。

## Release 动态
本次窗口内未见正式 Release 发布，但 PR #17517 显示 **v0.27.1rc1** 候选版本正在准备发布结论报告，且明确剔除了 Kimi K3 相关验证内容，暗示该模型在该 rc 中暂未达成发布门槛。

---

## 🐛 Issues

### #17520 — [[Bug][Attention] DFlash speculative verify: folded queries sharing causal KV length in FIA produce wrong outputs](https://github.com/vllm-project/vllm-ascend/issues/17520)
- **作者**: 982945902  **时间**: 2026-09-24 23:33 CST
- **摘要**: ## Summary  When running DFlash speculative decoding on Ascend, the target verify step folds multiple query tokens (e.g. 16 per sequence) into a single `npu_fused_infer_attention_score` call. We found that query groups in this call can share a causal KV length, even when the queries should see diffe…

### #17518 — [[Bug][MLA / torch_npu] npu_gather_pa_kv_cache 在聚集上下文跨 ≥12 个 KV block、或 KV 池为非连续交错视图时挂死（507035 / EZ9999）](https://github.com/vllm-project/vllm-ascend/issues/17518)
- **作者**: linnea-lin-00638949  **时间**: 2026-09-24 22:22 CST
- **摘要**: ## 问题概述  Kimi-K3 部署（MLA + MegaMoe，TP16 × 16 张 Ascend 910_93）在 prefill 跨 chunk 边界的第一步稳定打挂引擎。通过进程内探针定位到失败调用是 `MLAChunkedPrefill._compute_prefill_context`（`vllm_ascend/attention/mla_v1.py`，#15219 引入的 chunked-context 循环）里的 `DeviceOperator.kv_cache_load` → **`torch_npu.npu_gather_pa_kv_cache`**。  独立最小复现（…

## 🔀 Pull Requests

### #17526 — [[Doc] Translated Doc files 2026-09-25](https://github.com/vllm-project/vllm-ascend/pull/17526)
- **作者**: vllm-ascend-ci  **时间**: 2026-09-25 12:51 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **10** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/tutorials/models/DeepSeek-V4-Flash.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/tutorials/models/DeepS…

### #17525 — [Revert "[BugFix] Solve the d2h free_bubble in glm5.2 dspark" (#17184)](https://github.com/vllm-project/vllm-ascend/pull/17525)
- **作者**: LQDLove  **时间**: 2026-09-25 11:24 CST
- **标签**: module:tests, module:core, ready-precise
- **摘要**: ### What this PR does / why we need it?  Reverts #17184 ("[BugFix] Solve the d2h free_bubble in glm5.2 dspark").  The seq_lens source selection introduced by #17184 needs to be rolled back from main. Reverting restores the original behavior in `AscendAttentionBackend.build`:  - `_select_seq_lens` he…

### #17524 — [[Feature][P/D] Support FullAttn use TP parallel when DCP>1 and num_kv_heads > 1](https://github.com/vllm-project/vllm-ascend/pull/17524)
- **作者**: nwpu-zxr  **时间**: 2026-09-25 09:33 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it? Support FullAttention use TP parallel when DCP>1 and `num_kv_heads > 1`. For example, Kimi K3 with MHA dspark draft model, if DCP is enabled, the MLA layer will use DCP to extend block_size and reuse the Redundant kv_cache across DP, but Mamba and MHA dspark d…

### #17523 — [[CI] Improve nightly image build and publishing workflow](https://github.com/vllm-project/vllm-ascend/pull/17523)
- **作者**: MrZ20  **时间**: 2026-09-25 00:30 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? The manually triggered nightly image build workflow previously supported building a specific vLLM commit together with a specific vLLM Ascend commit. However, it had several limitations:  1. `vllm_commit` only accepted a commit hash and could not directly use …

### #17522 — [[BugFix][MRV2][P/D] Fix MRV2 kv_caches_dict register_kv_caches only use first tensor](https://github.com/vllm-project/vllm-ascend/pull/17522)
- **作者**: nwpu-zxr  **时间**: 2026-09-25 00:26 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it? Fix MRV2 initialization and KV-cache handling for Ascend’s per-layer K/V tuples and Conv/SSM lists. The previous workaround returned only the first tensor to satisfy upstream device filtering, dropping V/SSM from connector registration and breaking PD transfer…

### #17521 — [[Feat][Quantization] Support W4A8 dynamic linear layers](https://github.com/vllm-project/vllm-ascend/pull/17521)
- **作者**: q664171689  **时间**: 2026-09-24 23:46 CST
- **标签**: module:tests, module:ops, module:quantization
- **摘要**: ### What this PR does / why we need it?  ModelSlim checkpoints can describe standalone linear projections as `W4A8_DYNAMIC`, but vLLM Ascend currently registers that format only for fused MoE layers. Models whose shared experts are implemented as a regular MLP therefore cannot construct their W4A8 g…

### #17519 — [fix(attention): maintain per-query causal KV boundaries in FIA for DFlash](https://github.com/vllm-project/vllm-ascend/pull/17519)
- **作者**: 982945902  **时间**: 2026-09-24 23:32 CST
- **摘要**: ## Problem  When multiple queries are folded into a single `npu_fused_infer_attention_score` (FIA) call — which happens in DFlash speculative decoding verify, and in grouped prefill — query groups can incorrectly share a causal KV length. Queries that should see different KV extents end up in the sa…

### #17517 — [[0.27.1rc][Doc][Test] Add v0.27.1rc1 model validation and release conclusion report](https://github.com/vllm-project/vllm-ascend/pull/17517)
- **作者**: underfituu  **时间**: 2026-09-24 21:24 CST
- **标签**: documentation, module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  Remove all Kimi K3 model, performance, accuracy, configuration-link, and reliability entries from the release notes copied into tests/. Update the validation count and reliability numbering accordingly.  ### Does this PR introduce any user-facing change?  No.…

### #17516 — [[Docs][Misc] validate GLM-5.2 W8A8C8 A3 deployment](https://github.com/vllm-project/vllm-ascend/pull/17516)
- **作者**: Wyz-134  **时间**: 2026-09-24 21:14 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  - Update the A3 single-node GLM-5.2 example to the verified W8A8C8 checkpoint with DP1/TP16 and a 32K context. The previous 135K setting exceeded available KV-cache memory on the tested 64 GB × 16 setup. - Update the A3 dual-node co-located example to W8A8C8,…

### #17515 — [[CI] test](https://github.com/vllm-project/vllm-ascend/pull/17515)
- **作者**: xqchen7  **时间**: 2026-09-24 21:00 CST
- **标签**: ci/build, module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/ced6857afa0ea7b2e3f0846a62e1394e90f15607
