# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-07 11:32 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🔄 Issue (2 项)
本期 Issue 主要聚焦于 **GLM-5.2 模型的 DCP（解码上下文并行）功能**：
*   **DCP 前缀缓存失效 Bug**：在 v0.23.0rc1 版本下，A2/A3 设备开启 DCP 后，GLM5.2 出现前缀缓存不命中的问题，且可稳定复现 (#13750)。
*   **DCP 功能测试需求**：提出需对 GLM-5.2 的 DCP 功能进行正确性验证，以确保长序列推理（head-sharding 机制）可用 (#13747)。

---

### 🛠️ Pull Request (10 项)
PR 活动密集，涵盖新特性、性能优化、重要 Bug 修复及文档更新：

**✨ 新特性与性能优化**
*   **SiTU 动态 EPLB 支持**：移除了 `moe_mlp.py` 中的限制，支持 SiTU 动态 EPLB，并新增 W4A8 量化映射（同时合入 v0.26.0rc 分支）(#13753, #13733)。
*   **Kimi K3 性能提升**：将 Kimi K3 的 Q、K、V 投影融合为单一的列并行投影（Fuse QKV），提升推理性能 (#13749)。
*   **统一 RL 配置**：新增通过 `additional_config.rl_config` 进行统一强化学习配置的功能 (#13748)。

**🐛 重要 Bug 修复**
*   **推测解码 RNG 状态修复**：修复了 zero-draft 请求中 RNG 状态被错误消耗的问题，保障采样正确性 (#13755)。
*   **MRV2 启动崩溃修复**：修复了 MRV2 结合 FlashComm1 和 EAGLE3 推测解码时，A3 环境下的启动崩溃和全图捕获失败问题（涉及 4 个独立缺陷）(#13752)。

**📖 文档与 CI 更新**
*   **文档**：多节点部署文档新增 Atlas 950 服务器预检查步骤 (#13756)；自动翻译 51 个文档文件至中文 (#13754)。
*   **CI**：更新夜间/每周测试框架的多模态输入测试功能 (#13751)；调整部分 A3 测试分区在 752T 上的重跑策略 (#13734)。

---

### 🚀 Release
*   本期动态中**无新的 Release 发布**。
*   *注*：从 Issue 和 PR 中可见，社区当前正在积极修复 **v0.23.0rc1** 的遗留问题，并同步向 **v0.26.0rc** 分支合入新特性（如 SiTU 动态 EPLB），预计后续版本将重点支持这些新特性与修复。

---

## 🐛 Issues

### #13750 — [[Bug]: GLM5.2在0.23.0rc1版本下，A2/A3开启DCP之后，前缀缓存不命中/“Under vLLM version 0.23.0rc1, when DCP (decode-context-parallelism) is enabled on A2/A3 (Ascend devices), prefix caching does not hit.”](https://github.com/vllm-project/vllm-ascend/issues/13750)
- **作者**: liutt268-hash  **时间**: 2026-08-07 11:07 CST
- **标签**: bug, glm5, triaged, llm-model
- **摘要**: ### Your current environment  quay.io/ascend/vllm-ascend:v0.23.0rc1    A2/A23  ### 🐛 Describe the bug  稳定复现， 测试请求内容： curl -X POST http://localhost:11025/v1/chat/completions \   -H "Content-Type: application/json" \   -d '{     "model": "GLM-5.2",     "messages": [       {         "role": "user",    …

### #13747 — [[Test][DCP] GLM-5.2 DCP 功能测试（decode context parallelism）](https://github.com/vllm-project/vllm-ascend/issues/13747)
- **作者**: chengduxiaowu  **时间**: 2026-08-07 10:35 CST
- **摘要**: ## 背景  vllm-ascend 支持 DCP（Decode Context Parallelism），用于长序列 decode 阶段的并行加速（head-sharding 机制，见 `platform.py` 的 `decode_context_parallel_size` 约束）。需验证 DCP 在 **GLM-5.2** 上的功能正确性，确保长序列推理可用。  ## 任务  对 **GLM-5.2** 进行 DCP 功能测试，验证 decode context parallel 在该模型上的功能可用性。  - 模型：GLM-5.2 - 功能：DCP（decode context pa…

## 🔀 Pull Requests

### #13756 — [[Doc][v0.23.0] Add Atlas 950 server pre-checks for multi-node deployment](https://github.com/vllm-project/vllm-ascend/pull/13756)
- **作者**: MrZ20  **时间**: 2026-08-07 11:31 CST
- **摘要**: <!--  Thanks for sending a pull request!  BEFORE SUBMITTING, PLEASE READ https://docs.vllm.ai/en/latest/contributing/overview.html  --> ### What this PR does / why we need it? This PR adds an Atlas 950 series server pre-check section to the multi-node deployment documentation.  The new section:  - R…

### #13755 — [  [BugFix][Spec Decode] Preserve RNG state for zero-draft requests](https://github.com/vllm-project/vllm-ascend/pull/13755)
- **作者**: yaleyoou  **时间**: 2026-08-07 11:29 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?    `sample_recovered_tokens()` consumed per-request RNG for zero-draft   requests and then discarded the generated exponential samples with   `torch.where`. This made later rejection sampling depend on scheduling   and batch composition.    Use the existing Py…

### #13754 — [[Doc] Translated Doc files 2026-08-07](https://github.com/vllm-project/vllm-ascend/pull/13754)
- **作者**: vllm-ascend-ci  **时间**: 2026-08-07 11:29 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **51** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/versioning_policy.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Docume…

### #13753 — [[Ops][Feature] Support SiTU with dynamic EPLB and add W4A8 quantization mapping](https://github.com/vllm-project/vllm-ascend/pull/13753)
- **作者**: Spicy-Stick  **时间**: 2026-08-07 11:24 CST
- **摘要**: ### What this PR does / why we need it? This PR enables support for SiTU with dynamic EPLB by removing the `NotImplementedError` restriction in `moe_mlp.py`. Additionally, it adds the `(QuantType.W4A8, False)` configuration mapping to the EPLB adaptor in `vllm_adaptor.py` to support W4A8 quantizatio…

### #13752 — [[BugFix][MRV2] Fix EAGLE3 startup crash and FlashComm1 FULL-graph capture under MRV2](https://github.com/vllm-project/vllm-ascend/pull/13752)
- **作者**: xuchi-0808  **时间**: 2026-08-07 11:23 CST
- **标签**: module:ops, module:core
- **摘要**: ## Summary  Fixes the A3 nightly `A3_qwen3-30b-acc` (TP4) startup/runtime failures when Model Runner V2 is used together with FlashComm1 and Eagle3 speculative decoding. Four independent defects on the same MRV2 + FlashComm1 path are fixed in this PR so the nightly configuration can start and comple…

### #13751 — [[CI] Updating the multi-modal input testing feature in the nightly and weekly framework](https://github.com/vllm-project/vllm-ascend/pull/13751)
- **作者**: guxin108  **时间**: 2026-08-07 11:09 CST
- **标签**: module:tests, module:tools
- **摘要**: ### What this PR does / why we need it? we update the multi-modal input testing feature in the nightly and weekly framework  ### Does this PR introduce _any_ user-facing change? yes modify: tools/send_mm_request.py tests/e2e/nightly/single_node/models/scripts/test_single_node.py tests/e2e/nightly/si…

### #13749 — [[Perf][Model] Fuse Kimi K3 QKV projections](https://github.com/vllm-project/vllm-ascend/pull/13749)
- **作者**: Yaphets24  **时间**: 2026-08-07 10:54 CST
- **标签**: module:ops
- **摘要**: ## What this PR does / why we need it?  Fuses the Kimi K3 KDA Q, K, and V projections into one column-parallel projection while preserving checkpoint shard loading.  ## Does this PR introduce any user-facing change?  No.  ## How was this patch tested?  - Not run locally (NPU-dependent performance ch…

### #13748 — [[Feature] Unified RL Configuration via additional_config.rl_config](https://github.com/vllm-project/vllm-ascend/pull/13748)
- **作者**: Ronald1995  **时间**: 2026-08-07 10:40 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: <!--  Thanks for sending a pull request!  BEFORE SUBMITTING, PLEASE READ https://docs.vllm.ai/en/latest/contributing/overview.html  --> ### What this PR does / why we need it?  This implements rl config proposed in #https://github.com/vllm-project/vllm-ascend/issues/13706  ### Does this PR introduce…

### #13734 — [[CI] Rerun selected A3 partitions on 752T](https://github.com/vllm-project/vllm-ascend/pull/13734)
- **作者**: NaraFluorine  **时间**: 2026-08-07 10:29 CST
- **标签**: ci/build
- **摘要**: What this PR does / why we need it?  Temporarily limits E2E selected-tests to default A3 runners for comparison with the 560T run: - a3-2 card-(part 2-3) - a3-4 card-(part 3-3)  Does this PR introduce any user-facing change?  No.  How was this patch tested?  - python -m py_compile .github/workflows/…

### #13733 — [[v0.26.0rc][Ops][Feature] Support SiTU with dynamic EPLB and add W4A8 quantization mapping](https://github.com/vllm-project/vllm-ascend/pull/13733)
- **作者**: Spicy-Stick  **时间**: 2026-08-07 10:26 CST
- **标签**: module:ops
- **摘要**: ### What this PR does / why we need it? This PR enables support for SiTU with dynamic EPLB by removing the `NotImplementedError` restriction in `moe_mlp.py`. Additionally, it adds the `(QuantType.W4A8, False)` configuration mapping to the EPLB adaptor in `vllm_adaptor.py` to support W4A8 quantizatio…
