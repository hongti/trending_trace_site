# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-16 15:09 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🐛 Issue 动态
1. **Runtime Guard 可观测性缺陷与兼容性问题 (#21976)**
   - 在 Model Runner v2 上已接入 Runtime Guard（默认关闭），但目前存在特性兼容性差异、已知的 dump/attribution bug，且部分拓扑未经验证，产品文档缺乏对白盒结论和验证范围的相关说明。
2. **上游测试用例批量失败 (#21969 ~ #21975)**
   - 有用户集中反馈在 NPU 环境（torch-npu 2.10.0 / Cann 910B4）下运行多个上游 vLLM 测试用例时发生失败，涉及模块包括：Prefill 分类、结构化输出管理器、Hisparse KV Connector、MTP 推测解码、GLM 多模态、FP8 MoE 权重加载以及 RL 离线权重传输等。

### 🛠️ PR 动态
1. **性能与代码优化**
   - **Qwen4Exp 性能提升 (#21957)**：将三个针对 QSA（Query-Sparse Attention）准备路径的优化 PR 合并为一个集成 PR，以提升执行性能。
   - **清理 MRV2 遗留代码 (#21955)**：移除 Model Runner v2 中的旧版遗留代码。
2. **CI 与构建修复**
   - **构建内存优化与回传 (#21961, #21962, #21963, #21977)**：为控制集群构建节点的内存占用，将构建并行度限制为 `MAX_JOBS=32`。同时修复了此前将该参数硬编码进 Docker 镜像 `ENV` 的问题，改为仅在 CI 环境下生效，并将该修复回传至 `v0.30.0`、`v0.31.0rc` 和 `v0.32.0rc` 版本分支。
3. **文档更新**
   - **运维与诊断文档补充 (#21978)**：新增 A3 Memcache 软件兼容性、超级 Pod 检查及日志诊断的自查指南。
   - **多语言文档翻译 (#21956, #21959, #21960)**：CI 自动翻译并更新了数十个中文文档文件。

### 🚀 Release 动态
近期无新版本发布。

---

## 🐛 Issues

### #21976 — [[Observability] Runtime Guard: feature compatibility gaps, known dump/attribution bugs, and untested topologies](https://github.com/vllm-project/vllm-ascend/issues/21976)
- **作者**: djh95  **时间**: 2026-11-16 14:46 CST
- **摘要**: ## Summary  Runtime Guard（detect / report / `dump_kv`）在 **Model Runner v2** 上已接线，默认关闭。当前产品文档没有写清：哪些并行 / KV 特性会影响 Guard，白盒结论如何，验证到什么程度，以及已确认的诊断正确性问题。  **白盒** = 读代码推路径/假设是否成立。 **验证** = UT / live NPU / 端到端跑通并核对结果。  下文新确认的三条，是 **白盒 + CPU 隔离**；相关 **NPU 端到端尚未系统验证**。  **白盒列：** `未做` / `已做无问题` / `已做需改代码` / `…

### #21975 — [[Bug][Upstream]: test_gpu_model_runner_prefill_tail_classification case failed: tests/v1/worker/test_gpu_model_runner_prefill_tail_classification.py](https://github.com/vllm-project/vllm-ascend/issues/21975)
- **作者**: meiling6226  **时间**: 2026-11-16 14:40 CST
- **摘要**: ### Your current environment  Container `test_ml_1106`. vllm 0.32.0+empty, vllm-ascend 0.19.1rc2.dev3242+gb3b82c137. ASCEND_RT_VISIBLE_DEVICES=7, torch-npu 2.10.0.post4, Cann/kernel 910B4.  Job: `manual upstream rerun` shard singlecard.  ### 🐛 Describe the bug  `tests/v1/worker/test_gpu_model_runner…

### #21974 — [[Bug][Upstream]: test_structured_output_manager case failed: tests/v1/structured_output/test_structured_output_manager.py](https://github.com/vllm-project/vllm-ascend/issues/21974)
- **作者**: meiling6226  **时间**: 2026-11-16 14:40 CST
- **摘要**: ### Your current environment  Container `test_ml_1106`. vllm 0.32.0+empty, vllm-ascend 0.19.1rc2.dev3242+gb3b82c137. ASCEND_RT_VISIBLE_DEVICES=7, torch-npu 2.10.0.post4, Cann/kernel 910B4.  Job: `manual upstream rerun` shard singlecard.  ### 🐛 Describe the bug  `tests/v1/structured_output/test_struc…

### #21973 — [[Bug][Upstream]: test_hisparse_connector case failed: tests/v1/kv_connector/unit/test_hisparse_connector.py](https://github.com/vllm-project/vllm-ascend/issues/21973)
- **作者**: meiling6226  **时间**: 2026-11-16 14:40 CST
- **摘要**: ### Your current environment  Container `test_ml_1106`. vllm 0.32.0+empty, vllm-ascend 0.19.1rc2.dev3242+gb3b82c137. ASCEND_RT_VISIBLE_DEVICES=7, torch-npu 2.10.0.post4, Cann/kernel 910B4.  Job: `manual upstream rerun` shard singlecard.  ### 🐛 Describe the bug  `tests/v1/kv_connector/unit/test_hispa…

### #21972 — [[Bug][Upstream]: test_mtp case failed: tests/v1/e2e/spec_decode/mtp/gemma4/test_mtp.py](https://github.com/vllm-project/vllm-ascend/issues/21972)
- **作者**: meiling6226  **时间**: 2026-11-16 14:40 CST
- **摘要**: ### Your current environment  Container `test_ml_1106`. vllm 0.32.0+empty, vllm-ascend 0.19.1rc2.dev3242+gb3b82c137. ASCEND_RT_VISIBLE_DEVICES=7, torch-npu 2.10.0.post4, Cann/kernel 910B4.  Job: `manual upstream rerun` shard singlecard.  ### 🐛 Describe the bug  `tests/v1/e2e/spec_decode/mtp/gemma4/t…

### #21971 — [[Bug][Upstream]: test_glm case failed: tests/models/multimodal/generation_ppl_test/test_glm.py](https://github.com/vllm-project/vllm-ascend/issues/21971)
- **作者**: meiling6226  **时间**: 2026-11-16 14:39 CST
- **摘要**: ### Your current environment  Container `test_ml_1106`. vllm 0.32.0+empty, vllm-ascend 0.19.1rc2.dev3242+gb3b82c137. ASCEND_RT_VISIBLE_DEVICES=7, torch-npu 2.10.0.post4, Cann/kernel 910B4.  Job: `manual upstream rerun` shard singlecard.  ### 🐛 Describe the bug  `tests/models/multimodal/generation_pp…

### #21970 — [[Bug][Upstream]: test_fp8_tp_loading case failed: tests/kernels/moe/test_fp8_tp_loading.py](https://github.com/vllm-project/vllm-ascend/issues/21970)
- **作者**: meiling6226  **时间**: 2026-11-16 14:39 CST
- **摘要**: ### Your current environment  Container `test_ml_1106`. vllm 0.32.0+empty, vllm-ascend 0.19.1rc2.dev3242+gb3b82c137. ASCEND_RT_VISIBLE_DEVICES=7, torch-npu 2.10.0.post4, Cann/kernel 910B4.  Job: `manual upstream rerun` shard singlecard.  ### 🐛 Describe the bug  `tests/kernels/moe/test_fp8_tp_loading…

### #21969 — [[Bug][Upstream]: test_offline case failed: tests/entrypoints/rl/weight_transfer/test_offline.py](https://github.com/vllm-project/vllm-ascend/issues/21969)
- **作者**: meiling6226  **时间**: 2026-11-16 14:39 CST
- **摘要**: ### Your current environment  Container `test_ml_1106`. vllm 0.32.0+empty, vllm-ascend 0.19.1rc2.dev3242+gb3b82c137. ASCEND_RT_VISIBLE_DEVICES=7, torch-npu 2.10.0.post4, Cann/kernel 910B4.  Job: `manual upstream rerun` shard singlecard.  ### 🐛 Describe the bug  `tests/entrypoints/rl/weight_transfer/…

## 🔀 Pull Requests

### #21978 — [[Doc] Add Memcache software compatibility, super pod checks and log diagnosis](https://github.com/vllm-project/vllm-ascend/pull/21978)
- **作者**: wade-wang-bot  **时间**: 2026-11-16 14:51 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Resolves #21418.  The [Doc] request asks for A3 Memcache deployment self-check material that is currently missing from the KV Cache Pool feature guide: version compatibility, environment/support checks, cross-super-pod connectivity, and log-based judgment of …

### #21977 — [[CI][Misc] Only cap build parallelism when CI passes MAX_JOBS=32](https://github.com/vllm-project/vllm-ascend/pull/21977)
- **作者**: JavaPythonAIForBAT  **时间**: 2026-11-16 14:48 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? Follow-up to #21214, which baked `MAX_JOBS=32` / `CARGO_BUILD_JOBS=32` into the image `ENV` of all 8 Dockerfiles. That persisted the cap inside the image and forced `setup.py`'s cmake `-j=32` on low-core machines, which can OOM ordinary users building locally …

### #21963 — [[CI][v0.32.0rc] Cap build parallelism (MAX_JOBS=32) to bound memory on cluster buildkitd nodes](https://github.com/vllm-project/vllm-ascend/pull/21963)
- **作者**: JavaPythonAIForBAT  **时间**: 2026-11-16 14:35 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? Backport of #21214 to `releases/v0.32.0rc`.  Cap the vllm-ascend image build parallelism to `MAX_JOBS=32` / `CARGO_BUILD_JOBS=32` to bound memory on the shared buildkitd nodes of the Ascend CI cluster (unbounded nproc compile could OOM a 128Gi node and evict b…

### #21962 — [[CI][v0.31.0rc] Cap build parallelism (MAX_JOBS=32) to bound memory on cluster buildkitd nodes](https://github.com/vllm-project/vllm-ascend/pull/21962)
- **作者**: JavaPythonAIForBAT  **时间**: 2026-11-16 14:35 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? Backport of #21214 to `releases/v0.31.0rc`.  Cap the vllm-ascend image build parallelism to `MAX_JOBS=32` / `CARGO_BUILD_JOBS=32` to bound memory on the shared buildkitd nodes of the Ascend CI cluster (unbounded nproc compile could OOM a 128Gi node and evict b…

### #21961 — [[CI][v0.30.0] Cap build parallelism (MAX_JOBS=32) to bound memory on cluster buildkitd nodes](https://github.com/vllm-project/vllm-ascend/pull/21961)
- **作者**: JavaPythonAIForBAT  **时间**: 2026-11-16 14:34 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? Backport of #21214 to `releases/v0.30.0`.  Cap the vllm-ascend image build parallelism to `MAX_JOBS=32` / `CARGO_BUILD_JOBS=32` to bound memory on the shared buildkitd nodes of the Ascend CI cluster (unbounded nproc compile could OOM a 128Gi node and evict bui…

### #21960 — [[v0.32.0rc][Doc] Translated Doc files 2026-11-16](https://github.com/vllm-project/vllm-ascend/pull/21960)
- **作者**: vllm-ascend-ci  **时间**: 2026-11-16 14:26 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **10** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/contribution/doc_writing.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/…

### #21959 — [[v0.30.0][Doc] Translated Doc files 2026-11-16](https://github.com/vllm-project/vllm-ascend/pull/21959)
- **作者**: vllm-ascend-ci  **时间**: 2026-11-16 14:26 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **6** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/tutorials/models/DeepSeek-V4-Flash.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/tutorials/models/DeepSe…

### #21957 — [[Performance][Qwen4Exp] Fuse QSA preparation paths](https://github.com/vllm-project/vllm-ascend/pull/21957)
- **作者**: Lethobenthos20  **时间**: 2026-11-16 14:21 CST
- **标签**: module:tests, module:ops
- **摘要**: ## What this PR does / why we need it  Combines the three focused Qwen4Exp QSA preparation optimizations from #20943, #20944, and #20947 into one `main`-based integration PR so the stacked release work in #21636 can rebase on a single mainline dependency.  The runtime scope is limited to:  - fuse QS…

### #21956 — [[Doc] Translated Doc files 2026-11-16](https://github.com/vllm-project/vllm-ascend/pull/21956)
- **作者**: vllm-ascend-ci  **时间**: 2026-11-16 14:14 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **30** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/slash-commands.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents…

### #21955 — [[Misc][MRV2] Remove v2 legacy code](https://github.com/vllm-project/vllm-ascend/pull/21955)
- **作者**: zouzy5137  **时间**: 2026-11-16 14:13 CST
- **标签**: module:tests, module:ops, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…
