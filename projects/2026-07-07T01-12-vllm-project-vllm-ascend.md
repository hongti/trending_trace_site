# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-07 09:12 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue (问题反馈)
1. **NPU Graph 高并发崩溃问题 (#11514)**：在开启 NPU Graph 时，若并发请求超过 `max_num_seqs`，会触发 `aclnnScatterNdUpdate` 错误 (507011) 导致系统崩溃。
2. **Qwen3-Next 大模型精度异常 (#11509)**：`Qwen3-Next-80B-A3B-Instruct-W8A8` 量化模型在推理时存在精度问题。

### 🔧 Pull Request (代码合并)
**1. 核心修复**
- **修复 Qwen3.5+PCP+ChunkedPrefill 精度错误 (#11508)**：修正了 Qwen3.5 在 PCP 与 Chunked Prefill 堆叠时计算逻辑导致的精度丢失。
- **修复 PD Disaggregation Bug (#11517)**：解决了启用 Prefill-Decode 分离部署时出现的 Bug。

**2. 新特性与适配**
- **新增 W8A16 推理支持 (#11507)**：适配了 W8A16-FP8 格式的量化推理。
- **同步适配 vLLM 上游 Main 分支 (#11515, #11516)**：跟进 vLLM 上游最新代码（删除了 `DeepSeekV4ToolParser` 等废弃模块），保持分支兼容性。

**3. CI 与工程维护**
- **更新 Weekly 测试用例 (#11518)**：新增/更新了 Deepseekv3.2-w8a8、GLM5.1-W8A8、Qwen3.5-397B 等大模型的每周测试 YAML 配置。
- **回滚 CI 参数 (#11512)**：撤销了此前为 Nightly/Weekly CI 触发命令添加 `--aop_enabled` 参数的改动。
- **日常同步 (#11510, #11511, #11513)**：多处涉及 vLLM main 分支与 CANN 代码的日常同步与兼容性维护。

### 🚀 Release (版本发布)
- **近期无新版本发布**。当前重点动态集中在**向上游 vLLM main 分支同步适配**、**扩展量化支持 (W8A16)** 以及**修复并发与分离部署下的稳定性/精度问题**。

---

## 🐛 Issues

### #11514 — [Bug: NPU Graph crash with aclnnScatterNdUpdate error 507011 when concurrency > max_num_seqs](https://github.com/vllm-project/vllm-ascend/issues/11514)
- **作者**: xuejiakn  **时间**: 2026-07-06 23:06 CST
- **摘要**: ## Bug: NPU Graph (npugraph_ex) crash with `aclnnScatterNdUpdate` error 507011 when concurrency exceeds `max_num_seqs`  ### Summary  When using vLLM-Ascend with NPU Graph enabled (default), sending 32 concurrent requests with ~39K token context causes `aclnnScatterNdUpdate` to fail with NPU error co…

### #11509 — [[Bug]: Qwen3-Next-80B-A3B-Instruct-W8A8  has accuracy issue.](https://github.com/vllm-project/vllm-ascend/issues/11509)
- **作者**: menogrey  **时间**: 2026-07-06 21:06 CST
- **标签**: bug, qwen3-next, llm-model
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Versions of relevant libraries: [pip3] numpy==1.26.4 [pip3] pyzmq==27.1.0 [pip3] torch==2.10.0+cpu [pip3] torch_npu==2.10.0 [pip3] torchaudio==2.10.0+cpu [pip3] torchvision==0.25.0+cpu [pip3] tr…

## 🔀 Pull Requests

### #11518 — [[CI] update weekly cases: Deepseekv3.2-w8a8-A3.yaml GLM5.1-W8A8-a3-weekly.yaml Qwen3.5-397B-A17B-w8a8-mtp-A3-weekly.yaml](https://github.com/vllm-project/vllm-ascend/pull/11518)
- **作者**: guxin108  **时间**: 2026-07-07 09:09 CST
- **标签**: ci/build, module:tests, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  we update weekly cases: Deepseekv3.2-w8a8-A3.yaml GLM5.1-W8A8-a3-weekly.yaml Qwen3.5-397B-A17B-w8a8-mtp-A3-weekly.yaml  ### Does this PR introduce _any_ user-facing change? no  ### How was this patch tested? run the cases weekly  - vLLM version: v0.23.0 - vLL…

### #11517 — [fix p bugs while enabling PD Disaggregation](https://github.com/vllm-project/vllm-ascend/pull/11517)
- **作者**: chenhy97  **时间**: 2026-07-07 09:00 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.22.1 - vLLM main: https://github.com/vllm-project/vllm/commit/967c5c3bc38891f4465d3f4e99917ed837bb3833

### #11516 — [[Misc]feat: adapt to vLLM main (9fde043f)](https://github.com/vllm-project/vllm-ascend/pull/11516)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-07 05:00 CST
- **摘要**: ### What this PR does / why we need it?  vllm upstream `1f486d96...9fde043f` (13/13 steps).  #### vllm_ascend/core/recompute_scheduler.py  - Cause: Upstream deleted `vllm/tool_parsers/deepseekv4_tool_parser.py` (and `DeepSeekV4ToolParser` class) and introduced a new ParserEngine-based architecture (…

### #11515 — [[Misc]feat: adapt to vLLM main (0cd6f767)](https://github.com/vllm-project/vllm-ascend/pull/11515)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-07 00:27 CST
- **摘要**: ### What this PR does / why we need it?  vllm upstream `1f486d96...0cd6f767` (5/5 steps).  #### vllm_ascend/patch/platform/__init__.py  - Cause: `DeepSeekV4ToolParser` class deleted — replaced by engine-based parsers (`ParserEngine`, `DeepSeekV4ParserToolAdapter`, `DeepSeekV4EngineToolParser`). vllm…

### #11513 — [Rfc/vllm cann synchronize wuth main branch](https://github.com/vllm-project/vllm-ascend/pull/11513)
- **作者**: zzzkkk-zk  **时间**: 2026-07-06 22:16 CST
- **标签**: documentation, ci/build, module:tests, module:ops, module:core, module:quantization, module:tools
- **摘要**: ### What this PR does / why we need it?  Rfc/vllm cann synchronize wuth main branch  ### Does this PR introduce _any_ user-facing change? no ### How was this patch tested? with main branch test - vLLM version: v0.22.1 - vLLM main: https://github.com/vllm-project/vllm/commit/967c5c3bc38891f4465d3f4e9…

### #11512 — [[CI] Revert "[CI] Add `--aop_enabled` parameter to comment-triggered night…](https://github.com/vllm-project/vllm-ascend/pull/11512)
- **作者**: zhangxinyuehfad  **时间**: 2026-07-06 21:55 CST
- **标签**: documentation, ci/build
- **摘要**: …ly/weekly tests (#11475)"  This reverts commit 328a01ca15cb6cb7883e40e34af8a367f7459c3e.  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1…

### #11511 — [Rfc/vllm cann synchronize with  main branch](https://github.com/vllm-project/vllm-ascend/pull/11511)
- **作者**: zzzkkk-zk  **时间**: 2026-07-06 21:09 CST
- **标签**: documentation, ci/build, module:tests, module:ops, module:core, module:quantization, module:tools
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.22.1 - vLLM main: https://github.com/vllm-project/vllm/commit/967c5c3bc38891f4465d3f4e99917ed837bb3833

### #11510 — [[CI]main2main 0706](https://github.com/vllm-project/vllm-ascend/pull/11510)
- **作者**: zhao-stack  **时间**: 2026-07-06 21:09 CST
- **标签**: ci/build, module:tests, ready
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #11508 — [[BugFix]fix qwen3.5+pcp+chunkprefill accuracy error](https://github.com/vllm-project/vllm-ascend/pull/11508)
- **作者**: weiguihua2  **时间**: 2026-07-06 20:46 CST
- **标签**: module:tests, module:ops, ready
- **摘要**: ### What this PR does / why we need it? Fix the precision issue in qwen3.5 with pcp and chunkprefill stacking.  Original code: updated_state[i] = all_final_state[i] + matmul(all_final_h_update[i], updated_state[i-1]) Missing - s0 Expanding all_final_state[i] = Φ_i·s0 + p_i: Original code = (Φ_i·s0 +…

### #11507 — [add w8a16 support](https://github.com/vllm-project/vllm-ascend/pull/11507)
- **作者**: Jinxiao0302  **时间**: 2026-07-06 20:43 CST
- **标签**: module:ops, module:quantization
- **摘要**: <!--  Thanks for sending a pull request!  BEFORE SUBMITTING, PLEASE READ https://docs.vllm.ai/en/latest/contributing/overview.html  --> ### What this PR does / why we need it? <!-- - Please clarify what changes you are proposing. The purpose of this section is to outline the changes and how this PR …
