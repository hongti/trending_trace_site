# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-12 09:06 CST

## AI 总结

# vllm-ascend 仓库近期动态摘要

## 🐛 Issue

- **#11863** — Qwen3.6 在 PD 分离部署时抛出 `ValueError: Non-MoE models do not support external data parallel mode`，非 MoE 模型与外部数据并行模式的兼容性问题。
- **#11859** — GLM5.1-W4A8（16卡 910C，TP8/DP1/CP2）短上下文推理正常，但 **60k 长上下文**出现 `DDR address of MTE instruction is out of range` 错误，疑似长序列内存越界。

## 🔀 Pull Request

### Bug 修复
- **#11864** — 修复 SFA DCP 复制索引器在 KV 迁移（P/D transfer）时的处理逻辑，为跨 DCP rank 复制的稀疏索引 K cache 添加显式处理。
- **#11865 / #11866** — 新增 `store_kv_block_metadata` AscendC 算子，修正从 `slot_mapping.cpu` 而非 `.gpu` 取数据的问题（因 `slot_mapping.cpu != slot_mapping.gpu` 传入 sfa_v1 导致不一致）。

### 性能优化
- **#11862** (cherry-pick → v0.23.0) — 规避 CP proposer metadata 中的 H2D 同步开销，减少同步等待。

### 图融合 / 算子替换
- **#11867** (cherry-pick → v0.23.0) — 用 `torch_npu.npu_scatter_pa_kv_cache` / `npu_gather_pa_kv_cache` 替换已废弃的 PA KV cache 写/加载调用点。
- **#11861** (cherry-pick → v0.23.0) — 修复图融合（graph fusion）相关问题。

### CI / 测试
- **#11857 / #11858** — 降低 DeepSeek-V3.2-W8A8-DCP 夜间测试的 `max_out_len` 至 4096，修正配置错误。
- **#11856** — 新增 Gemma4 夜间单节点 CI 覆盖，包含 eager 执行和 ACLGraph `FULL_DECODE_ONLY` 两种模式（覆盖 MoE 与 dense 变体）。

### 文档
- **#11860** — 新增 310P 硬件相关文档。

## 🚀 Release

本期无新 Release 版本发布。多个 PR 正向 `releases/v0.23.0` 分支 cherry-pick，为 **v0.23.0** 版本做积聚准备，值得关注的关键亮点：
- PA KV cache 算子现代化替换（废弃算子清理）
- CP proposer metadata H2D 同步优化
- 图融合问题修复
- SFA DCP KV 迁移与 slot_mapping 一致性修复

---

## 🐛 Issues

### #11863 — [[Bug]: Qwen3.6 PD分离部署时出现ValueError: Non-MoE models do not support external data parallel mode.](https://github.com/vllm-project/vllm-ascend/issues/11863)
- **作者**: hei6775  **时间**: 2026-07-11 19:46 CST
- **标签**: bug
- **摘要**: ### Your current environment  根据官方文档：[Qwen3.5-27B-Qwen3.6-27B](https://docs.vllm.ai/projects/ascend/en/latest/tutorials/models/Qwen3.5-27B-Qwen3.6-27B.html#52-multi-node-pd-separation-deployment)的5.2 Multi-Node PD Separation Deployment章节，2台A2服务器PD分离部署Qwen3.6-27B报错。 vllm-ascend: v0.21.0rc1 2* A2 （2*8…

### #11859 — [[Bug]: GLM5.1-W4A8使用16卡910c部署，开启TP8，DP1，CP2的情况下，短上下文请求推理正常，但60k上下文请求出现The DDR address of the MTE instruction is out of range错误](https://github.com/vllm-project/vllm-ascend/issues/11859)
- **作者**: XiangyuBi0519  **时间**: 2026-07-11 17:54 CST
- **标签**: bug, glm5, llm-model
- **摘要**: ### Your current environment  GLM5.1-W4A8使用16卡910c，vllm版本0.19.0  ### 🐛 Describe the bug  如题，同时附上现在的启动参数: export HCCL_IF_IP=$local_ip export GLOO_SOCKET_IFNAME=$nic_name export TP_SOCKET_IFNAME=$nic_name export HCCL_SOCKET_IFNAME=$nic_name export HCCL_OP_EXPANSION_MODE="AIV" export PYTORCH_NPU_ALLOC_…

## 🔀 Pull Requests

### #11867 — [[Cherry-pick][releases/v0.23.0][Attention][Misc] Replace PA KV cache operators (from #11713)](https://github.com/vllm-project/vllm-ascend/pull/11867)
- **作者**: maoxx241  **时间**: 2026-07-11 23:25 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it?  Backport #11713 to `releases/v0.23.0`.  - Replace deprecated PA KV cache write/load call sites with `torch_npu.npu_scatter_pa_kv_cache` and `torch_npu.npu_gather_pa_kv_cache`. - Route the SFA CP KV write path through `DeviceOperator.reshape_and_cache`. - Upda…

### #11866 — [[BugFix]Added the store_kv_block_metadata ascendC operator](https://github.com/vllm-project/vllm-ascend/pull/11866)
- **作者**: ZT-AIA  **时间**: 2026-07-11 21:19 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it? 1. The operator obtains data from slot_mapping.cpu instead of slot_mapping.gpu. This is because slot_mapping.cpu!=slot_mapping.gpu is transmitted to sfa_v1. 2. Therefore, the `cpu_slotmapping` variable is deleted. It is not used as a placeholder in other place…

### #11865 — [[BugFix]Added the store_kv_block_metadata ascendC operator](https://github.com/vllm-project/vllm-ascend/pull/11865)
- **作者**: ZT-AIA  **时间**: 2026-07-11 20:36 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it? 1. The operator obtains data from slot_mapping.cpu instead of slot_mapping.gpu. This is because slot_mapping.cpu!=slot_mapping.gpu is transmitted to sfa_v1. 2. Therefore, the `cpu_slotmapping` variable is deleted. It is not used as a placeholder in other place…

### #11864 — [Fix SFA DCP replicated indexer KV transfer](https://github.com/vllm-project/vllm-ascend/pull/11864)
- **作者**: pisceskkk  **时间**: 2026-07-11 20:00 CST
- **标签**: module:tests
- **摘要**: ## What this PR does  Follow-up fix for #11696. This patch fixes SFA DCP `sfa_dcp_replicate_k` handling for P/D KV transfer when the sparse indexer K cache is replicated across DCP ranks.  It adds explicit SFA replicated-indexer cache layout metadata, applies the correct decode-to-prefill DCP rank o…

### #11862 — [[v0.23.0][Performance][SpecDecode] Avoid H2D sync in CP proposer metadata](https://github.com/vllm-project/vllm-ascend/pull/11862)
- **作者**: pisceskkk  **时间**: 2026-07-11 19:17 CST
- **标签**: module:tests, ready
- **摘要**: Cherry-pick of #11496 onto `releases/v0.23.0`.  This backports the CP proposer metadata sync reductions and follow-up lint fixes from the main PR.  Validation: - `ruff check --output-format github` on changed Python files - `ruff format --check` on changed Python files - `mypy --ignore-missing-impor…

### #11861 — [[Cherry-pick][releases/v0.23.0][Misc] Fix issues related to graph fusion](https://github.com/vllm-project/vllm-ascend/pull/11861)
- **作者**: kunpengW-code  **时间**: 2026-07-11 18:23 CST
- **标签**: documentation, module:tests, ready
- **摘要**: ### What this PR does / why we need it? Cherry-pick of PR https://github.com/vllm-project/vllm-ascend/pull/11776 onto releases/v0.23.0.  Original PR: https://github.com/vllm-project/vllm-ascend/pull/11776  ### Does this PR introduce _any_ user-facing change? no  ### How was this patch tested?  - vLL…

### #11860 — [Docs/310p documentation](https://github.com/vllm-project/vllm-ascend/pull/11860)
- **作者**: zyz111222  **时间**: 2026-07-11 18:07 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #11858 — [[Cherry-pick][releases/v0.23.0][Test](nightly): reduce max out len to 4096 in DeepSeek-V3.2-W8A8-DCP.yaml (from #11857)](https://github.com/vllm-project/vllm-ascend/pull/11858)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-11 17:22 CST
- **标签**: module:tests
- **摘要**: Cherry-pick of PR #11857 onto `releases/v0.23.0`.  Original PR: #11857 Original author: @pisceskkk  --- ### What this PR does / why we need it?  Fix error max out len in DeepSeek-V3.2-W8A8-DCP.yaml.   - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac761…

### #11857 — [[Test](nightly): reduce max out len to 4096 in DeepSeek-V3.2-W8A8-DCP.yaml](https://github.com/vllm-project/vllm-ascend/pull/11857)
- **作者**: pisceskkk  **时间**: 2026-07-11 17:19 CST
- **标签**: module:tests, ready
- **摘要**: ### What this PR does / why we need it?  Fix error max out len in DeepSeek-V3.2-W8A8-DCP.yaml.   - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #11856 — [[CI] Add Gemma4 graph and eager nightly coverage](https://github.com/vllm-project/vllm-ascend/pull/11856)
- **作者**: Alex-stack-hub  **时间**: 2026-07-11 17:18 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This PR adds Gemma4 nightly single-node CI coverage for both eager execution and ACLGraph `FULL_DECODE_ONLY` execution.  The new coverage includes: - Gemma4 MoE eager and full-decode graph cases. - Gemma4 31B dense eager and full-decode graph cases. - GPQA-D …
