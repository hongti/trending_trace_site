# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-29 14:03 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
近期主要集中报告了镜像编译与推理运行时的兼容性及崩溃 Bug：
- **扩展缺失导致崩溃**：发布的 `v0.26.0rc2-310p` 镜像缺失编译好的 `vllm_ascend_C` 扩展，导致所有基于 GDN 的模型（如 Qwen3.5 / Qwen3-Next 家族）在 310P 上直接崩溃 (#17713)。
- **上游接口不兼容**：上游 vLLM 更新了 `GPUModelRunner.prepare_inputs()` 接口，新增了第四个参数 `num_active_loras`，导致 MRV2 签名不兼容 (#17710)。
- **编译与推理失败**：9月28日的 `main` 镜像在编译安装阶段失败 (#17709)；此外，在 `main` 镜像下运行 `kimi-k3` 时，同时开启 DCP、MRV2 和 gmmsituquant 会导致推理失败（关闭 DCP 可正常运行）(#17706)。

### 🛠️ Pull Request 动态
PR 活动涵盖关键 Bug 修复、新模型支持与架构重构：
- **关键修复**：
  - 修复构建盲区：当设置 `COMPILE_CUSTOM_KERNELS=1` 但 CMake 未生成 `vllm_ascend_C*.so` 时，`setup.py` 将直接快速失败，避免静默通过导致后续运行崩溃 (#17714)。
  - 修复 fa quant 问题 (#17712)。
  - 在 KV offload 场景下，旧请求结束后显式重置请求级状态，修复潜在异常 (#17708)。
- **新特性**：
  - **模型支持**：支持 GLM-5.3-Flash KPool 索引器（带有 PCP 和 DCP），将完成的四 Token 池压缩至 BF16 缓存中，取代了旧版 GLM-5.2 的 LightningIndexer (#17705)。
  - **心跳机制**：为 MooncakeConnector V2 新增心跳机制，取代固定延迟释放时间，防止 P 节点过早释放 KV 缓存块 (#17704)。
  - **Decode 隔离**：隔离核心 decode 请求分片适配 (#17701)。
- **重构与优化**：
  - 弃用非 FP8 的 Triton RoPE，统一改用 Ascend 原生算子实现，优化普通 Q/K RoPE 调度 (#17703)。
  - 向 `releases/v0.28.0rc` 分支挑选合并代码：移除 `batch_matmul_transpose` 自定义算子，替换为 aclnn TransposeBatchMatMul 路径，并修复 ACL graph 的 SFA 问题 (#17707)。
- **文档与 CI**：更新 DSV4 Flash 文档中的 Ascend 950DT 镜像标签（针对 v0.27.1rc）(#17711)；为单节点 CI 测试用例添加 `config_base_path` 并引入 `TEST_FREQUENCY` 环境变量 (#17702)。

### 🚀 Release 动态
本期未包含正式的 Release 发布记录，但 PR 记录显示，团队正在积极维护 `v0.27.1rc`（文档更新）和 `releases/v0.28.0rc`（代码重构与挑选合并）这两个候选版本分支。

---

## 🐛 Issues

### #17713 — [[Bug]: v0.26.0rc2-310p release image is missing the vllm_ascend_C extension — all GDN models (Qwen3.5 family) crash on 310P](https://github.com/vllm-project/vllm-ascend/issues/17713)
- **作者**: green-hand-shandong  **时间**: 2026-09-29 14:02 CST
- **标签**: 310p, qwen-3.5, multimodal-understanding
- **摘要**: ### Bug Description  The published image `quay.io/ascend/vllm-ascend:v0.26.0rc2-310p` does not contain the compiled `vllm_ascend_C` extension. As a result, any GDN-based model (Qwen3.5 / Qwen3-Next family) crashes on 310P at the first forward pass:  ``` (EngineCore pid=36) ERROR ...   File "/vllm-wo…

### #17710 — [[Bug]: MRV2 prepare_inputs signature is incompatible with updated vLLM num_active_loras argument](https://github.com/vllm-project/vllm-ascend/issues/17710)
- **作者**: Cancan0320  **时间**: 2026-09-29 12:38 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Your output of above commands here ```  </details>   ### 🐛 Describe the bug   Upstream vLLM changed the `GPUModelRunner.prepare_inputs()` interface to add a fourth argument, `num_active_loras`. …

### #17709 — [[Bug]: 9月28日main镜像编译安装失败](https://github.com/vllm-project/vllm-ascend/issues/17709)
- **作者**: Sfeching  **时间**: 2026-09-29 11:54 CST
- **标签**: bug
- **摘要**: ### Your current environment  A3  ### 🐛 Describe the bug    [2026-09-28 09:46:53] [ 93%] Linking CXX static library libabsl_flags_parse.a   [2026-09-28 09:46:53] [ 93%] Built target flags_parse   [2026-09-28 09:46:53] [ 93%] Building CXX object third_party/abseil-cpp/absl/log/CMakeFiles/log_flags.di…

### #17706 — [[Bug]: main镜像 kimi-k3 开启dcp、mrv2、gmmsituquant 推理失败。关闭dcp正常推理](https://github.com/vllm-project/vllm-ascend/issues/17706)
- **作者**: Sfeching  **时间**: 2026-09-29 11:37 CST
- **标签**: bug
- **摘要**: ### Your current environment  A3 vllm-ascend b64b4d71 vllm 0.30.0   prefill： `DP_RANK=$4 KV_PORT=$((31000 + DP_RANK * 100)) export LOCAL_IP=$(ifconfig eth0 2>/dev/null | awk '/inet /{print $2; exit}') export DRAFT_MODEL_PATH="/data1/models/inferact-kimi-k3-dspark-block5" export VLLM_RPC_TIMEOUT=3600…

## 🔀 Pull Requests

### #17714 — [[BugFix][Build] Fail fast when the C extension is silently not built](https://github.com/vllm-project/vllm-ascend/pull/17714)
- **作者**: green-hand-shandong  **时间**: 2026-09-29 14:02 CST
- **摘要**: ### What This PR Does / Why We Need It?  `setup.py` currently completes `pip install` successfully even when `COMPILE_CUSTOM_KERNELS=1` but the CMake build produced no `vllm_ascend_C*.so` (e.g. building from a clean checkout without the CANN environment sourced). The resulting package imports fine; …

### #17712 — [[BugFix] fix fa quant](https://github.com/vllm-project/vllm-ascend/pull/17712)
- **作者**: lcfenglinwan  **时间**: 2026-09-29 13:39 CST
- **标签**: module:quantization
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/ced6857afa0ea7b2e3f0846a62e1394e90f15607

### #17711 — [[v0.27.1rc][Doc] Update DSV4 Flash Doc](https://github.com/vllm-project/vllm-ascend/pull/17711)
- **作者**: lcfenglinwan  **时间**: 2026-09-29 12:48 CST
- **标签**: documentation
- **摘要**: <!--  Thanks for sending a pull request!  BEFORE SUBMITTING, PLEASE READ https://docs.vllm.ai/en/latest/contributing/overview.html  --> ### What this PR does / why we need it? <!-- - Please clarify what changes you are proposing. The purpose of this section is to outline the changes and how this PR …

### #17708 — [[Bugfix] Explicitly reset request level states when old requests are finished in kv offload](https://github.com/vllm-project/vllm-ascend/pull/17708)
- **作者**: Angazenn  **时间**: 2026-09-29 11:49 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/ced6857afa0ea7b2e3f0846a62e1394e90f15607

### #17707 — [[Cherry-pick][releases/v0.28.0rc][Refactor][Ops] Remove batch_matmul_transpose custom operator (from #17141)](https://github.com/vllm-project/vllm-ascend/pull/17707)
- **作者**: ZT-AIA  **时间**: 2026-09-29 11:48 CST
- **标签**: module:tests, ready-precise
- **摘要**: Cherry-pick of #17141 onto `releases/v0.28.0rc`.  Remove the `batch_matmul_transpose` custom operator and replace its usage with the aclnn TransposeBatchMatMul path, including SFA fixes for ACL graph and N*L/K*B constraint handling.  Changes: - Delete `csrc/batch_matmul_transpose/` operator sources …

### #17705 — [[Feature][Model] Support GLM-5.3-Flash KPool indexer with PCP and DCP](https://github.com/vllm-project/vllm-ascend/pull/17705)
- **作者**: Zhaocen  **时间**: 2026-09-29 11:30 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  GLM-5.3-Flash replaces the GLM-5.2 LightningIndexer cache layout with a pooled KPool indexer. Completed four-token pools are compressed into BF16 cache while the causal suffix remains in an FP32 tail cache. The original Ascend path did not support this layout…

### #17704 — [[Feature][P/D] MoocakeConnector V2 supports heartbeat](https://github.com/vllm-project/vllm-ascend/pull/17704)
- **作者**: nwpu-zxr  **时间**: 2026-09-29 11:23 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it? Add heartbeat for `MooncakeConnectorV2`. The heartbeat mechanism can be used to replace the fixed delayed free time, preventing the P node from prematurely releasing kv_cache due to request congestion, which could lead to potential precision issues.  Due to ve…

### #17703 — [[Refactor][Op] Retire non-FP8 Triton RoPE in favor of Ascend ops](https://github.com/vllm-project/vllm-ascend/pull/17703)
- **作者**: AuroraEmiya  **时间**: 2026-09-29 11:19 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Route non-FP8 RoPE through the existing Ascend implementations and retire the corresponding Triton kernels:  - Dispatch ordinary Q/K RoPE, including Q-only calls, to `torch_npu.npu_mrope`. Use its supported `rotary_mode="interleave"` value for non-NeoX layout…

### #17702 — [[CI] single node cases add config_base_path](https://github.com/vllm-project/vllm-ascend/pull/17702)
- **作者**: chen-commits  **时间**: 2026-09-29 11:16 CST
- **标签**: documentation, ci/build, module:tests, module:tools, ready-precise
- **摘要**: ### What this PR does / why we need it? This pull request introduces a TEST_FREQUENCY environment variable to explicitly configure scheduled-test frequencies and refactors the multi-node test path resolution in the bisect tool. It also updates documentation regarding DP mode selection. ### Does this…

### #17701 — [[Feature][PCP] Isolate the core decode request sharding adaptation](https://github.com/vllm-project/vllm-ascend/pull/17701)
- **作者**: recky-c  **时间**: 2026-09-29 11:10 CST
- **标签**: module:tests
- **摘要**: This PR preserves the previous implementation from #17456, swapped at the user's request. Current code: `745c2f1853504d56b974290825a055d68a6fe949`. The historical validation notes below refer to their explicitly named revisions.  ### What this PR does / why we need it?  Assign PCP decode requests to…
