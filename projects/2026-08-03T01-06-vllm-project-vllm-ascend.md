# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-03 09:06 CST

## AI 总结

# vllm-project/vllm-ascend 近期动态摘要

---

## 📋 Issue

| # | 标题 | 要点 |
|---|------|------|
| #13343 | main2main 自动化流程中断 | CI 自动化未完成所有计划步骤，需人工介入审查 |
| #13339 | DeepSeek-V4-Flash w8a8 缓存命中为 0 | nightly v0.25.1rc-openeuler 部署后 KV-cache 命中率异常，疑似缓存机制缺陷 |
| #13338 | PD 负载均衡代理丢失 prefiller active_tokens | `SharedProxyScheduler` 重构后，`_pick_server` 仅走 KV-cache 路径，导致 prefiller 的 active_tokens 计数丢失 |
| #13336 | 请求适配 Unlimited-OCR 模型 | 社区询问鲲鹏 920 + 昇腾 910B3 是否计划支持 PaddlePaddle/Unlimited-OCR |

---

## 🔀 Pull Request

### 🚀 新特性 / 功能增强

- **#13333 / #13332 — LoRA 支持 MoE 模型 W8A8 动态多 LoRA 推理**（重点）
  - 面向 DeepSeek-V4-Flash 类模型，支持 routed experts + shared experts + LoRA
  - 提供通用量化 MoE LoRA 推理框架，补齐昇腾在 MoE+LoRA 场景的空白

- **#13335 — CGDR 中 mamba_ssm_dtype 支持 fp32 精度**
  - A2/A3 设备上 CGDR（连续生成解码）的 Mamba 状态数据类型扩展至 fp32

### ⚡ 性能优化

- **#13340 — 融合多组 slot mapping 为单一 2D-grid Triton kernel**（重点）
  - 新增 `compute_slot_mapping_fused_kernel`，使用 `(num_reqs+1, num_groups)` 二维网格
  - 预缓存参数，减少 kernel launch 开销，提升调度效率

### 🐛 Bug 修复

- **#13341 — 恢复 prefiller active token 计数**
  - 修复 #13338 对应问题，在 disaggregated prefill 负载均衡代理中恢复 active_tokens 统计，无需额外 RPC 调用

- **#13337 — MTP 在线更新时保留 MoE weight_loader**
  - 修复在线 RL 权重更新时，MoE 权重 layout 变化导致 weight_loader 元数据丢失的问题

- **#13334 — 重命名自定义 SparseFlashAttention**
  - 避免与 CANN 内置稀疏注意力算子命名冲突，消除两层集成中的符号重复

### 🔄 上游适配 & CI

- **#13344 — 适配 vLLM main（截至 7 月 27 日）** — 成功适配
- **#13342 — 适配 vLLM main（09061239）** — main2main 适配失败，无步骤完成
- **#13345 — 为 a5 Dockerfile 添加 buildkit fuse-overlayfs 测试** — 同步 nv-action/vllm-benchmarks#278 的 Dockerfile 变更

---

## 🏷️ Release

> 本周期内无新版本发布。

---

### 📌 总体观察

1. **MoE + LoRA 是当前核心方向**：DeepSeek-V4-Flash 类模型的 W8A8 量化 + 多 LoRA 推理支持是本次最突出的功能推进。
2. **调度层重构引入回归**：`SharedProxyScheduler` 重构后导致 prefiller active_tokens 丢失（#13338），已通过 #13341 修复，说明 disaggregated prefill 架构仍在快速迭代中。
3. **性能优化持续深入**：slot mapping kernel 融合（#13340）和算子命名规范化（#13334）体现出对底层算子性能和稳定性的持续打磨。
4. **上游同步存在波动**：vLLM main 分支适配一次成功、一次失败，需关注后续 CI 稳定性。

---

## 🐛 Issues

### #13343 — [[main2main] main2main manual review required ()](https://github.com/vllm-project/vllm-ascend/issues/13343)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-03 01:05 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/13342 - Commit range: `394beb633b0b5b5d68aed3d2f2b5c1477e756d80`...`` - Status: `failed`  ## Final Summary  main2main adaptation failed — no steps c…

### #13339 — [[Bug]: nightly-releases-v0.25.1rc-openeuler部署DeepSeek-V4-Flash-0731-w8a8，缓存命中为0](https://github.com/vllm-project/vllm-ascend/issues/13339)
- **作者**: ponyioy  **时间**: 2026-08-02 21:17 CST
- **标签**: bug, llm-model, deepseek
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Your output of above commands here ```  </details>   ### 🐛 Describe the bug  A2机器一台，部署脚本：  ``` docker run -dit \   --name model-DeepSeek-V4-Flash-0731-w8a8-6390 \   --shm-size 512g \   --privile…

### #13338 — [[Bug]: PD load balance proxy drops prefiller active_tokens after shared-scheduler refactor](https://github.com/vllm-project/vllm-ascend/issues/13338)
- **作者**: mashuiping  **时间**: 2026-08-02 20:45 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Your output of above commands here ``` </details>   ### 🐛 Describe the bug  ## Background In the post-#9645 `SharedProxyScheduler`:  - `begin_request` / `reserve_prefill_kv` call `_pick_server(.…

### #13336 — [[New Model]: https://modelscope.cn/models/PaddlePaddle/Unlimited-OCR](https://github.com/vllm-project/vllm-ascend/issues/13336)
- **作者**: WuYueH2  **时间**: 2026-08-02 19:08 CST
- **标签**: new model
- **摘要**: ### The model to consider.  请问一下是否适配或者有没有计划：鲲鹏920 + 昇腾910B3 适配：Unlimited-OCR ？？  ### The closest model vllm already supports.  _No response_  ### What's your difficulty of supporting the model you want?  _No response_

## 🔀 Pull Requests

### #13345 — [feat(ci): add buildkit fuse-overlayfs test for a5 Dockerfiles](https://github.com/vllm-project/vllm-ascend/pull/13345)
- **作者**: JavaPythonAIForBAT  **时间**: 2026-08-03 09:01 CST
- **标签**: ci/build
- **摘要**: Sync Dockerfiles from [nv-action/vllm-benchmarks#278](https://github.com/nv-action/vllm-benchmarks/pull/278).  - Add workflow `buildkit-dockerfile-test-group3.yaml` - Sync `Dockerfile.a5`, `Dockerfile.a5.openEuler` - runner labels: linux-aarch64-cpu-4-buildkit / linux-amd64-cpu-4-buildkit - vLLM ver…

### #13344 — [[Misc]feat: adapt to vLLM main (04502dec)](https://github.com/vllm-project/vllm-ascend/pull/13344)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-03 03:01 CST
- **标签**: module:core
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to July 27.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | `vllm_ascend/__init__.py` | [09061239](https://github.com/vllm-project/vll…

### #13342 — [[Misc]feat: adapt to vLLM main (09061239)](https://github.com/vllm-project/vllm-ascend/pull/13342)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-03 01:04 CST
- **摘要**: main2main adaptation failed — no steps completed.  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/d02df748bf9efd99022f1a062597dc3cb3808485

### #13341 — [[BugFix][Misc] Restore prefiller active token accounting without extra RPC](https://github.com/vllm-project/vllm-ascend/pull/13341)
- **作者**: mashuiping  **时间**: 2026-08-02 23:24 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This PR restores prefiller `active_tokens` accounting in the disaggregated prefill load-balance proxy.  After the shared-scheduler refactor, prefiller selection reserved only `active_kv_cache`. As a result, `active_tokens` remained zero and the prefiller scor…

### #13340 — [[Performance]: Fuse multi-group slot mapping into a single 2D-grid Triton kernel with pre-cached params](https://github.com/vllm-project/vllm-ascend/pull/13340)
- **作者**: frankie-ys  **时间**: 2026-08-02 23:01 CST
- **标签**: module:ops
- **摘要**: ### What this PR does / why we need it? ### 1. Fused 2D-grid slot-mapping kernel (`ops/triton/slot_mapping.py`) - New `compute_slot_mapping_fused_kernel` uses a 2D grid   `(num_reqs + 1, num_groups)`. - Each program handles exactly one `(req_idx, group_idx)` pair, so all KV   cache groups execute co…

### #13337 — [[MTP][BugFix] Preserve MoE weight loaders during online updates](https://github.com/vllm-project/vllm-ascend/pull/13337)
- **作者**: Mengyuyang  **时间**: 2026-08-02 19:56 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  This PR prevents existing `weight_loader` metadata from being dropped when unquantized Ascend MoE weights change layout during online RL weight updates with MTP.  The target model and the MTP drafter receive actor weights through `model.load_weights()`. After…

### #13335 — [[WIP][Feat][Ops] mamba_ssm_dtype in CGDR support fp32 precision in A2/A3](https://github.com/vllm-project/vllm-ascend/pull/13335)
- **作者**: leijie-ww  **时间**: 2026-08-02 17:20 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.26.0 - vLLM main: https://github.com/vllm-project/vllm/commit/d02df748bf9efd99022f1a062597dc3cb3808485

### #13334 — [[BugFix][Ops] Rename custom SparseFlashAttention to avoid CANN collision](https://github.com/vllm-project/vllm-ascend/pull/13334)
- **作者**: lxb007981  **时间**: 2026-08-02 14:49 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  CANN already provides a built-in sparse-attention operator. The vLLM Ascend custom implementation duplicated its names at two different integration layers:  - Both operators used `SparseFlashAttention` as their GE OpType, causing an operator-registration coll…

### #13333 — [[Feature][LoRA] 1.Support W8A8 dynamic multi-LoRA inference for MoE models; 2. support shared expert + lora;3. Gereneral quant MOE FrameWork for Lora inference ](https://github.com/vllm-project/vllm-ascend/pull/13333)
- **作者**: SkychenLee  **时间**: 2026-08-02 11:55 CST
- **标签**: module:tests, module:ops, module:quantization
- **摘要**: ## Motivation  This PR adds Ascend support for multi-LoRA inference on MoE models with a W8A8_DYNAMIC quantized base model.  The main target is DeepSeek V4 Flash-style models containing routed experts and shared experts. LoRA computation must preserve floating-point activation boundaries while the b…

### #13332 — [[Feature][LoRA] 1.Support W8A8 dynamic multi-LoRA inference for MoE models; 2. support shared expert + lora;3. Gereneral quant moe frameWork with Lora inference for Adaption](https://github.com/vllm-project/vllm-ascend/pull/13332)
- **作者**: SkychenLee  **时间**: 2026-08-02 11:47 CST
- **标签**: documentation, ci/build, module:tests, module:ops, module:core, module:quantization
- **摘要**: ## Motivation  This PR adds Ascend support for multi-LoRA inference on MoE models with a W8A8_DYNAMIC quantized base model.  The main target is DeepSeek V4 Flash-style models containing routed experts and shared experts. LoRA computation must preserve floating-point activation boundaries while the b…
